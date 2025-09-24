# Project: React SPA Deployment on AWS (S3 + CloudFront + HTTPS + Route 53 Domain)

## 1. Project Overview
This project demonstrates deploying a React Single Page Application (SPA) on AWS S3 with CloudFront CDN for secure and scalable delivery, using a custom domain purchased from Route 53.

     - Frontend: React SPA

     - Hosting: AWS S3 (private bucket)

     - CDN + Security: CloudFront with Origin Access Control (OAC)

     - Domain: Route 53

     - HTTPS: ACM certificate


## 2. Prerequisites

     - AWS Account

     - React project ready (npm run build)

     - AWS CLI (optional)

     - Budget for domain purchase (Route 53)



## 3. Steps

### Step 1: Buy a Domain in Route 53

     - Go to Route 53 → Domains → Register Domain

     - Search for desired domain, e.g., wasayali.com

     - Complete registration and payment

     - Domain becomes available in Route 53 hosted zones


### Step 2: Create S3 Bucket

     - Go to S3 → Create Bucket


     - Name: awss3deploy (bucket name must be globally unique)


     - Region: us-east-1


     - Block all public access → Yes (we will use CloudFront OAC)


     - Enable static website hosting (optional for testing)









### Step 3: Upload React Build
In React project, run:

     - npm run build

     - Upload all contents of build/ to S3 bucket

<img width="1104" height="548" alt="Screenshot from 2025-09-24 17-09-31" src="https://github.com/user-attachments/assets/631a330c-c294-4568-8b80-cbd592e22654" />


     - Confirm files exist:


     index.html, favicon.ico, static/ (JS, CSS), manifest.json







### Step 4: Create CloudFront Distribution

     - Origin Domain: awss3deploy.s3.amazonaws.com (NOT website endpoint)

<img width="837" height="307" alt="image" src="https://github.com/user-attachments/assets/38079d07-fda1-42f6-98c6-fea6aa9bf6e2" />


     - Origin Access Control (OAC): Create → attach → allows CloudFront to fetch private bucket content




     - Default Root Object: index.html


     - Viewer Protocol Policy: Redirect HTTP to HTTPS


     - Path Pattern * → origin = S3 bucket + OAC


     - Enable Origin Shield (optional for caching)















### Step 5: Configure Bucket Policy

     - Make bucket private


     - Add CloudFront OAC access policy (You can directly copy it while attaching (OAC) to Cloud front, screenshot attached in Step 4):



```bash
   {
  "Version": "2008-10-17",
  "Id": "PolicyForCloudFrontPrivateContent",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::awss3deploy/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<ACCOUNT_ID>:distribution/<DISTRIBUTION_ID>"
        }
      }
    }
  ]
}
```


### Step 6: Test the Cloudfront Endpoint and security

     - Go to your Cloudfront distribution and copy the Distribution Domain Name

  
     - Access it from a browser, you should see your react application:




     - Inspect the site, go to Networks tab you should see the Rerouting from S3 to Cloudfront:
 


     - To check that S3 is only exposed to cloudfront, copy the object url of index.html from S3 bucket and browse it, it should give an error:



### Step 7: Request SSL Certificate via ACM

     - Go to AWS Certificate Manager (ACM) → US East (N. Virginia)


     - Request certificate for your domain:
      Eg “wasayali.com”

     - Validate DNS (CNAME) → ACM provides CNAME → add in Route 53 hosted zone


     - Certificate status becomes Issued







### Step 8: Configure CloudFront for Custom Domain + HTTPS

     - CloudFront → Distribution → General → Edit


     - Alternate Domain Names (CNAMEs): wasayali.com


     - SSL Certificate: Custom SSL → select ACM certificate


     - Viewer Protocol Policy: Redirect HTTP to HTTPS




### Step 9: Configure Route 53 DNS

     - Go to Route 53 → Hosted Zones → Select domain


     - Create a CNAME record:


      Name: wasayali.com
      Value: d1a4nc1qowxorh.cloudfront.net (Your Cloudfront endpoint)
      TTL: 300

     - Optional: Create A Record (Alias) → points directly to CloudFront distribution

















### Step 10: CloudFront Invalidation (Optional)

     - After each update to build → CloudFront → Invalidations → Object Paths: /*



     - Ensures updated content served to users



### Step 11: Testing
     - Visit: https://wasayali.com

   Expected behavior:


     - Loads index.html → JS renders SPA pages


     - S3 bucket is private → direct access denied









