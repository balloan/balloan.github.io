---
layout: default
title: SSH Overview
parent: Ubuntu Project
grand_parent: Projects
nav_order: 2
---

# SSH - Overview & Initial Install

I’m going to begin with a Nmap scan of the Ubuntu server, just so we can compare the changes throughout the process of installing and configuring SSH.

![Initial Nmap Scan](../../assets/images/ubuntuproject/4.png)

As expected, all ports are closed as there are currently no services running on the Ubuntu server.

Before diving into configuration, I think it’s important to consider (at a high level), what SSH is, and how it works. From there we’ll be able to consider the possible ways it can be attacked, and more importantly, defended.

SSH is a protocol that allows remote logins. Communications through SSH are encrypted. It has numerous functions - remote shell access, remote command execution, file transfer port forwarding and more. It’s extremely versatile, and is present on the vast majority of Linux distros. The most common usage, and then one I’ll be using for this example, is a client server model. The Ubuntu server will be running OpenSSH server, and any machine that connects to it is the client.

The client needs to authenticate to the server to connect - the most common methods are passwords and public key authentication. 

OpenSSH also supports a variety of other methods (ie Kerberos integration, challenge/response for PAM) but for the sake of simplicity I won’t be covering those.

Right away, it’s extremely apparent that this service is valuable to attackers. Gaining access through SSH is gaining a shell, allowing a great deal of access in the system. It’s a service that allows direct access, and it can be publicly exposed to the internet as a whole; we need to ensure it is secure.

# Installation

We'll begin by installing the SSH service.

```
sudo apt install openssh-server

systemctl status ssh
ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-01-15 01:54:17 UTC; 1min 19s ago
```

```
ss -tnlp

State            Recv-Q           Send-Q                     Local Address:Port                       Peer Address:Port           Process           
LISTEN           0                128                              0.0.0.0:22                              0.0.0.0:*                                
LISTEN           0                4096                       127.0.0.53%lo:53                              0.0.0.0:*                                
LISTEN           0                128                                 [::]:22                                 [::]:*                          
```

The SSH server is now running, and the server is listening on port 22.
