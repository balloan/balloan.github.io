---
layout: default
title: SSH Attacking & Hardening
parent: Ubuntu Project
grand_parent: Projects
nav_order: 3
---

## Password Attacks

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
