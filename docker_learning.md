
# 🐳 Docker Command Guide — Interactive & Clean

Badges:  
![Docker](https://img.shields.io/badge/Docker-Engine-blue) ![Platform](https://img.shields.io/badge/Platform-Ubuntu%20%7C%20Windows-lightgrey)

Short, practical guide with copy-ready commands, quick explanations, tips and small exercises you can run in VS Code integrated terminal.

---

## Table of contents
- [Quick start](#quick-start)
- [Install & prepare](#install--prepare)
- [Containers: create, run, manage](#containers-create-run-manage)
- [Networking & web servers](#networking--web-servers)
- [Images: pull, build, save](#images-pull-build-save)
- [Maintenance & troubleshooting](#maintenance--troubleshooting)
- [Cheatsheet (copy-ready)](#cheatsheet-copy-ready)
- [Interactive exercises](#interactive-exercises)

---

## Quick start
Run these to confirm Docker is installed and the daemon is running:

```bash
docker --version
sudo systemctl status docker --no-pager
```

If Docker is not running on Ubuntu:

```bash
sudo systemctl enable --now docker
```

---

## Install & prepare
Commands to add Docker repo and install (Ubuntu):

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/trusted.gpg.d/docker.asc
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y docker-ce docker-ce-cli
sudo systemctl enable --now docker
sudo apt-mark hold docker-ce docker-ce-cli
```

Tip: On Windows use Docker Desktop; these apt commands are for Linux.

---

## Containers — create, run & manage
Create (does not start):

```bash
docker container create nginx
docker container create --name web01 nginx
```

Run (create & start):

```bash
docker container run --name server01 ubuntu
docker container run --name server02 -it ubuntu     # interactive shell
docker container run --name server03 -dit ubuntu   # detached + pseudo-tty
```

Start / Stop / Restart / Kill:

```bash
docker container start server03
docker container stop server03
docker container restart server03
docker container kill server03
```

Attach / Exec:

```bash
docker container attach server03                # attach to TTY (Ctrl+P, Ctrl+Q to detach)
docker container exec -it server03 /bin/bash    # run a shell in a running container
docker container exec server03 cat /etc/resolv.conf
```

Rename / Remove:

```bash
docker container rename server03 server003
docker container rm server003
docker container rm $(docker container ls -a -q) -f   # remove ALL containers
docker container prune -f                             # remove stopped containers
```

Inspect & stats:

```bash
docker container inspect web02 | jq '.[0].NetworkSettings'
docker container logs server003 --timestamps
docker container stats --no-stream
```

---

## Networking & Web servers
Run nginx mapped to host port:

```bash
docker container run --name web02 -dit -p 8080:80 nginx
```

Publish random host port:

```bash
docker container run --name web03 -dit -P nginx
```

Inspect to find host port / IP:

```bash
docker container inspect web02 --format '{{json .NetworkSettings}}' | jq
# Or quick grep:
docker container inspect web02 | grep -E '"HostPort"|"IPAddress"' | uniq
```

Copy content into nginx:

```bash
docker container cp ./index.html web02:/usr/share/nginx/html/index.html
```

Tip: After copying, reload nginx inside the container or restart the container if needed.

---

## Images: search, pull, build, commit, save
Search and pull:

```bash
docker search alpine --filter=stars=100
docker image pull alpine
docker image pull docker.io/alpine/git
```

Build from Dockerfile in current dir:

```bash
docker image build -t my-ubuntu-nginx:latest .
```

Commit a running container state to an image:

```bash
docker container commit webserver1 customweb:v1
```

Remove images:

```bash
docker image rm $(`docker image ls -a -q`) -f
```

Save / load image to tar:

```bash
docker image save alpine -o alpine_backup.tar
tar -tvf alpine_backup.tar
docker image load --input alpine_backup.tar
```

---

## Update container resources
View memory settings and update:

```bash
docker container inspect web02 --format '{{.HostConfig.Memory}}'
docker container update --memory 200M --memory-swap 200M web02
```

---

## Maintenance & troubleshooting
Disable local firewall/AppArmor (only when appropriate):

```bash
sudo systemctl disable ufw --now
sudo systemctl disable apparmor --now
```

Prune and reclaim space:

```bash
docker system prune -a --volumes
```

Clear dangling images:

```bash
docker image prune -f
```

Check service logs:

```bash
sudo journalctl -u docker --no-pager -n 200
```

---

## Cheatsheet (copy-ready)
- List containers: docker container ls -a  
- Run detached: docker container run -dit --name NAME IMAGE  
- Exec shell: docker container exec -it NAME /bin/bash  
- Copy file: docker container cp localfile NAME:/path/in/container  
- Build image: docker image build -t TAG .  
- Commit container: docker container commit NAME TAG  

---

## Interactive exercises
1) Start an nginx container and open it in browser:
   - Run: docker container run --name try-nginx -dit -p 8081:80 nginx
   - Visit: http://localhost:8081

2) Copy a simple index.html and verify:
   - Create index.html in your project root
   - Run: docker container cp index.html try-nginx:/usr/share/nginx/html/index.html
   - Visit http://localhost:8081 — confirm content changed

3) Save an image and reload:
   - docker image save nginx -o nginx.tar
   - docker image rm nginx -f
   - docker image load --input nginx.tar

---
