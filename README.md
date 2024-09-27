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

## Step 1: Create a Resource Group
![image](https://github.com/user-attachments/assets/78f36e59-433f-47a6-b539-d8c519d79cbd)




1. Open the Azure portal.
2. Use the search bar to find "Resource Groups" and click on `Create`.
3. Enter `ThreeTier-RG` as the **Resource Group Name**.
4. Choose `West US 2` (or your preferred region) for the **Region**.
5. Click `Review + Create`, then select `Create`.

---

## Step 2: Create a Virtual Network with 3 Subnets
![image](https://github.com/user-attachments/assets/96a2dfe2-39af-49d4-870c-d814742f9805)


1. In the search bar, look for `Virtual Networks` and click on `Create`.
2. Enter `ThreeTierVNet` as the **Name**.
3. Select `West US 2` for the **Region**.
4. Set the **Address Space** to `10.0.0.0/24`.
5. Click on `Next: IP Addresses`.

6. Add 3 Subnets:
   - **Web-Subnet**: Address range `10.0.0.0/27`
   - **App-Subnet**: Address range `10.0.0.32/27`
   - **Db-Subnet**: Address range `10.0.0.64/27`
7. Click `Review + Create`, then `Create`.

---

## Step 3: Create NSGs and Associate them with Subnets
![image](https://github.com/user-attachments/assets/c80d62db-c491-4739-beaa-5427ef048bdb)


### Create NSGs for Web, App, and DB Subnets

1. In the search bar, look for **Network Security Groups** and click on `Create`.
2. Create an NSG for each tier:
   - **WebNSG** for the Web Tier
   - **AppNSG** for the App Tier
   - **DbNSG** for the DB Tier
3. For each NSG, set the **Region** to `West US 2`.
4. Click on `Review + Create`, and then select `Create`.

### Create Rules for NSGs

1. **WebNSG**: ![image](https://github.com/user-attachments/assets/51d478a8-a058-4641-b639-011a3008e395)


   - Add inbound security rules to allow HTTP (port 80) and HTTPS (port 443) traffic from any source.

2. **AppNSG**: ![image](https://github.com/user-attachments/assets/6e80e5f8-cb3c-4dbc-8807-742b5fbd6be9)



   - Add inbound security rules to allow traffic from `WebSubnet` to `AppSubnet` on port 8080.

3. **DbNSG**: ![image](https://github.com/user-attachments/assets/14e19146-ce29-42d4-8a91-750a370ee817)


   - Add inbound security rules to allow traffic from `AppSubnet` to `DbSubnet` on port 1433.

### Associate NSGs with Subnets
![Screenshot 2024-09-27 100031](https://github.com/user-attachments/assets/ad51dacf-2599-4ca5-bee3-7069a2fc9c89)



1. Go to the `Network Security Groups` you created.
2. Select the `WebNSG` and then Select the `Subnets` tab.
3. For each NSG, associate the respective Subnet:
   - `WebNSG` -> `Web-Subnet`
   - `AppNSG` -> `App-Subnet`
   - `DbNSG` -> `DB-Subnet`

---

## Step 4: Create Virtual Machines for Each Tier
![image](https://github.com/user-attachments/assets/f2ae0f97-b197-479b-96cd-9714b20206b5)



### Create Web VM

1. In the search bar, look for `Virtual Machines` and select `Create`.
2. Enter `USW-WEB-VM` as the **Name**.
3. Choose `WestUS 2` for the **Region**.
4. Pick your preferred OS image (e.g., Ubuntu Server or Windows).
5. Under the `Networking` section, select `ThreeTierVNet` and `Web-Subnet`.
6. For the `Network Security Group`, choose `WebNSG`.
7. Click `Review + Create`, and then hit `Create`.

### Create App VM

1. Repeat the steps for creating `AppVM`.
2. Set the **Subnet** to `App-Subnet`.
3. Set the **NSG** to `AppNSG`.

### Create DB VM

1. Repeat the steps for creating `DbVM`.
2. Set the **Subnet** to `DB-Subnet`.
3. Set the **NSG** to `DbNSG`.

---

## Step 5: Create Load Balancers for Web, App, and DB Tiers
![image](https://github.com/user-attachments/assets/040b9b39-ebc6-4157-82f8-cdba32c403fc)



### Create Public Load Balancer for Web Tier

1. In the search bar, find `Load Balancers` and select `Create`.
2. Enter `PublicLB-Web` as the **Name**.
3. Set the **Type** to `Public`.
4. Create a new public IP address for the load balancer.
5. Click `Review + Create`, then hit `Create`.
6. Navigate to the `Backend Pools` section of the load balancer and add `WebVM` to the pool.

### Create Internal Load Balancers for App and DB Tiers

1. Follow the steps above to create an internal load balancer for `InternalLB-App`:
   - Set the **Type** to `Internal`.
   - Assign it a private IP within `App-Subnet`.
   - Add `USW-APP-VM` to the backend pool.

2. Repeat the steps to create an internal load balancer for `InternalLB-DB`:
   - Set the **Type** to `Internal`.
   - Assign it a private IP within `DB-Subnet`.
   - Add `USW-DB-VM` to the backend pool.

---

## Step 6: Host `index.html` on Web VM (IIS Windows Server)

1. **Promote WebVM to IIS Windows Server** ![Screenshot (1)](https://github.com/user-attachments/assets/b7fee1a0-83a4-4ce3-8691-4ae49046ffdb)


   - After your Web VM is deployed, connect to it using Remote Desktop (RDP).
   - Launch `Server Manager` on the Windows VM.
   - In `Server Manager`, choose `Add roles and features`.
   - Proceed through the wizard, and in the `Roles` section, select `Web Server (IIS)`.
   - Finish the wizard and wait for the IIS role to install.

2. **Navigate to the wwwroot Directory and Upload your HTML File**                                                                                                 
    ![Screenshot (3)](https://github.com/user-attachments/assets/3d95b0c4-698b-4475-8854-dcbe1e4c67a6)


   - Once IIS is installed, go to the following path on your VM:
      `C:\inetpub\wwwroot`.
   - In the `wwwroot` folder, delete the default `index.html` file (if it exists).
   - Upload your frontend HTML file into the `wwwroot` folder.
   - Rename your HTML file to `index.html`.
    
3. **Verify the Website** ![Screenshot 2024-09-26 111106](https://github.com/user-attachments/assets/ff3c7b51-2655-4da1-8ee1-189e88cfc646)


  
   - Open `Internet Information Services (IIS) Manager` to verify that the IIS service is running.
   - If necessary, start or restart the website through IIS Manager.
   - In a browser, navigate to your Web VM’s public IP address:
      `http://<your-web-vm-public-ip>`.
   - You should now see your frontend site, accessible via the Web VM's public IP.

For example, if your Web VM's public IP is 52.175.225.251, enter http://52.175.225.251 in your browser to view the site.

Congratulations! Your site is now live on IIS using Windows Server.

Now, anyone with internet access can open the site using the public IP or domain name. If you have set up DNS, users can access the site through a domain name; otherwise, they will need to use the public IP address directly in their browser.

## Contact

For any questions, doubts, or clarifications regarding this repository, feel free to reach out:

- Email: mailto:tomarankitsingh2000@gmail.com
- LinkedIn: https://www.linkedin.com/in/tomarankitsingh/
