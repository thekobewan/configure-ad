<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines. This tutorial demonstrates how to set up virtual machines within Azure, install Active Directory, and create additional users to simulate the use of Active Directory in educational and/or organizational environments.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Step 1: Set up Azure Resources: Domain Controller, Client-1 (Windows VM), setting Domain Controller's NIC Private IP to static
- Step 2: Ensure & Establish Connectivity between the Client & Domain Controller
- Step 3: Install Active Directory
- Step 4: Create an Admin account and normal user account in Active Directory
- Step 5: Join Client-1 (Windows VM) to the domain
- Step 6: Setup and Establish Remote Desktop for non-administrative users on Client-1
- Step 7: Create additional users and log into Client-1 with those users


<h2>Deployment and Configuration Steps</h2>


  <h3>🟢 Step 1: Set Up Resources in Azure</h3>
  <p>In step 1, we’re going to create a Domain Controller, a client (Microsoft VM), and set the Domain Controller’s NIC Private IP Address to be static.</p>
  
  <h4>Creating the Domain Controller</h4>
  
  <strong>What is a domain controller?</strong>
  
  <p>A domain controller is a server that manages and authenticates user access to a network domain. Domain controllers store user account information such as usernames and passwords and controls access to network resources. They help authorize and authenticate users, facilitate efficient and streamlined network administration, and improve overall security.</P>
  
  <h4>🔵 Process for Creating DC-1:</h4>
  
  1. Naviagte to Microsoft Azure and select 'Virtual Machines'
  2. Within the Virtual Machines creation portal, create a Resource Group to hold both the Domain Controller and Client. In this instance, we'll name it: "AD-Lab" (ActiveDirectory-Lab)
  3. We'll create the Virtual Machine Name: "DC-1" (DomainController-1)
  4. Select the image: Windows Server 2022 Database Azure Edition - x64 Gen2
  5. Size: 2vcpus is recommended as it allows for the system to still run efficiently without eating up too many resources
  6. Create a username/password (Example: darinstathos)
  7. Take note that Azure automatically created a virtual network. Both our DC-1 and Client-1 will exist on the same virtual network connectivity. There is nothing left for us to edit, so we will select “Review + Create”
  
<img src="https://i.imgur.com/0cDyCbn.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/OX1wCfK.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/cjHYVJ1.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<p>We have now created the Domain Controller Virtual Machine. DC-1 will later be used as the central point for storing user accounts we create and managing/authenticating user access.</p>
<br />

  <h4>Creating our Client</h4>
  
  <strong>What is the client?</strong>
    
 <p>The client, in relation to the domain controller, is a device that connects to the network domain and relies on the domain controller for user authentication, access permissions, etc. The domain controller is the 'big brain' that lets the device/client know what to do/who's allowed to do what. The client interacts with the domain controller to log in, access shared resources, etc.</p>
    <p>We're creating a client device (Microsoft Windows) to witness the borrowing of user account information from our Domain Controller.</P>
    
 <h4>🔵 Process for Creating the Client:</h4>
    
1. Navigate to Microsoft Azure & select 'Virtual Machines'
2. Select the Resource Group: 'AD-Lab'
3. Virtual Machine Name: 'Client-1'
4. Select image: 'Windows 10 Pro, version 22H2 -x64Gen'
5. Size: 2vcpus
6. Create a username/password: (Example: darinstathos)
7. Under Licensing, check the box: "I can confirm I have eligible Windows 10/11 license with multi-tenant hosting rights."
8. Forward to the next page: Next: Disks >, Next: Networking > Take note that Client-1 was automatically put on the same network as DC-1. This is important so that the two can later communicate with one another. 

<br>

<img src="https://i.imgur.com/gOn48pL.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/13dDsO0.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/MT99vXJ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
    
<br />

<h4>Setting Domain Controller's NIC Private IP Address to Static</h4>

<strong>Why is it necessary to change the Private IP Address to Static?</strong>

<p>Whenever we create resources such as Virtual Machines within Azure, there are several other resources created alongside it. One of these resources are NICs (Network Interface Cards) which have IP addresses automatically assigned to them via a DHCP server hidden within Azure. As mentioned previously, domain controllers provide centralized management and organization over user accounts, access permissions, and security policies within a network.</p> 

<p>Given that it plays such a crucial role in centralizing and streamlining networking, we want our network devices to always be able to locate the Domain Controller for consistent instruction and communication. A static IP address prevents potential disruptions caused by a dynamic IP address. Thus, we must change the IP address from dynamic to static (meaning the IP address numerical values never change).</p>

<h4>🔵 Process for Setting DC-1 NIC Private IP to Static:</h4>
  
1. Navigate to Azure > 'Virtual Machines' > and select our Domain Controller "DC-1"
2. Select 'Networking' > Select 'Network Interface'
3. Select 'IP Configurations' > Select the current/only IPConfig there
4. Switch the assignment from 'Dynamic' to 'Static' and press 'Save'

<br>

<img src="https://i.imgur.com/dYzWLhZ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/g3IL4nQ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/FHp5pT4.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br> 

We have now switched the Domain Controller's NIC Private IP Address to be static. Now, there is peace of mind in knowing that devices connected to the domain can rely on the Domain Controller for user information with secure connection and communication. 

<br>

  <h3>🟢 Step 2: Ensure & Establish Connectivity between the Client and Domain Controller</h3>
  
  <p>The next step is to ensure connectivity between the Client and the Domain Controller. We want to make sure that our machines can talk to one another so that once we add users accounts to DC-1, Client-1 will be able to access them.</p>
  
  <h4>Log into Client-1 VM</h4>
  
  <p>First, we’re going to log into the Client-1 VM. To do this, we must retrieve the Public IP Address and connect it to Remote Desktop Connection (if you’re using WindowsOS) or Microsoft Remote Desktop (if you’re using MacOS).</p>
  
  <h4>🔵 Process for Logging into Client-1:</h4>
  
  1. In the Azure Portal, navigate to 'Virtual Machines'
  2. Select 'Client-1'
  3. Copy the Public IP Address to clipboard (Example shown: 20.163.26.26)
  4. Open Remote Desktop Connection (Windows) or Microsoft Remote Desktop (MacOS: can be downloaded from App Store)
  5. Paste the IP address (Example shown: 20.163.26.26)
  6. Log in using the username/password created earlier in Step 1. (Example: darinstathos)

<br>

<img src="https://i.imgur.com/ZIh6gvb.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/NuYn2WR.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/kPeUFHN.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>

<h4>Check Connectivity to Domain Controller (will fail at first attempt)</h4>

Now that we’ve logged into Client-1, we want to “ping” to our Domain Controller to make sure our machines can talk to one another/that there’s connectivity.

However, most likely, this “ping” will fail due to the Domain Controller’s firewall blocking ICMP traffic.

<strong>Why do firewalls sometimes block ICMP traffic?</strong>

Firewalls sometimes block ICMP (Internet Control Message Protocol) traffic as a security measure to protect against potential network vulnerabilities. ICMP traffic includes various types of messages, such as ping requests and error messages, which can be exploited for network scanning, denial-of-service attacks, or information disclosure. Blocking ICMP can help prevent these types of attacks and limit potential exposure of sensitive information. However, it's worth noting that ICMP is also used for legitimate network troubleshooting and diagnostic purposes, so firewall rules should be carefully configured to balance security and operational needs.

<h4>🔵 Process for Checking Connectivity:</h4>

1. We need to get DC-1's private IP Address: Azure Portal > 'Virtual Machines' > DC-1
2. Copy the NIC Private IP Address (Example: 10.0.0.4)
3. Navigate back to our Client-1 VM and open the command-line in the search bar
4. Type: "ping -t 10.0.0.4" (-t means that it will ping continuously)
5. We can see the request failed/timed out because DC-1's firewall is preventing/blocking ICMP traffic from coming through

<br>
  
<img src="https://i.imgur.com/3b3sYC2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/2l07H1G.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>

<h4>Open Up the Firewall</h4>
  
<p>Since we were unable to “ping”/connect to our Domain Controller, we must open up the firewall to allow ICMP traffic. To do this, we must connect/log into our Domain Controller VM and alter the settings.</p>

<h4>🔵 Process for Opening Up Firewall:</h4>
  
1. Navigate to Azure > 'Virtual Machines' > DC-1: Copy the Public IP Address (Example: 20.169.82.100)
2. Open Remote Desktop Connection (Windows) or Microsoft Remote Desktop (MacOS) & paste the IP address and use the username/password created in Step 1 (Example: 20.169.82.100 ; darinstathos)
3. Inside of DC-1 VM, navigate to the search bar and type in “wf.msc” or “firewall”

<br>
  
<img src="https://i.imgur.com/I8myAP2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>

4. Select “Inbound Rules” and filter by “Protocol” (since we’re looking for ICMPv4)
5. Enable Core Networking Diagnostics: ICMP Echo Request (ICMPv4-In)
6. Right-click the two inbound rules > select “enable rule"

<br>
  
<img src="https://i.imgur.com/ALQ2Tqc.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/9emoF0X.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>
  
<p>Now, when we go back to our Client-1 VM, if we left the command-line open, we can now see that the “pings” are coming through because we opened up the DC-1’s firewall to allow ICMP traffic.</p>

<br>

<img src="https://i.imgur.com/cWVAUuA.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>

<h3>🟢 Step 3: Install Active Directory</h3>
  
<Strong>What is Active Directory and why is it important?</strong>
  
<p>Now that there has been connectivity established between DC-1 and Client-1, it’s time to install Active Directory.

In simple terms, Active Directory is like a phonebook or a directory for a network. It keeps track of all the users, computers, and other resources within a network domain. It helps manage user accounts, control access to resources, and enforce security policies, making it easier for network administrators to organize and manage their network effectively.</p>

<h4>🔵 Process for Installing Active Directory:</h4>
  
1. Log into DC-1 VM using the Public IP Address & username/password (if not already)
2. Once inside DC-1, use the search bar to go to Service Manager
3. Select “Add roles and features” > Hit ‘Next’ on everything until you reach & select <strong>'Active Directory Domain Services'</strong>
4. Add the features > continue to press ‘Next’ on everything > select ‘Install’ 

<br>
  
<img src="https://i.imgur.com/Qk6wPjg.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/E6J427g.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/UFYfRkq.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>
  
We now have Active Directory installed on DC-1. However, we are not finished yet. We need to set up an actual domain. 

<h4>Promote as a DC: Setup a New Forest as mydomain.com (Domain can actually be anything. Mydomain.com is an example)</h4>

<p>We have installed Active Directory but the next step is to set up an actual domain. Setting up a domain means there is an address for a particular organization to unite under. This domain is attached to permitted users, allowing users to access various clients/devices on the network.</p> 
  
<p>For example, if you work for a large accounting firm or you're in school, you're given a username with a domain attached to it. You can then take your username and log into any computer on campus or in the office. Example: dstathos@univerity.edu or dstathos@accountingfirm.com.</p>
  
  <h4>🔵 Process for Promoting to Domain Controller:</h4>
  
  For this example, we are using "mydomain.com". However, any domain is suitable. 
  
  1. Go to DC-1 VM > open Service Manager > select the Notifications flag in the top right-hand corner
  2. Select: "Promote this server to be a domain controller"
  3. Select: “Add a new forest” > Input the domain name of choice (example: mydomain.com) > Select: “Next” 
  4. Input a password for the sake of continuing forward with “Next” although it won’t be used in this lab > Continue pressing ‘Next’ until you reach the ‘Install’ button > 'Install'

  <br>
  
<img src="https://i.imgur.com/3qgGfvS.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/PRP9sSq.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/c1m2IIM.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
  
  <br>

<h4>Restart & Log back into DC-1 as user: mydomain.com\[username]</h4>

1. It is normal for the computer to need to restart. When the computer restarts, if the IP address changes, we are sure to navigate back to Azure Portal and copy the DC-1 Public IP address. We set the Private IP address to be static, but it’s possible for the Public IP to change.</p>
2. Once booted out, we can log back into DC-1 using the domain we have now set up: mydomain.com\darinstathos (or) darinstathos@mydomain.com
3. DC-1 is now officially a domain controller 

<br>
  
<img src="https://i.imgur.com/TuOxlzE.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/EBmChAn.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>
  
<h3>🟢 Step 4: Create an Admin and a Normal User Account in Active Directory</h3>
  
<p>Now that Active Directory has been installed, our Domain Controller became official with its domain, it’s now time to create an admin and normal user account in AD.</p>
<p>Admins are able to create, modify, and delete user accounts, manage access permissions, etc. They can configure and enforce security policies such as password requirements and account lockout settings. They can also manage the structure of the domain, delegate admin tasks, etc.</p>
  
<h4>🔵 Process for Creating Organizational Units: Employees and Admins</h4>
  
  1. Inside DC-1 VM, navigate to ‘Active Directory Users and Computers’

<p>We can do this two ways:</p>

* Go to Service Manager > Tools (upper right-hand corner) > Active Directory Users and Computers
* Search bar > Type: Active Directory Users and Computers

2. We can see the domain we created: "mydomain.com". Now, we’re going to create organizational units to place our employees and admins.
3. Right click on “mydomain.com” (or whatever name you decided to give your domain) > 'New' > 'Organizational Unit'
4. We will create an organizational for _EMPLOYEES and for _ADMINS

<br>
  
<img src="https://i.imgur.com/RSgMaLR.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/m4OE8Wr.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/vPE2TyC.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>
  
<h4>🔵 Process for Creating an Employee/Admin & then Assigning Role of Admin</h4>
  
<p>Now that we have our organizational units, we want to create an Admin and assign them the role of “Admin” so that they can fulfill admin duties.</p>

1. Inside of _ADMINS: right click > ‘New’ > ‘Users’
2. We will give our admin the name of “Jane Doe” and her login username shall be jane_admin@mydomain.com. 
3. Ideally, we’d like for the user to change their password every time for security purposes. However, for the sake of this exercise, we will select for the password to never expire. 

<br>
  
<img src="https://i.imgur.com/wHJMccp.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/o8OgZsq.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>

<p>Now that jane_admin has been created, we must give her the role of ‘Admin’. Currently, just because she was created inside of an organizational unit folder called “_ADMINS” doesn’t mean her title carries substance or authority yet. We shall grant her authority right now. We’ll add Jane Doe to the Domain Admins Security Group.</p>

<br> 
  
4. Right-click Jane Doe > 'Properties' > 'Member Of' > 'Add' > type 'Domain Admins' [Enter Key & Check Names] > 'Apply' & 'OK'
  
<br>

<strong>What are Domain Admins in Active Directory?</strong>

<p>In Active Directory Users and Computers, "Domain Admins" is a default group that holds administrative privileges over the entire domain. Members of the Domain Admins group have full control and unrestricted access to all resources within the domain. They can perform tasks such as creating, modifying, or deleting user accounts, managing group memberships, configuring security policies, and managing domain-wide settings. Essentially, Domain Admins have the highest level of administrative authority within the Active Directory domain and are responsible for managing and maintaining the domain infrastructure.</p>

<br> 
  
<img src="https://i.imgur.com/5prj2Ju.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/IPTd2qT.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>
  
<h4>🔵 Log Out of DC-1 and Log In as New Admin</h4>

<p>Now that Jane Doe is an admin, we can log out of our Domain Controller and log back in as Jane Doe.</p>
username: jane_admin@mydomain.com (or) mydomain.com\jane_admin

<br>

<img src="https://i.imgur.com/WcNCaDu.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<h3>🟢 Step 5: Join Client-1 (Windows VM) to the Domain</h3>

<h4>Set Client-1's DNS settings to the DC-1's Private IP Address + Restart</h4>

<p>It’s time to now connect Client-1 to the domain (mydomain.com). During this entire time, Client-1 hasn’t been doing much as we’ve been setting up DC-1. Once Client-1 is added to the domain, Client-1 can retrieve/connect to user information stored in DC-1. Client-1 is like one of the computers you'd find in an office building or on a university campus. Once connected to the domain, any/all authorized registered users may log into the computers if their data is stored on the Domain Controller. </p>

<p>Setting a client's DNS settings to the domain controller's private IP address is important because it helps the client find and connect to important resources within a network. It ensures that the client can communicate with the domain controller for things like logging in, accessing shared files and printers, and finding other computers on the network. It's like having the right address book entry to reach the central hub of the network, making everything work smoothly.</p>

  <h4>🔵 Process for Setting Client-1 DNS Settings:</h4>
    
1. Go to Azure Portal > 'Virtual Machines' > 'DC-1 VM' > Copy the Private IP address in the Overview (Example: 10.0.0.4)
2. Go to Client-1 VM > 'Networking' > 'Network Interface' 
3. Select 'DNS Servers' > 'Custom' > Type/paste DC-1’s Private IP address (Example: 10.0.0.4) > 'Save'

<p>Now that we’ve changed Client-1’s DNS settings, we want to solidify this. We are going to flush any/all previous local DNS cache on Client-1 so they we know it’ll definitely be set to our DC-1 Private IP.</p>

4. Go to Client-1 VM Overview > Press "Restart" to flush DNS cache 

<br>
    
<img src="https://i.imgur.com/cHfZnzg.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/tB82xBL.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/YGVZQkF.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/RkYeWsW.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>
    
<h4>🔵 Process for Joining Client-1 to the Domain and Re-login as Admin</h4>

<p>Now it’s time to actually add Client-1 onto the domain. We’re going to login with the original user we created in the beginning and join Client-1 to the domain. Once Client-1 is on the domain, we are able to use our admin Jane Doe.</p>

1. Log back into Client-1 VM with original user from Step 1. (Example: darinstathos)
2. Right-click the Windows start icon > 'System' 
3. 'Rename this PC (advanced)' > 'Change' > 'Domain'
4. Change the domain to: mydomain.com (or whatever domain you'd like)

<br>
    
<img src="https://i.imgur.com/kbowguZ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/2owVYZj.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/bxXLQHp.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>
    
<p>Since we've altered the DNS settings earlier so that it matches DC-1’s private IP address, Client-1 will be able to recognize this domain.</p>

5. We’ll be prompted to login as Jane Admin (mydomain.com\jane_admin) and when this happens, our computer will be prompted to restart
6. We can now log into Client-1 as Jane Admin because Client-1 shares the same domain network as the Domain Controller.

<br>
    
<img src="https://i.imgur.com/dHpcgqe.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/dsMpci3.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>
    

<h3>🟢 Step 6: Setup and Establish Remote Desktop for Non-Administrative Users on Client-1</h3>

<p>We're going to set it up so all domain users can remote login into Client-1. Right now, only admins can do this.</p>

<h4>🔵 Process for Setting Up Remote Desktop for Non-Admins:</h4>
    
1. Inside Client-1 VM, right-click Windows start icon at bottom of screen > 'system' properties
2. Select 'Remote desktop' > 'Select Users that can remotely access PC' > 'Add' > 'Domain Users' > 'OK'
3. You can now log into Client-1 as a normal, non-administrative user now

<img src="https://i.imgur.com/JjCFyjm.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/k7QDAua.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<p>**Side notes:</p>

* If you go inside DC-1 VM > Active Directory Users and Computers > Users > Domain Users > Members: that’s where everyone is and where everyone will get added automatically 
* Group Policy allows us to do this remotely and with MANY computers (hundred or thousands of computers instantly instead of logging into each computer one by one), but that’s beyond the scope of this exercise


<h3>🟢 Step 7: Create additional users and log into Client-1 with those users</h3>

<p>Now that we’ve set it up so non-administrative users (regular users) can access Client-1, we’re going to put this into action by creating many users and logging into Client-1 with one of those users.</p>

<h4>🔵 Process for Creating Additional Users:</h4>
    
1. Log into DC-1 VM as Jane Doe: mydomain.com\jane_admin
2. Use the search bar to open up 'Powershell ISE' > right-click to 'run as administrator'
<br>

<strong>What is Powershell ISE?</strong>

<p>PowerShell ISE (Integrated Scripting Environment) is a user-friendly graphical tool provided by Microsoft for writing, testing, and executing PowerShell scripts. It provides an interactive and simplified environment for individuals, including IT professionals and system administrators, to automate tasks, manage systems, and perform various administrative tasks using the PowerShell scripting language. Think of it as a coding workspace with helpful features that make it easier to create and run PowerShell commands and scripts.</p>
<br>
    
3. Create a new file > Copy/Paste the script below. This code allows us to create random usernames quickly > Run the script
4. Random usernames are now being created

<br>

<img src="https://i.imgur.com/PusOjdD.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/AEEJPFH.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br>

 # ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 10000
# ------------------------------------------------------ #

Function generate-random-name() {
    $consonants = @('b','c','d','f','g','h','j','k','l','m','n','p','q','r','s','t','v','w','x','z')
    $vowels = @('a','e','i','o','u','y')
    $nameLength = Get-Random -Minimum 3 -Maximum 7
    $count = 0
    $name = ""

    while ($count -lt $nameLength) {
        if ($($count % 2) -eq 0) {
            $name += $consonants[$(Get-Random -Minimum 0 -Maximum $($consonants.Count - 1))]
        }
        else {
            $name += $vowels[$(Get-Random -Minimum 0 -Maximum $($vowels.Count - 1))]
        }
        $count++
    }

    return $name

}

$count = 1
while ($count -lt $NUMBER_OF_ACCOUNTS_TO_CREATE) {
    $fisrtName = generate-random-name
    $lastName = generate-random-name
    $username = $fisrtName + '.' + $lastName
    $password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force

    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $firstName `
               -Surname $lastName `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_EMPLOYEES,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
    $count++
}

<br>
    
5. If we go back to DC-1 VM > Open up 'Active Directory Users and Computers' > '_EMPLOYEES' > Refresh: all the names are being dumped in there 
6. We will choose one of these users and log into Client-1 VM with it. (Example username: Duna.lin ; password: Password1)

<br>
    
<img src="https://i.imgur.com/2YJiMsr.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="https://i.imgur.com/Sv9iB4e.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
