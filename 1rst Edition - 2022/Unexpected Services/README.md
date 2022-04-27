<div align="center">
  <a href="https://hackynov.fr"><img src="https://i.imgur.com/XGJF8Xu.png" alt="Hacky'Nov" width="50%"></a>
  <br><br>
  
  ![Discord](https://img.shields.io/discord/897766049099956284?label=Discord&style=for-the-badge)
  ![Twitter Follow](https://img.shields.io/twitter/follow/HackyNov?color=%231d9bf0&label=Twitter&style=for-the-badge)
  ![Twitch Status](https://img.shields.io/twitch/status/hackynov?color=%23772ce8&style=for-the-badge)
  
  <p>Follow us on</p>
  <a href="https://www.linkedin.com/company/hacky-nov/">Linkedin</a>
  •
  <a href="https://twitter.com/HackyNov">Twitter</a>
  •
  <a href="https://discord.gg/JGue7PhV">Discord</a>
  •
  <a href="https://www.twitch.tv/hackynov">Twitch</a>
  •
  <a href="">Youtube (coming soon)</a>
</div>

----
## Contact me

Pseudo : Cursecure<br/>
Pro : [Ludovic Mullot](https://www.linkedin.com/in/ludovic-mullot/)

## Write-up

Below you will see the content of the suspicious service detected :

![](Img/Capture1)

Note that the blue part is compressed and converted to Base64.
The first step will be to determine the compression mode. To do this we will use the CyberChef web application. Simple and intuitive, it will be enough do the necessary operations for the analysis of this PowerShell.

![](Img/Capture2)

As shown in the capture above, by adding the operation "From Base 64" and the operation "Detect File Type", the compression format is revealed to us. (In this case GZip).
The next step will be to unzip the text with the “Gunzip” operation.

![](Img/Capture3)

As shown in the capture above, the "Detect File Type" operation must be changed to "Gunzip". This allows to obtain the next part of the PowerShell, which you will find below.

![](Img/Capture4)

Note that there is another section in base 64, which is executed by kernel32 (as shown in the orange part).

Using CyberChef we decode the part in base 64.

![](Img/Capture5)

The figure above shows that the result is not exploitable. So we are still missing one or more steps. 
As mentioned above, the converted part must be executed by the Kernel (orange part of the PowerShell). We can therefore assume that the code is a Shellcode.
To check if it is a Shellcode, the result previously obtained in Hexadecimal with the operation "To Hex" must be converted into CyberChef. The objective is to obtain the code in an interpretable version by a disassembler. 

![](Img/Capture6)

After an online search search with part of the beginning of the hexadecimal obtained.  We use this part for our research because the first few bytes contain the magic numbers. Detecting these constants in files is a simple and effective way to distinguish between many file formats.

![](Img/Capture7)

Unfortunately our code does not have «magic numbers». 
However we find that this code is already known and we can even find another Shellcode with similarities.

![](Img/Capture8)
![](Img/Capture9)

The many similarities between the two hexa make it possible to determine that it is a shellcode from msfvenom.
The differences between the two codes are probably due to changes made by the hacker to alter the signature of the code, in order to avoid being detected by an antivirus.

We now need to disassemble the Shellcode to get the following information:
• Determine his actions; 
• Check the presence of an information hard coded like an IP or a port;
 To do this we add in CyberChef the operation «Disassemble x86».

After analysis of the differencies between the two shellcode we can find an IP and a port hardcoded in the shellcode.

![](Img/Capture10)

The IP address and destination port are green on the illustration above.

----

Once the IP address and port are retrieved, it is now necessary to determine whether the port is still open and what it is associated with.
For this we use the command :
````shell
nmap -Pn -sV 5.196.224.102 -p 4444
````
![](Img/Capture11)

From the information gathered it seems that port 4444 is an http service. You can now  display the content of the default page with your web-browser or with curl :
````shell
curl 5.196.224.102:4444 to
````

![](Img/Capture12)
![](Img/Capture13)

Flag : HACKYNOV{You_Have_Benn_Pawned}
