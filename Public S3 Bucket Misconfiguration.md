# Public S3 Bucket Misconfiguration — HackerOne Report #2715239

## Overview

In September 2024, I discovered a vulnerability on a major NFT marketplace platform’s creator dashboard that allowed unauthenticated users to upload and access content on a public Amazon S3 bucket. This misconfiguration exposed a wide range of potential risks, including sensitive data leakage, reputational damage, and abuse through arbitrary file uploads.

---

## Vulnerability Summary

**Target:** creators.magiceden.io  
**Asset:** creator-hub-prod.s3.us-east-2.amazonaws.com  
**Type:** Improper Access Control / Public File Upload  
**Report ID:** #2715239  
**Platform:** HackerOne  
**Status:** Closed as Duplicate  

---

## Root Cause

The backend endpoint responsible for generating signed S3 URLs (`/api/upload/signed-url`) did not enforce access restrictions or validate user authentication at the file access level. As a result, files uploaded using signed URLs became publicly accessible via predictable S3 URLs, and the bucket itself lacked restrictions to prevent enumeration or read access from unauthenticated sources.

---

## Steps to Reproduce

### 1. Navigate to Creator Dashboard
Visit `https://creators.magiceden.io/dashboard` and initiate a new listing.

### 2. Upload a File and Intercept Request
Use Burp Suite to intercept the file upload process. Look for a POST request to:

```
POST /api/upload/signed-url HTTP/2
Host: api-creators.magiceden.io
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "fileName": "example_image.jpeg"
}
```

### 3. Extract Signed URL
The server responds with a signed S3 URL like:

```
https://creator-hub-prod.s3.us-east-2.amazonaws.com/example_image_1726204966358.jpeg?X-Amz-Algorithm=...
```

### 4. Upload Arbitrary File
Upload a file directly via `curl`, bypassing the frontend:

```bash
curl -X PUT -T yourfile.php.jpeg "https://creator-hub-prod.s3.us-east-2.amazonaws.com/example_image_1726204966358.jpeg?X-Amz-Algorithm=..."
```

### 5. Public Access Without Authentication
Visit the resulting URL directly in a browser. No login is required to access the file.

Example:
```
https://creator-hub-prod.s3.us-east-2.amazonaws.com/aasdfasdf_pfp_1726204966358.jpeg
```

### 6. Enumerate Other Files (Optional)
Because filenames followed predictable patterns and access controls were not enforced, it was possible to guess and access other files within the bucket.

---

## Impact

- **Sensitive Data Exposure:** Uploaded assets including images and documents were publicly viewable.
- **Arbitrary File Upload:** Malicious actors could upload disguised payloads such as `.php.jpeg`.
- **Bucket Enumeration:** Public access allowed attackers to guess and list other files in the bucket.
- **Reputation Damage:** Public exposure of internal or user-uploaded content could harm user trust and brand credibility.

---

## Resolution Status

While my submission was valid and demonstrated a real vulnerability, HackerOne marked the report as a **duplicate** of a previously submitted issue. Their team responded professionally and acknowledged the issue’s legitimacy.

---

## Lessons Learned

- **Always restrict public read access on S3 buckets.**
- **Implement validation on file uploads (content-type, file name, size).**
- **Ensure signed URLs expire quickly and are tied to authenticated sessions.**
- **Do not expose internal file structure or naming conventions.**

---

## Reflections

Although this report was closed as a duplicate, it was a strong learning experience that deepened my understanding of AWS S3 security, API abuse, and responsible disclosure workflows. It remains a great portfolio example for real-world cloud misconfiguration testing.

