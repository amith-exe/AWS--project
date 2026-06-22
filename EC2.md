# EC2
<img width="743" height="422" alt="image" src="https://github.com/user-attachments/assets/ca7c3870-6ac8-4f90-a4ff-a74e7b656574" />
# SSH Key Generation & Importing to AWS EC2

*A Complete Guide for Windows and Linux*

---

# What is an SSH Key?

SSH (Secure Shell) is a protocol used to securely connect to remote machines over a network.

Instead of logging in with passwords, SSH uses **public-key cryptography**.

When you generate an SSH key pair, two files are created:

```
Private Key  ← Keep Secret
Public Key   ← Safe to Share
```

Example:

```
patientping-key       → Private Key
patientping-key.pub   → Public Key
```

---

# How SSH Authentication Works

```text
Your Computer                     AWS EC2 Instance
-------------                    -----------------
Private Key  ------------------>  Verifies using Public Key
                Authentication Successful
```

1. You keep the **private key** on your computer.
2. The **public key** is stored on the server (EC2).
3. During login, the server verifies that you own the matching private key.
4. No password needs to be transmitted over the internet.

---

# Why Use SSH Keys Instead of Passwords?

### More Secure

* Resistant to brute-force attacks
* Uses strong cryptography

### Convenient

* No need to type passwords repeatedly
* Easy automation and scripting

### Widely Supported

Used by:

* AWS EC2
* GitHub
* GitLab
* Linux Servers
* Docker Hosts
* Kubernetes Nodes

---

# Step 1: Generate an SSH Key Pair

## Windows (PowerShell)

```powershell
ssh-keygen -t ed25519 -C "patientping-key" -f $HOME\.ssh\patientping-key
```

---

## Linux/macOS

```bash
ssh-keygen -t ed25519 -C "patientping-key" -f ~/.ssh/patientping-key
```

---

# Command Breakdown

```bash
ssh-keygen
```

SSH utility used to:

* Generate keys
* Manage keys
* Convert key formats

---

## `-t ed25519`

Specifies the key algorithm.

```bash
-t ed25519
```

### Why Ed25519?

* Faster than RSA
* Smaller key size
* Strong security
* Modern recommended standard

AWS, GitHub, and modern Linux systems all support Ed25519.

---

## `-C`

Adds a comment inside the public key.

```bash
-C "patientping-key"
```

Examples:

```bash
-C "github-amith"
-C "production-server"
-C "aws-key"
```

The comment helps identify what the key is used for.

---

## `-f`

Specifies where to save the key files.

Windows:

```powershell
-f $HOME\.ssh\patientping-key
```

Linux:

```bash
-f ~/.ssh/patientping-key
```

---

# Understanding `$HOME` and `~`

## Windows

```powershell
$HOME
```

Example:

```text
C:\Users\amith
```

Therefore:

```powershell
$HOME\.ssh\patientping-key
```

becomes:

```text
C:\Users\amith\.ssh\patientping-key
```

---

## Linux

```bash
~
```

or

```bash
$HOME
```

Example:

```text
/home/amith
```

Therefore:

```bash
~/.ssh/patientping-key
```

becomes:

```text
/home/amith/.ssh/patientping-key
```

---

# Files Created

After running:

```bash
ssh-keygen -t ed25519 -C "patientping-key" -f ~/.ssh/patientping-key
```

Two files are created:

### Private Key

```text
patientping-key
```

Contains:

```text
-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
```

Keep this secret.

Never:

* Upload to GitHub
* Email it
* Share it

---

### Public Key

```text
patientping-key.pub
```

Contains something like:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA.... patientping-key
```

This file is safe to share.

---

# Viewing the Public Key

## Windows

```powershell
Get-Content $HOME\.ssh\patientping-key.pub
```

or

```powershell
cat $HOME\.ssh\patientping-key.pub
```

---

## Linux

```bash
cat ~/.ssh/patientping-key.pub
```

---

# Step 2: Import Public Key into AWS EC2

## Windows

```powershell
aws ec2 import-key-pair `
  --key-name "patientping-key" `
  --public-key-material fileb://$HOME/.ssh/patientping-key.pub
```

---

## Linux

```bash
aws ec2 import-key-pair \
  --key-name "patientping-key" \
  --public-key-material fileb://~/.ssh/patientping-key.pub
```

---

# Command Breakdown

## `aws ec2 import-key-pair`

Tells AWS:

> "Upload my locally generated public key and create an EC2 Key Pair."

---

## `--key-name`

```bash
--key-name "patientping-key"
```

This is simply the name shown inside AWS.

Example:

```text
patientping-key
production-key
github-server-key
```

You select this key when launching EC2 instances.

---

## `--public-key-material`

```bash
--public-key-material
```

Specifies that public key data is being provided.

---

## `fileb://`

```bash
fileb://~/.ssh/patientping-key.pub
```

Means:

> Read the file locally and send its contents as raw binary data.

This avoids encoding issues, especially on Windows.

---

# What Happens Internally?

## Step 1

Generate keys:

```text
Private Key
Public Key
```

↓

## Step 2

Upload Public Key:

```text
AWS EC2 Key Pair
```

↓

## Step 3

Launch EC2:

```text
EC2 Instance
```

↓

## Step 4

Connect:

```text
Private Key → Authentication → EC2
```

---

# Connecting to an EC2 Instance

## Linux/macOS

```bash
ssh -i ~/.ssh/patientping-key ubuntu@13.51.24.110
```

---

## Windows PowerShell

```powershell
ssh -i $HOME\.ssh\patientping-key ubuntu@13.51.24.110
```

---

# Command Breakdown

## `ssh`

Starts an SSH session.

---

## `-i`

Specifies which private key to use.

```bash
-i ~/.ssh/patientping-key
```

---

## `ubuntu`

Username on the remote server.

Different AMIs use different users:

| Distribution | Username |
| ------------ | -------- |
| Ubuntu       | ubuntu   |
| Amazon Linux | ec2-user |
| Debian       | admin    |
| CentOS       | centos   |

---

## `13.51.24.110`

Public IP address of the EC2 instance.

---

# Real-World Example: Production and Staging Servers

## Production Key

```bash
ssh-keygen -t ed25519 -C "production-server" -f ~/.ssh/prod_key
```

Import:

```bash
aws ec2 import-key-pair \
  --key-name "prod-key" \
  --public-key-material fileb://~/.ssh/prod_key.pub
```

---

## Staging Key

```bash
ssh-keygen -t ed25519 -C "staging-server" -f ~/.ssh/staging_key
```

Import:

```bash
aws ec2 import-key-pair \
  --key-name "staging-key" \
  --public-key-material fileb://~/.ssh/staging_key.pub
```

This keeps environments isolated and improves security.

---

# Using SSH Keys with GitHub

Generate:

```bash
ssh-keygen -t ed25519 -C "github-amith" -f ~/.ssh/github_ed25519
```

View public key:

```bash
cat ~/.ssh/github_ed25519.pub
```

Copy and paste the output into:

```
GitHub
↓
Settings
↓
SSH and GPG Keys
↓
New SSH Key
```

After that:

```bash
git clone git@github.com:username/repository.git
git push
git pull
```

No passwords required.

---

# Best Practices

✅ Use `ed25519`

✅ Keep private keys secret

✅ Create separate keys for different environments

✅ Backup important private keys securely

❌ Never upload private keys to GitHub

❌ Never share private keys with anyone

❌ Never store private keys inside your project repository

---

# Quick Revision

```text
ssh-keygen
    ↓
Private Key + Public Key
    ↓
Import Public Key to AWS
    ↓
Launch EC2
    ↓
Connect using Private Key
    ↓
Authentication Successful
```
