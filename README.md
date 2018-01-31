# Mr-Robot-CTF
Will Pfleger
1/30/18

This is my write-up for the Mr Robot CTF hosted on vulnhub.com I will be installing 
the CTF VM in Oracle VirtualBox and I will run my penetration test from a Kali Linux
VM also intalled in VirtualBox. The 2 virtual machines are connected via a host-only 
NAT network hosted by VirtualBox.

The inital login screen of the vulnerable VM immedately prompts you for a username
and password. I tried several typical combinations (e.g. "admin/admin") but could't
log on successfully. I then moved to my Kali VM to run a port scan of the subnet
(the virtual network) "nmap -sP 10.0.2.15/24" and found the Mr Robot VM's local IP address to 
be. A more detailed port scan ("nmap -A 10.0.2.15") revealed an HTTP service running on port 80.
After running "firefox 10.0.2.15" and finding a Mr. Robot-based intro video, I check for a 
robots.txt file on the web server and successfully found one. The conents were:

"User-agent: *
fsocity[sic].dic
key-1-of-3.txt"

Running a "wget 10.0.2.15/key-1-of-3.txt" yielded the first key file, which contained the string
"073403c8a58a1f80d943455fb30724b9". I then ran a Nikto scan to further inspect the running web
server "nikto -h 10.0.2.15". This showed an active Wordpress account running on the target machine
(per Nikto: "/wp-login/: Admin login page/section found."). I opened 10.0.2.15/wp-login in Firefox
and tried to guess a few username/password combinations with no luck. However, it being a Mr.
Robot-themed CTF, one of my username guesses was "elliot" and the error message I got for that
username gave me a clue. Instead of giving me the usual "incorrect password" message, I got
"The password you entered for the username elliot is incorrect". This gave me the idea to use
the username "elliot" against the dictionary file I downloaded from the webserver (10.0.2.15/
fsocity.dic) in a brute force attack. I initally attempted to use Hydra to brute force the
password but was unsuccessful. I then tried wpscan instead and ran "wpscan -u 10.0.2.15 --wordlist
./fsocity.dic --username elliot" and after a lengthy scan came up with the password "ER28-0625"

I used this username/password combination to log into Wordpress. I played around with the functionality
of the Wordpress and guessed that since I could add/modify plugins, I was a privileged user. Since Nikto
found a vulnerable version of Wordpress on the VM and since I had admin credentials, I decided to open Metasploit.
The exploit I chose was a Wordpress PHP reverse shell (msfconsole>"use exploit/unix/webapp/wp_admin_reverse_shell.php").
I set all the options accordingly (RHOST and RPORT were set to the victim's IP and target port, respectively. 
USERNAME and PASSWORD were set to the credentials I brute forced earlier. I then ran the exploit (msfconsole>"exploit")
and after letting it run for a while, it successfully opened a Meterpreter session between my machine and the vicim. 
The payload was executed at "/wp-content/plugins/blMwULtTPK/tpvYlKfvcv.php". The Meterpreter session was spawned in the 
"/opt/bitnami/apps/wordpress/htdocs/wp-content/plugins/blMwULtTPK" directory and after finding nothing interesting in that 
directory, I moved to the user's home directory. There, I found 2 files: "key-2-of-3.txt" and "password.raw-md5". 

Now that I had the 2nd key, I inspected the hash file closer (note: in my Meterpreter session
I didn't have permission to cat the 2nd key, but I did have permission to cat the 
hash file. I will later describe how I gained user access to gain permission to view
the key file. Its contents were "robot:c3fcd3d76192e4007dfb496cca67e13b" which looks like
a username and hash pair. I moved over to haskiller.co.uk to run the given hash against theirprecomputed rainbow tables. The site successfully reversed the hash to abcdefghijklmnopqrstuvwxyz. I ran "meterpreter>shell"
to drop into a shell on the machine. The shell given was extremely limited in functionality, so
I spawned a Bash shell inside my current session with "python -c 'import pty; pty.spawn("/bin/
bash")' ". I then logged in as the "robot" user given the password I got from the MD5 hash. Now I
have permission to view the 2nd key (822c73956184f694993bede3eb39f959).

My final step was to gain root access to the system in order to find the 3rd and final key. I ran
"nmap localhost" in hopes of gaining root access through a running service but despite finding several
services (FTP, HTTP/S, and MySql), I had no luck. My next idea was to look through all the commands
I had access to as the "robot" user. Running "echo $PATH" revealed my shell's path was "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games".
I inspected each of those directories and found that nmap was installed on the system in "/usr/local/bin".
I then ran "nmap --interactive" to enter nmap's interactive command state in hopes of spawning a root shell from inside
nmap since nmap is located in "/usr/local/bin" and owned by root. I ran "nmap>!sh" which successfully spawned a root shell 
(confirmed by "whoami" and "groups" commands). I then moved to the root directory discovering the 3rd and final key 
(04787ddef27c3dee1ee161b21670b4e4). Now we have successfully found all 3 keys and rooted the system.










