# [Linux post-installation steps for Docker Engine](https://docs.docker.com/engine/install/linux-postinstall/)
## Manage Docker as a non-root user
### 1. To create the docker group and add your user:
```bash
sudo groupadd docker
```
### 2. Add your user to the docker group.
```bash
sudo usermod -aG docker $USER
```
### 3. Log out and log back in so that your group membership is re-evaluated.

If you're running Linux in a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.