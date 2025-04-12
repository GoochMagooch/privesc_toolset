# copy /etc/shadow hashes from first : to the second :
# copy hash to text file and crack with john
sudo john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# for ssh keys use ssh2john to convert key to crackable hash
sudo ssh2john id_rsa > hash.txt

# crack hash file
sudo john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
