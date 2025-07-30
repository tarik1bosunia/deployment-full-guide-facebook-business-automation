# [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/) 
### for [clean installation](https://docs.docker.com/engine/install/ubuntu/#uninstall-docker-engine)

#### [How to Uninstall Docker Engine](./uninstall_docker_engine.md)


## Prerequisites
### Firewall limitations
#### Warning
Before you install Docker, make sure you consider the following security implications and firewall incompatibilities.
### Uninstall old versions
#### Run the following command to uninstall all conflicting packages:
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

---

## Install using the apt repository
1. Set up Docker's apt repository.
    - Add Docker's official GPG key:
    ```bash
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    ```
    - Add the repository to Apt sources:
    ```bash
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    ```
2. Install the Docker packages.
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
3. Verify that the installation is successful by running the hello-world image:
```
sudo docker run hello-world
```    