---
layout: post
title: "Proving Grounds Craft: Walkthrough"
---
Today's CTF blog will cover Proving Grounds machine Craft.

# Enumeration

I'll start by kicking off an `nmap` scan of all 65535 TCP ports to see what we're working with.

![nmap TCP Scan 1](/assets/img/PG-Craft-Walkthrough_Nmap-Tcp-Scan-1.png)

It looks like the target is blocking ICMP packets. Not a problem, `nmap` can be instructed to skip host discovery using the `-Pn` flag. Let's try that again.

![nmap TCP Scan 2](/assets/img/PG-Craft-Walkthrough_Nmap-Tcp-Scan-2.png)

The scan only returned port 80, which is serving some sort of HTTP service. Let's gather more information about this service with an aggressive scan.

![nmap TCP Scan 3](/assets/img/PG-Craft-Walkthrough_Nmap-Tcp-Scan-3.png)

The scan didn't return anything too noteworthy, but I'll jot down the service names and versions to research later. As a side note, make sure to enumerate UDP ports as well. In this particular case, the scan didn't return any ports.

![nmap UDP Scan](/assets/img/PG-Craft-Walkthrough_Nmap-Udp-Scan.png)

## Web Application Enumeration

HTTP(S) services call for directory/file brute forcing. I'll use the `gobuster` **dir** mode and provide the `-x` flag to specify the file extensions I want to test. It doesn't take long to find some interesting stuff like `/uploads` and `/upload.php`.

![gobuster Scan](/assets/img/PG-Craft-Walkthrough_Gobuster-Scan.png)

I'll browse out to the root directory in Firefox to see what the rendered page looks like. Google Translate wasn't any help, but it's safe to assume that this is some sort of business website. Further down the page, I found an upload feature for submitting resumes!

![Craft Landing Page](/assets/img/PG-Craft-Walkthrough_Craft-Landing-Page.png)

![Craft Upload](/assets/img/PG-Craft-Walkthrough_Craft-Upload.png)

To test the upload function, I will provide a simple text file to see what happens. It appears this `upload.php` file checks the file extension to ensure only `.odt` files are allowed.

![Craft Upload Test 1](/assets/img/PG-Craft-Walkthrough_Craft-Upload-Test-1.png)

![Craft Upload Test 2](/assets/img/PG-Craft-Walkthrough_Craft-Upload-Test-2.png)

Now it's time to enlist the help of BurpSuite to modify the POST request. I spent a lot of time messing with different PHP file extensions and some null byte shenanigans, but nothing worked. If I want to upload a file to this web server, it must have a `.odt` file extension.

![Craft Upload BurpSuite](/assets/img/PG-Craft-Walkthrough_Craft-Upload-BurpSuite.png)

## .odt File Research & Creation

Having created malicous Office 365 macros in the past, I assume LibreOffice may have similar functions that can be abused. I did some Googling and found this [Medium article](https://medium.com/@akshay__0/initial-access-via-malicious-odt-macro-ac7f5d15796d) detailing malicious ODT file macros. The syntax for this macro is targeted for Linux machines, so the script itself is not useful to me, but the instructions for creating a LibreOffice macro are still of value.

Knowing this, I searched for Microsoft VBA Macro reverse shell scripts and found another [Medium Article](https://medium.com/@mavrogiannispan/phishing-2-0-9f49654de4a6) written by Mavrogiannis Panagiotis that contained proper syntax for Windows targets. First, I need to create a malicious document in LibreOffice. Select **File > New > Text Document** to create a new file. Once the document window opens, navigate to **File > Save As..** and save the file, ensuring the `.odt` extension is specified.

![Odt Step 1](/assets/img/PG-Craft-Walkthrough_Odt-Step-1.png)

![Odt Step 2](/assets/img/PG-Craft-Walkthrough_Odt-Step-2.png)

Next, I will select **Tools > Macros > Organize Macros > Basic...** to open up the *BASIC Macros* window. In the window, select the name of the saved document and then click the **New** button on the right-hand side to create a new module. In the pop-up window, provide a name for the module and click **Ok**.

![Odt Step 3](/assets/img/PG-Craft-Walkthrough_Odt-Step-3.png)

![Odt Step 4](/assets/img/PG-Craft-Walkthrough_Odt-Step-4.png)

![Odt Step 5](/assets/img/PG-Craft-Walkthrough_Odt-Step-5.png)

Now I can copy the script structure from the Medium article and paste it into the macro window.

![Odt Step 6](/assets/img/PG-Craft-Walkthrough_Odt-Step-6.png)

You may have noticed that I left out the entire string from the `objshell.Run` line. I am slightly modifying the VBA script example by generating a PowerShell Base64-encoded reverse shell command using `msfvenom`. I prefer this format because I don't have to fiddle with quotations within quotations which can be a headache at times. I'll copy everything starting from `powershell.exe` and paste into the empty double quotes in my script.

![Odt Step 7](/assets/img/PG-Craft-Walkthrough_Odt-Step-7.png)

![Odt Step 8](/assets/img/PG-Craft-Walkthrough_Odt-Step-8.png)

The last step is to tie this macro to an event. I want this script to execute immediately after the document is open. To do this, I'll navigate to **Tools > Customize**. Inside the Customize window, click the  **Events** button, select the **Open Document** event, and click **Macro...** on the right-hand side.

![Odt Step 9](/assets/img/PG-Craft-Walkthrough_Odt-Step-9.png)

Expand the document tree until you find the macro name and select it. Click **Ok** on the Macro Selector and Customize windows. Once in the document window, press `Ctrl + S` one more time for good measure and close LibreOffice.

![Odt Step 10](/assets/img/PG-Craft-Walkthrough_Odt-Step-10.png)

# Initial Access

Now it's time to upload the `.odt` file. Before I do that, I first need to setup a listener on Kali to catch the reverse shell connection. I am prepending my `nc` command with `rlwrap` which will help stabilize the Windows shell. Browse out to the root web directory again and select the saved `.odt` file and select the **Upload** button.

![InitialAccess Step 1](/assets/img/PG-Craft-Walkthrough_InitialAccess-Step-1.png)

![InitialAccess Step 2](/assets/img/PG-Craft-Walkthrough_InitialAccess-Step-2.png)

After a few seconds pass, a reverse shell is received from the target!

![InitiAccess Step 3](/assets/img/PG-Craft-Walkthrough_InitialAccess-Step-3.png)

# Post-Exploitation Enumeration

Let's see if there are any flags we can access as `thecybergeek`. This is a PowerShell one-liner that I use to scrape flag content in the `C:\Users\` directory.

```powershell
Get-ChildItem -Path C:\Users\* -Include "local.txt","proof.txt" -Recurse -ErrorAction SilentlyContinue | % {Write-Host "[!] $($PSItem.FullName)`r`n`t$(Get-Content $PSItem.FullName)"}
```

![Local](/assets/img/PG-Craft-Walkthrough_Local.png)

Local is now captured! Moving onto further enumeration, I'll start with examining service information to check for any unquoted service paths using another PowerShell one-liner.

```powershell
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$PSItem.State -eq 'Running'}
```

I immediately notice a custom service named **ResumeService1** with an unquoted service path of `C:\Program Files\nssm-2.24\win64\nssm.exe`.

![Post Exploit Enum 1](/assets/img/PG-Craft-Walkthrough_Post-Exploit-Enum-Unquoted-1.png)

If my current user has write access to the `C:\` directory, I could place a malicious executable there, restart the service, and send myself an NT Authority\SYSTEM shell. Unfortunately, my user cannot create files in that directory.

![Post Exploit Enum 2](/assets/img/PG-Craft-Walkthrough_Post-Exploit-Enum-Unquoted-2.png)

I didn't find anything noteworthy my current user's home directory, nor do I have any interesting privileges. I decided to enumerate all of the users on the target and noticed an **apache** username. What if I have write access to the Apache root directory?

![Post Exploit Enum Apache 1](/assets/img/PG-Craft-Walkthrough_Post-Exploit-Enum-Apache-1.png)

## Lateral Movement

To find the root directory, I remembered from my `gobuster` enumeration that I found a file called `upload.php`. I can search for that filename to locate the root directory of the Apache web server. Once I discovered the `C:\xampp\htdocs\` root directory, I attempted to write a text file to that location and it worked! I further verified this by curling the new web server file from Kali.

```powershell
Get-ChildItem -Path C:\ -Filter "upload.php" -Recurse -File -ErrorAction SilentlyContinue | % {$PSItem.FullName}
```

![Post Exploit Enum Apache 2](/assets/img/PG-Craft-Walkthrough_Post-Exploit-Enum-Apache-2.png)

![Post Exploit Enum Apache 3](/assets/img/PG-Craft-Walkthrough_Post-Exploit-Enum-Apache-3.png)

I've confirmed my write access on the Apache web server. Now I just need to write a PHP backdoor file instead of a text file. I'll host the `simple-backdoor.php` file from Kali using a Python HTTP server and download the file using the `Invoke-WebRequest` Cmdlet.

![Post Exploit Enum Apache 4](/assets/img/PG-Craft-Walkthrough_Post-Exploit-Enum-Apache-4.png)

![Post Exploit Enum Apache 5](/assets/img/PG-Craft-Walkthrough_Post-Exploit-Enum-Apache-5.png)

I now have code execution as **apache**!

![Post Exploit Enum Apache 6](/assets/img/PG-Craft-Walkthrough_Post-Exploit-Enum-Apache-6.png)

Following the same form as the `.odt` file macro, I'll once again lean on `msfvenom` to generate a PowerShell Base64-encoded reverse shell command to avoid URL encoding shenanigans. I can't avoid URL encoding entirely, I'll need to replace the spaces in my command with either `%20` or `+` characters to make the URL valid. If you are entering this command from the browser, this step will be automated. Don't forget to setup the `nc` listener on Kali before executing the command!

![Post Exploit Enum Apache 7](/assets/img/PG-Craft-Walkthrough_Post-Exploit-Enum-Apache-7.png)

![Post Exploit Enum Apache 8](/assets/img/PG-Craft-Walkthrough_Post-Exploit-Enum-Apache-8.png)

I now have a shell as the **apache** user.

![Post Exploit Enum Apache 9](/assets/img/PG-Craft-Walkthrough_Post-Exploit-Enum-Apache-9.png)

# Privilege Escalation