# MultiCloud, DevOps & AI Challenge - Challenge 2 - Part 1 - Docker - Experienced

# CloudMart Setup Guide

## Step 1: Create CloudMart Tables using Terraform

Log in to the EC2 instance

Remove the [`main.tf`](http://main.tf) file used in Challenge 1

```bash
cd terraform-project
rm main.tf
```

Create a new Terraform file with the content below

```bash
nano main.tf
```

Copy and paste the Terraform code below into the file and save it.

```
provider "aws" {
  region = "us-east-1"  # Change to your preferred region
}

# DynamoDB Tables
resource "aws_dynamodb_table" "cloudmart_products" {
  name           = "cloudmart-products"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "id"

  attribute {
    name = "id"
    type = "S"
  }
}

resource "aws_dynamodb_table" "cloudmart_orders" {
  name           = "cloudmart-orders"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "id"

  attribute {
    name = "id"
    type = "S"
  }
}

resource "aws_dynamodb_table" "cloudmart_tickets" {
  name           = "cloudmart-tickets"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "id"

  attribute {
    name = "id"
    type = "S"
  }
}

```

Initialize Terraform:

```
terraform init

```

Review the plan:

```
terraform plan

```

Apply the configuration:

```
terraform apply

```

1. Type "yes" when prompted to create the resources.

## Step 2: Install Docker on EC2

Run the following commands:

```bash
sudo yum update -y
sudo yum install docker -y
sudo systemctl start docker
sudo docker run hello-world
sudo systemctl enable docker
docker --version
sudo usermod -a -G docker $(whoami)
newgrp docker

```

## Step 3: Create the CloudMart Docker Image

### Backend

Create a folder and download the source code:

```bash
mkdir -p challenge-day2/backend && cd challenge-day2/backend
wget https://tcb-public-events.s3.amazonaws.com/mdac/resources/day2/cloudmart-backend.zip
unzip cloudmart-backend.zip

```

Create a `.env` file:

```bash
nano .env

```

Content of the `.env` file:

```
PORT=5000
AWS_REGION=us-east-1
BEDROCK_AGENT_ID=<your-bedrock-agent-id>
BEDROCK_AGENT_ALIAS_ID=<your-bedrock-agent-alias-id>
OPENAI_API_KEY=<your-openai-api-key>
OPENAI_ASSISTANT_ID=<your-openai-assistant-id>

```

Create a Dockerfile:

```bash
nano Dockerfile

```

Content of the Dockerfile:

```
FROM node:18
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["npm", "start"]

```

Build and run the Docker image:

```bash
docker build -t cloudmart-backend .
docker run -d -p 5000:5000 --env-file .env cloudmart-backend

```

### Frontend

Create a folder and download the source code:

```bash
cd ..
mkdir frontend && cd frontend
wget https://tcb-public-events.s3.amazonaws.com/mdac/resources/day2/cloudmart-frontend.zip
unzip cloudmart-frontend.zip

```

Create a `.env` file:

```bash
nano .env

```

Content of the `.env` file:

```
VITE_API_BASE_URL=http://<your-ec2-ip>:5000/api

```

Create a Dockerfile:

```bash
nano Dockerfile

```

Content of the Dockerfile:

```
FROM node:16-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:16-alpine
WORKDIR /app
RUN npm install -g serve
COPY --from=build /app/dist /app
ENV PORT=5001
ENV NODE_ENV=production
EXPOSE 5001
CMD ["serve", "-s", ".", "-l", "5001"]

```

Build and run the Docker image:

```bash
docker build -t cloudmart-frontend .
docker run -d -p 5001:5001 cloudmart-frontend

```

## Step 4: Configure Firewall Rules

Open the following ports in the EC2 security group:

1. Port 5000 (for the backend)
2. Port 5001 (for the frontend)

Now you can access CloudMart remotely using the public IP of your EC2 instance.