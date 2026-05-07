**PHP Website Hosted on AWS — Two-Tier Cloud Architecture**

A fully deployed PHP website hosted on Amazon Web Services, built using a secure two-tier architecture with isolated public and private subnets inside a custom VPC. This project demonstrates real-world cloud networking, EC2 provisioning, database configuration, and inter-subnet communication — all configured from scratch.

Website link: http://54.197.24.61/

 **Architecture Overview**

The infrastructure is built inside a **custom VPC (CIDR: 10.0.0.0/16)** and split into two isolated subnets:
<img width="1764" height="2324" alt="Image" src="https://github.com/user-attachments/assets/63376de7-4068-4738-a42d-294b464b7b2d" />

Traffic from the internet enters through an **Internet Gateway**, hits the public subnet, and the frontend communicates with the backend database over a **private internal connection** between the two subnets.

<img width="607" height="210" alt="Image" src="https://github.com/user-attachments/assets/f8ffb3a1-1dbf-4086-a99d-cbce707c5a40" />


 **Step-by-Step Breakdown**

**1. VPC Setup**
A Virtual Private Cloud (VPC) is created with a custom CIDR block of `10.0.0.0/16`. This gives the project its own isolated network space within AWS — nothing outside can reach resources inside unless explicitly allowed. Think of it as the walls of the entire building before rooms are added.

<img width="1561" height="405" alt="Image" src="https://github.com/user-attachments/assets/603e3585-4d73-4adb-960f-9ee5a7bfebb0" />


 **2. Subnets**
Two subnets are carved out of the VPC's IP range:

- **Public subnet** — accessible from the internet. This is where the frontend PHP files live.
- **Private subnet** — completely shielded from direct internet access. This is where the database server lives.

Separating frontend and backend into different subnets is a core security principle: even if the public-facing server is compromised, the database remains unreachable from the outside.

<img width="1455" height="253" alt="Image" src="https://github.com/user-attachments/assets/5556a0ca-6ce2-44a5-a697-91b133d059f6" />

<img width="1478" height="260" alt="Image" src="https://github.com/user-attachments/assets/05c8150c-c731-4b91-97fc-a0d31a262e1b" />

**3. Internet Gateway**
An **Internet Gateway (IGW)** is attached to the VPC and associated with the public subnet. It acts as the front door — all inbound traffic from users on the internet enters through here and is routed to the public subnet's EC2 instance. Without it, the public subnet would have no path to the outside world.

<img width="1597" height="317" alt="Image" src="https://github.com/user-attachments/assets/703317fc-916d-4cb2-8aad-da538634711a" />

**4. NAT Gateway**
A **NAT (Network Address Translation) Gateway** is attached to the private subnet. While the private subnet has no inbound internet access, the NAT gateway allows the private EC2 to make *outbound* requests — for example, to download software updates or packages — without exposing it to incoming traffic. It translates the private IP to a public one for outbound calls only.

**5. Route Tables**
Two separate **route tables** are configured — one per subnet:

- **Public route table** — has a route sending all internet-bound traffic (`0.0.0.0/0`) to the Internet Gateway.
- **Private route table** — has a route sending outbound traffic (`0.0.0.0/0`) through the NAT Gateway instead.

Route tables are what tell traffic *where to go*. Without them, even with a gateway in place, packets would have no direction.

<img width="1403" height="182" alt="Image" src="https://github.com/user-attachments/assets/d81debe7-35fb-4a8a-b1f5-ab7cd12f1775" />

<img width="1395" height="171" alt="Image" src="https://github.com/user-attachments/assets/17248617-5e2f-4209-85b2-f1504f5e94d7" />

**6. Security Group**
A **security group** is applied to the public subnet's EC2, acting as a virtual firewall. The inbound rules are:

| Type | Protocol | Port | Purpose |
|---|---|---|---|
| SSH | TCP | 22 | Secure terminal access to the EC2 |
| HTTP | TCP | 80 | Serves the website to users |
| RDP | TCP | 3389 | Remote Desktop access to configure the server |

All outbound traffic is allowed. The private subnet has no public-facing security group — it's only reachable via the internal VPC network.

<img width="1428" height="180" alt="Image" src="https://github.com/user-attachments/assets/4234797b-a465-43fc-9c97-7e1228bfda64" />

<img width="1387" height="185" alt="Image" src="https://github.com/user-attachments/assets/8cd61740-c856-494d-a969-6198632968ff" />

**7. EC2 — Public Subnet (Web Server)**
An **EC2 instance** in the public subnet hosts the PHP website files. It has a public IP address (routed through the IGW) and is the only resource users interact with directly. The PHP frontend is uploaded here and served over HTTP.

<img width="1406" height="686" alt="Image" src="https://github.com/user-attachments/assets/567e9367-7d5a-4601-8838-2f7aea52f779" />

8. EC2 — Private Subnet (Database Server)
A **Windows Server 2022 EC2 instance** in the private subnet runs the backend. It has no public IP — it's only accessible from within the VPC. Access is secured using an **encrypted `.pem` key file**, and the instance is managed via **Remote Desktop Connection (RDP)**.

XAMPP is installed on this instance, providing:
- **Apache** — web server layer for phpMyAdmin
- **MySQL** — relational database storing the PHP app's data
- **phpMyAdmin** — browser-based GUI for managing the database

<img width="833" height="543" alt="Image" src="https://github.com/user-attachments/assets/1dd59599-b5ed-4407-85fe-4738299c46b6" />

<img width="1721" height="818" alt="Image" src="https://github.com/user-attachments/assets/132af5af-6b91-4abe-aa32-27d08e366201" />

<img width="1897" height="865" alt="Image" src="https://github.com/user-attachments/assets/f40e7d20-1c8f-4703-9163-232d522fc7e6" />

<img width="1882" height="853" alt="Image" src="https://github.com/user-attachments/assets/ca40071f-7090-4b7f-bc0e-f7cb73b748b6" />

<img width="1917" height="858" alt="Image" src="https://github.com/user-attachments/assets/a3c83148-93b9-4731-bd86-d646251516d8" />

<img width="1896" height="867" alt="Image" src="https://github.com/user-attachments/assets/dffec5a4-92a7-45b2-831c-92ed67962f4e" />

<img width="1915" height="865" alt="Image" src="https://github.com/user-attachments/assets/5774d423-f136-4699-b693-ba8354120294" />

**9. Inter-Subnet Connection**
Once both EC2 instances are running, a connection is established between the public and private subnets using their **private IP addresses** (within the 10.0.0.0/16 range). The PHP frontend queries the MySQL database on the private EC2 over this internal link. No traffic leaves AWS — it stays within the VPC, keeping the database isolated and secure.

How It All Works Together

```
User (browser)
     │
     ▼
Internet Gateway
     │
     ▼
Public Subnet — EC2 (PHP frontend)
     │  Security group: SSH + HTTP + RDP
     │
     │ private IP connection (internal VPC)
     ▼
Private Subnet — EC2 Windows Server 2022
     │  XAMPP: Apache + MySQL + phpMyAdmin
     │
     ▼
NAT Gateway (outbound updates only)
```

The user hits the public IP → the PHP site loads from the public EC2 → PHP queries MySQL on the private EC2 via internal IP → the database responds → the page renders. All backend traffic stays inside the VPC.


**Technologies Used**

- **AWS VPC** — custom network isolation
- **AWS EC2** — compute instances (Linux frontend, Windows Server backend)
- **AWS Internet Gateway** — public internet routing
- **AWS NAT Gateway** — outbound-only private subnet access
- **AWS Security Groups** — firewall rules
- **XAMPP** — Apache + MySQL + phpMyAdmin stack
- **PHP** — server-side scripting language
- **Remote Desktop Protocol (RDP)** — Windows Server management
- **Windows Server 2022** — backend OS
