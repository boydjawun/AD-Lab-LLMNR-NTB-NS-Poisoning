# Building AD Lab, LLMNR Poisioning, and NTLMv2 Cracking w / Hashcat

# Building AD Lab

To set up an AD(Active Directory) Lab you will need will need:

- Virtualization software: VM Workstation Player or Oracle Virtual Box
- Microsoft Windows Server 2022
- Microsoft Windows 11

# LLMNR Poisioning

- Start off on Windows Server 2022 to File and Storage Services, then click shares

![Screenshot 2025-12-16 102059.png](Screenshot_2025-12-16_102059.png)

- Open your files and go to the local C: drive on your computer and create a folder called hackme

![Screenshot 2025-12-16 102257.png](Screenshot_2025-12-16_102257.png)

- Back on the Server Manager screen, click tasks in the top right corner, then click new share in the dropdown menu

![Screenshot 2025-12-16 102710.png](Screenshot_2025-12-16_102710.png)

- Select SMB Share quick, then Next >

![Screenshot 2025-12-16 103037.png](Screenshot_2025-12-16_103037.png)

- Select the type a custom path for your share location

![Screenshot 2025-12-16 103231.png](Screenshot_2025-12-16_103231.png)

- Select the hackme folder that was made on the C: drive, then click Next>

![Screenshot 2025-12-16 103411.png](Screenshot_2025-12-16_103411.png)

- In the Share Name section, we are able to see all of or share information including the share’s local and remote path locations

![Screenshot 2025-12-16 103625.png](Screenshot_2025-12-16_103625.png)

- After clicking Next> click the box to enable access based enumeration

![Screenshot 2025-12-16 104116.png](Screenshot_2025-12-16_104116.png)

- In the Permissions section you can see that Users and Administrators have both read and write permissions

![Screenshot 2025-12-16 104339.png](Screenshot_2025-12-16_104339.png)

- Now click create

![Screenshot 2025-12-16 104515.png](Screenshot_2025-12-16_104515.png)

- Share was successfully created

![Screenshot 2025-12-16 104640.png](Screenshot_2025-12-16_104640.png)

- Now we head back to Windows 11 VM to add that share we just created. Log in as created user
- Head to this PC in the file explorer and select to map a network drive
    
    ![image.png](image.png)
    
- Enter a drive and enter the name of the server and the folder for the share and click finish

![image.png](image%201.png)

- Now we have access to the JAWUNSERVER hackme share

![image.png](image%202.png)

# LLMNR(Link Local Multicast Name Resolution)/NBT-NS Poisioning

- ( Write and tell about LLMNR Poininonsg). NBTNS - NetBIOS name service
- They are used to identify hosts when the DNS fails
- Flaw - Both the services use the Users’ Username and their hash(NTLMv2 sometimes NTLMv1)
- Using this name resolution to access a share

### Man-In-The-Middle Attack

- While User hashes are being used over and over, typically the hashes are going out to a known share or a know device. Sometimes there is a mistake with where the hashes need to be sent and a man in the middle could accept that hash
- Go to Kali and open the root terminal to locate [Responder.py](http://Responder.py) then cd into it. The responder picks up network traffic

![image.png](image%203.png)

- While still in the responder folder run the command python [Responder.py](http://Responder.py) -I eth0 -wd to start the listener

![image.png](image%204.png)

- Notice how the LLMNR and NBT-NS poisiners are on and also the servers that [Responder.py](http://Responder.py) is listening on
- Now head back to the windows VM logged in as a created user and head to the hackme share

![image.png](image%205.png)

- The folder is empty but the responder captures this instance

![image.png](image%206.png)

- The response by the Responder on the Linux VM

![image.png](image%207.png)

- Open a new tab in the Linux VM terminal and get your IP address

![image.png](image%208.png)

- Place the IP address in the path name box

![image.png](image%209.png)

- You are not going to be able to get into it but

![image.png](image%2010.png)

- There is the hash sent to the the Linux Terminal for user bfranks
- Includes the Domain name(JAWUN), the username(bfranks), and the user’s NTLMv2 hash then skips the hashes that have already been captured

![image.png](image%2011.png)

- Responder stores the hashes in /usr/share/responder/logs

![image.png](image%2012.png)

- Now copy the entire hash including the user and domain

![image.png](image%2013.png)

- Navigate to the root folder and create a file to post the hash in, Nano is used here

![image.png](image%2014.png)

- Now to cracking the hashes
- Using Hashcat on Kali Linux to find the password in the hashes.txt file created with the hashes from the LLMNR poisioning
- After typing hashcat —help see, in the Network Protocol section, that NetNTLMv2 is the is the protocol that is going to be used to crack this hash becasuse this is a NTLMv2 hash

![image.png](image%2015.png)

- Type in the terminal
    - hashcat -m 5600 hashes.txt /usr/share/wordlists/rockyou.txt
    - -m 5600 specifies the protocol to use when cracking the hash
    - Also use a good password list, in this example rockyou.txt.gz is used. A built in wordlist on Kali Linux
    
    ![image.png](image%2016.png)
    
- It didnt work, so we found another alternative

![image.png](image%2017.png)

- Head back to the Users profile on Windows 11 and open the command prompt and Run as Administrator
- For the Username I used Administrator and password is the password you set for your account when setting up Windows 11

![image.png](image%2018.png)

- Head to hashcat’s official website and download the binaires which will include a .7z file

![image.png](image%2019.png)

- Extract the file, my have to use 7zip

![image.png](image%2020.png)

- Go back to Kali and get the hash from the responder logs to copy it in the notepad on Windows

![image.png](image%2021.png)

- type notepad in the Commad Prompt to open the Notepad app. Paste the hash in there and save the file as Hashes.txt

![image.png](image%2022.png)

- Using notepad wouldn’t so I started a Python HTTP server and used wget on the Windows Machine to retrieve the hashes.txt file with Powershell running as Administrator

![image.png](image%2023.png)

![image.png](image%2024.png)

- View the file using the type command in Windows

![image.png](image%2025.png)

- I have multiple hashes of the same profile saved into this file from accessing the file and requesting Kali machine multiple times. So I chose to edit the file with notepad. I used Resolve-Path -path “hashes.txt” to reveal the location of hashes.txt. Then Used notepad.exe and the file path to edit the hashes.txt file

![image.png](image%2026.png)

- After deleting the extra hashes, I click save to save the the file under the same name and in the dame location

![image.png](image%2027.png)

- Hashes.txt has been edited now. There is only one username, domain name, and hash for that user
- Below I use Reslove-Path -path “hashes.txt” to find the path of hashes.txt in Powershell running as Administrator

![image.png](image%2028.png)

- Now go back to the commnd prompt and run as administrator

![image.png](image%2029.png)

- Navigate back to the created user

![image.png](image%2030.png)

- Now with the command prompt running as Administrator run hashcat.exe