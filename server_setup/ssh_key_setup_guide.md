# SSH Key Authentication Only: Full Setup Guide

This guide helps you create a secure deployment user on a VPS that uses **only SSH key authentication** (no passwords).

---

## 🔧 Step 1: Create a `deploy` User on the VPS

SSH into your VPS as `root` or an existing user:

```bash
sudo adduser deploy
sudo usermod -aG sudo deploy
```

---

## 🔑 Step 2: Generate SSH Key on Local Machine

If you don’t already have an SSH key:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
ssh-keygen -t rsa -b 4096 -C "tarik.cse.ru@gmail.com"
```

This creates:
- Private key: `~/.ssh/id_rsa`
- Public key: `~/.ssh/id_rsa.pub`
- C:\Users\tarik/.ssh/id_rsa_fba_deploy

---

## 📤 Step 3: Copy Your Public Key to the VPS
it is currently not working on windows,
if windows device use gitbash
```bash
ssh-copy-id deploy@your_server_ip
```

If you're using a custom key:

```bash
ssh-copy-id -i ~/.ssh/my_custom_key.pub deploy@your_server_ip
```

---

## 🔐 Step 4: Disable Password Authentication (VPS-Side)

Edit the SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```

Find and update or add these lines:

```conf
PasswordAuthentication no
PermitRootLogin no
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

## ✅ Step 5: Test Login

From your local machine:


```bash
ssh deploy@your_server_ip
ssh -i ~/.ssh/id_rsa_fba_deploy deploy@your_server_ip
```

You should log in **without password prompt**.

---
## Edit/Create SSH Config File for custom file
To simplify your future SSH connections, you can create a config file so you don’t need to specify -i every time.
On your local machine, run:
```bash
nano ~/.ssh/config
```
for windows
```bash
~/.ssh/config

```
### add the following block
```ini
Host fba-vps
    HostName server_ip_address
    User deploy
    IdentityFile ~/.ssh/id_rsa_fba_deploy
```
### Now connect with just:
```bash
ssh fba-vps
```

## 🧠 Why SSH Key Only?

- Stronger security than passwords
- Prevents brute-force attacks
- Enables automation tools (e.g., Ansible, Git, deploy scripts)

---

## 🚨 Important Tips

- Always back up your private key (`~/.ssh/id_rsa`)
- Do **not** share the private key with anyone
- Use a passphrase for extra security