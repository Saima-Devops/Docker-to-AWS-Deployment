# Docker to AWS Deployment | ECS + ECR Real-World Project 

## A Node.js-based Todo List

### Step#1: Launch an EC2 for Docker

1. Create a simple **EC2** with Ubuntu OS
2. t2.micro (free-tier)
3. No inbound Rules needed except SSH
4. Create **Instance**

<img width="1899" height="385" alt="image" src="https://github.com/user-attachments/assets/485ed8de-a2c7-45c5-b641-3a6de49e67dc" />

----

###  Step#2: Connect EC2 with SSH Client

1. Copy the ssh command and paste into **ssh client** (like Putty, MobaXterm, iTerm etc..)
2. update the server.
```bash
sudo apt update
```

----

###  Step#3: Clone the entire code base of your project: 

```bash
git clone <repo link>
ls
cd <project folder>
```
----

###  Step#4: Build the Dockerfile & Push to ECR

1. Go to **AWS** dashboard> **ECR** > **Create a Repository**
2. Create a **Public Repository** with any name, with Linux all types of architectures (ARM, ARM 64, x86, x86-64)

<img width="1913" height="763" alt="image" src="https://github.com/user-attachments/assets/01af7815-d498-4d75-bca8-c8339abe47a1" />

<br>
<br>

3. Click on **View Push Commands**
<br>
<img width="1908" height="333" alt="image" src="https://github.com/user-attachments/assets/7e075319-b993-446a-9be2-b03abba85fbe" />

<img width="1058" height="367" alt="image" src="https://github.com/user-attachments/assets/e613cfbf-c01e-45e6-8621-5a57011819be" />

---

### Step#5: To install Docker 

Go to EC2 instance's client shell

   ```bash
   sudo apt install docker.io
   ```
 <br>
 
> But `docker` will not run, beacause `ubuntu user` has no permission. To give permission, **add ubuntu user into docker group**:

```bash
sudo usermod -aG docker ubuntu
```
<br>
 
OR if you don't know the name of user, check with whoami or just use `$USER` with the above command like:

```bash
sudo usermod -aG docker $USER

After that,

sudo reboot
```
<br>

Connect with the **instance** again

 <br>

<img width="1130" height="379" alt="image" src="https://github.com/user-attachments/assets/e19672c7-3c70-4678-b486-dfa4207ac497" />

---

### Step#5: Install aws cli on Linux ec2 instance

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

✅Docker insatlled\
✅aws CLI insatlled

----

### Step#5: Retrieve an authentication token and authenticate your Docker client 

Retrieve an authentication token and authenticate your Docker client to your registry. Use the AWS CLI:

<img width="1058" height="367" alt="image" src="https://github.com/user-attachments/assets/e613cfbf-c01e-45e6-8621-5a57011819be" />

Copy the cammand and paste into docker-ec2 CLI

> But the command needs `credentials` from the AMI user to run this command, so:

 - Go to `AMI` > `Security` > `Create Access Key` > Command Line Interface > Create Key
 
 - `aws login`/ `aws configure`

 - Give the `Access Key` and `Secret Access Key` here
   
 - Give ECR permissions to the user
   
 - Permission Policies > Add Permissions > Attach Permissions directly

 - Search `AmazonElasticContainerRegistryPublicFullAccess`

 - Search `AWSServiceRoleForECS`
   
 - You should have `IAM Full Access` to give these permissions

 - Assign all permssions to this Role.

---

#### Troubleshooting
 
1. `iam:CreateServiceLinkedRole` is a permission (not a role)
2. You add it via IAM policy (JSON)
3. If blocked → organization/admin restriction (Request to grant the permissions)

<img width="1469" height="246" alt="image" src="https://github.com/user-attachments/assets/c539bee7-5e0a-47fe-9c43-9eadc70d5bed" />

---

**What if you can't find `AWSServiceRoleForECS`**

- Go to Amazon Web Services Console → IAM
- Click Users > find your user for eg. user-01
- Go to Permissions
- Click Add permissions
- Choose Attach policies directly
- Click Create policy
- Create custom policy

Go to the **JSON tab** and paste:

```bash
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:CreateServiceLinkedRole",
      "Resource": "*"
    }
  ]
}
```
<br>

Click: `Next`
Name it: `AllowServiceLinkedRoleCreation`
Create policy

---

#### Attach the policy

Go back to your user
Click Add permissions
Attach the new policy you just created

✅ All Permissions Done.

---

- Copy the command from **public ECR Registry** `push commands for node-app`
- Paste the command, this time it will be successful

<img width="1064" height="284" alt="image" src="https://github.com/user-attachments/assets/93b39ab9-2f68-4e94-813f-bc4eb57b0698" />

----

### Step#6: Build your Docker image

Build your Docker image using the following command.
 
```bash
cd Docker-to-AWS-Deployment
docker build -t node-app .
```

✅Docker Image Built

----

### Step#7: Tag the Image to Push to Repository

After the build completes, tag your image so you can push the image to this repository:

```bash
docker tag node-app:latest public.ecr.aws/m0********/node-app:latest
docker images # To view all images
```

✅Image Tag Done.

---

#### Push this tagged image to public ECR repository

```bash
docker push public.ecr.aws/m0********/node-app:latest
```

<img width="1905" height="449" alt="image" src="https://github.com/user-attachments/assets/ca5fec9c-4883-4f7c-88f2-3809aac0dcb8" />

----

###  Step#8: Now Deploy & Manage your container to ECS

Create a **Cluster**

---
<img width="1913" height="493" alt="image" src="https://github.com/user-attachments/assets/daf7febc-aa7d-4c5b-8fa4-c5517d497dad" />

<img width="540" height="334" alt="image" src="https://github.com/user-attachments/assets/7fe7e22d-4f47-4411-bd50-d54abf13d8f3" />

---

- Farget Only✅
- Use Container Insights✅ 
- Choose Recommended Option✅ 
- Done✅

---
<img width="1915" height="513" alt="image" src="https://github.com/user-attachments/assets/2a530722-b380-4eb5-b10f-78129523b4f7" />

---

###  Step#9: How to Create a Task Definition

Now there's no task running:

---

<img width="1911" height="810" alt="image" src="https://github.com/user-attachments/assets/1f940b22-ccd2-4032-b798-b61f7b925ef6" />

-----

#### Create a New Task Definition

<img width="1919" height="479" alt="image" src="https://github.com/user-attachments/assets/371d7270-98b7-41b3-b245-dc7242ecee02" />

<img width="1856" height="811" alt="image" src="https://github.com/user-attachments/assets/e95d153f-4954-4a1e-9486-1e23d9cfc2ce" />

---

> If your app has a chance to get more traffic, then choose 8 GB memory else 3 GB is enough.

-----

#### Create Task Role

go to IAM > Roles > Use Existing Policy `AWSServiceRoleforECS` > Create Role

<img width="1544" height="171" alt="image" src="https://github.com/user-attachments/assets/b5a57e9e-b815-4d02-966d-dc4f44dba17d" />

<img width="1905" height="857" alt="image" src="https://github.com/user-attachments/assets/abb6e74d-1bca-48ce-9c6a-7f36c9712782" />

<img width="1898" height="845" alt="image" src="https://github.com/user-attachments/assets/cadedc9f-0a26-4b4f-bc89-dbfba2a8038a" />

---

#### To get ImageURI:

- Select Image

- Click `View in Gallery`

---

<img width="1187" height="612" alt="image" src="https://github.com/user-attachments/assets/b8a9b49f-57e1-45b7-a2c9-4e3641d4409c" />

---

Copy the **URI** and paste in **Task Role**

<br>

<img width="1505" height="622" alt="image" src="https://github.com/user-attachments/assets/98cd5d59-85c3-41cd-b075-367edb718531" />

<br>

---

**Port Mapping**: 80 and 8000

<br>

<img width="1276" height="284" alt="image" src="https://github.com/user-attachments/assets/b6735dbe-637f-4630-b0c7-494a2d643e31" />

<br>

---


#### Resource allocation limits

<img width="1473" height="219" alt="image" src="https://github.com/user-attachments/assets/6f3818d8-4c78-49c6-8371-2cafd07fbc3d" />

<br>

---


#### Use Log Collection

<img width="1380" height="545" alt="image" src="https://github.com/user-attachments/assets/52d7cbb4-b5a8-41d6-b177-6749fe2e1f54" />

<br>
<br>

**✅ Task Definition is ready**


<img width="1901" height="856" alt="image" src="https://github.com/user-attachments/assets/fe09722a-e441-486f-82cc-3d80c576eff4" />

-----

###  Step#10: Run the Node Cluster

Now we have to run this task on our `Node Cluster`:

1. Select **task definition**
2. Deploy > Run Task
3.Select the task or it will be autoselected
4. Leave all settings as default
5. **Fargate** is already selected
6. Create
7. Refresh the Task if it's in Pending state
8. Task is Running

✅Done!

<br>

<img width="1524" height="412" alt="image" src="https://github.com/user-attachments/assets/41da9cfd-63e9-4b56-bd01-942ee00cf3a2" />

<img width="1504" height="394" alt="image" src="https://github.com/user-attachments/assets/b9f3c8d1-23bc-4e80-8e7c-03415159bc68" />

-----

###  Step#10: Check  the app in Browser

**To View:**

- Click on the Task

<img width="1898" height="749" alt="image" src="https://github.com/user-attachments/assets/86598ed0-a4f5-463b-8a05-f6480c8ddfdf" />

---

Select the **Public IP** address and open with port# **8000**

---

<img width="1894" height="512" alt="image" src="https://github.com/user-attachments/assets/deec9875-d72f-4164-a57c-12ac1c098b88" />

<img width="1919" height="858" alt="image" src="https://github.com/user-attachments/assets/d14d33a2-3bfd-48f0-bede-f2966ad6d24d" />


----

#### Troubleshooting:

**Case# 01: If your app is not opening:**

Select `ENI ID`

---

<img width="1480" height="256" alt="image" src="https://github.com/user-attachments/assets/8e1b266d-be83-4386-8c42-f030b51a8b05" />

---

Select **Security Group** edit **Inbound Rules** and **add port 8000** as custom TCP from anywhere

---

<img width="1906" height="638" alt="image" src="https://github.com/user-attachments/assets/1168a26a-96d8-49b6-82ef-6c7729b59bf1" />

----



**Case# 02: If you the Code base has been changed by dev team**

**Redeploy (same as before)**

After changes:

```bash
docker build -t node-app .
docker tag node-app:latest 121846058867.dkr.ecr.ap-south-1.amazonaws.com/node-app:latest
docker push 121846058867.dkr.ecr.ap-south-1.amazonaws.com/node-app:latest
```

Then:

- New ECS task revision
- Update service

---

✅Done!

---

###  Step#10: To View the Logs

1. Select **CloudWatch**
2. Logs > Log Management
3. Select your Log & Log Stream

<img width="1919" height="722" alt="image" src="https://github.com/user-attachments/assets/c0b7276c-a634-498a-b0c1-e61f1e9a930e" />

---

#### To Check how my serverless infrastructure is performing:

<br>

Select the Logs for **Node Cluster Performance**

---

<img width="1911" height="812" alt="image" src="https://github.com/user-attachments/assets/6abfe26e-809c-410e-8c2e-8a51eb0911a3" />

<img width="1529" height="378" alt="image" src="https://github.com/user-attachments/assets/416ba359-3206-445b-90c2-eb64a02af4a5" />

<img width="1893" height="814" alt="image" src="https://github.com/user-attachments/assets/5a868306-0722-41a4-bbdc-96b6b0de421a" />

----

