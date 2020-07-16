# Windows Server

Installing:
https://www.youtube.com/watch?v=dd0VNFz-2nk

DC Config:
https://www.youtube.com/watch?v=WVSjA7wbh1M

### Create DC stuff (basics)

Install the server roles

1. Go to to *Manage -> Add Roles and Features*
2. Install DNS and AD Domain Services

#### Promote to DC

1. Click on the flag and click the "promote to domain controller"
2. Add a new forrest
3. The root domain name can be whatever. EG. lab.lan or network.com
4. Set a DSRM password
5. Keep clicking next and install. You will get warning but that is expected

#### Set up DNS zones.

Create a Reverse lookup zone to get the Domain Name of the DC from its IP

1. Under Reverse Lookup Zone Create a new zone
2. Select *primary zone*
3. Select *To all DNS server running on domain controllers in this domain*
4. Set the network ID to the network you are on. EG. 192.168.0
5. Select *Only allow secure dynamic updates*

#### Add the client computer to the domain

On a windows computer that can contact the network set its *Primary DNS* accordingly. EG. My DC is on 192.168.0.254 so the DNS of the client will the 192.168.0.254, you should then get prompted to log in.

But wait! You dont have an account in AD yet. That part is covered next

### Create AD stuff (basics)

Under *Tools, AD Users and Computers* complete the following.

1. expand your domain dropdown (eg. lab.lan)
2. Right click on your domain (lab.lan) and Create a *new* -> *Organizational Unit* called Network (or any other suitable name)
3. In your new OU create 2 sub OUs called *Users* and *Security Groups*

Create some users

1. Create a user for yourself
2. Create an admin user by right clicking on the user and clicking *Add to a group* and adding them th the *Administrators* group.

### Remote admin

Microsoft offers *Windows Admin Center* as a downloadable application for windows computers to easily manage multiple windows server instances from multiple locations.

To make Windows Admin Center work you need to read [this](https://docs.microsoft.com/en-us/windows/win32/winrm/installation-and-configuration-for-windows-remote-management) article about Installation and Configuration for Windows Remote Management.

Download Windows Admin Center [here](https://www.microsoft.com/en-us/windows-server/windows-admin-center).

You can read more about Windows Admin Center [here](https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/overview).

Once installed you can search for and start *"Server Manager"* and navigate to *manage* -> *Add Servers* too add the DC that we created above.

#### Refresh error

```output
The refresh failed because WinRM is not running and could not be started
```

The solution is to do the following the the remote windows machine to enable the WinRM services.

1. Run cmd as admin
2. navigate to `cd \WIndows\System32\wbem\AutoRecover`
3. run `for /f %s in ('dir /b \*.mof \*.mfl') do mofcomp %s
4. Restart the *Server Manager*

### GPO

Following [this video for reference](https://www.youtube.com/watch?v=00t18BsXl9I)

Under *tools -> Group Policy Management -> Forrest -> Domains -> your_domain.com -> Your_network -> Group Policy Objects* You can create a new group policy object, then follow the video to create the policy.

Once you have created the policy object, drag and drop it into the Users folder that you want to apply the policy to. Remember that policy is hierarchal so sub organizational units inside the  Users directory will also inherit this policy.
