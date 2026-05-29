
class: title

# Our training environment

![SSH terminal](images/title-our-training-environment.jpg)

---

## Our training environment

- 2 VMs has been provisioned for three students

- They were created just before the training.

- They will stay up during the whole training.

- They will be destroyed shortly after the training.

- They do not come pre-loaded with Docker, so you need to install everything.

<img src="images/qrcode_docs.google.com.png" alt="VMs Sheets QR Code" width="200" align="right"/>

---

## What *is* Docker?

- "Installing Docker" really means "Installing the Docker Engine and CLI".

- The Docker Engine is a daemon (a service running in the background).

- This daemon manages containers, the same way that a hypervisor manages VMs.

- We interact with the Docker Engine by using the Docker CLI.

- The Docker CLI and the Docker Engine communicate through an API.

- There are many other programs and client libraries which use that API.

---

## Why don't we run Docker locally?

- We are going to download container images and distribution packages.

- This could put a bit of stress on the local WiFi and slow us down.

- Instead, we use remote VMs that has a good connectivity

- In some rare cases, installing Docker locally is challenging:

  - no administrator/root access (computer managed by strict corp IT)

  - 32-bit CPU or OS

  - old OS version (e.g. CentOS 6, OSX pre-Yosemite, Windows 7)

- It's better to spend time learning containers than fiddling with the installer!

---

## Connecting to your Virtual Machine

You need an SSH client.

* On OS X, Linux, and other UNIX systems, just use `ssh`:

```bash
$ ssh <login>@<ip-address>
```

* On Windows, if you don't have an SSH client, you can download:

  * Putty (www.putty.org)

  * Git BASH (https://git-for-windows.github.io/)

  * MobaXterm (https://mobaxterm.mobatek.net/)

---

## Checking your Virtual Machine

Once logged in, goal is to be able to run a basic Docker command:

.small[
```bash
$ docker version
Client: Docker Engine - Community
 Version:           29.2.1
 API version:       1.53
 Go version:        go1.25.6
 Git commit:        a5c7197
 Built:             Mon Feb  2 17:17:24 2026
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          29.2.1
  API version:      1.53 (minimum version 1.44)
  Go version:       go1.25.6
  Git commit:       6bc6209
  Built:            Mon Feb  2 17:17:24 2026
  OS/Arch:          linux/amd64
  Experimental:     false
```
]

If this doesn't work, raise your hand so that an instructor can assist you!

???

:EN:Container concepts
:FR:Premier contact avec les conteneurs

:EN:- What's a container engine?
:FR:- Qu'est-ce qu'un *container engine* ?
