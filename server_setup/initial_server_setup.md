# 1. Initial Server Setup

## 1.1 Update and Upgrade Packages
```bash
sudo apt update && sudo apt upgrade -y
```

## 1.2 [Create Deployment User](./deploy_user_setup.md)
```bash
man sudo_root # for details learn about sudo
sudo adduser deploy
sudo usermod -aG sudo deploy
su - deploy
```

##  1.3 [Set Up SSH Key Authentication](.//ssh_key_setup_guide.md)
on your local machine
```bash
ssh-copy-id deploy@your_server_ip
```

## 1.4 Basic Firewall Configuration
```bash
sudo ufw allow OpenSSH
sudo ufw allow http
sudo ufw allow https
sudo ufw enable
```