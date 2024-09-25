# Azure Three-Tier Application
This repository provides a detailed, step-by-step guide for manually deploying a three-tier application on Azure. It includes clear explanations and code snippets for setting up the Web, Application, and Database tiers. The project utilizes Azure Virtual Machines, Virtual Networks, Subnets, Load Balancers, and Network Security Groups (NSGs) to create a secure and scalable environment.

## Prerequisites

- An active Azure subscription is required.
- A basic knowledge of Azure Networking, Virtual Machines, Load Balancers, and NSGs is necessary.
- Access to the Azure Portal is needed.

## Architecture Overview
![Screenshot 2024-09-25 231523](https://github.com/user-attachments/assets/fb1e70a1-d0be-4dc1-b636-1b6ecd03cd7b)



The project uses a three-tier architecture with distinct subnets for each layer:

1. **Web Tier:** Exposed to the public, responsible for serving the frontend.
2. **App Tier:** Acts as the middle layer, handling business logic.
3. **DB Tier:** Contains the database and is only accessible through the App Tier.

Each tier is protected using Azure NSGs and is load-balanced using Azure Load Balancers.

## What is 3-Tier Architecture?
A 3-tier architecture is a software design pattern commonly used in application and system development. It organizes the software into three separate layers, each with its specific roles and responsibilities.

1. **Web Tier (Frontend):** This is the top layer, responsible for managing the user interface and user interactions.

2. **Application Tier (Backend):** Also known as the business logic tier, this layer contains the core business and processing logic. It manages the application's main functions and communicates with the data tier to retrieve or update information as needed.

3. **Data Tier (Database):** The bottom layer, this tier focuses on data management, including storage, retrieval, and manipulation.

The 3-tier architecture is widely utilized in the development of web applications and services.

Let’s begin constructing this architecture!

## Step 1: Creating a Resource Group
![image](https://github.com/user-attachments/assets/78f36e59-433f-47a6-b539-d8c519d79cbd)



1. Navigate to the Azure portal.
2. In the search bar, search for "Resource Groups" and click `Create`.
3. Set the **Resource Group Name** to `ThreeTierResourceGroup`.
4. Set the **Region** to `East US` (or your preferred region).
5. Click `Review + Create`, then `Create`.

---

## Step 2: Creating a Virtual Network with 3 Subnets
![Screenshot 2024-08-16 031519](https://github.com/user-attachments/assets/4df1a882-9862-44f2-a6e0-09710f18eb2a)

1. In the search bar, search for `Virtual Networks` and click `Create`.
2. Set the **Name** to `ThreeTierVNet`.
3. Set the **Region** to `East US`.
4. Set the **Address Space** to `10.0.0.0/16`.
5. Click `Next: IP Addresses`.

6. Add 3 Subnets:
   - **WebSubnet**: Address range `10.0.1.0/24`
   - **AppSubnet**: Address range `10.0.2.0/24`
   - **DbSubnet**: Address range `10.0.3.0/24`
7. Click `Review + Create`, then `Create`.

---

## Step 3: Create NSGs and Associate with Subnets
![Screenshot 2024-08-16 031847](https://github.com/user-attachments/assets/46438f24-0582-4ff2-b858-dae3c8d272a2)

### Create NSGs for Web, App, and DB Subnets

1. In the search bar, search for `Network Security Groups` and click `Create`.
2. Create an NSG for each tier:
   - **WebNSG** for Web Tier
   - **AppNSG** for App Tier
   - **DbNSG** for DB Tier
3. Set the **Region** to `East US` for each NSG.
4. Click `Review + Create`, then `Create`.

### Create Rules for NSGs

1. **WebNSG**:![Screenshot 2024-08-16 031947](https://github.com/user-attachments/assets/3112689d-5a1e-4291-9a57-45e4208cf5bf)

   - Add inbound security rules to allow HTTP (port 80) and HTTPS (port 443) traffic from any source.

2. **AppNSG**:![Screenshot 2024-08-16 032011](https://github.com/user-attachments/assets/1f6de6a2-277b-429e-8dac-8a1d596d5de5)

   - Add inbound security rules to allow traffic from `WebSubnet` to `AppSubnet` on port 8080.

3. **DbNSG**:![Screenshot 2024-08-16 032029](https://github.com/user-attachments/assets/9e991fc7-ca85-465b-b3bd-297ce14dd7ca)

   - Add inbound security rules to allow traffic from `AppSubnet` to `DbSubnet` on port 1433.

### Associate NSGs with Subnets

1. Go to the `Virtual Network` you created.
2. Select the `Subnets` tab.
3. For each subnet, associate the respective NSG:
   - `WebSubnet` -> `WebNSG`
   - `AppSubnet` -> `AppNSG`
   - `DbSubnet` -> `DbNSG`

---

## Step 4: Create Virtual Machines for Each Tier
![Screenshot 2024-08-16 032044](https://github.com/user-attachments/assets/1a23c45a-fd75-4d3a-b108-8379cc132370)


### Create Web VM

1. In the search bar, search for `Virtual Machines` and click `Create`.
2. Set the **Name** to `WebVM`.
3. Set the **Region** to `East US`.
4. Select your desired OS image (e.g., Ubuntu Server or Windows).
5. Under `Networking`, select the `ThreeTierVNet` and `WebSubnet`.
6. Select `WebNSG` for the Network Security Group.
7. Click `Review + Create`, then `Create`.

### Create App VM

1. Repeat the steps for creating `AppVM`.
2. Set the **Subnet** to `AppSubnet`.
3. Set the **NSG** to `AppNSG`.

### Create DB VM

1. Repeat the steps for creating `DbVM`.
2. Set the **Subnet** to `DbSubnet`.
3. Set the **NSG** to `DbNSG`.

---

## Step 5: Create Load Balancers for Web, App, and DB Tiers
![Screenshot 2024-08-16 032150](https://github.com/user-attachments/assets/0af47943-d19b-482a-b104-e5584fea0dba)


### Create Public Load Balancer for Web Tier

1. In the search bar, search for `Load Balancers` and click `Create`.
2. Set the **Name** to `WebLB`.
3. Set the **Type** to `Public`.
4. Create a new public IP for the load balancer.
5. Click `Review + Create`, then `Create`.

6. Go to the `Backend Pools` section of the load balancer and add `WebVM` to the pool.

### Create Internal Load Balancers for App and DB Tiers

1. Follow the steps above to create an internal load balancer for `AppLB`:
   - Set the **Type** to `Internal`.
   - Assign it a private IP within `AppSubnet`.
   - Add `AppVM` to the backend pool.

2. Repeat the steps to create an internal load balancer for `DbLB`:
   - Set the **Type** to `Internal`.
   - Assign it a private IP within `DbSubnet`.
   - Add `DbVM` to the backend pool.

---

## Step 6: Host `index.html` on Web VM (IIS Windows Server)

1. **Promote WebVM to IIS Windows Server**![Screenshot 2024-08-16 033429](https://github.com/user-attachments/assets/d0eb7900-3371-4af6-aac4-ffdc0f7a6e91)

   - Once your Web VM is deployed, log in via Remote Desktop (RDP).
   - Open `Server Manager` on your Windows VM.
   - In `Server Manager`, select `Add roles and features`.
   - Follow the wizard, and in the `Roles` section, check the box for `Web Server (IIS)`.
   - Complete the wizard and wait for the IIS role to be installed.

2. **Navigate to the wwwroot Directory**![Screenshot 2024-08-16 033523](https://github.com/user-attachments/assets/a065d8f2-7e32-404a-bc64-d6e8377bfbf8)

   - Once IIS is installed, go to the following path on your VM:  
     `C:\inetpub\wwwroot`.
   
3. **Upload Your HTML File**![Screenshot 2024-08-16 033545](https://github.com/user-attachments/assets/8b380837-54de-46ec-8bc8-443e10169488)

   - In the `wwwroot` folder, delete the default `index.html` file (if it exists).
   - Upload your frontend HTML file into the `wwwroot` folder.
   - Rename your HTML file to `index.html`.

4. **Verify the Website**![Screenshot 2024-08-16 033623](https://github.com/user-attachments/assets/ad75df38-da61-41fb-abb6-9613f14c5319)

   - Ensure the IIS service is running by opening `Internet Information Services (IIS) Manager`.
   - Start or restart the website from within IIS Manager if needed.
   - Go to your Web VM's public IP address in a browser:  
     `http://<your-web-vm-public-ip>`.
   - You should now see your frontend site hosted and accessible via the public IP of the Web VM.

   Example:  
   If your Web VM's public IP is `13.92.140.123`, open `http://13.92.140.123` in a browser to view the site.

Congratulations! Your site is now live using IIS on Windows Server.

Now, any person with internet access should be able to open this site using the public IP or domain name[DNS or Direct Access: Users will access the site via the public IP address of the Web VM unless you set up a DNS to point a domain name to this IP. If you only provide the public IP, other users will need to enter that IP address directly into their browser].













## Further Reading

For detailed insights and learnings from this project, please refer to the [Insights and Learnings](INSIGHTS_AND_LEARNINGS.md) document.


