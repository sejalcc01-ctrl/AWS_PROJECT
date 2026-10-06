# Question 1: Static Website Hosting on Amazon S3

**AWS Project Assignment, Set 1 (AWS Only)**

## Problem Statement
Design and deploy a static website on AWS using Amazon S3. Configure the S3 bucket for static website hosting, upload HTML/CSS/JavaScript files, configure the required permissions, and demonstrate the working website URL.

## Project Overview
| Item | Details |
|---|---|
| AWS service | Amazon S3 (static website hosting) |
| Bucket name | `q1-project-buckket` |
| Region | Asia Pacific (Mumbai), `ap-south-1` |
| Website files | `index.html`, `style.css`, `script.js` |
| Access | Public read through a bucket policy |
| Website URL | http://q1-project-buckket.s3-website.ap-south-1.amazonaws.com |

## Architecture
```
 Website files            Amazon S3 bucket                 Visitor's browser
 index.html     Upload   +--------------------------+  HTTP
 style.css    --------->  q1-project-buckket        | <-------------  Website endpoint
 script.js               |  - Static website hosting|
                         |  - Public-read policy    |
                         +--------------------------+
```
S3 stores the files and serves them directly as a website, so no web server has to be managed.

## The Website Files
| File | Purpose |
|---|---|
| `index.html` | Page structure: heading, message text and a **Show current time** button |
| `style.css` | Styling: Arial font, centred layout, light background, orange button |
| `script.js` | Behaviour: `showTime()` writes the current date and time into the page |

## Implementation Steps

### 1. Create the S3 bucket
1. Open the **S3** service in the AWS Management Console and click **Create bucket**.
2. Bucket name: `q1-project-buckket`.
3. Region: **Asia Pacific (Mumbai) ap-south-1**.
4. Untick **Block all public access** and acknowledge the warning.
5. Click **Create bucket**.

### 2. Upload the website files
1. Open the bucket and click **Upload**.
2. Add `index.html`, `style.css` and `script.js` (keep all files in the bucket root).
3. Click **Upload** to finish.

### 3. Enable static website hosting
1. Bucket, **Properties** tab, **Static website hosting**, **Edit**.
2. Choose **Enable** and **Host a static website**.
3. Index document: `index.html`.
4. **Save changes** to get the website endpoint.

### 4. Add the public-read bucket policy
Bucket, **Permissions** tab, **Bucket policy**, **Edit**, then paste:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::q1-project-buckket/*"
    }
  ]
}
```
Click **Save changes**. The policy gives visitors read-only access, so they can view but not change files.

## Testing and Results
Open the website endpoint in a browser:

http://q1-project-buckket.s3-website.ap-south-1.amazonaws.com

| Test | Expected result | Result |
|---|---|---|
| Open the website URL | Page loads with heading and button | Passed |
| Click **Show current time** | Current date and time appear on the page | Passed |

**Before clicking the button**

![Website before click](screenshots/01_website_before_click.png)

**After clicking Show current time**

![Website after click](screenshots/02_website_after_click.png)

## Conclusion
A static website was hosted on Amazon S3 using only a bucket, three files and a bucket policy. Static website hosting serves the files at a public endpoint, the policy grants read-only access, and the setup is low cost, scalable and needs no server to maintain.

## Cleanup
Empty the bucket, then delete it to avoid charges.
