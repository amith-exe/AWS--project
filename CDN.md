# CDN
<img width="735" height="342" alt="image" src="https://github.com/user-attachments/assets/9cd5cfc3-6507-450d-a839-4f8b45c9aa9e" />
<img width="725" height="407" alt="image" src="https://github.com/user-attachments/assets/f3e7cfcf-5aab-40c6-832e-dde0a05f9701" />
<img width="732" height="367" alt="image" src="https://github.com/user-attachments/assets/74b11465-c273-4c6d-aec5-05a94bd0ea9d" />
# AWS CloudFront Distribution Notes

## What is CloudFront?

**Amazon CloudFront** is AWS's **Content Delivery Network (CDN)**.

A CDN stores copies of your content at AWS Edge Locations around the world so users receive content from the nearest location instead of the origin server.

Example:

```text
User (India)
      ↓
Nearest Edge Location
      ↓
Cached File
```

instead of

```text
User (India)
      ↓
S3 Bucket (US-East-1)
```

This makes websites faster and reduces load on your origin.

---

# Why Do We Need CloudFront?

Suppose your favicon is stored in an S3 bucket.

Without CloudFront:

```text
User
    ↓
S3 Bucket
```

Problems:

* Higher latency for distant users
* Every request reaches S3
* More bandwidth usage
* No edge caching

---

With CloudFront:

```text
User
    ↓
Nearest Edge Location
    ↓
(Cache Hit)
```

If the file isn't cached:

```text
Edge Location
      ↓
S3 Bucket
      ↓
Stores Copy
      ↓
Serves User
```

Future users receive the cached copy.

---

# What is a CloudFront Distribution?

A **CloudFront Distribution** is the configuration that tells CloudFront:

* Where the content is
* How to cache it
* How users access it
* Which security settings to use

When created, CloudFront provides a domain like:

```text
d123456abcdef.cloudfront.net
```

You can immediately use this domain to access your content.

---

# Components of a Distribution

## 1. Origin

The original source of the content.

Examples:

```text
S3 Bucket
EC2
Application Load Balancer
API Gateway
```

Example:

```text
S3 Bucket
      ↓
Origin
```

---

## 2. Edge Locations

AWS data centers distributed worldwide.

They cache content closer to users.

```text
S3
   ↓
CloudFront
   ↓
Edge Locations
   ↓
Users
```

---

## 3. Cache

CloudFront stores frequently accessed files.

Example:

```text
favicon.ico

logo.png

style.css

script.js
```

The first request downloads from S3.

Later requests are served directly from cache.

---

## 4. Distribution Domain

Example:

```text
d12abc34xyz.cloudfront.net
```

Example request:

```text
https://d12abc34xyz.cloudfront.net/favicon.ico
```

---

# Key Configuration Options

## Origin

Where CloudFront fetches content.

Example:

```text
patientping-favicon-bucket
```

---

## Default Root Object

The page served when visiting:

```text
https://domain.com/
```

Usually:

```text
index.html
```

---

## Cache Settings

Controls:

* Cache duration
* Compression
* Cache behavior

---

## Price Class

Determines which AWS Edge Locations are used.

```text
More Edge Locations
      ↓
Better Performance
      ↓
Higher Cost
```

---

# Request Flow

## First Request

```text
User
    ↓
Edge Location

(Cache Miss)
    ↓
S3 Bucket
    ↓
Return File
    ↓
Store In Cache
```

---

## Later Requests

```text
User
    ↓
Edge Location

(Cache Hit)
    ↓
Return File
```

S3 is not contacted.

---

# Benefits of CloudFront

### Faster Content Delivery

Content comes from the nearest edge location.

---

### Reduced S3 Load

Most requests never reach S3.

---

### Lower Latency

Less network distance.

---

### Lower Bandwidth Cost

Cached content reduces requests to the origin.

---

### Global Availability

AWS has hundreds of edge locations worldwide.

---

# Common CloudFront Use Cases

Serve:

```text
Images

Videos

CSS

JavaScript

PDFs

Fonts

Static Websites
```

Can also front:

```text
REST APIs

Load Balancers

EC2 Applications
```

---

# CloudFront Architecture

```text
               Users
                  │
                  ▼
        CloudFront Edge Location
                  │
        Cache Hit │ Cache Miss
                  ▼
             S3 Bucket
                  │
              Static Files
```

---

# Step-by-Step Guide

## Step 1

Navigate:

```text
AWS Console
 ↓
CloudFront
```

Click:

```text
Create Distribution
```

---

## Step 2

Distribution Name

Enter:

```text
patientping-favicon-distro
```

Distribution Type:

```text
Single Website or App
```

Click:

```text
Next
```

---

## Step 3

Configure Origin

Origin Type:

```text
Amazon S3
```

Click:

```text
Browse S3
```

Select:

```text
patientping-favicon-bucket
```

Origin Settings:

```text
Use Recommended Origin Settings
```

---

## Step 4

Configure Cache

Select:

```text
Use Recommended Cache Settings
Tailored For Serving S3 Content
```

---

## Step 5

Security

Leave:

```text
WAF
Default
```

Click:

```text
Next
```

---

## Step 6

Review

Verify all settings.

Click:

```text
Create Distribution
```

Deployment usually takes:

```text
5–15 Minutes
```

Status changes:

```text
In Progress
      ↓
Enabled
```

---

## Step 7

Get Distribution Domain

Example:

```text
d12abc34xyz.cloudfront.net
```

---

## Step 8

Test CloudFront

Open:

```text
https://YOUR_DISTRIBUTION.cloudfront.net/favicon.ico
```

Expected:

The favicon loads successfully.

---

## Step 9

Check Performance

Open Browser Developer Tools:

```text
F12
 ↓
Network
 ↓
Reload Page
```

Observe:

* Load Time
* Response Time
* Cache Behavior

Requests are typically much faster because they're served from a nearby edge location.

---

# Real-World Example

BookMyVenue stores:

```text
Venue Images
Profile Pictures
Icons
Documents
```

Instead of:

```text
User
     ↓
S3
```

Users access:

```text
User
     ↓
CloudFront
     ↓
Nearest Edge
     ↓
S3 (Only if Needed)
```

This improves performance and reduces costs.

---

# CloudFront vs S3

| S3                | CloudFront              |
| ----------------- | ----------------------- |
| Stores files      | Delivers files globally |
| Single AWS Region | Global Edge Network     |
| No caching        | Built-in caching        |
| Higher latency    | Lower latency           |
| Origin storage    | CDN layer               |

---

# Key Takeaways

### CloudFront

```text
AWS Content Delivery Network (CDN)
```

### Distribution

```text
Configuration that tells CloudFront
how to deliver your content.
```

### Origin

```text
Original source of the content
(S3, EC2, ALB, API Gateway)
```

### Edge Location

```text
AWS server that caches content
closer to users.
```

### Cache Hit

```text
Content served from the Edge Location.
```

### Cache Miss

```text
CloudFront fetches content
from the origin.
```

### Distribution Domain

```text
CloudFront-generated URL
used to access your content.
```

---

# Big Picture

```text
            User
              │
              ▼
      CloudFront Edge Location
         │              │
   Cache Hit       Cache Miss
         │              │
         ▼              ▼
      Return File    S3 Bucket
                          │
                   Store Original Files
```

**Rule of Thumb**

```text
S3 Stores Files

CloudFront Delivers Files

Edge Locations Cache Files

Users Receive Content From
The Nearest Edge Location
```
# AWS CloudFront Cache Invalidation Notes

## Why Cache Invalidation is Needed

CloudFront is a **CDN (Content Delivery Network)**.

It caches files at AWS Edge Locations around the world.

Example:

```text
S3 Bucket
      ↓
CloudFront Edge Cache
      ↓
Users
```

This makes websites much faster.

---

# The Problem

Suppose your S3 bucket contains:

```text
favicon.ico
```

CloudFront caches it.

Now you upload a new favicon.

```text
Old Favicon
      ↓
Upload
      ↓
New Favicon
```

Problem:

```text
S3
      ↓
New Version

CloudFront Edge
      ↓
Still Has Old Version
```

Users continue seeing the old cached file.

---

# What is Cache Invalidation?

Cache Invalidation tells CloudFront:

```text
Discard the cached copy.

Fetch a fresh copy
from the origin (S3).
```

Think of it as:

```text
Delete Old Cache
      ↓
Download Latest Version
```

---

# Why Doesn't CloudFront Update Automatically?

CloudFront follows the cache rules (TTL).

Until the cached object expires:

```text
Edge Location
      ↓
Uses Cached Copy
```

It does **not** check S3 every request.

This reduces latency and bandwidth costs.

---

# When Should You Invalidate?

Whenever you update files like:

```text
Images

CSS

JavaScript

HTML

PDFs

Videos
```

Example:

```text
logo.png

favicon.ico

index.html
```

---

# Invalidate Specific Files

Best Practice:

```text
/favicon.ico
```

instead of

```text
/*
```

Why?

Because:

```text
Only One File Changes
      ↓
Only Refresh That File
```

This is faster and more efficient.

---

# CloudFront Request Flow

Before Invalidation

```text
User
     ↓
CloudFront
     ↓
Cached favicon.ico
```

After Updating S3

```text
S3
     ↓
New File

CloudFront
     ↓
Still Old File
```

After Invalidation

```text
Invalidate Cache
      ↓
Edge Deletes Cached Copy
      ↓
Next Request
      ↓
CloudFront Downloads
Latest File From S3
```

---

# Hard Refresh

Browsers also cache files.

Sometimes even after CloudFront is updated, the browser still uses its local cache.

Use:

```text
Windows:
Ctrl + Shift + R

Mac:
Cmd + Shift + R
```

This forces the browser to download the newest file.

---

# Step-by-Step Guide

## Step 1

Download the new favicon.

```bash
curl -o favicon.ico https://storage.googleapis.com/qvault-webapp-dynamic-assets/course_assets/patientping-bw-favicon.ico
```

This saves:

```text
favicon.ico
```

to your current directory.

---

## Step 2

Open:

```text
AWS Console
 ↓
S3
```

Open your bucket:

```text
patientping-favicon-bucket
```

---

## Step 3

Upload the new file.

Click:

```text
Upload
```

Select:

```text
favicon.ico
```

Replace the existing file.

---

## Step 4

Test through CloudFront.

Open:

```text
https://YOUR_DISTRIBUTION.cloudfront.net/favicon.ico
```

Hard refresh:

```text
Ctrl + Shift + R
```

Expected:

```text
Still Old Favicon
```

because CloudFront is still serving the cached version.

---

## Step 5

Open:

```text
AWS Console
 ↓
CloudFront
```

Select your distribution.

---

## Step 6

Open:

```text
Invalidations
```

Click:

```text
Create Invalidation
```

---

## Step 7

Object Path

Enter:

```text
/favicon.ico
```

> Avoid using `/*` unless you need to invalidate the entire cache.

Click:

```text
Create Invalidation
```

---

## Step 8

Wait

Status:

```text
In Progress
      ↓
Completed
```

This usually takes a few minutes.

---

## Step 9

Test Again

Open:

```text
https://YOUR_DISTRIBUTION.cloudfront.net/favicon.ico
```

Hard refresh:

```text
Ctrl + Shift + R
```

Expected:

```text
New Black & White Favicon
```

---

## Step 10

Wait a few minutes before running tests.

CloudTrail needs time to record the invalidation request.

Typically:

```text
2–5 Minutes
```

---

# Why CloudTrail Matters

Every invalidation request is an AWS API call.

CloudTrail records:

```text
Who Created It

When

Which Distribution

Which Paths
```

This helps with auditing and troubleshooting.

---

# Real-World Example

Suppose BookMyVenue updates:

```text
logo.png
```

Without invalidation:

```text
Users
      ↓
Still See Old Logo
```

With invalidation:

```text
Update Logo
      ↓
Invalidate /logo.png
      ↓
CloudFront Downloads
Latest Logo
      ↓
Users See New Logo
```

---

# Best Practices

✅ Invalidate only the files that changed.

```text
/logo.png

/style.css

/favicon.ico
```

instead of:

```text
/*
```

---

Use versioned file names for large applications whenever possible.

Example:

```text
style-v2.css

app-v3.js

logo-v5.png
```

This avoids frequent cache invalidations because the URL changes.

---

Always perform a hard refresh after invalidation to bypass your browser cache.

---

# Key Takeaways

### Cache

```text
CloudFront stores copies of files
at Edge Locations.
```

### Cache Invalidation

```text
Deletes cached files so CloudFront
fetches the latest version from S3.
```

### Hard Refresh

```text
Forces the browser to ignore
its local cache.
```

### Best Practice

```text
Invalidate only the files
that changed.
```

### CloudTrail

```text
Records cache invalidation
API events.
```

---

# Big Picture

```text
          Upload New File
                │
                ▼
            S3 Bucket
                │
      Old Cache Still Exists
                │
                ▼
     Create CloudFront Invalidation
                │
                ▼
     Edge Cache Removes Old Copy
                │
                ▼
      Next User Request Arrives
                │
                ▼
 CloudFront Downloads Latest File
                │
                ▼
        New Content Served
```

## Rule of Thumb

```text
Update File
      ↓
Upload To S3
      ↓
Invalidate Cache
      ↓
Wait Until Completed
      ↓
Hard Refresh Browser
      ↓
Users Receive Latest Version
```
# AWS S3 Presigned URLs Notes

## The Problem

Normally, files in S3 are either:

```text
Public
```

or

```text
Private
```

### Public Bucket

```text
Anyone With The URL
        ↓
Can Access File
```

Problems:

* Anyone can download the file.
* Search engines can index it.
* Users can share the URL forever.
* No access control.

---

### Private Bucket

```text
Private Bucket
      ↓
Access Denied
```

Only AWS users with permission can access it.

But what if we want **only authenticated users** to download a file?

---

# What is a Presigned URL?

A **Presigned URL** is a **temporary, secure URL** that allows access to a private S3 object.

Think of it like:

```text
Movie Ticket
```

It works for a limited time.

Example:

```text
Valid For 15 Minutes
      ↓
Expires
      ↓
Cannot Be Used Again
```

---

# How Presigned URLs Work

Normal Access

```text
User
   ↓
Private S3 Bucket
   ↓
Access Denied
```

Using Presigned URL

```text
Application
      ↓
Generate Presigned URL
      ↓
User Receives URL
      ↓
Downloads File
      ↓
URL Expires
```

---

# What Does a Presigned URL Contain?

Example:

```text
https://bucket.s3.amazonaws.com/file.pdf?
X-Amz-Algorithm=...
X-Amz-Date=...
X-Amz-Expires=3600
X-Amz-Signature=...
```

The URL contains:

* Authentication Signature
* Expiration Time
* AWS Credentials (signed)
* Security Information

AWS verifies these before allowing access.

---

# Why Is It Secure?

The S3 object remains:

```text
Private
```

Only users with the generated URL can access it.

Once the expiration time passes:

```text
Access Denied
```

---

# URL Lifecycle

```text
Generate URL
      ↓
Share With User
      ↓
User Downloads File
      ↓
Expiration Time Reached
      ↓
URL Invalid
```

---

# Common Expiration Times

Examples:

```text
15 Seconds

5 Minutes

1 Hour

24 Hours
```

Choose an expiration based on your application's needs.

---

# Real Production Flow

Example:

BookMyVenue

```text
User Purchases Ticket
         ↓
Backend Verifies Payment
         ↓
Generate Presigned URL
         ↓
User Downloads PDF Ticket
         ↓
URL Expires
```

The ticket stays private in S3.

---

# Why Not Make Everything Public?

Suppose you sell:

```text
Course Videos
PDF Notes
Images
Software
```

If the bucket is public:

```text
One User Pays
      ↓
Shares URL
      ↓
Everyone Downloads Free
```

Bad for business.

---

# Presigned URL Solution

```text
User Logs In
      ↓
Backend Checks Permission
      ↓
Generate Presigned URL
      ↓
Download File
      ↓
URL Expires
```

Even if someone shares the URL:

```text
Expired URL
      ↓
Access Denied
```

---

# Advantages

### Secure

```text
Private Files Stay Private
```

---

### Temporary Access

```text
Access Automatically Expires
```

---

### No Public Bucket Required

```text
Bucket Remains Private
```

---

### Easy To Share

Only users with the generated URL can download the file.

---

# Typical Use Cases

### Online Courses

```text
Students Download Purchased PDFs
```

---

### Media Streaming

```text
Purchased Videos

Music Downloads
```

---

### Software Distribution

```text
Licensed Software Downloads
```

---

### Document Sharing

```text
Contracts

Invoices

Reports
```

---

### Backup Downloads

```text
Export User Data
```

---

# Presigned URL Architecture

```text
User
   │
   ▼
Application Server
   │
Check Authentication
   │
Generate Presigned URL
   │
   ▼
Amazon S3
   │
Generate Signed URL
   │
   ▼
User Downloads File
   │
   ▼
URL Expires
```

---

# Step-by-Step Guide

## Step 1

Open a terminal.

---

## Step 2

Generate a Presigned URL.

```bash
aws s3 presign s3://YOUR_BUCKET_NAME/favicon.ico --expires-in 15
```

Example:

```bash
aws s3 presign s3://patientping-favicon-bucket-x7k9m2/favicon.ico --expires-in 15
```

---

## Step 3

AWS returns a long URL.

Example:

```text
https://bucket.s3.amazonaws.com/favicon.ico?X-Amz-...
```

Copy the URL.

---

## Step 4

Open the URL in your browser.

Expected:

```text
favicon.ico downloads
```

or

```text
favicon loads successfully
```

---

## Step 5

Wait for the expiration time.

Example:

```text
15 Seconds
```

---

## Step 6

Open the same URL again.

Expected:

```text
Access Denied

or

Request Expired
```

because the URL has expired.

---

# How Applications Use Presigned URLs

Example:

```text
User Clicks Download
        ↓
Backend Checks Login
        ↓
Generate Presigned URL
        ↓
Return URL
        ↓
Browser Downloads File
```

The backend never exposes the bucket publicly.

---

# CloudFront + Presigned URLs

Presigned URLs can also be used with CloudFront.

```text
Private S3 Bucket
       ↓
CloudFront
       ↓
Signed URL
       ↓
Fast Download
```

This provides:

* Global CDN speed
* Secure private access
* Temporary authorization

---

# Best Practices

✅ Keep S3 buckets private.

✅ Generate URLs only after authentication.

✅ Use short expiration times.

```text
5 Minutes

15 Minutes

1 Hour
```

instead of:

```text
30 Days
```

---

Never share presigned URLs publicly.

Anyone with the URL can use it until it expires.

---

Generate a new URL whenever a user requests a download instead of creating long-lived links.

---

# AWS CLI Command

Generate a Presigned URL:

```bash
aws s3 presign s3://YOUR_BUCKET_NAME/OBJECT_NAME --expires-in SECONDS
```

Example:

```bash
aws s3 presign s3://patientping-favicon-bucket/favicon.ico --expires-in 300
```

Generates a URL valid for:

```text
5 Minutes
```

---

# Key Takeaways

### Presigned URL

```text
Temporary URL for accessing
a private S3 object.
```

### Security

```text
Bucket Remains Private.
```

### Expiration

```text
Automatically Stops Working
After The Time Limit.
```

### Best Practice

```text
Authenticate User
      ↓
Generate URL
      ↓
Download File
      ↓
Expire URL
```

### Common Use Cases

```text
Course Downloads

Private Documents

Media Files

Software Distribution

User Data Exports
```

---

# Big Picture

```text
          User Requests File
                  │
                  ▼
         Application Server
                  │
     Verify User Permission
                  │
                  ▼
      Generate Presigned URL
                  │
                  ▼
          Amazon S3 Bucket
                  │
                  ▼
         User Downloads File
                  │
          URL Expires Automatically
                  │
                  ▼
            Access Denied
```

## Rule of Thumb

```text
Public Bucket
      ↓
Anyone Can Access

Private Bucket
      ↓
Nobody Can Access

Private Bucket
      +
Presigned URL
      ↓
Only Authorized Users
Can Access Temporarily
```
# AWS CloudFront with Route 53 DNS Notes

## Why Use DNS with CloudFront?

When you create a CloudFront distribution, AWS gives it a domain like:

```text
d1234567890abcdef.cloudfront.net
```

This works, but it is long and difficult to remember.

Instead, create your own domain name:

```text
cdn.patientping.internal
        ↓
CloudFront Distribution
```

Users access the friendly domain while Route 53 forwards the request to CloudFront.

---

# CNAME vs ALIAS

### CNAME Record

Maps one domain name to another domain name.

Example:

```text
cdn.patientping.internal
        ↓
d1234567890abcdef.cloudfront.net
```

Use when pointing a subdomain to CloudFront.

---

### ALIAS Record

AWS-specific DNS record.

Benefits:

* Can point the root domain directly to CloudFront
* No additional Route 53 query cost
* Better integration with AWS

Example:

```text
patientping.com
       ↓
CloudFront
```

**Note:** ALIAS records require a **Public Hosted Zone**. Since `patientping.internal` is a **Private Hosted Zone**, use a **CNAME** instead.

---

# Architecture

```text
User
   ↓
cdn.patientping.internal
   ↓
Route 53 (CNAME)
   ↓
CloudFront Distribution
   ↓
S3 Bucket
```

---

# Step-by-Step Guide

## Step 1

Navigate:

```text
AWS Console
 ↓
Route 53
 ↓
Hosted Zones
```

---

## Step 2

Open your hosted zone:

```text
patientping.internal
```

---

## Step 3

Click:

```text
Create Record
```

---

## Step 4

Configure the record.

Record Type:

```text
CNAME
```

Name:

```text
cdn
```

Value:

```text
YOUR_CLOUDFRONT_ID.cloudfront.net
```

Example:

```text
d1234567890abcdef.cloudfront.net
```

---

## Step 5

Click:

```text
Create Record
```

---

## Verify

Your DNS record should look like:

```text
cdn.patientping.internal
            ↓
d1234567890abcdef.cloudfront.net
```

---

# Production Note

For a real public website:

```text
cdn.example.com
```

you would also configure:

* Alternate Domain Name (CNAME) in CloudFront
* SSL/TLS Certificate using AWS Certificate Manager (ACM)

This allows users to securely access:

```text
https://cdn.example.com
```

instead of the default CloudFront domain.

---

# Key Takeaways

### CloudFront Domain

```text
AWS-generated CDN URL.
```

### CNAME

```text
Maps one domain name
to another domain name.
```

### ALIAS

```text
AWS Route 53 record
used mainly with Public Hosted Zones.
```

### Best Practice

```text
Private Hosted Zone
→ Use CNAME

Public Hosted Zone
→ Prefer ALIAS
```
