# 1. Initial Server Setup

## 1.1 Update and Upgrade Packages
```bash
sudo apt update && sudo apt upgrade -y
```

## 1.2 Create Deployment User
```bash
sudo adduser deploy
sudo usermod -aG sudo deploy
su - deploy
```

# Description

## `sudo adduser deploy`
- This creates a new user named `deploy`.
- It prompts you to enter a password and optionally some user details.
- The user will have a home directory like `/home/deploy/`.
- It does not have sudo (admin) rights by default.

## `sudo usermod -aG sudo deploy`
- This adds the `deploy` user to the `sudo` group, which gives it administrative privileges.
- `-aG`: Append (`a`) the user to supplementary groups (`G`)
- `sudo`: Name of the group
- After this, `deploy` can run commands as root using `sudo`
- Now `deploy` can do things like `sudo apt update` or `sudo docker-compose up`.

## `su - deploy`
- This switches to the `deploy` user account in the current shell session.
- `su` = substitute user
- `-` loads the full environment (like login shell)

You're now acting as the `deploy` user, with its home, permissions, and shell.

---

## 🧠 Why use a `deploy` user?
- **Security**: Avoid deploying apps directly as `root`
- **Isolation**: Keeps deployment and app files in `/home/deploy/`
- **Control**: You can limit what this user can do if needed