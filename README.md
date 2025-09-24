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

<img width="1279" height="637" alt="Screenshot from 2025-09-24 14-50-16" src="https://github.com/user-attachments/assets/e417db55-663e-4a4b-a3ef-3d822e7bb515" />


- Default Root Object: index.html

<img width="1279" height="637" alt="Screenshot from 2025-09-24 14-50-03" src="https://github.com/user-attachments/assets/8007ca9c-81f7-449a-abdb-bcf8ea288f77" />


- Viewer Protocol Policy: Redirect HTTP to HTTPS


- Path Pattern * → origin = S3 bucket + OAC


- Enable Origin Shield (optional for caching)


<img width="1279" height="637" alt="Screenshot from 2025-09-24 14-50-03" src="https://github.com/user-attachments/assets/7b12c325-e6cb-4119-a5ba-6e69be3a82be" />



### Step 5: Configure Bucket Policy

- Make bucket private
     
<img width="1279" height="637" alt="Screenshot from 2025-09-24 14-49-50" src="https://github.com/user-attachments/assets/26447e21-37b7-4f71-bc8f-c3b3bc0b7092" />

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

<img width="531" height="99" alt="image" src="https://github.com/user-attachments/assets/63b4e257-3cd3-46e8-befd-9aaffee03ecc" />

  
- Access it from a browser, you should see your react application:

<img width="1279" height="637" alt="Screenshot from 2025-09-24 14-49-19" src="https://github.com/user-attachments/assets/a41302ea-21a3-4d72-bd50-529625ffe643" />


- Inspect the site, go to Networks tab you should see the Rerouting from S3 to Cloudfront:

 <img width="812" height="249" alt="Screenshot from 2025-09-24 13-41-46" src="https://github.com/user-attachments/assets/b1a26374-c8f5-4513-97e8-0b6adcb9c8b6" />

- To check that S3 is only exposed to cloudfront, copy the object url of index.html from S3 bucket and browse it, it should give an error:

<img width="880" height="238" alt="image" src="https://github.com/user-attachments/assets/14a93892-6b59-4a17-9a92-f5cc0b34cd30" />


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

<img width="1157" height="231" alt="image" src="https://github.com/user-attachments/assets/b38fc488-f0d8-4e72-9f7e-9633bb89dca5" />

- Ensures updated content served to users



### Step 11: Testing
   - Visit: https://wasayali.com

   Expected behavior:


   - Loads index.html → JS renders SPA pages


   - S3 bucket is private → direct access denied


<img width="1279" height="637" alt="Screenshot from 2025-09-24 14-56-33" src="https://github.com/user-attachments/assets/b30fe8dd-845b-4c65-8a0b-28b057db5339" />








