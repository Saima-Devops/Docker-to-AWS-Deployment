# Docker to AWS Deployment | ECS + ECR Real-World Project 

## A Node.js-based Todo List

### Step#1: Launch an EC2 for Docker

1. Create a simple EC2 with Ubuntu OS
2. t2.micro (free-tier)
3. No inbound Rules needed except SSH
4. Create Instance

----

###  Step#2: Connect EC2 with SSH Client

1. Copy the ssh command and paste into ssh client (like Putty, MobaXterm, iTerm etc..)
2. update the server.
```bash
sudo apt update
```
----

###  Step#3: Clone the entire code base of your project: 

git clone <repo link>

