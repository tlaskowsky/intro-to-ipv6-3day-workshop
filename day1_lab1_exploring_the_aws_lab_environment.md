---
theme: white # Or your preferred theme
revealjs:
  width: 960
  height: 700
  margin: 0.1 # Adjust margin as needed
  center: true
---

# Lab 1: Exploring the AWS Lab Environment

*Focus: Service Provider VPC (Initial State)*

<aside class="notes">
Welcome to Lab 1! Before we start configuring and testing IPv6 in Lab 2, it's crucial to understand the starting point – the initial environment automatically set up for you using AWS CloudFormation. This lab focuses on exploring the **Service Provider VPC** and identifying its components as they exist *before* we enable IPv6. We will also connect to the instances and perform a basic IPv4 connectivity test.
</aside>

---

## Lab Objectives

* Log into your provided AWS student account.
* Navigate the AWS Management Console, focusing on the VPC service.
* Identify the key network resources created by the CloudFormation template for the **Service Provider VPC**.
    * VPC details (IPv4 CIDR only)
    * Subnet types (Public/Private IPv4-only)
    * Gateways (IGW, NGW)
    * Route Tables and key IPv4 routes
    * Associated EC2 Instances & Security Groups
* Understand the initial **IPv4-only** network layout.
* **Connect** to the EC2 instances using Session Manager.
* **Perform** an IPv4 ping test between instances.
* **Capture** ICMP packets and identify L3 header information.

<aside class="notes">
Our goals for this first lab are straightforward. Ensure access, navigate the VPC console, and find the resources for the 'Service Provider VPC'. We'll look at its IPv4 addressing, subnets, gateways, and routing. We'll locate the EC2 instances and security groups. Then, we'll connect to the instances, verify basic IPv4 connectivity between them with ping, and use tcpdump to capture those packets, looking specifically at the IPv4 header information. This sets the stage for Lab 2.
</aside>

---

## Step 1: Logging In

* Access the AWS Management Console using the credentials provided for your student account.
    * *(Instructor Note: Provide specific login instructions based on how accounts are distributed - e.g., Workshop Studio link, IAM user credentials, etc.)*
* Familiarize yourself with the main console dashboard.
* **Select the correct AWS Region** specified for the workshop (e.g., us-east-1, eu-west-1). This is important!

<aside class="notes">
First things first, let's get logged in. Please use the specific instructions and credentials provided to you to access the AWS Management Console. Once you're in, take a quick look around the main dashboard if you're unfamiliar with it. Crucially, make sure you are operating in the correct AWS Region where the CloudFormation stack was deployed – check the top-right corner of the console. All the resources we need are in that specific region.
</aside>

---

## Step 2: Finding Your VPC

* In the AWS Console search bar, type `VPC` and select the **VPC** service (Virtual Private Cloud).
* In the VPC dashboard navigation pane (usually on the left), click on **Your VPCs**.
* You will see three VPCs created by the template. Locate the one named **Service Provider VPC** (or similar, based on the `ServiceProviderVPCName` parameter used during deployment [cite: 1116]).
    * Use the filter bar or 'Name' tag.
    * Note its VPC ID.

<aside class="notes">
Now, let's find the core network container – the VPC. Use the search bar at the top to find and navigate to the VPC service. Once the VPC dashboard loads, look for 'Your VPCs' in the left-hand navigation menu. You should see three VPCs listed [cite: 1114]. Our focus for this lab and the next is the 'Service Provider VPC'. Use the filter bar or look at the 'Name' tag to find it [cite: 1116]. Note its VPC ID.
</aside>

---

## Step 3: VPC Details (IPv4 Only)

* Select the **Service Provider VPC**.
* Examine the **Details** tab in the bottom pane.
* Note the following:
    * **VPC ID:** (e.g., `vpc-0abc...`)
    * **IPv4 CIDR:** (e.g., `10.1.0.0/16` - from `ServiceProviderVPCCIDR` parameter [cite: 1119])
    * **IPv6 CIDRs:** *None* (This VPC was *not* created with an IPv6 block by the template)

<aside class="notes">
With the 'Service Provider VPC' selected, look at the 'Details' tab below the list. Find the main IPv4 address block assigned to this VPC – the template default is `10.1.0.0/16` [cite: 1119]. Critically, notice that there is **no IPv6 CIDR block** listed for this VPC. The CloudFormation template intentionally created this VPC as IPv4-only, setting the stage for Lab 2 where you will enable IPv6.
</aside>

---

## Step 4: Subnets Overview

* In the VPC navigation pane, click on **Subnets**.
* Filter the list by the **Service Provider VPC ID**.
* Observe the subnets created:
    * Two Public Subnets (AZA/AZB) [cite: 1135, 1136]
    * Two Private Subnets (AZA/AZB) [cite: 1137]
* Note they are all currently **IPv4-only**.

<aside class="notes">
Now let's look at how this VPC is divided. Click 'Subnets' in the left navigation. Use the filter bar to show only the subnets belonging to our 'Service Provider VPC'. You'll see four subnets listed [cite: 1135-1137]: two named 'Public Subnet' (one in AZA, one in AZB) and two named 'Private IPv4 Subnet' (one in AZA, one in AZB). At this stage, all these subnets are configured only for IPv4.
</aside>

---

## Step 5: Public IPv4 Subnets

* Identify the subnets named `Service Provider VPC Public Subnet AZA`/`AZB`.
* Select one and examine its **Details**:
    * **IPv4 CIDR:** (e.g., `10.1.0.0/24` - from `ServiceProviderVPCPublicSubnet1CIDR` [cite: 1123])
    * **IPv6 CIDR:** *None*
    * **Auto-assign public IPv4 address:** Yes (`MapPublicIpOnLaunch: true` [cite: 1135])

<aside class="notes">
Find the 'Public Subnets'. Select one. In the details, you'll see its IPv4 CIDR block (like `10.1.0.0/24` [cite: 1123]). Confirm there is no IPv6 CIDR block listed. Note that 'Auto-assign public IPv4 address' is enabled [cite: 1135], meaning instances launched here can get a public IPv4 address for direct internet communication (via the IGW).
</aside>

---

## Step 6: Private IPv4 Subnets

* Identify the subnets named `Service Provider VPC Private IPv4 Subnet AZA`/`AZB`.
* Select one and examine its **Details**:
    * **IPv4 CIDR:** (e.g., `10.1.2.0/24` - from `ServiceProviderVPCPrivateSubnet1CIDR` [cite: 1125])
    * **IPv6 CIDR:** *None*
    * **Auto-assign public IPv4 address:** No (`MapPublicIpOnLaunch: false` [cite: 1137])

<aside class="notes">
Now find the 'Private IPv4 Subnets'. Select one. These have only an IPv4 CIDR block (like `10.1.2.0/24` [cite: 1125]) and no IPv6 CIDR. 'Auto-assign public IPv4 address' is set to No [cite: 1137]. Instances here only get private IPv4 addresses and need a NAT Gateway for outbound internet access.
</aside>

---

## Step 7: Internet Gateway (IGW)

* In the VPC navigation pane, click on **Internet Gateways**.
* Find the IGW attached to the **Service Provider VPC**. (Resource name: `ServiceProviderVPCInternetGateway` [cite: 1132]).
* **Purpose:** Enables **IPv4** communication between instances in public subnets and the internet. (Will also handle IPv6 later if configured).

<aside class="notes">
How does this VPC talk to the internet via IPv4? Click 'Internet Gateways'. Find the one attached to our Service Provider VPC [cite: 1132]. An Internet Gateway allows resources in public subnets to communicate with the internet. Currently, it's configured only for IPv4 traffic for this VPC.
</aside>

---

## Step 8: NAT Gateway (NGW)

* In the VPC navigation pane, click on **NAT Gateways**.
* Find the NGWs associated with the **Service Provider VPC** (one in Public Subnet AZA, one in AZB). (Resource names: `ServiceProviderVPCNatGateway1`, `ServiceProviderVPCNatGateway2` [cite: 1158, 1159]).
* **Purpose:** Allows instances in **private subnets** to initiate **outbound-only** connections over **IPv4** to the internet via the IGW.

<aside class="notes">
Now click 'NAT Gateways'. Find the two NAT Gateways associated with the Service Provider VPC [cite: 1158, 1159]. Note that they are placed *in* the public subnets. Their purpose is to allow instances in the *private* subnets (which only have private IPv4 addresses) to initiate outbound connections to the internet over IPv4. Traffic goes from the private instance to the NGW, which then uses its Elastic IP address [cite: 1158, 1159] to talk to the internet via the IGW.
</aside>

---

## Step 9: Route Tables Overview

* In the VPC navigation pane, click on **Route Tables**.
* Filter by the **Service Provider VPC ID**.
* Note the route tables created:
    * A **Public** Route Table (`ServiceProviderVPCPublicRouteTable` [cite: 1162]).
    * Two **Private** Route Tables (`ServiceProviderVPCPrivateRouteTable1`, `ServiceProviderVPCPrivateRouteTable2` [cite: 1164, 1165]), one for each AZ.

<aside class="notes">
Let's look at IPv4 routing. Click 'Route Tables' and filter for the Service Provider VPC. You should see three main route tables: one public [cite: 1162] and two private (one for each AZ's private subnet [cite: 1164, 1165]).
</aside>

---

## Step 10: Public Route Table Analysis

* Select the `ServiceProviderVPCPublicRouteTable`.
* Go to the **Routes** tab.
* Observe the key routes:
    * `10.1.0.0/16` -> `local` (Traffic within the VPC)
    * `0.0.0.0/0` -> `igw-xxxx` (IPv4 internet traffic via IGW [cite: 1170])
    * ***No IPv6 Routes***
* Check **Subnet Associations**: Associated with the two public subnets.

<aside class="notes">
Select the 'Public Route Table' [cite: 1162]. Look at the 'Routes'. You see the local IPv4 route and the default IPv4 route (`0.0.0.0/0`) pointing to the Internet Gateway [cite: 1170]. There are no IPv6 routes present yet. This table is associated with the public subnets, allowing instances there to reach the internet directly via the IGW.
</aside>

---

## Step 11: Private Route Table Analysis

* Select `ServiceProviderVPCPrivateRouteTable1` (or 2).
* Go to the **Routes** tab.
* Observe the key routes:
    * `10.1.0.0/16` -> `local`
    * `0.0.0.0/0` -> `nat-xxxx` (IPv4 internet traffic via the NGW in the corresponding AZ [cite: 1171])
    * ***No IPv6 Routes***
* Check **Subnet Associations**: Associated with the private subnet in the corresponding AZ.

<aside class="notes">
Now select one of the 'Private Route Tables' (e.g., AZA's [cite: 1164]). Look at its routes. It has the local IPv4 route, but the default IPv4 route (`0.0.0.0/0`) points to the NAT Gateway (`nat-xxxx`) located in the same AZ [cite: 1171]. Again, there are no IPv6 routes. This table is associated with the private subnet in that AZ, ensuring instances there send outbound internet traffic via the NAT Gateway.
</aside>

---

## Step 12: EC2 Instances & Security Groups (Brief Look)

* Navigate to the **EC2** service dashboard.
* Click on **Instances (running)**.
* Find instances named `Service Provider VPC IPv4 EC2 Instance AZA`/`AZB` [cite: 1205, 1208].
* Note their **Subnet ID** (should be the private IPv4 subnets) and lack of Public IP.
* Select an instance and view the **Security** tab. Note the Security Group (e.g., `ServiceProviderVPCName IPv4 EC2 Instance SG` [cite: 1189]). Controls IPv4 traffic.

<aside class="notes">
Let's locate the instances in this VPC. Go to the EC2 dashboard and find the instances named 'Service Provider VPC IPv4 EC2 Instance...' [cite: 1205, 1208]. Confirm they are located in the private IPv4 subnets and don't have public IP addresses. Check the 'Security' tab – they are associated with a security group configured for IPv4 traffic [cite: 1189]. We will connect to these next.
</aside>

---

## Step 13: Connecting to EC2 Instances

* In the EC2 Console, select **Instance A** (`...AZA`).
* Click the **Connect** button (top right).
* Select the **Session Manager** tab.
* Click **Connect**. A terminal window should open in your browser.
* Repeat the process to open a separate Session Manager connection to **Instance B** (`...AZB`).
* *(Troubleshooting: Ensure SSM Agent is running and IAM permissions are correct - the CFN template sets up the IAM role `IAMRoleForEC2` [cite: 1183] and SSM Endpoint `SSMInterfaceEndpointVPCA` [cite: 1197] which should allow this)*.

<aside class="notes">
Now we'll connect to the two instances launched in the private IPv4 subnets. We'll use AWS Systems Manager Session Manager, which provides secure shell access without needing SSH keys or bastion hosts, thanks to the IAM role and VPC endpoint configured by the template. Select the first instance (AZA), click 'Connect', choose the 'Session Manager' tab, and click 'Connect' again. Do the same in a separate browser tab or window for the second instance (AZB). You should now have terminal access to both instances.
</aside>

---

## Step 14: Finding Private IPv4 Addresses

* In the Session Manager terminal for **Instance A**, run:
  `ip addr show eth0`
* Look for the `inet` line (excluding 169.254...) - this is the private IPv4 address (e.g., `10.1.2.x/24`). Note it down.
* In the Session Manager terminal for **Instance B**, run:
  `ip addr show eth0`
* Find and note down its private IPv4 address (e.g., `10.1.3.x/24`).

<aside class="notes">
In each terminal window, we need to find the instance's private IPv4 address. The command `ip addr show eth0` (or `ifconfig eth0` if you prefer) will display the network interface configuration. Look for the line starting with `inet` followed by the IPv4 address and prefix length (like `inet 10.1.2.100/24`). Ignore the `inet6` lines for now. Record the private IPv4 address for both Instance A and Instance B. They should be in different subnets (e.g., 10.1.2.x and 10.1.3.x).
</aside>

---

## Step 15: Running Packet Capture (Instance B)

* In the Session Manager terminal for **Instance B**:
* Become root: `sudo su -`
* Install tcpdump if needed: `yum install -y tcpdump` (Might already be installed)
* Start capturing ICMP packets coming from Instance A (replace `[IP_of_Instance_A]` with the actual IP):
  `tcpdump -i eth0 icmp and src host [IP_of_Instance_A] -n`
* Leave this running.

<aside class="notes">
Now, on Instance B, we want to capture the ping packets we're about to send from Instance A. First, switch to the root user using `sudo su -` for permissions. Then, run tcpdump. The command `tcpdump -i eth0 icmp and src host [IP_of_Instance_A] -n` tells it to:
- `-i eth0`: Listen on the primary network interface.
- `icmp`: Only capture ICMP packets (which ping uses).
- `and src host [IP_of_Instance_A]`: Only capture packets *from* Instance A's IP address.
- `-n`: Don't resolve IP addresses to names (faster, clearer).
Press Enter to start the capture. It will wait for packets.
</aside>

---

## Step 16: Performing the Ping (Instance A)

* In the Session Manager terminal for **Instance A**:
* Ping the private IPv4 address of Instance B (replace `[IP_of_Instance_B]` with the actual IP):
  `ping -c 4 [IP_of_Instance_B]`
* Observe the replies. You should see successful ICMP echo replies.
* *(Troubleshooting: If ping fails, check Security Group `IPv4EC2InstanceSecurityGroupVPCA` [cite: 1189] - it should allow all traffic within the VPC by default, but verify)*.

<aside class="notes">
Switch to the terminal for Instance A. Now, send 4 ping packets (`-c 4`) to the private IPv4 address you noted for Instance B. Since both instances are within the same VPC and the default security group usually allows internal traffic, this ping should succeed. You'll see replies confirming connectivity. While the ping is running, tcpdump on Instance B should be showing the incoming ICMP echo request packets.
</aside>

---

## Step 17: Analyzing Captured Header (Instance B)

* In the Session Manager terminal for **Instance B**:
* Stop `tcpdump` by pressing `Ctrl+C`.
* Examine the output lines. Each line represents a captured packet.
* Example Output Line:
  `14:10:05.123456 IP 10.1.2.100 > 10.1.3.200: ICMP echo request, id 123, seq 1, length 64`
* **Identify L3 (IPv4) Header Info:**
    * **`IP`**: Indicates an IPv4 packet.
    * **Source IP:** `10.1.2.100` (Instance A's IP)
    * **Destination IP:** `10.1.3.200` (Instance B's IP)
    * **Protocol:** Implied by `ICMP` later in the line (Protocol number 1)
    * *(To see TTL, add `-v` to tcpdump: `tcpdump -v ...`)*

<aside class="notes">
Go back to Instance B's terminal and stop tcpdump with Ctrl+C. Look at the output lines. Each line summarizes a captured packet. You can clearly see the L3 IPv4 header information: the `IP` keyword confirms IPv4, the source IP (Instance A), the destination IP (Instance B). The protocol is identified as ICMP (echo request). To see other L3 fields like the TTL (Time To Live, equivalent to IPv6 Hop Limit), you would need to run tcpdump with the `-v` (verbose) flag. This confirms basic IPv4 connectivity and packet structure.
</aside>

---

## Step 18: Lab 1 Summary & Next Steps

* **Identified:** Service Provider VPC (IPv4-only), Public/Private IPv4 Subnets, IGW, NGWs, IPv4 Route Tables, EC2 Instances, IPv4 Security Groups.
* **Performed:** Connected to instances via Session Manager, verified IPv4 connectivity with `ping`, captured packets with `tcpdump`, identified basic L3 IPv4 header info.
* **Key Concepts:** This VPC is currently configured *only* for IPv4 communication. Basic network tools confirm connectivity and packet structure.
* **Next Lab (Lab 2):** We will take this IPv4-only VPC and manually enable IPv6, configure dual-stack and IPv6-only subnets, add an EIGW, and adjust routing - essentially building out the IPv6 capabilities ourselves.

<aside class="notes">
So, in this lab, we've explored the initial IPv4 state of the Service Provider VPC and confirmed basic IPv4 connectivity between its private instances using standard tools like ping and tcpdump. We've seen the essential L3 header information in action. This provides our baseline. In Lab 2, we will begin the process of adding IPv6 capabilities to this very VPC and comparing the results. Any questions about this initial IPv4 setup or the connectivity tests?
</aside>
