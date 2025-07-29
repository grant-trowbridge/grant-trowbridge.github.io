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

# .odt File Research & Creation

Having created malicous Office 365 macros in the past, I assume LibreOffice may have similar functions that can be abused. I did some Googling and found this [Medium article](https://medium.com/@akshay__0/initial-access-via-malicious-odt-macro-ac7f5d15796d) detailing malicious ODT file macros. The syntax for this macro is targeted for Linux machines, so the script itself is not useful to me, but the instructions for creating a LibreOffice macro are still of value. Knowing this, I searched for Microsoft VBA Macro reverse shell scripts and found another [Medium Article](https://medium.com/@mavrogiannispan/phishing-2-0-9f49654de4a6) written by Mavrogiannis Panagiotis that contained proper syntax for Windows targets. First, I need to create a malicious document in LibreOffice. Select **File > New > Text Document** to create a new file. Once the document window opens, navigate to **File > Save As..** and save the file, ensuring the `.odt` extension is specified.

![Odt Step 1](/assets/img/PG-Craft-Walkthrough_Odt-Step-1.png)

![Odt Step 2](/assets/img/PG-Craft-Walkthrough_Odt-Step-2.png)