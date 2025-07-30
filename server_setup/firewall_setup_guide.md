# 🔥 UFW Firewall Setup Guide for Ubuntu VPS

## 1. Introduction

UFW (Uncomplicated Firewall) is a user-friendly frontend to `iptables`, the default firewall management tool on Ubuntu. It allows you to easily configure which connections are allowed or blocked.

This guide helps you set up basic firewall rules necessary for running a Django app over SSH, HTTP, and HTTPS.

---

## 2. Enable Required Ports

### Run the following commands as root or with `sudo`:

```bash
sudo ufw allow OpenSSH
sudo ufw allow http
sudo ufw allow https
sudo ufw enable
# then press y if sure OpenSSH allowed
```

---

## 3. Explanation of Each Command

### ✅ `sudo ufw allow OpenSSH`
- Opens port `22` for incoming SSH connections.
- This is **critical** so you don’t get locked out of your server after enabling the firewall.

### ✅ `sudo ufw allow http`
- Opens port `80` to allow unencrypted web traffic (HTTP).

### ✅ `sudo ufw allow https`
- Opens port `443` to allow encrypted web traffic (HTTPS).

### ✅ `sudo ufw enable`
- Activates the firewall with the configured rules.
- Once enabled, any traffic not explicitly allowed will be denied by default.

---

## 4. Check Firewall Status

To verify which rules are currently active:

```bash
sudo ufw status verbose
```

---

## 5. Disable Firewall (if needed)

If you ever need to disable UFW temporarily:

```bash
sudo ufw disable
```

---

## 6. Additional Tips

- To deny specific ports or services: `sudo ufw deny 8080`
- To remove a rule: `sudo ufw delete allow http`

---

## ✅ Summary

| Rule           | Port | Purpose              |
|----------------|------|----------------------|
| OpenSSH        | 22   | Remote shell access  |
| HTTP           | 80   | Web traffic          |
| HTTPS          | 443  | Secure web traffic   |

---

Always verify access (especially SSH) **before enabling the firewall** in production environments.