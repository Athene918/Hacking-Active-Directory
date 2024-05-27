# Hacking Active Directory with AS-REP Roasting

![image](https://miro.medium.com/v2/resize:fit:640/format:webp/1*0H8fU7OUFtZ-E47H1zQslQ.png)

## Introduction
AS-REP Roasting is an attack that exploits a weakness in the Kerberos authentication protocol. Specifically, it targets the AS-REP (Authentication Server Response) message used during the initial authentication process when a user logs into a Kerberos-protected network.

During the AS-REP Roasting attack, an attacker captures a valid AS-REP message from the Kerberos authentication server, typically sent when a user logs into the network. The attacker then uses brute-force techniques to crack the encryption to protect the AS-REP message.

Once the attacker has successfully decrypted the AS-REP message, they can obtain the user’s password hash, which can be used to launch further attacks against the network. With the password hash in hand, the attacker can attempt to crack the hash using tools such as John the Ripper or Hash Cat, which can often reveal the plaintext password.

## Creating Users

In a Domain Controller, open **Windows Server Manager.**

Click on **Tools → Active Directory Users and Computers.**

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*wT8Ts90s2RwYECBp2ah-VA.png)

Expand **Local Domain (MARVEL.local).**

Click **Users.**

![image](https://miro.medium.com/v2/resize:fit:640/format:webp/1*VlBtUJB2WjkD2yDzF36RZg.png)

Right-click **Users** → click **New** → click **User.**

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*823rrqxPJjf-pU-ejgzvFQ.png)

Giver new User a **First/Last name** and a User logon **name.**

In my case, I gave the user the following:

First name: **Peter**

Lastname: **Parker**

User logon name: **pparker**

Click **Next.**

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*LNsQY9iCw9RxtsCXrkZRoQ.png)

Give the new User a password.

Password: **Password1234**

Uncheck **User must change password at next logon.**

Click **Next.**

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*SWGWiBX1i7psTsii2LpQ1Q.png)

Click **Finish.**

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*GWC29LtxceRoeATLvnzW1g.png)

The new user account, **Peter Parker**, has been successfully created.

Go to **Peter Parker Properties** by double-clicking on **Peter Parker.**

![image](https://miro.medium.com/v2/resize:fit:640/format:webp/1*TnBGzqv_wFx0y-7GobWP5w.png)

In Peter Parker Properties, click **Account.**

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*woaNwvjrGnnplngRThEpFQ.png)

Kerberos preauthentication is enabled by default.

In the **Account options**, check **Do not require Kerberos preauthentication.**

Click Apply → **OK.**

This will simulate a misconfiguration since it’s highly recommended not to disable Kerberos preauthentication. With Kerberos preauthentication disabled, we’re more vulnerable to AS-REP Roasting. In some cases, it may be necessary to disable it to integrate with specific applications or whatever the cause might be. Still, for the most part, there’s no reason to disable Kerberos preauthenticaiton.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*knuPKZiUMpD9o5quc4KDRA.png)

## Performing AS-REP Roasting

When performing AS-REP Roasting, you must have a list of usernames or know the target’s username. So, using a tool called GetNPUsers.py through Impacket allows us to acquire domain users who have a “Do not require Kerberos preauthentication” set and ask for the ticket-granting tickets (TGT) without knowing their passwords.

The TGT contains information that the client can use to request additional tickets from the ticket-granting server (TGS), which is used to authenticate the client to specific network services. You can then try to crack the session key sent along with the ticket to retrieve the user password.

Open **Kali Linux terminal.**

The command format: **GetNPUsers.py local domain name/user logon name -dc-ip ip address of the domain controller.**

For example, type **GetNPUsers.py MARVEL.local/pparker -dc-ip 10.60.0.9**

Click **Enter.**

Enter **Kali’s host password.**

Click **Enter.**

An encrypted TGT hash is generated.

To crack it, I first highlighted the hash.

Then **right-click → Copy Selection.**

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*FWN16pm_CUWcWxcA7I8N7Q.png)

Open a new file by typing **vim tgt**

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*BDVex7YY-oHq9wCL3AGEEA.png)

**Right-click** → click **Paste Selection.**

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*Ex3ESd-Rtu4sCCBB0xs2Yg.png)

I pasted the hash in the tgt file.

![image](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*MP45cUq3Vt31pvTpghaM2Q.png)

To exit the file, **click Ctrl + Copy → type :wq → click Enter.**

![image](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*yfLlybE4e6XXH7GY-96Uqw.png)

Type **sudo john tgt.**

Enter Kali’s host password.

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*F1-S2Ki2GBlvC5zEMS8ggw.png)

I attempted to Brute Force, but because of the lengthy process, I aborted and used John the Ripper since I already knew the password.

To abort the session, click **Ctrl + C.**

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*dow1vIzIWkACnpfrtel1gg.png)

Open a new password file: Type **vim thepass.txt**

![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*S8yfyIFw6WXDDDimAnMc3Q.png)

Type the password: **Password1234.**

To exit the file, click **Ctrl + Copy** → type :**wq → click Enter.**

![image](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*tjJI1kmjY4w45IxColLnVw.png)

Run John the Ripper

````bash
sudo john tgt --wordlist=thepass.txt
````
![image](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*vZPgNR5s_H5VIXo8S3-8Eg.png)

The password was cracked!!!

**Password1234**
![image](https://miro.medium.com/v2/resize:fit:720/format:webp/1*ZVCIRQRZmkBuD2h0Yf433g.png)
