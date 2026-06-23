# SSH & AWS EC2 Complete Study Notes

**A Beginner to Intermediate Guide (Windows + Linux + AWS)**

---

# Table of Contents

1. What is SSH?
2. Why Do We Need SSH?
3. How SSH Works Internally
4. SSH Architecture
5. Public Key vs Private Key
6. AWS Key Pairs and `.pem` Files
7. Connecting to EC2 Without Config File
8. SSH Config File Explained
9. Connecting to EC2 With Config File
10. Windows vs Linux SSH Setup
11. SSH First-Time Connection and Fingerprints
12. Message of the Day (MOTD)
13. Common SSH Commands
14. Useful SSH Use Cases
15. SSH Troubleshooting
16. Production Best Practices

---

# 1. What is SSH?

SSH stands for:

**Secure Shell**

SSH is a secure protocol used to remotely access and manage another computer over a network.

Think of it like this:

```text
Your Laptop
      ↓
Secure Encrypted Tunnel
      ↓
Remote Linux Server
```

SSH gives you:

* Remote terminal access
* Secure file transfer
* Server administration
* Secure command execution

---

# Real-Life Analogy

Imagine:

```text
Your Laptop = Remote Control
Server = Television
SSH = Secure Wireless Connection
```

Without SSH:

You must physically sit in front of the server.

With SSH:

You can control the server from anywhere.

---

# Why Do Companies Use SSH?

Almost every cloud server is managed through SSH:

* AWS EC2
* Azure VMs
* Google Cloud Compute Engine
* DigitalOcean Droplets
* Raspberry Pi servers
* Corporate Linux servers

---

# 2. Why Do We Need SSH?

Suppose you deploy:

BookMyVenue Backend
NestJS API
PostgreSQL
Nginx

The server may be:

```text
Mumbai
Singapore
Germany
USA
```

You cannot physically visit the server.

SSH lets you:

```text
Laptop
   ↓
Internet
   ↓
AWS Server
```

and administer it remotely.

---

# Things You Can Do Using SSH

Install software:

```bash
sudo dnf install nodejs
```

Clone projects:

```bash
git clone https://github.com/user/project.git
```

Start applications:

```bash
npm install
npm run build
pm2 start dist/main.js
```

Configure servers:

```bash
sudo nano /etc/nginx/nginx.conf
```

Restart services:

```bash
sudo systemctl restart nginx
```

Everything happens through SSH.

---

# 3. How SSH Works Internally

When you run:

```bash
ssh ec2-user@13.207.19.76
```

This happens:

```text
Laptop
    ↓
Find server IP
    ↓
Connect to Port 22
    ↓
Exchange encryption keys
    ↓
Authenticate user
    ↓
Open secure shell session
```

After authentication:

```text
Your Keyboard
      ↓
Encrypted Commands
      ↓
Remote Linux Terminal
```

---

# 4. SSH Architecture

```text
Your Laptop
(SSH Client)
        │
        │ Port 22
        ▼
Internet
        │
        ▼
EC2 Instance
(SSH Server)
```

Client:

Your laptop.

Server:

EC2 Linux machine.

Protocol:

SSH.

Default Port:

22/TCP

---

# 5. Public Key vs Private Key

SSH uses asymmetric cryptography.

Two keys are generated:

## Public Key

Safe to share.

Example:

```text
id_ed25519.pub
```

Installed on server.

---

## Private Key

Must remain secret.

Example:

```text
id_ed25519
bmv-key-1.pem
```

Stored on your laptop.

---

# Analogy

```text
Server = Lock
Private Key = Physical Key
```

Anyone can see the lock.

Only you possess the key.

---

# AWS Key Pair

AWS stores:

Public Key

You download:

Private Key (`.pem`)

AWS never stores your private key.

If you lose it:

You cannot SSH into the server.

---

# 6. What is a .pem File?

PEM:

Privacy Enhanced Mail

In AWS:

`.pem` contains your private SSH key.

Example:

```text
bmv-key-1.pem
```

Think:

```text
.pem = Digital Key
```

Without it:

```text
Permission denied (publickey)
```

---

# 7. Connecting WITHOUT Config File

Windows:

```powershell
ssh -i "C:\Users\amith\Downloads\aws_iam\bmv-key-1.pem" ec2-user@13.207.19.76
```

Linux:

```bash
ssh -i ~/.ssh/bmv-key-1.pem ec2-user@13.207.19.76
```

---

# Breaking the Command

```bash
ssh
```

SSH program.

---

```bash
-i
```

Identity file.

Use this private key.

---

```bash
bmv-key-1.pem
```

Private key file.

---

```bash
ec2-user
```

Linux username.

---

```bash
13.207.19.76
```

Server IP address.

---

# Internally

SSH does:

```text
Use key
      ↓
Connect to server
      ↓
Authenticate
      ↓
Open terminal
```

---

# Why This Gets Annoying

Imagine:

Production:

```bash
ssh -i prod.pem ubuntu@54.xx.xx.xx
```

Staging:

```bash
ssh -i stage.pem ubuntu@3.xx.xx.xx
```

Testing:

```bash
ssh -i test.pem ubuntu@18.xx.xx.xx
```

Very repetitive.

---

# 8. SSH Config File

SSH config file stores connection settings.

Think:

```text
Contacts App
```

Instead of:

```text
+91 9876543210
```

you save:

```text
Mom
```

Similarly:

Instead of:

```bash
ssh -i key.pem ec2-user@13.207.19.76
```

you write:

```bash
ssh patientping
```

---

# Config File Locations

Windows:

```text
C:\Users\amith\.ssh\config
```

Linux:

```text
~/.ssh/config
```

No extension.

Not:

```text
config.txt
```

Correct:

```text
config
```

---

# Example Config

```sshconfig
Host patientping
    HostName 13.207.19.76
    User ec2-user
    IdentityFile C:\Users\amith\Downloads\aws_iam\bmv-key-1.pem
```

---

# What Each Line Means

## Host

Alias.

```text
patientping
```

Can be:

```text
prod
aws
server
backend
```

---

## HostName

Actual server address.

```text
13.207.19.76
```

Could also be:

```text
api.bookmyvenue.com
```

---

## User

Linux account.

Amazon Linux:

```text
ec2-user
```

Ubuntu:

```text
ubuntu
```

CentOS:

```text
centos
```

Debian:

```text
admin
```

---

## IdentityFile

Path to private key.

```text
C:\Users\amith\Downloads\aws_iam\bmv-key-1.pem
```

---

# How SSH Uses Config

You type:

```bash
ssh patientping
```

SSH:

```text
Open config
      ↓
Find Host patientping
      ↓
Get IP
Get User
Get Key
      ↓
Connect
```

Internally:

```bash
ssh -i key.pem ec2-user@13.207.19.76
```

---

# 9. Connecting WITH Config File

Without config:

```bash
ssh -i key.pem ec2-user@13.207.19.76
```

With config:

```bash
ssh patientping
```

Exactly the same thing.

Config is just a shortcut.

---

# 10. First-Time Connection

You saw:

```text
The authenticity of host can't be established.
ED25519 key fingerprint...
```

SSH says:

"I have never seen this server before."

The fingerprint is:

Server's digital identity.

When you type:

```text
yes
```

SSH saves it in:

Windows:

```text
C:\Users\amith\.ssh\known_hosts
```

Linux:

```text
~/.ssh/known_hosts
```

---

# Why Does SSH Do This?

To prevent:

Man-in-the-Middle attacks.

---

# 11. Message Of The Day (MOTD)

File:

```text
/etc/motd
```

Meaning:

Message Of The Day.

Command:

```bash
echo 'Welcome, PatientPing Server Admin!' | sudo tee /etc/motd > /dev/null
```

---

# Breaking It Down

## echo

Creates text.

```bash
echo 'Hello'
```

Output:

```text
Hello
```

---

## Pipe |

Take output from left.

Send to right.

---

## tee

Writes data into file.

---

## sudo

Administrator privileges.

Required because:

```text
/etc
```

belongs to root.

---

## /dev/null

Discard unnecessary output.

---

# Result

Whenever someone logs in:

```bash
ssh patientping
```

Linux shows:

```text
Welcome, PatientPing Server Admin!
```

---

# Real Company Examples

```text
PRODUCTION SERVER
Changes require approval.
```

```text
Unauthorized access prohibited.
```

---

# 12. Useful SSH Commands

Connect:

```bash
ssh patientping
```

Connect directly:

```bash
ssh user@ip
```

Specify key:

```bash
ssh -i key.pem user@ip
```

Exit:

```bash
exit
```

Copy file TO server:

```bash
scp file.txt user@ip:/home/user
```

Copy file FROM server:

```bash
scp user@ip:/home/user/file.txt .
```

Execute remote command:

```bash
ssh patientping "ls -la"
```

---

# 13. Common Real-World SSH Use Cases

Deploy NestJS:

```bash
ssh patientping
git pull
npm install
npm run build
pm2 restart app
```

Install Node.js:

```bash
sudo dnf install nodejs
```

Configure Nginx:

```bash
sudo nano /etc/nginx/nginx.conf
```

Restart service:

```bash
sudo systemctl restart nginx
```

Check logs:

```bash
pm2 logs
```

---

# 14. Troubleshooting

Connection refused:

```text
Port 22 blocked
```

Timeout:

```text
Security Group
Route Table
Internet Gateway
Public IP
```

Permission denied:

```text
Wrong .pem
Wrong username
```

Host identification changed:

```text
Server recreated
IP changed
```

---

# Production Security

Bad:

```text
SSH
22
0.0.0.0/0
```

Better:

```text
SSH
22
My IP
```

Website:

```text
80
0.0.0.0/0

443
0.0.0.0/0
```

---

# Final Big Picture

```text
Your Laptop
       ↓
SSH Client
       ↓
Internet
       ↓
AWS Security Group
       ↓
Port 22
       ↓
EC2 Linux Server
       ↓
Authenticate using .pem key
       ↓
Open Remote Terminal
       ↓
Administer Linux Server
```

SSH is essentially:

**A secure, encrypted remote terminal that allows you to administer servers anywhere in the world using cryptographic keys instead of passwords.**
