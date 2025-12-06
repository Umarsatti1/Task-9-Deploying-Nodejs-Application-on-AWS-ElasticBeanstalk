
# Deploying a Simple Node.js Application on AWS Elastic Beanstalk

This project demonstrates how to deploy a lightweight Node.js application using **AWS Elastic Beanstalk**, configure environment settings, enable auto scaling, and monitor application health through AWS CloudWatch.

---

## Architecture Diagram

<p align="center">
  <img src="./diagram/Architecture Diagram.png" alt="Architecture Diagram" width="850">
</p>

---

## Project Overview

This guide walks through the complete process of preparing, deploying, updating, monitoring, and finally cleaning up a Node.js application hosted on Elastic Beanstalk. It is designed for beginners who want hands‑on experience deploying applications in a managed AWS environment.

---

## Features Implemented

- Deployment of a Node.js Express app  
- Elastic Beanstalk application & environment creation  
- Auto Scaling configuration using CPU‑based CloudWatch Alarms  
- Enhanced Health Monitoring  
- Application versioning & redeployment  
- Environment variable configuration  
- Testing via Elastic Beanstalk DNS endpoints  
- Cleanup of environments & resources  

---

## Project Structure

```
├── app.js
├── package.json
├── public/
│   └── index.html
└── README.md
```

---

## Application Code Summary

### `app.js`
A simple Express server that:
- Serves static files from a `public/` directory  
- Listens on the port provided by Elastic Beanstalk via the `PORT` environment variable  

### `public/index.html`
A simple static web page served by the Express and Node.js application.

---

## Prerequisites

- AWS Account  
- IAM user with EB & EC2 access  
- Node.js installed locally  
- AWS CLI (optional, but useful)  

---

## Task Breakdown

### **Task 1: Prepare Node.js Application**
Install dependencies and ensure the app runs locally.

---

### **Task 2: Create Elastic Beanstalk Application**
Using the AWS Console:
- Navigate to Elastic Beanstalk
- Click **Create Application**
- Provide application name, description, and optional tags

---

### **Task 3: Create Elastic Beanstalk Environment**
- Select **Web Server Environment**
- Choose **Node.js** platform
- Upload initial application bundle (ZIP file)
- Launch environment and wait until health = **OK**

This step creates:
- EC2 instance  
- Load balancer  
- Auto Scaling group  
- CloudWatch alarms  
- Service role  

---

### **Task 4: Update Application Code & Redeploy**

1. Make changes to the application code  
2. Compress the updated application  
3. Go to Elastic Beanstalk → Environment → **Upload and Deploy**  
4. Select the new ZIP and deploy  
5. Verify updates using the Elastic Beanstalk environment URL  

---

### **Task 5: Set Up Monitoring & Alerts**

Elastic Beanstalk automatically created two CloudWatch CPU alarms:

| Alarm | Condition | Purpose |
|-------|-----------|---------|
| **High CPU Alarm** | CPU > 70% for 2 datapoints in 2 minutes | Triggers scale‑up |
| **Low CPU Alarm** | CPU < 30% for 2 datapoints in 2 minutes | Triggers scale‑down |

Auto Scaling uses these alarms to add or remove EC2 instances.

---

### **Task 6: Trigger Auto Scaling (Test)**

Using SSH, CPU load was simulated with:

```
stress --cpu 4 --timeout 180
```

or

```
stress-ng --cpu 0 --cpu-load 95 --timeout 300
```

When CPU remained above 70% for 2 minutes:
- **Auto Scaling launched a new instance**
- Load balancer routed traffic across both instances  

---

### **Task 7: Enhanced Monitoring**

Enabled **Enhanced Health Reporting**, which provides:
- P50–P99 latency metrics  
- HTTP response counts  
- CPU user/sys/idle breakdown  
- Live request throughput  

---

### **Task 8: Clean Up**

To avoid ongoing charges:

1. Go to Elastic Beanstalk → Environments  
2. Select the environment  
3. Click **Actions → Terminate Environment**  
4. Confirm deletion  

---

## Troubleshooting

### **Issue 1: Cannot Enable Managed Platform Updates**
**Error**
```
You can't enable managed platform updates when using the service-linked role AWSServiceRoleForElasticBeanstalk.
```

**Root Cause**  
Default service-linked role does not include required policies for managed updates.

**Solution**  
Create a custom EB service role:  
`aws-elasticbeanstalk-service-role`

Attach:
- `AWSElasticBeanstalkEnhancedHealth`
- `AWSElasticBeanstalkManagedUpdatesCustomerRolePolicy`

After using this custom role, the environment allowed managed updates.

---

### **Issue 2: Auto Scaling Did Not Trigger Immediately**
**Root Cause**
CloudWatch alarms require:
- 2 datapoints  
- Over a 2‑minute period  
- Data delivery latency (up to 1–2 minutes)

**Solution**
Sustain CPU load for **at least 3–4 minutes** to guarantee alarm evaluation.

---

### **Issue 3: Stress-ng Temp Path Error**
**Error**
```
aborting: temp-path '.' must be readable and writeable
```

**Root Cause**  
`stress-ng` required a writable temp path.

**Solution**
Run:
```
stress-ng --temp-path /tmp --cpu 0 --cpu-load 90 --timeout 300
```

---

## Testing the Application

You can test the app with:

```
curl http://nodejs-env.eba-igqtyv3m.us-east-1.elasticbeanstalk.com/ -I
```

This returns HTTP headers confirming successful deployment.

---

## Cleanup Summary

Terminate the environment to shut down:
- EC2 instances  
- Auto Scaling groups  
- Load Balancer  
- CloudWatch alarms  
- EB metadata resources  

This prevents unnecessary AWS charges.

---

## Conclusion

This project successfully demonstrates:
- Packaging & deploying a Node.js application  
- Auto Scaling based on CPU metrics  
- CloudWatch alarm‑driven scaling  
- Environment management through the Elastic Beanstalk Console  

Elastic Beanstalk greatly simplifies infrastructure, while still giving control over scaling, logs, and environment configuration.

---
