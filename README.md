# 🚀 AWS Auto Scaling Group Setup using Ubuntu & Launch Template

## 🧭 Objective

Create a fully functional **Auto Scaling Group (ASG)** using a **Launch Template** with **Ubuntu**, integrated with an **Application Load Balancer (ALB)**, and implement **Dynamic Scaling Policies** based on CPU utilization.

---

## 🧩 Step 1 — Create Launch Template

1. Go to **EC2 → Launch Templates → Create Launch Template**

<img width="1918" height="968" alt="Screenshot 2025-11-11 111538" src="https://github.com/user-attachments/assets/d8504a68-4386-4966-81d9-8371ac464ee0" />

<img width="1919" height="921" alt="Screenshot 2025-11-11 111753" src="https://github.com/user-attachments/assets/1b378c72-8a55-46d5-933e-a585d3d72d0f" />




2. Fill details:
   - **Name:** `ubuntu-web-template`
   - **AMI:** Ubuntu Server 22.04 LTS
   - **Instance Type:** `t2.micro`
   - **Key Pair:** Choose your key
   - **Security Group:** Allow **HTTP (80)** and **SSH (22)**


 <img width="1919" height="967" alt="Screenshot 2025-11-11 111937" src="https://github.com/user-attachments/assets/06cbd00d-eb2a-4ea7-bf4d-5968fede0546" />
     
3. In **Advanced Details → User Data**, paste:

 ```
   #!/bin/bash
   apt update -y
   apt install apache2 -y
   systemctl enable apache2
   echo "<h1>Hello from Auto Scaling - Ubuntu Instance</h1>" > /var/www/html/index.html
   systemctl start apache2
```
   
<img width="1919" height="971" alt="Screenshot 2025-11-11 112015" src="https://github.com/user-attachments/assets/72b006c3-191c-4f17-96bc-9658536a709b" />

   

4.Click Create Launch Template

<img width="1919" height="964" alt="Screenshot 2025-11-11 112053" src="https://github.com/user-attachments/assets/f18a9614-4719-4a76-8505-2e585dfe3d35" />


🏗️ Step 2 — Create Auto Scaling Group

Go to Auto Scaling Groups → Create ASG

<img width="1918" height="919" alt="Screenshot 2025-11-11 112139" src="https://github.com/user-attachments/assets/13988023-20d1-44b5-8a6f-e61911b55808" />

Select the Launch Template you just created.


Choose your VPC and 2 Availability Zones (e.g., ap-south-1a, ap-south-1b)


Set:

Min capacity: 1

Desired capacity: 1

Max capacity: 3

Skip Load Balancer for now.

Create the ASG.

Verify that one instance is launched and running.


🌐 Step 3 — Create Load Balancer and Target Group
3.1 Create Target Group

Go to EC2 → Target Groups → Create target group

Select:

Target type: Instances

<img width="1919" height="910" alt="Screenshot 2025-11-11 112306" src="https://github.com/user-attachments/assets/086b3f8d-08ca-495d-8830-b42c815798a9" />


Protocol: HTTP

Port: 80

Name: ubuntu-tg

<img width="1919" height="948" alt="Screenshot 2025-11-11 112341" src="https://github.com/user-attachments/assets/b3dc130f-c92c-44e9-a0c8-841cab369677" />


Click Next → Create target group

<img width="1919" height="959" alt="Screenshot 2025-11-11 112356" src="https://github.com/user-attachments/assets/b898d00c-631e-4c61-8d53-1bfecd935c36" />

3.2 Create Application Load Balancer (ALB)

Go to EC2 → Load Balancers → Create Load Balancer → Application Load Balancer

Configure:

Scheme: Internet-facing

Listener: HTTP (Port 80)


<img width="1918" height="919" alt="Screenshot 2025-11-11 112139" src="https://github.com/user-attachments/assets/fd0f5602-4124-4248-8df4-674ec1ff0c42" />


Choose same VPC and Subnets (multi-AZ)

<img width="1918" height="904" alt="Screenshot 2025-11-11 112157" src="https://github.com/user-attachments/assets/318d871c-68db-4986-b117-45ff1938c6b4" />

Attach Target Group (ubuntu-tg)

<img width="1919" height="910" alt="Screenshot 2025-11-11 112306" src="https://github.com/user-attachments/assets/28e0e5fc-193f-4e52-b9fc-6add241e8eda" />

<img width="1919" height="903" alt="Screenshot 2025-11-11 112750" src="https://github.com/user-attachments/assets/6d8bf4ad-265d-49bb-a547-1396c750a067" />

<img width="1919" height="959" alt="Screenshot 2025-11-11 112356" src="https://github.com/user-attachments/assets/2fbc8a7d-67fa-4dee-82ef-a7409ea65ada" />

Create the ALB

<img width="1919" height="923" alt="Screenshot 2025-11-11 112500" src="https://github.com/user-attachments/assets/8aa3a4af-dbc9-4166-8662-4cb7a4838866" />

<img width="1919" height="924" alt="Screenshot 2025-11-11 112532" src="https://github.com/user-attachments/assets/17839481-6a75-4584-b19d-a8e3f8ac88e5" />


3.3 Attach ALB to Auto Scaling Group

Go to your Auto Scaling Group → Edit → Load balancing → Attach to existing target group

<img width="1919" height="899" alt="Screenshot 2025-11-11 112414" src="https://github.com/user-attachments/assets/cca0f60b-dd59-48a0-9468-dbfc6d04a7c0" />



Select ubuntu-tg

Save

✅ Verify:

Visit your ALB DNS name
→ You should see: “Hello from Auto Scaling - Ubuntu Instance”

Health checks should show healthy in the Target Group.

📈 Step 4 — Configure Dynamic Scaling Policy

Go to Auto Scaling Group → Automatic Scaling → Add Policy

<img width="1919" height="926" alt="Screenshot 2025-11-11 114202" src="https://github.com/user-attachments/assets/ca03af12-182b-46e4-bd47-76e0ba215881" />

Choose Target Tracking Scaling Policy

Configure:

Metric: Average CPU Utilization

Target value: 60%

<img width="1919" height="919" alt="Screenshot 2025-11-11 112849" src="https://github.com/user-attachments/assets/1c4a41ed-4e7a-4d92-ba36-94b960663ced" />


Cooldown: 120 seconds

Save

🧪 Step 5 — Test Auto Scaling (Scale-Out and Scale-In)
5.1 SSH into one of your EC2 instances:

ssh -i "your-key.pem" ubuntu@<instance-public-ip>

5.2 Install the stress tool:
sudo apt install stress -y


5.3 Generate high CPU load (fast trigger):

stress --cpu 8 --timeout 90

<img width="1457" height="780" alt="Screenshot 2025-11-11 113317" src="https://github.com/user-attachments/assets/ed992c5c-0b08-4bfa-9a80-0e177d405f15" />


Monitor EC2 → Auto Scaling → Activity

You should see new instances launch (scale-out)

<img width="1919" height="915" alt="Screenshot 2025-11-11 113033" src="https://github.com/user-attachments/assets/1a6d952e-7811-40be-bfbd-8d060367c5ec" />

🏁 Result

✅ Launch Template (Ubuntu + User Data)
✅ Multi-AZ Auto Scaling Group
✅ ALB with Target Group and Health Checks
✅ Dynamic Scaling Policy (CPU-based)
✅ Validation with stress test and cleanup

You can now include this setup in your project report or portfolio as a complete AWS Auto Scaling demonstration.
