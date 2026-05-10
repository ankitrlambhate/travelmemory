Project Overview:
TravelMemory is a full-stack web application deployed on AWS using EC2 instances, Nginx reverse proxy, and an Application Load Balancer (ALB). The application architecture supports scalability by distributing traffic across multiple EC2 instances.

Tech Stack:
Frontend
React.js
Axios

Backend
Node.js
Express.js
Database
MongoDB
Infrastructure
AWS EC2
AWS Application Load Balancer (ALB)
Nginx Reverse Proxy
GitHub

Architecture Overview
User Browser
      ↓
Custom Domain / ALB DNS
      ↓
AWS Application Load Balancer
      ↓
Target Group (Port 80)
      ↓
Multiple EC2 Instances
      ↓
Nginx Reverse Proxy
      ↓
Frontend (Port 3000) + Backend (Port 3001)

Prerequisites
Before deployment, ensure the following:
AWS Account
EC2 Key Pair (.pem file)
Security Groups configured
Node.js installed
Nginx installed
Git installed
MongoDB connection string

Security Group Configuration for EC2 instance:
Allow the following inbound rules:
SSH
22
Anywhere
HTTP
80
Anywhere
Custom TCP
3000
Anywhere
Custom TCP
3001
Anywhere


Steps to setup the deployment:

EC2 Instance Setup
Step 1: Connect to EC2
ssh -i ~/your-key.pem ubuntu@<EC2-PUBLIC-IP>

Step 2: Update Packages
sudo apt update && sudo apt upgrade -y

Step 3: Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
Verify installation:
node -v
npm -v

Step 4: Install Nginx
sudo apt install nginx -y
Start and enable Nginx:
sudo systemctl start nginx
sudo systemctl enable nginx

Clone Repository
git clone https://github.com/UnpredictablePrashant/TravelMemory
cd TravelMemory

Backend Setup
Navigate to Backend
cd backend
Install Dependencies
npm install
Configure Environment Variables
Create a .env file:
MONGO_URI=your_mongodb_connection_string
PORT=3001
Start Backend
npm start
Verify backend:
curl http://<PUBLIC_IP>:3001
curl http://<PUBLIC_IP>:3001/hello
curl http://<PUBLIC_IP>:3001/trips

Frontend Setup
Navigate to Frontend
cd ../frontend
Install Dependencies
npm install

Update Backend URL
Update urls.js with:
export const baseUrl = "const BASE_URL = "http://<PUBLIC_IP>";";

Build Frontend
npm run build

Nginx Reverse Proxy Configuration
Create Nginx Configuration
sudo nano /etc/nginx/sites-available/default
Replace contents with:
server {
    listen 80;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}

Restart Nginx
sudo nginx -t
sudo systemctl restart nginx

Scaling the Application
Create AMI
Open AWS EC2 Console
Select configured EC2 instance which has MERN app deployed
Actions → Image and Templates → Create Image
Create AMI

Launch Multiple Instances
Launch new EC2 instances using the AMI
Ensure security groups are correctly configured and same VPC is selected
Verify application is running on all instances

Application Load Balancer Setup
Create Target Group
Go to EC2 → Target Groups
Create Target Group
Type: Instances
Protocol: HTTP
Port: 80
Health Check Path: /

Register Targets
Add all EC2 instances to the target group.
Verify all targets become healthy.

Create Application Load Balancer
Go to EC2 → Load Balancers
Create Application Load Balancer
Internet-facing
Listener: HTTP (80)
Select subnets
Attach target group

Domain Setup
CNAME Record
Point domain to ALB DNS:
example.com -> my-alb-123.ap-south-1.elb.amazonaws.com

A Record
Point subdomain to EC2 instance:
test.example.com -> EC2_PUBLIC_IP


