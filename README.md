# Self-Hosted Linux Server with a Containerized App

A Dockerized Node.js/PostgreSQL application deployed on a self-managed Ubuntu server, built from scratch in VirtualBox—including real-world disk and networking troubleshooting along the way.

## Objective

Build a Linux server from scratch using VirtualBox and progressively expand the environment over time. The project incorporates Linux installation, remote administration, networking, containerization, and application deployment. Along the way, I encountered and resolved issues involving disk space failures, networking misconfigurations, and a partition table problem that ultimately required rebuilding the virtual machine.

## Skills Learned

* Installing and configuring a Linux server (Ubuntu Server) in a virtualized environment
* Configuring and securing remote administration using SSH
* Resolving network configuration issues (NAT vs. Bridged Adapter modes in VirtualBox)
* Linux disk and partition management, including diagnosing a partially resized partition
* Recognizing when to repair an environment versus rebuilding it from scratch
* Installing and operating Docker
* Writing a REST API and connecting it to a PostgreSQL database
* General Linux command-line proficiency (file operations, package management, and systemd service management)

## Tools Used

* Oracle VirtualBox (Hypervisor)
* Ubuntu Server 26.04 LTS (Virtual Machine Operating System)
* OpenSSH (Remote Administration)
* Docker (Containerization and Container Networking)
* Node.js and Express (Application Layer)
* PostgreSQL 16 (Database)
* Windows 11 (Host Operating System)

## Steps

### Ref 1: VM Creation in VirtualBox

Created a new virtual machine in VirtualBox, allocating **4096 MB of RAM** and a **25 GB dynamically allocated virtual disk**, matching the operating system type and version to the Ubuntu Server ISO.

### Ref 2: Ubuntu Server Installation

Booted from the Ubuntu Server ISO and completed the installation using the default **ext4** filesystem, full-disk partitioning, and no disk encryption (planned for a future enhancement).

### Ref 3: First Login and System Update

Logged into the console and performed the initial system updates:

```bash
sudo apt update
sudo apt upgrade
```

### Ref 4: SSH Server Installed and Verified

Installed and enabled OpenSSH so the server could be managed remotely from the Windows host instead of the VirtualBox console.

```bash
sudo apt install openssh-server
sudo systemctl enable --now ssh
```

### Ref 5: Remote Connection from Windows Host

Connected to the virtual machine from Windows Terminal using:

```bash
ssh user@<vm-ip>
```

Successfully confirmed remote administration was fully functional.

### Ref 6: Nginx Installed and Verified in Browser

Installed Nginx and confirmed the default welcome page loaded successfully from a web browser on the Windows host, proving the virtual machine was reachable as a normal network device and not just through SSH.

### Ref 7: Docker Installed

Installed Docker and verified the Docker daemon was active.

```bash
sudo apt install docker.io
sudo systemctl enable --now docker
sudo systemctl status docker
```

### Ref 8: Docker Container Deployed and Verified

Ran a prebuilt Nginx container, mapping the container's port **80** to port **8080** on the virtual machine.

```bash
docker run -d --name web -p 8080:80 nginx
```

Confirmed the container was running with:

```bash
docker ps
```

The output showed the container status as **Up** with the port mapping:

```text
0.0.0.0:8080->80/tcp
```

### Ref 9: End-to-End Verification

Opened:

```text
http://192.168.1.30:8080
```

from a browser on the Windows host and confirmed the Nginx welcome page rendered successfully, verifying that networking, Docker, and the web server were all functioning correctly.

## Problems Encountered: Disk Space and Partition Recovery

Partway through the project, `apt upgrade` failed with a **"No space left on device"** error. Later, pulling a Docker image resulted in the same issue. Investigation using `lsblk` and `fdisk -l` revealed an orphaned **16.4 GB** partition from an earlier installation attempt occupying most of the **25 GB** virtual disk. The partition was unmounted and unused.

After confirming it was safe to remove, the partition was deleted and an attempt was made to extend the active partition into the newly freed space using `parted` and `resize2fs`. The filesystem resize was unsuccessful because the active partition's **starting boundary**, not just its ending boundary, would have needed to move. Moving the start of a mounted root partition is unsafe without booting from external media.

Given the uncertainty introduced by multiple partition modifications, the decision was made to rebuild the virtual machine from a clean installation rather than continue repairing the existing environment. The rebuilt VM resulted in a single correctly sized partition with no additional manual resizing required.

### Ref 10: `lsblk` Output Identifying the Orphaned Partition

### Ref 11: Clean Reinstallation Confirming Full Disk Usage After Rebuild

                                  
