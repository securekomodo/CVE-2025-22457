# CVE-2025-22457
CVE-2025-22457: Python Exploit POC Scanner to Detect Ivanti Connect Secure RCE

<img width="947" alt="image" src="https://github.com/user-attachments/assets/51b82d3e-cdb9-4587-918f-f6a4e2471eaf" />



This script helps identify Ivanti Connect Secure, Policy Secure, or ZTA Gateways that may be vulnerable to **CVE-2025-22457**, a critical unauthenticated stack-based buffer overflow in the web process triggered via a crafted `X-Forwarded-For` header. It can also identify the remote targets version based on scraping the version information from the HTTP response.

> Inspired by early work to build a [Nuclei template](https://github.com/projectdiscovery/nuclei-templates/pull/11895), this script was created after discovering that Nuclei could not handle cases where **no response is received from the target**, which is essential to reliably detect this issue.

## 📖 Background

Original research and disclosure:
- [watchTowr Labs Blog](https://labs.watchtowr.com/is-the-sofistication-in-the-room-with-us-x-forwarded-for-and-ivanti-connect-secure-cve-2025-22457)

- [Redline Blog](https://www.redlinecybersecurity.com/blog/cve-2025-22457-python-exploit-poc-scanner-to-detect-ivanti-connect-secure-rce))

Shodan Query: `http.favicon.hash:-485487831`

*4,260 Results as of April 9, 2025*


When the payload is successful, the target system logs:
`ERROR31093: Program web recently failed.`


This can be used to build log-based detections in addition to the scan.

## 🔍 How It Works

The script sends a long `X-Forwarded-For` header to vulnerable `.cgi` endpoints. It supports two modes:

### Quick Mode (default)
- Fingerprints the target and grabs the version
- Determines if vulnerable based on version only, no attempt at triggering buffer overflow.

### Detailed Mode
- Sends a pre-check request and expects HTTP 200.
- Sends the crash payload and expects **no response**.
- Sends a follow-up request to ensure the crash wasn’t incidental.
- Only if all three steps behave as expected is the system marked as **vulnerable**.

## 🛠 Usage

Install requirements:
```bash
pip install requests
```

### Scan a single target
```bash
python cve_2025_22457_check.py --target https://example.com
```

### Use detailed mode
```bash
python cve_2025_22457_check.py --target https://example.com --mode detailed
```

### Scan a list of targets from file
```bash
python cve_2025_22457_check.py --input targets.txt --mode detailed
```

### Output results to a file
```bash
python cve_2025_22457_check.py --input targets.txt --mode detailed --output vulnerable.txt
```

## 📌 How It Works
The default path `/dana-na/auth/url_default/welcome.cgi` **can be changed** by Ivanti administrators and may result in false negatives.

A secondary hardcoded path `/dana-na/setup/psaldownload.cgi` is included, which **cannot be modified** and still triggers the vulnerability. Any `.cgi` endpoint appears potentially affected.

## ✅ Example Output
```bash
Pre-check successful (HTTP 200)
No response to payload (expected crash behavior).
Follow-up request returned HTTP 200. Crash condition verified.
VULNERABLE: https://example.com/dana-na/setup/psaldownload.cgi
```
