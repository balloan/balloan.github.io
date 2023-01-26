---
layout: default
title: SSH Attacking & Hardening
parent: Ubuntu Project
grand_parent: Projects
nav_order: 3
---

# Attacking and Hardening SSH

I’ll start by attacking the default configuration of this server by performing a dictionary attack against the password of the user `victim` on the Ubuntu server by using the tool Hydra.

Afterwards, I’ll begin hardening the SSH service by looking at solutions such as Fail2Ban, only allowing authentication with keys, and enabling multi-factor authentication. I’ll also look at potentially dangerous SSH settings.

Example bruteforce of a user with a weak password

![Hydra Attack](../../assets/images/ubuntuproject/8.png)

Obviously, the above example is using an extremely weak password. Just for fun, let’s generate a quick mutated password list based on the word “Microsoft” (pretend that Victim worked there for this example). I used a couple of default word lists, and one custom one I had saved. I quickly generated 480 unique passwords that are various permutations of Microsoft. 

```bash
hashcat password.txt -r /usr/share/hashcat/rules/best64.rule --stdout | sort -u > mutatedlist1.txt
hashcat password.txt -r /usr/share/hashcat/rules/Incisive-leetspeak.rule --stdout | sort -u > mutatedlist2.txt 
hashcat password.txt -r new.rule --stdout | sort -u > mutatedlist3.txt

cat mutatedlist1.txt mutatedlist2.txt mutatedlist3.txt | sort -u > passwords.txt
wc -l passwords.txt   
480 passwords.txt
```

![Hydra Attack](../../assets/images/ubuntuproject/9.png)

The password `Micr0s0ft2022!` is 14 characters long, with a mix of uppercase, lowercase, symbols and numbers. Most password guidelines would consider that to be a very strong password, but the existence of a dictionary word makes it weak. People tend to be lazy about their passwords; permutations of their company name plus easy to remember symbols and numbers aren’t exactly rare.

It's important to note that there are *many* tools out there to generate password lists. You can use hashcat, as I did here, tools such as CUPP (Common User Passwords Profiler), Mentalist and more. It's extremely easy to make long lists that add various levels of complexity to a password, appending common numbers to the end, etc. Truly random passwords are very difficult to guess. Passwords that are based upon words that can be linked to an individual (company name, spouse name, pet name, sports teams, anything posted about frequently on social media) are significantly more vulnerable, even if it's a seemingly strong password.

* * *
## Hardening SSH

The above attack only works if I'm allowed to attempt to authenticate as many times as I like - let's fix that.

`Fail2Ban` is a program that scans log files such as `/var/log/auth.log` and blocks IP addresses who have too many failed authentications (ie bruteforce attacks). It’s a good way to harden services and discourage bruteforce attacks, while not sacrificing convenience. The default ban time is only 10 minutes, you can adjust the timeout to a value that is appropriate.

The default installation will enable fail2ban for our SSH service. I’m not going to do any in-depth configuration here; it’s more just to show that this exists, and is a good option to help harden the server.

```bash
sudo apt install fail2ban -y

sudo systemctl status fail2ban
fail2ban.service - Fail2Ban Service
    Loaded: loaded (/lib/systemd/system/fail2ban.service; disabled; vendor preset: enabled)
    Active: inactive (dead)
```

I’ll be looking at the configuration file `/etc/fail2ban/jail.conf`, as this file has configuration options such as the services to defend, ban duration, etc.

I’ll make a copy of it named `jail.local`. You aren’t supposed to edit the existing `jail.conf`, as it could get wiped during package updates.

I’ll change the bantime to 1 minute, as I’m going to lock myself out by attempting to bruteforce the SSH service again.

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

sudo vim /etc/fail2ban/jail.local
# "bantime" is the number of seconds that a host is banned.
bantime  = 1m
```

Starting the fail2ban service: `sudo systemctl start fail2ban`

Let’s confirm it’s working, and is protecting our SSH server. By default, it should time out an IP address after 5 incorrect login attempts.

![Hydra Attack Blocked](../../assets/images/ubuntuproject/10.png)

As you can see, it successfully blocked my connection after a few failed login attempts. This alone would make bruteforcing the service substantially more time consuming, and a much less appealing target.

* * *

## Configuring SSH - Public Key Authentication

Password based authentication is less secure than using keys. It’s possible to guess a password - guessing a key isn’t really viable. In addition,  if we want we can encrypt the private key with a passphrase so it’s required every time we authenticate with the key - this adds another layer of security.

It’s relatively simple to configure public key authentication.

First, we generate a public and private key pair.  Next, we will need to add the public key to the `/home/<user>/.ssh/authorized_hosts` file on the server (creating the directory and file if it doesn’t exist), and keep our private key secure on our client.

That’s it; we’ll be able to authenticate via public keys afterwards.

Note: This example will overwrite the existing contents of the `authorized_keys` file. Append the public key to the end of the existing file to avoid erasing existing allowed keys.

```bash
ssh-keygen -b 4096   
scp id_rsa.pub balloan@10.0.0.10:/home/victim/.ssh/authorized_keys
```

![Public Key Authentication](../../assets/images/ubuntuproject/11.png)

We’re now able to SSH into the server using our private key `id_rsa` instead of a password. It’s important to note that password authentication is still enabled - let’s disable it and only allow authentication to the server via public keys. 

We might as well disable allowing EmptyPasswords to authenticate here as well - it won’t affect public key authentication, but if we had a user account with an empty password for some reason, this blocks it.

```bash
sudo vim /etc/ssh/sshd_config

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no  
PermitEmptyPasswords no
```

Next, we need to restart the SSH service, and we should be good to go: `sudo systemctl restart sshd`

Let’s verify that the server will no longer allow me to connect with a password. I temporarily moved my `id_rsa` on my Kali machine so it wouldn’t default to using my private key to authenticate.

![Permission Denied](../../assets/images/ubuntuproject/12.png)

![Nmap Scan](../../assets/images/ubuntuproject/13.png)