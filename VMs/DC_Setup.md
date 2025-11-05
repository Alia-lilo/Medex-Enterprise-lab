## Create VM (Medex_DC)  

open VM Settings:   
add VMnet5

<a href="/images/Medex_DC/DC-settings.png">
  <img src="/images/Medex_DC/DC-settings.png" width="600"/>
</a>

The first step after completing the network setup is to create the **Domain Controller**.  
I chose **Windows Server 2016** because it’s lighter on my laptop compared to 2019, and the differences between the two are not significant for our lab purposes.  
Since we’ve already created a custom network `vmnet5` in **VMware Network Editor**, we’ll continue using the same setup.  
You can also add another **network adapter in Bridged mode** if you want to have internet access before connecting to EVE-NG or FortiGate.  

I also assigned the minimum possible RAM and disk space to this VM to optimize performance, as I plan to run multiple virtual machines and want to preserve system resources. I’ll increase resources later if needed. 

We connected the switch to the vmnet5 cloud (just to allow connectivity between the VM and EVE) using port e2/0, which is part of VLAN10 — exactly like what happens in a real data center.  
So, we’re simulating the connectivity between the physical host and VLAN10.  
In a production environment, this would typically be handled by a physical switch with proper VLAN configuration.  

### To install VM Tools from Windows Server itself
https://packages.vmware.com/tools/esx/latest/windows/x64/VMware-tools-13.0.0-24696409-x64.exe

➡️ ## DC Network Setup:  
from server manager →  Local Server  
Ethernet0: click on IPv4  

![DC IP Config](/images/Medex_DC/DC_IP.png)


```
The Default Gateway should be the same HSRP virtual IP we configured earlier in the routing setup — this will allow the server to communicate properly within the enterprise network.
```

➡️ ## Add Active Directory Domain Services

After installing **Windows Server**, open **Server Manager**, then go to:  
Manage → Add Roles and Features   
select: Active Directory Domain Services (AD DS)   

Continue through the wizard by clicking **Next** until you reach the final step.

✅ Make sure to check the option:

> "Restart the destination server automatically if required"
> 

Then click **Install** to begin the installation.

➡️ **Promote This Server to a Domain Controller: Deployment Configuration:**

After the **AD DS role** is installed, click:

> “Promote this server to a domain controller”
> 
- Select: **"Add a new forest"** — because this is a new domain setup
- Set the **Root domain name** to: `medex.local`
- Enter and confirm a **DSRM (Directory Services Restore Mode) password**

Then, keep clicking **Next** through the remaining steps, leaving the default values as is unless needed.

Finally, click **Install** to begin the promotion process.

The server will automatically restart once it's done.

**Congratulations! You've Successfully Created a Domain** for our **humble company**  
now let’s configure **IPv4 network settings** for the Domain Controller

🔹Now that the domain is ready, it’s time to add users and create Organizational Units (OUs).

## Adding Users to AD

Now that the domain is ready, it’s time to add users and create Organizational Units (OUs).

I need to add a large number of users and groups, but doing this manually would take too much time.
So, I will use a **PowerShell** script along with a **CSV file** to add them automatically.

This method is much faster and easier, whether for labs or real-world environments.

> 💡 Let's say that HR department sends you an Excel file with all employees information.
> 

Here is the CSV file. I will add it to C:\ADusers in our DC.
[ADusers.csv file](/images/Medex_DC/ADusers.csv)  

<a href="/images/Medex_DC/CSV-file.png">
  <img src="/images/Medex_DC/CSV-file.png" width="600"/>
</a>


I generated this CSV using GPT.  
It includes **250 users**, **5 OUs [IT, HR, Finance, Legal, Marketing]**, and **8 groups** — with **50 users per group**.

The groups are:  
- IT_Admins
- IT_Support
- HR_Users
- HR_ReadOnly
- Finance_Users
- Finance_ReadOnly
- Legal_Users
- Marketing_Users

```
🗝️ Of course, t**hese details are usually defined by the management department**.  
So it’s just an example and not the rules. I wanted to divide each department into two:   
**high-access and low-access,** to help with our policies and scenarios in this project  

Now. I will create these OUs and groups first by PowerShell, they must exist before we add the 250 users.
```

Now, I’ll create the **OUs** and **groups** using PowerShell. These must exist **before** adding the 250 users.

### 🛠️ **Pre-setup Requirements (Before Running Any Commands):**

You can add them from Tools → Active Directory Users and Computers, or by using PowerShell.  
I personally prefer PowerShell because it helps me practice more, and the GUI method is quite easy anyway.

- Open **PowerShell as Administrator**.
- Make sure the **Active Directory PowerShell module** is installed — it’s required to run AD-related commands.
- The script should be executed on a **Domain Controller** or on a **domain-joined machine** with sufficient **privileges**.

```powershell

Import-Module ActiveDirectory 

# Create OUs
New-ADOrganizationalUnit -Name "HR" -Path "DC=medex,DC=local"
New-ADOrganizationalUnit -Name "IT" -Path "DC=medex,DC=local"
New-ADOrganizationalUnit -Name "Finance" -Path "DC=medex,DC=local"
New-ADOrganizationalUnit -Name "Legal" -Path "DC=medex,DC=local"
New-ADOrganizationalUnit -Name "Marketing" -Path "DC=medex,DC=local"

# Create groups inside the OUs
New-ADGroup -Name "HR_Users" -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=medex,DC=local"
New-ADGroup -Name "HR_readonly" -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=medex,DC=local"
New-ADGroup -Name "Finance_Users" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=medex,DC=local"
New-ADGroup -Name "Finance_readonly" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=medex,DC=local"
New-ADGroup -Name "IT_Admins" -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=medex,DC=local"
New-ADGroup -Name "IT_Support" -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=medex,DC=local"
New-ADGroup -Name "Legal_Users" -GroupScope Global -GroupCategory Security -Path "OU=Legal,DC=medex,DC=local"
New-ADGroup -Name "Marketing_Users" -GroupScope Global -GroupCategory Security -Path "OU=Marketing,DC=medex,DC=local"

```

>🔹Import-Module ActiveDirectory 

> It loads the AD tools into PowerShell so we can manage users, groups, and computers from the command line.”
> In short: it enables PowerShell to work with and manage Active Directory objects.

<a href="/images/Medex_DC/Powershell.png">
  <img src="/images/Medex_DC/Powershell.png" width="600"/>
</a>

You can make sure it has been created by checking Active Directory Users and Computers.

I'm used to setting a simple password when working on any VM to speed up the process. So for now, I’ll disable password-complexity requirements and use an easy password during setup.


```powershell
Import-Module GroupPolicy

Set-ADDefaultDomainPasswordPolicy `
  -ComplexityEnabled $false `
  -MinPasswordLength 3 
  
  # or in one line
  Set-ADDefaultDomainPasswordPolicy -Identity "aaa.local" -MinPasswordLength 3 -ComplexityEnabled $false 

```
➡️ From GUI
go to Tools → Group Policy Management
Default Domain Policy > Edit
  Computer Configuration >
    Policies >
      Windows Settings >
        Security Settings >
          Account Policies >
            Password Policy 


## 📄 Script to Import Users from CSV File


> 💡 Note:
> All accounts are created as **enabled.**
To disable specific users, simply change the `"Enabled"` column in the CSV file from `True` to `False`,
> and the script will handle it accordingly.



```powershell
Import-Module ActiveDirectory 

# CSV Path
$users = Import-Csv -Path "C:\ADusers.csv"

foreach ($user in $users) {
    $Name = "$($user.FirstName) $($user.LastName)"
    $sam = $user.SamAccountName
    $ou = $user.OU
    $group = $user.Group
    $enabled = [bool]::Parse($user.Enabled)

    $newUser = New-ADUser `
        -Name $name `
        -SamAccountName $sam `
        -UserPrincipalName "$sam@medex.local" `
        -Path $ou `
        -AccountPassword (ConvertTo-SecureString "123" -AsPlainText -Force) `
        -Enabled $enabled `
        -ChangePasswordAtLogon $true `
        -PassThru

    Add-ADGroupMember -Identity $group -Members $sam

    Write-Host "✅ Created user: $name - Enabled: $enabled - Group: $group"
}

```


### ➡️ Explanation of the Script:


> 
> 
> ```powershell
> Write-Host "✅ Created user: $name - Enabled: $enabled - Group: $group"
> ```
> 
> This line simply means that script will display a message on the screen during execution to inform you that the user has been successfully created, along with some key details.
> 
> ---
> 
> ### 🧩 Breakdown:
> 
> - **`Write-Host`**
>     
>     Displays text directly to the PowerShell console. It only outputs text.
>     
>     > It only displays the message; it doesn't store or return a value.
>     > 
> - **`"✅ Created user: $name - Enabled: $enabled - Group: $group"`**
>     
>     This is the actual message displayed. It contains variables that change for each user created.
>     
> 
> ---
> 
> ### 📦 Variables explained:
> 
> - **`$name`**: Full name of the user (First Name + Last Name).
> - **`$enabled`**: Indicates whether the user account is enabled.
>     - `True` → Enabled
>     - `False` → Disabled
> - **`$group`**: The AD group that the user is assigned to
> 
> ---
> 
> ### 🧠 Additional logic:
> 
> ```powershell
> $enabled = [bool]::Parse($user.Enabled)
> ```
> 
> This line converts the  `Enabled` column from the CSV file (a string such as `"true"` or `"false"`) into a proper Boolean type (`$true` or `$false`).   
> This ensures the -Enabled parameter in New-ADUser is set correctly.  
>  `-ChangePasswordAtLogon $true ` force user to change Password At Logon  
>


### ➡️ Create an individual User with PowerShell

```powershell
New-ADUser -Name "Sara Brown" -SamAccountName "sbrown" -UserPrincipalName "sbrown@medex.com" -AccountPassword (ConvertTo-SecureString "P@ssword123" -AsPlainText -Force) -Enabled $true -Path "OU=Finance,DC=medex,DC=com"
```


### ➡️ Add that user to a group

```powershell
Add-ADGroupMember -Identity "Finance_Users" -Members "sbrown"
```


###  ➡️ Change Password at Logon for a specific OU

```powershell
Get-ADUser -Filter * -SearchBase "OU=IT_Admins,DC=medex,DC=local" | ForEach-Object {
Set-ADUser $_ -ChangePasswordAtLogon $true
}
```

`Get-ADUser -Filter *` → show all users.

**`SearchBase`** → specifies the OU or part of the domain to search for users. If you want to show all users in the entire domain, simply remove the `SearchBase` parameter.

`Set-ADUser -ChangePasswordAtLogon $true` → enables the "User must change password at next logon" checkbox for the selected users.

> 💡 **Tip:**
If you ever need to enforce **Change Password At Logon** for users in all or specific OUs,
you can automate the process using a simple PowerShell script.
>

```powershell

$OUs = @(
"OU=IT_Admins,DC=medex,DC=local",
"OU=IT_Support,DC=medex,DC=local",
"OU=HR_Users,DC=medex,DC=local",
"OU=HR_ReadOnly,DC=medex,DC=local",
"OU=Finance_Users,DC=medex,DC=local",
"OU=Finance_ReadOnly,DC=medex,DC=local",
"OU=Legal_Users,DC=medex,DC=local",
"OU=Marketing_Users,DC=medex,DC=local"
)

## foreach ($ou in $OUs) {
Get-ADUser -Filter * -SearchBase $ou | ForEach-Object {
Set-ADUser $_ -ChangePasswordAtLogon $true
  }
}

```

-----

### 🔄 **Still in Progress**

⏳ This isn’t the end!  

This documentation is currently under development.
