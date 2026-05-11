# NetworkDevicesLogOut
Disconnects SMB shares and mapped network drives without removal unless they are stuck. <br/>
It's the worlds first automated tool to remove even so-called "ghost drives" (unaccessible mapped drives in Windows Explorer). <br/>
(Release Date: 11.05.2026, Publisher: Dragodraki alias Dreamland, Notice: no fork) <br/>

[<img src="https://user-images.githubusercontent.com/76787321/197257488-1b7aa8e9-9b6f-4600-949e-8ff477cb4bf4.png" width="23%"></img>](https://github.com/Dragodraki/NetworkDevicesLogOut/releases/latest/download/Network_devices_log-out)
<br>
<br>

-------------------------------
EXPLANATION
-------------------------------
At some point all of us have faced the SMB protocol implemented in Windows with NetBIOS/CIFS implementation, CIFS began after Windows 2000 and is newer. As soon as we connect to a local (intranet) device's file system, it means we use SMB. This protocol is very well-known because of two characteristics: practical with ease to use as like as popular target of hackers.

By connecting to another devices filesystem in Windows, the Windows/File Explorer gains the same rights on the files as if we were directly connected to it. Mounting it by assign an own drive letter gives the additional benefit of accessing it with the same file path even from mundane batch scripts - making it automatically reconnect at logon and storing the credentials in Windows fault spare us the effort to type in server path, user name and password again next time. Questionable advantage, since it is precisely the one that hackers are most likely to exploit for their criminal purposes. Sure, they can try to break the authentication method and execute kernel code on remote systems without even the system ever connected to it before like they did with WannaCry on SMBv1 in 2017 or SMBGhost on compressed SMBv3 in 2020 (what is called a worm) – but the most malware is not so sophisticated and focus on existing connections. Not having the SMB connection(s) open all the time and deleting saved credentials from Windows vault is a crucial step for isolating intranet systems to protect lateral movement.

That's where my software comes in. The tool „Network devices log-out.exe“ combines several steps and is completely harmless to use, as long you don’t have open network file transfers open (local ones does not count), but a message prompt notices you about that anyway. In simple terms, it makes Windows forget about you successfully logged in to any SMB share. While it removes open shares (UNC paths) and deletes stored SMB credentials from the Windows vault (only domain-related, all other credentials like MS Teams or others will never be deleted), it does not remove mapped drive letters. The SMB paths are unconnected now, without doing a reboot. On the next windows logon you will see the 'error' message complaining about restoring network drives  not being posssible (yes, that is correct, because exactly that is what we wanted to achieve).

After execution of „Network devices log-out.exe“ you can double click the drive again to enter the credentials again (better not permanent save it this time). It’s really recommended to use this tool again as soon you finished file transfer and don’t need the SMB share/drive be open anymore! Not because it has to but your remote devices (like another Windows client, server or NAS) remains much safer this way in general.
<br>
<br>

-------------------------------
LICENSE (FREEWARE)
-------------------------------
In one sentence: It's classic FREEWARE for everyone

Permissions:
+ Private and Commcercial usage is allowed as long you don't demand money for it ;)
+ Forks are allowed, but they have to kept free of charge ;)
+ Free Distribution to friends or strangers is allowed, even wanted ;)

Limitations:
- Use the app at your own risk!
- Don't sell it as product or pretend to be its developer!
- Software must NOT be altered/hacked/decompiled (or similar)!
- You have to follow the EULAs from the "3rd party apps" as well!
- If you copy/fork my project, you have to contribute me as original author as well as everyone from subfolder "3rd party apps"
- Don't abuse it for malicious purposes!
<br/>
ANY DISBEHAVIOUR AGAINST THESE RESTRICTIONS OR DAMAGE TO YOUR SYSTEM BY MY SOFTWARE I ASSUME NO LIABILITY !!!
<br/>
<br/>

-------------------------------
USAGE
-------------------------------
-	normal run by double clicking the exeuctable. 
-	silent/hidden run by launching with parameter /VERYSILENT

Important:
If you cannot see your previous mapping, you indeed have to kill and restart explorer.exe yourself! It would to aggressive to put that into my script (cause it interferes with copying transfer windows), so I sticked to an attempt refreshing it without killing the process.
<br>
<br>

-------------------------------
WINDOWS SUPPORT
-------------------------------
This setup was build to support all Windows OS beginning with Windows 2000.
The OS I actually tested the software are labeled with "(verified)" at the end:

- Windows XP (verified)
- Windows Server 2003
- Windows Vista
- Windows Server 2008 R1
- Windows 7 (verified)
- Windows Server 2008 R2
- Windows 8/8.1
- Windows Server 2012 R1
- Windows Server 2012 R2
- Windows 10 (verified)
- Windows Server 2016
- Windows Server 2019
- Windows 11 (verified)
- Windows Server 2022
- Windows Server 2025
- ... probably next future Windows OS too
<br>
<br>

-------------------------------
HOW IT WORKS TECHNICALLY
-------------------------------
This time I not only to keep it simple (no promise its to be short though): The executable is actually an inno setup wizard. Instead of installing anything, the script routine performs all steps needed to disconnect the SMB shares/drives properly. First, it saves the content of HKEY_CURRENT_USER\Network\ to apply it later again (without the mappings would not be disconnected but removed).
At next all mapped drives are removed gracefully by command net use * /delete /y. This has to be done with medium integrity level (not elevated!, it does not even work elevated if you are member of the administrator group - just dont ask why). To achieve that, the tool itself has to be launched as de-elevated another instance in the context of the logged-in session user (that was really tricky, I use my own tool „RunasMe.exe“ for that, you can find in my entire own repository „RunasMe“). But that is not enough – the very same command hast o be executed with SYSTEM privileges too, to stuck mapped drives that were initiated by a service. To all administrators out there: Distributing SMB drives as SYSTEM/NT-Authority won’t work, it just makes it appear is inaccessible drive that can not be removed by normal elevated rights.
The service "LanmanWorkstation" should be restarted at this point (crucial if net use did not got applied due to open file transfers or open explorer.exe windows). Without it, we cannot be sure that the session is really terminated immediately. Therefore we just run the command with normal elevated rights. Why we did the previous step in the first place if we force termination anyway? - Because otherwise we would create so-called "ghost drives" which are remnants of earlier connections but cannot be deleted. They would need a account re-logon to disappear and I don't want that to happen.
Now the registry information from the first step is being re-applied. It results in previous mapped drives will show again (but disconnected ths time) in Windows Explorer.
Finally, the explorer.exe process should be restarted. In a clean way it would  mean, killing explorer.exe and restart the process again. But I am not fond of it since it could break any open file transfers mid-operation. Thats why I implemented my own refresh routine that has a success rate of approximately 80% by using Rexplorer.exe from Sordum and one of my own coded programs for that. If you still cannot see your previous mapping, you indeed have to kill and restart explorer.exe by yourself (but it would to aggressive to do this automatically for my taste).
<br>
<br>

-------------------------------
SPECIAL THANKS
-------------------------------
- Thank you very much for your arsenal of tools, NirSoft! Especially your product "AdvancedRun" comes in handy for me to ignore the UAC, which is somethat tricky. Another very, very useful feature is your GUI - as RunasMe not only use "AdvancedRun" of course, but also others that invoked or be invoked by it, I had to do a lot quotation mark escaping that gave me headache even with your app as 'debugger'. Maybe it would be helpful if you could add it to your website, because without quotation marks only one parameter without spaces would have been working.
- Anyway: As RunasMe's project code is much more comprehensive than only calling AdvancedRun.exe, I decided against naming them within my app as developer.
- Thanks you too, Uwe Sieber, for giving me an excellent alternative to psexec.exe with your RunAsSystem.exe which even works on Windows XP and has a much better reputation on VirusTotal than Sysinternals for years.
- Guillaume’s notifu.exe was a solid solution for me to include a nice customizable notification toast / pop-up balloon at the end oft he script. Inno Setup does not offer any code to do it by itself and is not compatible with known Lazarus routines. So, great work!
<br>
