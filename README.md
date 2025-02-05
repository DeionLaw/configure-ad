<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>Pre-Reqs</h2>

- 2 machines (VMs in this case), one running Windows Server (our domain controller), and another running Windows itself (our client machine)
- Both machines on the same network (a virtual network through Azure in my example)
- Set the DC's private IP address to be static [in Azure it is found @ (your VM name) > networking > network settings]
- Set the clients DNS server to the domain controllers private IP address

<h2>Deployment and Configuration Steps</h2>

![Screenshot 2025-02-05 090901](https://github.com/user-attachments/assets/8d6b0e02-f113-46da-bd14-03069c91ec5c)
![Screenshot 2025-02-05 091012](https://github.com/user-attachments/assets/7417a407-f690-46fa-839d-e3c29a5fb7d9)


<p>
INSIDE DC1: Open 'Server Manager', click 'Add roles and features', following the prompts the only thing you should have to do is check 'Active Directory Domain Services' in the 'Server Roles' section
</p>

---

![Screenshot 2025-02-05 091610](https://github.com/user-attachments/assets/777670f4-b719-49f5-aee4-0d3b95d4b9a8)

<p>
  STILL INSIDE DC1: Next a flag warning should have came up top of the Server Manager where we can click it and then 'Promote this server to a domain controller', there we will 'Add a new forest' and I will name it 'mydomain.com', it does not matter unless in a real world environment.
</p>

---

  ![Screenshot 2025-02-05 092820](https://github.com/user-attachments/assets/4eda420b-25d7-4ce2-a0d6-0379f5f8d788)
  ![Screenshot 2025-02-05 092851](https://github.com/user-attachments/assets/2c505f95-831e-442f-8f4c-aad33af1a591)
  ![Screenshot 2025-02-05 092917](https://github.com/user-attachments/assets/926b76b5-0e5c-4ff2-9667-bc8c8ead494f)
  
<p>
  [If using RDP, you will have to specify that you want to now log in as a domain user (mydomain.com\username)]

  Now, we are going to create a Domain Admin in the domain. Go ahead and open 'Active Directory Users and Computers' inside the DC1 machine.

  Inside the 'mydomain.com' drop down we are going to create two new organizational units (OUs), one called '_EMPLOYEES' and another called '_ADMINS'. To do this just click right click the 'mydomain.com' and select 'New' > 'Organization Unit'
  </p>

---
![Screenshot 2025-02-05 093403](https://github.com/user-attachments/assets/fc823935-8ba3-4b9b-a191-69d415528461)
![Screenshot 2025-02-05 093512](https://github.com/user-attachments/assets/07bc67ac-e405-4f61-8857-85c2f5258d42)

<p>
  Now, inside the '_ADMINS' OU we are going to create a new admin employee, named Jane Doe here. Inside the '_ADMINS' OU, we can right click and then 'New' > 'User' and we can fill out our information for Jane Doe, our admin employee.
</p>

---

![Screenshot 2025-02-05 094311](https://github.com/user-attachments/assets/f40c0f58-0491-4c8a-b98c-0789693b8a87)

<p>
  Jane's account is not yet actually an admin, let us give her the permissions that make her so. Right click on the new 'Jane Doe' user found within our OU '_ADMINS', go ahead and click 'Properties' and head to the 'Member Of' tab. Here, we will add her to to 'Domain Admins' built in group by hitting 'Add...' and typing the name and hitting 'Check Names', it should capitalize and underline the 'Domain Admins', revealing to us it has found the built in permissions. Now Jane has a bunch of permissions that a domain admin would need to use. From now on, I will be logging into the DC server as the Jane admin account we created. (mydomain.com\jane_admin).
</p>

---
![Screenshot 2025-02-05 095756](https://github.com/user-attachments/assets/38206145-7075-45e9-ae75-ff6adc28afe6)
![Screenshot 2025-02-05 100018](https://github.com/user-attachments/assets/bc58dba4-30fd-466e-b9e3-08309530733d)
![Screenshot 2025-02-05 100043](https://github.com/user-attachments/assets/e43fa956-af54-40a1-b88b-0480933e651b)



<p>
  INSIDE CLIENT 1: Now we are going to add the client machine to the domain. Inside the system settings of windows, we can click the 'About' tab at the left, and choose 'Rename this PC (advanced) from the tab at the right. Here we can click the 'Change...' button available and set the 'Domain:' to your domain name, 'mydomain.com' for me. In the pop up, we can now log in as the Jane admin account we created, from the client machine since our client machine is now connected to the domain we created. It will now make you restart the client machine.
</p>

---

![Screenshot 2025-02-05 100429](https://github.com/user-attachments/assets/a199f159-e853-4fcd-bd72-1d9f7f8ca4b4)

<p>
  INSIDE DOMAIN CONTROLLER (LOGGED IN AS JANE ADMIN): We can now see that the client machine has been added to the Active Directory (AD), as shown in the screenshot above.
</p>

---

![Screenshot 2025-02-05 102904](https://github.com/user-attachments/assets/355e20f2-ab5c-42f3-a6f4-6a691f795086)
![Screenshot 2025-02-05 103143](https://github.com/user-attachments/assets/ed1333ff-0e48-41e0-b38c-ac2047506254)
![Screenshot 2025-02-05 103437](https://github.com/user-attachments/assets/a918d34e-0097-423c-b86e-1dc45f362227)


<p>
  INSIDE CLIENT 1 (LOGGED IN AS JANE ADMIN): Back in the 'About' system settings of windows, now logged into the client machine as Jane Admin, open the Remote Desktop settings. We are going to allow users to log into the domain using client machines. At the bottom choose 'Select users that can remotely access this PC'. In the pop up, click 'Add...' and you can choose type 'domain users', click 'Check Names', and like before the name will be capitalized and underlined, showing the system knows which group of people we are talking about. At this point no users actually exist, but we will create some using a powershell script.
</p>

---

![Screenshot 2025-02-05 103725](https://github.com/user-attachments/assets/2c31d9f7-388e-4c09-a59f-eea733a45d68)
![Screenshot 2025-02-05 104513](https://github.com/user-attachments/assets/831ebcfa-8c94-41d1-b9ea-e296a19ef536)



<p>
  INSIDE DOMAIN CONTROLLER (LOGGED IN AS JANE ADMIN): We are going to open 'Windows PowerShell ISE' as an administrator, click the 'new script' option at the top left under 'File' and we will paste the contents of the script provided below, and hit the green 'run' button at the top, this will create 40 random "employees" for us, if you would prefer you could create your own employees with more real names one by one, by hand.
</p>

<details>
  <summary>PowerShell create user script:</summary>
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 40

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
    $username = $firstName + '.' + $lastName
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
</details>

---


![Screenshot 2025-02-05 105244](https://github.com/user-attachments/assets/2f51bd18-de9d-4b8c-9d29-46ab3e807c6f)


<p>
  If you used the script above, you will find something like this in the '_EMPLOYEES' OU inside AD Users and Computers
</p>

---
![Screenshot 2025-02-05 105544](https://github.com/user-attachments/assets/110c9452-14f7-4cda-a368-0df2294b81ec)
![Screenshot 2025-02-05 110240](https://github.com/user-attachments/assets/d21e57bb-c810-42a3-b17e-b8edfa1e27e0)


<p>
Now, we can log into the Client machine as a random user that was created, the password will be 'Password1' if you used the script above. The first screenshot is showing the user inside of the Domain Controller machine. The second one is showing us logging into the client machine as that user, using RDP.
</p>

---

![Screenshot 2025-02-05 113516](https://github.com/user-attachments/assets/31e64661-5e16-43b0-9d24-353c84612ef1)


<p>
Now, we are going to create a group policy that makes it so users have a password attempt limit.
</p>

<p>
  This is going to take place INSIDE THE DOMAIN CONTROLLER (LOGGED IN AS JANE ADMIN): We are going to open 'Group Policy Management' and inside we can use the drop downs to go to 'Foreset: mydomain.com' > 'Domains' > 'mydomain.com' > Default Domain 'Policy'. We will right click this and click 'Edit...'.
</p>

<p>
  From there we will go to the drop downs of 'Computer Configuration' > 'Policies' > 'Windows Settings' > 'Security Settings' > 'Account Policies' > 'Account Lockout Policy'.
</p>

<p>
  Here, we will edit 2 different options available for customizing account lockouts on the domain.
</p>

<p>
  We will change the 'Account lockout duration' to 30 minutes. We will also change the 'Account lockout threshold' to 5 attempts. This will make it so if someone tries to log into their account wrong 5 times, they will be locked out for 30 minutes.
</p>

---
![Screenshot 2025-02-05 114945](https://github.com/user-attachments/assets/f035bc2c-abaf-40cc-81c3-dee0c413e133)



<p>
  INSIDE CLIENT 1 (LOGGED IN AS JANE ADMIN): The changes we just made will take effect in some time, to get them to change now we can open 'Command Prompt' and type 'gpupdate /force'. The changes that we made to the policies are now in effect.
</p>

---

![Screenshot 2025-02-05 115809](https://github.com/user-attachments/assets/91fc4e84-14b1-4d4f-b146-ba14b6d37368)
![Screenshot 2025-02-05 115829](https://github.com/user-attachments/assets/5cd7dbb4-8596-414e-ab6d-5edd229debfd)


<p>
Now, let's attempt to log into the client machine as a user, and fail the password prompt more times than allowed. After we hit the threshold amount, we should see the error pop up that is shown in the second screenshot, the account is now locked, based on the rules and policies we set.
</p>

---

![Screenshot 2025-02-05 120212](https://github.com/user-attachments/assets/952c5b26-8fbe-4451-9565-1ee902931778)
![Screenshot 2025-02-05 120412](https://github.com/user-attachments/assets/b160c0cc-6d4e-4ee3-a0bc-6ba79f6fce6c)
![Screenshot 2025-02-05 120506](https://github.com/user-attachments/assets/86972899-a13e-42ba-bee2-0401f7e43ed4)




<p>
  Now, inside of the users properies from inside of AD Users and Computers, we can see that the account has been locked out, checking the unlock account tick shown in the screenshot will unlock the account and let the user attempt log-ins again.
</p>

<p>
  If you would need to reset the password, you could right click the user and select 'Reset Password...', this will give you a prompt to change their password as shown in the second and third screenshot.
</p>

---

![Screenshot 2025-02-05 120705](https://github.com/user-attachments/assets/5530c62c-774f-478a-a6f5-a382232dd81e)
![Screenshot 2025-02-05 120743](https://github.com/user-attachments/assets/e0df8b77-f09d-48af-8241-8dc4e1ea3e5d)



<p>
  Next, we will go through disabling and enabling accounts, it is rather simple, just right click the user and select 'Disable Account', you will see attempts to log in will be denied until someone with power has reenabled the account.
</p>

---

<p>
  This ends the AD tutorial. You should now understand for the most part, how to handle user log ons using a domain and client setup.
</p>
