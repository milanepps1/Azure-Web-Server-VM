# Azure-Web-Server-VM
Deploying a virtual machine to a web server in Microsoft Azure

## Objective

This Azure project involves creating a VM (virtual machine) that has a NIC (network interface controller) that is connected to the subnet of that VNET (virtual network). The subnet is protected by the filtering rules of an NSG (network security group). The VNET is connected to Azure Bastion, which holds a different subnetwork. Bastion is an Azure service that allows us to connect to our VM without exposing public SSH (secure shell) ports. In this way, it is possible to create an internal domain using Azure's Nextcloud resource to avoid exposing it to any potentially dangerous incoming traffic from the internet.   This hands-on lab will help to deepen understanding of which resources are needed to create and maintain an internal network and to gain familiarity with some of Azure's services to support securing a network in addition to other defensive strategies.


### Skills Learned
[Bullet Points - Remove this afterwards]

- Advanced understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting network logs.
- Ability to generate and recognize attack signatures and patterns.
- Enhanced knowledge of network protocols and security vulnerabilities.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used
[Bullet Points - Remove this afterwards]

- Security Information and Event Management (SIEM) system for log ingestion and analysis.
- Network analysis tools (such as Wireshark) for capturing and examining network traffic.
- Telemetry generation tools to create realistic network traffic and attack scenarios.

## Steps
drag & drop screenshots here or use imgur and reference them using imgsrc

Every screenshot should have some text explaining what the screenshot is about.

Example below.

*Ref 1: Network Diagram*
The purpose of this project is to example the process of using Microsoft Azure to create a virtual machine and to deploy said virtual machine to a web server.

The image below illustrates the desired architecture of this entire setup.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/1.png">

Starting from the bottom left of the image, you can see that we are creating a VM (virtual machine) that has a NIC (network interface controller) that is connected to the subnet of our VNET (virtual network). The subnet is protected by the filtering rules of our NSG (network security group). The VNET is connected to Azure Bastion, which holds a different subnetwork. Bastion is an Azure service that allows us to connect to our VM without exposing public SSH (secure shell) ports.  
The steps to successfully create a web server in Azure and deploy a virtual machine to it are as follows:

Create a Resource Group
Create a Virtual Network and a subnet
Protect a subnet using a Network Security Group
Deploy Bastion to connect to a Virtual Machine
Create an Ubuntu Server Virtual Machine
Install Nextcloud by connecting via SSH using Bastion
Publish an IP
Create a DNS label

Creating a Resource Group

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/2.png">

To create a resource group, you can click the Create a Resource button and then, in the search box, type “resource group.” You will see an option for a resource group pop up. Click it and then, when you see the resource group box appear with the Create button, click Create.

You will then have the ability to name your resource group and adjust any necessary details. The name doesn’t matter. Just choose a name you can easily remember and distinguish from others if you decide to create more later. For my group, I chose RG-USEast-Nextcloud since I am creating a Resource Group that is in the US East region and it will be creating a Nextcloud server.


At the bottom-left of the screen, you should see the button “Review + Create.” Click that button, then click the Create button that follows.
If you click on the notification icon at the top right, you will see the confirmation that your resource group was successfully created. If you click the “Go to resource group button,” you will be taken to your resource group.


Creating a Virtual Network and a Subnet

The process is very similar to how the resource group was just created. Click Create a Resource again and type “virtual network” in the search box. Then, click the virtual network option that pops up and follow the same steps for creating your virtual network. 

You’ll notice that the resource group you just created is already pre-populated as the name of the resource group that will be tied to your new virtual network. You only need to name the virtual network to your liking. The steps up until here will be very similar to how you just created your resource group, except you will need to also create your subnet before clicking the “Review + Create” button. To set up your subnet, you will first need to click on the IP addresses option. 

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/3.png">

You should find that there is already a designated network and subnet. But we don’t want to use that. We want to delete the default subnet and add our own IP address and subnet. 

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/4.png">

In this example, I’m using 172.10.0.0/16 as the IPv4 address. From there, a subnet is added.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/5.png">

Using a similar naming convention, I chose SNET-USEast-Nextcloud. The range for this would be 172.10.0.0/24, but this information might already be pre-filled as soon as you click the Add a subnet button. From there, you can click Add at the bottom left of the page to add the subnet.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/6.png">

Once you’ve added the subnet, you can review what you have to make sure everything looks right and then click the review + Create button at the bottom of the page.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/7.png">

Once you click Create and the deployment has been completed, you will see a new notification that confirms the deployment was successful.

You can click on Go to resource if there is something else that you would like to configure on it ( like DDoS protection or firewall settings). If instead, you want to confirm that the deployment is showing up in the right place, click on the resource group itself (in notifications, you should also see a Go to resource group button right under the Go to resource button. Use that to navigate back to your resource group).

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/8.png">

If you see it, you can move on to the actual resource itself by clicking on the virtual network (VNET) you just created or clicking on Go to resource in the notifications section. From there, on the far right, there is a Monitoring tab that you can click to open up a dropdown list. At the bottom of the list is the Diagram option. If you click this, it will display the topology of what you have created so far. You can use this along the way to track how much of the entire virtual setup you have completed.

Creating a Network Security Group
Next we want to create a network security group to ensure that our network is protected by filtering inbound and outbound traffic to and from Azure resources and the internet.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/9.png">

The process is just like before. Once you create a new resource and type in “network security group” and select that option, you can create a network security group and name it with the same naming convention (seeing a trend here?). You will see that it is automatically connected to your resource group instance just like the virtual network was. After naming it, click Review + Create and then click Create. Like before, you will see a notification that confirms when the deployment is complete.

If you click the Go to resource button, you can view and edit the details of your network security group.As you can see in the image below, Azure has already provided some default rules for our inbound and outbound filtering. By default, all ports and protocols are open inbound and outbound inside the virtual network. It can receive data from the Azure Load Balancer and send data to the internet. Everything else is denied by default. So, nothing can enter the virtual network by default.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/10.png">

From here, we want to assign the network security group to our subnet. Click the resource group to navigate back to it, where you will find all of the assets you’ve created so far (subnet and network security group) listed under the resource group.

Click Subnets in the far left column to navigate to our subnet. 

Then, click your new subnet to pull up the settings to adjust. From here, only the Network Security Group needs to be changed from None to the one that was just created. Once it’s selected, click Save to save your changes.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/11.png">

So far we’ve created a network security group to protect the internal subnet of the virtual machine that we’re going to set up within our virtual network. Our next step is to create a Bastion instance to connect to the virtual machine that we’re going to create. It can take a while for Bastion to deploy, which is why we’re doing this before creating the virtual machine so that we make better use of our time. 

Bastion will need its own subnetwork. So, we need to create that before creating the Bastion resource. To do so, click Subnets in the left panel and then click the button to add a subnet.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/12.png">

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/13.png">

Nothing else needs to be changed. Click Add (or Save depending on the options that are presented). You should see it listed under your other subnet now.

Next we need to create our Bastion resource. To do so, click back into your resource group, then click the button to create a resource and type “bastion” in the search box to see the bastion option to select. Click it, and then click Create > Bastion. From there, I have marked the fields to edit. Aside from the name field being updated, the most important field is the virtual network option. Be sure to choose your virtual network and not the resource group, which is also listed as an option. Then, click Review + create, and click Create again.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/14.png">

Our next step will be to create the virtual machine. For this, we will once again click the Add button to pull up the search box, and type “ubuntu” to pull up a list of options. We want the Ubuntu Server. I chose the 22.04 LTS ARM64 Gen2 option.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/15.png">

Take note of all changes on this page. 
Size: You will likely want to choose a different size if you are assigned a larger (and more costly) than necessary size by default. Since this VM won’t require much processing, we can go for the lowest available option in this tier. 

Authentication account: Ensure that SSH public key is checked since we will need that to be generated in order to make this VM more secure. 

Username: You can use your own name or any name you want.

Key pair name: Used the same naming conventions as before when naming this VM.

Public inbound ports: None. We don’t want this SSH port to be available over the internet since we are connecting to our VM inside our virtual network using Bastion.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/16.png">

Once everything is adjusted, click Next: Disks > to adjust the disk size. You don’t have to do anything here except allow the default disk size to auto-populate. Then, click Next: Networking to move to the next page.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/17.png">

Here we also don’t have much to do. The settings should be correct, but double-check the marked areas. Public IP should be set to None since we want to set one up manually later. NIC network security group should also be set to None since we already created one and don’t need a new one applied. As long as those settings look right, click Review + create.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/18.png">

If everything looks correct, click Create and the deployment will be created and provide a pop-up prompting you to Generate a new key pair with the option to download the private key and create the desired resource. Click that option as you will need that key pair later in order to access the virtual machine.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/19.png">

Once the resource is created, you will see confirmation and the option to access it.

Now that the virtual machine is created, the next step is to connect to it using Bastion via SSH and install a simple Nextcloud server on our virtual machine. Here’s what you should see if you click Monitoring > Diagram again for the topology.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/20.png">

You can reference the image below to make sure that everything looks right. The main areas marked are worth paying attention to (Private IP address and OS). One thing to note is that the Start button (next to Conect) is grayed out meaning that the VM is on. Be sure to turn it off by clicking the Stop button when you’re finished, otherwise, it will keep running and you will be billed for the extra time used. If everything looks good, click the Connect button (circled at the top left) and choose Connect via Bastion to connect to Bastion through the VM.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/21.png">

In the Username field, enter whichever username you chose when setting this VM up. Then, in Authentication Type, choose SSH Private Key from Local File because that was the option we chose to work with instead of setting it up through a VM password. You should have downloaded the Private key pair to a local file, so you need to click that blue square with the file icon next to the Local File field to find it and allow it to authenticate you.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/22.png">

If you try clicking Connect it should connect unless you have pop-ups blocked. If so, you will see this error.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/23.png">

To fix this, click on the pop-up blocked icon in the URL bar at the top of the page and change it to Always Allow.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/24.png">

Now when you click Connect, it will open up a new window for the Ubuntu server. Here you need to install Nextcloud with root level permissions. So, you will type:
sudo snap install nextcloud
And press Enter.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/25.png">

Once it’s installed, we need to create an admin account with a sample username and password. The purpose of this is to show how it works, so the username and password chosen here are very weak. In use for real accounts, it is highly recommended to create a much more secure username and password! To add the sample user and password, type:
sudo nextcloud.manual-install (desired username, followed by a single space, then desired password).

 <img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/26.png">
 
Depending on your computer specs, this could take a few minutes or seconds. Once this is complete, we need to create a self-sign certificate. A self-sign certificate isn’t vital, but it’s the most simple to use for the context of this project (unless you are trying to learn Nextcloud).

To do so, type:
sudo nextcloud.enable-https self-signed
and press Enter.

Once this task is complete,m we can close this Bastion instance by typing: 
exit
And pressing Enter.
Then, click the Close option that pops up.


Publish an IP

The next step is to access our Nextcloud instance on the web. In order to see our Nextcloud working we need to create a public IP and only allow https connections to it. To do this, we can click on the Network settings option under the Networking tab and then click onto our Network Interface.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/27.png">

From there, we should navigate to IP configuration, where we will see our current setup for a private IP address, but no public IP (which we now will create).

Now, click on your IP config (mine is named ipconfig1) and it will pull up the option to edit it. From here, we need to check the Associate public IP address box, click the Create a public IP address option, name our new public IP, and make sure it’s set to standard before clicking OK and then Save.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/28.png">

Once Azure has created our new public IP address and associated it with our Network Interface Controller (NIC), you can check the update by clicking into your VM and then clicking Overview in the left panel. You should see a public IP address has been added right above the private IP.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/29.png">

Generally, it would be possible to copy the public IP and navigate to it by typing:
https://172.191.142.65 (the IP address would be whatever your public IP is on that page). However, it won’t work at this point because we didn’t add a rule to allow inbound https traffic in our network security group. To resolve this, we will need to allow inbound traffic to our Nextcloud server, but only if it’s coming from our current IP.

One way to find the current public IP is by navigating to www.whatsmyip.com , but if that doesn’t provide anything (shown as a Not Detected message) you may need to login to your router (using the IP address on the back of your router/modem) to view the Broadband tab’s info that includes your router’s IPv4 address.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/30.png">

Next you want to copy your IPv4 address and add it as an inbound rule in your network settings. From here, click Networking > Network settings (in the left panel of your Azure portal) and then click Create port rule > Inbound port rule.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/31.png">

The fields that are marked should be filled out as exemplified in the image below. The main fields to note are the Source IP addresses being your copied public IPv4 address, the Destination IP being the private IP for your VM, the service being HTTPS, and preferably (optional) changing the name to be something you can easily identify. Once you have everything entered, click Add. In a few seconds, that Allow rule will be added to your list of inbound port rules.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/32.png">

Now that the rule has been created, copy the VM’s public IP address and try navigating to it again:
https://172.191.142.65

Because the certificate created earlier was a self-signed certificate, your browser will likely flag it as not being safe, and you will need to click the Advanced button that appears and click forward to accept the risk. Once you do, you will be taken to the untrusted domain you created (which was the risk).

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/33.png">

Creating a DNS Label


Our final step is creating a DNS label. We can start by taking one more look at our topology to see what we have so far and make sure that we’re on the right track.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/34.png">

We’ll need to click into our public IP and edit the configuration first. If you navigate to your resource group, you should see a list of all of the resources you’ve created thus far. We’ll want to click into the Virtual Machine IP which is the resource that holds the public IP.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/35.png">

After clicking into your VMIP, click into Settings > Configuration. You will now be able to edit your configuration. 

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/36.png">

This is where we will create a DNS label. To do this, you only need to name the label (needs to be a unique name) and click the Save button at the top.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/37.png">

Once that is done, click Overview in the left panel. Then, click into your resource group again and then click into your virtual machine.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/38.png">

Looking at the details on the far right or under networking, we can see that we now have the public IP address and the DNS name associated with this virtual machine.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/39.png">

Our next step is to open our virtual machine to make sure Nextcloud is aware of it. Connect via Bastion by clicking Connect > Connect via Bastion in the top left of that page. Then sign in with your username and SSH key saved on  a local file again and click Connect to open Bastion.
From here, type:
sudo nextcloud.occ config:system:set trusted_domains 1 --value=milannextcloud.eastus.cloudapp.azure.com
and press Enter.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/40.png">

Once that task is complete, type: exit
Press Enter, and click Close to close out that tab when the close prompt opens.

Now, navigate that previous tab where the untrusted domain page was (or open a brand new tab) and type your new domain in the URL bar and press enter. Mine was:
https://milannextcloud.eastus.cloudapp.azure.com
Once you navigate to this site, you will get the same warnings as before because of the self-signed certificate. Continue forward to see your new site page.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/41.png">

You can log in with the credentials you created (the sample admin creds). That will bring you to your admin page, where you can interact with the site as desired.

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/42.png">

<img src= "https://github.com/milanepps1/Azure-Web-Server-VM/blob/main/43.png">

With that, the project is complete. Feel free to play around with the site for a bit if you’d like. Otherwise, if you are done with using Bastion and your VM, be sure to shut down your virtual machine and delete your Bastion instance from your resource group so that you aren’t charged for it.


