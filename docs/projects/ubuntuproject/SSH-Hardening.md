---
layout: default
title: SSH Attack & Defend
parent: Ubuntu Project
grand_parent: Projects
nav_order: 3
---

## Password Attacks

I’ll start by attacking the default configuration of this server by performing a dictionary attack against the password of the user `victim` on the Ubuntu server by using the tool Hydra.

Afterwards, I’ll begin hardening the SSH service by looking at solutions such as Fail2Ban, only allowing authentication with keys, and enabling multi-factor authentication. I’ll also look at potentially dangerous SSH settings.