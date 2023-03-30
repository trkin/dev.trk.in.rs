---
layout: post
---

## Install on ubuntu

Follow instructions on <https://docs.docker.com/engine/install/ubuntu/>
After adding GPG key and repo you can run

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world
Hello from Docker!
```

or you can use convenience script
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh --dry-run
```

Uninstall with
```
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

To enable users to run use docker engine without sudo
https://docs.docker.com/engine/install/linux-postinstall/
```
sudo usermod -aG docker $USER
```

* run ubuntu shell
```
docker run -it ubuntu /bin/bash
```

https://docs.docker.com/engine/install/linux-postinstall/
