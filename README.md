# Login to Ubuntu/Zorin OS Using Active Directory (AD) Credentials

## ✔️ Prerequisites
- Windows Server Active Directory must be working
- Ubuntu/Zorin client DNS **must point to AD DNS**
- Client must reach Domain Controller (`ping` works)

---

## 1. Install Required Packages
```bash
sudo apt update
sudo apt install sssd realmd adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
```

---

## 2. Discover the Domain
Replace `MYDOMAIN.LK` with your actual domain.
```bash
sudo realm discover MYDOMAIN.LK
```
If successful, domain information will be displayed.

---

## 3. Join the Domain
```bash
sudo realm join -U Administrator MYDOMAIN.LK
```
Enter the AD Administrator password when prompted.

---

## 4. Enable Automatic Home Directory Creation
```bash
sudo pam-auth-update --enable mkhomedir
```
Verify permissions:
```bash
ls -ld /home
```
Should be:
```
drwxr-xr-x  root root ...
```

---

## 5. Verify Domain Join
```bash
realm list
```
Expected output includes:
```
domain-name: MYDOMAIN.LK
configured: kerberos-member
login-formats: %U@MYDOMAIN.LK
```

---

## 6. Test Domain User Lookup
Example: usernetop3
```bash
getent passwd usernetop3
```
Expected format:
```
usernetop3:*:611022138:611000513:Data Network Section op 3:/home/usernetop3:/bin/bash
```
Then domain integration is working.

---

## 7. Check SSSD Service
```bash
systemctl status sssd
```
If not running:
```bash
sudo systemctl restart sssd
```

---

# Kerberos Configuration

## 1️⃣ Install Kerberos Client
```bash
sudo apt install krb5-user
```
During setup:
- Default Kerberos realm → `MYDOMAIN.LK`

## 2️⃣ Test Kerberos Login
```bash
kinit usernetop3@MYDOMAIN.LK
klist
```
You should see a valid ticket.

## 3️⃣ Restart SSSD
```bash
sudo systemctl restart sssd
```

---

# Configure SSSD

Edit SSSD configuration:
```bash
sudo nano /etc/sssd/sssd.conf
```

Recommended config:
```ini
[sssd]
services = nss, pam
config_file_version = 2
domains = mydomain.lk

[domain/mydomain.lk]
ad_domain = mydomain.lk
krb5_realm = MYDOMAIN.LK
realmd_tags = manages-system joined-with-samba
id_provider = ad
auth_provider = ad
chpass_provider = ad
access_provider = ad
default_shell = /bin/bash
fallback_homedir = /home/%u
use_fully_qualified_names = False
cache_credentials = True
ad_site = Default-First-Site-Name
```

Set correct permissions:
```bash
sudo chmod 600 /etc/sssd/sssd.conf
sudo chown root:root /etc/sssd/sssd.conf
```

Restart:
```bash
sudo systemctl restart sssd
```

---

# Test Login

### CLI Login
```bash
su - usernetop3
```
Or:
```bash
su - usernetop3@mydomain.lk
```

### GUI Login (Zorin / Ubuntu Desktop)
1. Go to login screen
2. Click **Not Listed?**
3. Enter username:
   ```
   usernetop3
   ```
   (Only username. because use_fully_qualified_names=False in sssd.conf)
4. Enter AD password:

### Verify User Info
```bash
id usernetop3
```
Expected:
```
uid=611022138(usernetop3) gid=611000513(domain users) groups=611000513(domain users)
```

---

# ✔️ Tools & Services Used

| Tool / Service | Purpose |
|----------------|---------|
| **SSSD** | Authentication + user info retrieval from AD |
| **realmd** | Domain discovery and join helper |
| **adcli** | Performs actual join to AD |
| **Kerberos (krb5-user)** | Secure authentication and ticketing |
| **Samba / samba-common-bin** | NetBIOS/SMB helpers for AD environment |
| **PAM (mkhomedir)** | Auto-create home directories on first login |
| **Netplan / NetworkManager** | DNS configuration for AD resolution |
| **dig / nslookup** | Troubleshooting DNS & Kerberos SRV records |

---

# 🎉 Outcome
- Zorin/Ubuntu successfully joined to AD
- AD users can login using GUI and CLI
- Home directories auto-created
- Kerberos authentication working
- No local user management needed

---

**Author:** Nalaka326  
**Last Updated:** November 2025  
**Tested On:** Zorin OS 18 Core / Ubuntu Server 24.04

