## ***Exercises*** 

# **Finding Files with Linux**

1. Create a file named ``my-file.txt`` with the touch command. Then execute the ``locate my-file.txt`` command. Do you find the file? 
    > Your response : NO
2. Run the command sudo ``updatedb``. And run the locate my-file.txt command again. Do you find your file ?
    > Your response : YES
3. With the command ``which``, find the executable file nc and indicate the path
    > Path :/bin/nc
4. With the command ``which``, find the executable file becode. What is the flag ?
    > Flag : BC{WH1CH_FL4G_EXECUTLE_FILE}
5. Search with ``find ``command for a file that contains the name "Edgar Allan Poe". What is the flag ?
    > Flag : BC{3d54r_4ll4n_P03_FL45}
6. Using the ``find`` command, find the file password.txt and specify the flag.
    > Flag : BC{PASSWORD_FILE}
7. With the command ``find``, find a file that starts with ``becode-`` and ends with ``.sh``.
    > Flag : BC(YOU_C4N_FIND_ME_WITH_WICH_IF_AM_EXEC) 
8. Using the ``find`` command to identify any file (not directory) modified in the last day, NOT owned by the root
user and execute ls -l on them. **Chaining/piping commands is NOT allowed!**
    > Your command :find / -type f -mtime 0 ! -user root -exec ls -l {} + 
9. With the find command, find all the files that have an authorization of ``0777``.
    > Your command: find / -type f -perm 0777
10. With the find command, find all the files in the folder ``/home/student/findme/`` that have an authorization of ``0777`` and change the rights of these files to ``0755``
    > Your command: find /home/student/findme/ -type f -perm 0777 -exec chmod 0755 {} +

# **Text manipulation with Linux**

1. Search all sequences containing "Loxondota" in ``/home/student/lorem.txt``
    > Flag : BC{GREP_ME_LOREM_FL4G}
2. Copy the file /etc/passwd to your home directory. Display the line starting with ``student`` name.
    > Your commands : cp /etc/passwd ~/passwd && grep '^student' ~/passwd
3. Display the lines in the passwd file starting with login names of 3 or 4 characters.
    > Your commands : grep -E '^[a-zA-Z0-9]{3,4}:' ~/passwd
4. In the file ``/home/student/sample.txt`` how many different values are there in the first column? in the second?
    > Your response : first column-8, second column-8 
    > Your command : first column: cut -d ' ' -f 1 /home/student/sample.txt | sort | uniq | wc -l
                     second column: cut -d ' ' -f 2 /home/student/sample.txt | sort | uniq | wc -l
5. In the file ``/home/student/sample.txt`` sort the values in the second column by frequency of occurrence. (uniq -c can be useful)
    > Your response :1 ind4,20
					 1 ind4,11
					 1 ind3,20
					 1 ind3,10
					 1 ind2,30
					 1 ind2,20
					 1 ind1,20
      		 1 ind1,10 
    > Your command : cut -d ' ' -f 2 /home/student/sample.txt | sort | uniq -c | sort -nr
6. In the file ``/home/student/iris.data`` Change the column separator (comma) to tab (make sure that the changes are applied to the file)
    > Your command : sed -i 's/,/\t/g' /home/student/iris.data
7. In the file ``/home/student/iris.data``, extract from this file the column 3 (petal length in cm) (use cut )
    > Your command : cut -d ',' -f 3 /home/student/iris.data
8. In the file ``/home/student/iris.data``, count the number of flower species (cut and uniq)
    > Your response : 148 
    > Your command : cut -d ',' -f 5 /home/student/iris.data | sort | uniq | wc -l
9. In the file ``/home/student/iris.data``, sort by increasing petal length (see sort options)
    > Your command : sort -t ',' -k 3n /home/student/iris.data
10. In the file ``/home/student/iris.data``, show only lines with petal length greater than the average size
    > Your response :  
    > Your command : cut -d ',' -f 3 /home/student/iris.data | awk '{sum += $1; count++} END {avg = sum / count} {if ($1 > avg) print}' | grep .
11. Using ``/etc/passwd``, extract the user and home directory fields for all users on your student
machine for which the shell is set to ``/bin/false``. 
    > Your response : systemd-timesync /run/systemd
				systemd-network /run/systemd/netif
				systemd-resolve /run/systemd/resolve
				systemd-bus-proxy /run/systemd
				syslog /home/syslog
				_apt /nonexistent
				lxd /var/lib/lxd/
				mysql /nonexistent
				messagebus /var/run/dbus
				uuidd /run/uuidd
				dnsmasq /var/lib/misc
				postfix /var/spool/postfix
				dovecot /usr/lib/dovecot
				dovenull /nonexistent
				colord /var/lib/colord
    > Your command :awk -F ':' '/\/bin\/false$/ {print $1, $6}' /etc/passwd

# **Linux : Piping and Redirection**

Read the following [article](https://ryanstutorials.net/linuxtutorial/piping.php) and answer the questions below. Some questions will require additional research.

1. Write the message "hello everyone" in a file called "test" by redirecting the output of the echo command.
    > Your command : echo "hello everyone" > test
2. Write the message "goodbye" in the same file "test" by redirecting the output of the echo command and without overwriting the content of "test" and check with the cat command
    > Your command : echo "goodbye"  >> test
3. Make the ``ls -la`` command redirect to the ``foo`` file
    > Your command : ls-la > foo
4. Execute ``find /etc -name *conf*`` command  and redirect errors (only errors) to a file named err.txt 
    > Your command : find /etc -name *conf* 2> err.txt
5. Repeat the previous exercise, this time redirecting the errors to the linux nothingness.
    > Your command : find /etc -name *conf* > /dev/null 2>&1
6. Now redirect the standard output and the error output of the ``find /etc -name *conf*`` command to two different files (std.out and std.err)
    > Your command : find /etc -name *conf* > std.out 2> std.err
7. What does the mkfifo command do?
    > The mkfifo command in Kali Linux is used to create a special type of file known as a "named pipe" or a FIFO (First-In-First-Out). 
8. Create a pipe named "MyNammedPipe". Then execute the pwd command which will transmit the data in this pipe. Then use the cat command to read the contents of your "MyNammedPipe" pipe.
    > Your commands : mkfifo MyNamedPipe, pdw > MyNamedPipe, cat MyNamedPipe
9. With cat command, add number the lines in the file /etc/passwd with the command ``nl``
    > Your commands : cat -n /etc/passwd
10. Using the previous nl command, the head and tail commands, display the lines of /etc/passwd between line 7 and line 12
    > Your commands : nl /etc/passwd | head -n 12 | tail -n 6
11. On your student machine what is the value of the FLAG environment variable ?
    > Flag: FLAG=BC{EXPORT_B4SH_FLAG}

# **Protocols and servers**

1.  On your kali (or other) , install ``ngnix`` to have an http server on port 8080. Replace the default page of ngnix by an html page displaying a hello world.
    > No answer required

2. What other well-known service could be used instead of nginx? 
    > Your answer : Apache HTTP Server, Microsoft Internet Information Services (IIS)

3. On your student machine, create a temporary http server with python, on port ``5000``. Then on your kali machine, open a browser and go to the address ``10.12.181.X:``.
    > Your command : python3 -m http.server 5000

4. Let's imagine that a hacker owns the domain name ``g00gle.com``, which tool would allow him to obtain an ssl certificate (https) very easily?
    > Your answer : He can obtain an ssl certificate by using https://www.sslforfree.com/

5. On a linux machine, what tool could you use to have a self-signed SSL certificate on your local machine (localhost) ? 
    > Your answer : we can install openSSL by using command: "sudo apt-get install openssl", then navigate directory where you want to generate your ssl, then generate private key using: "openssl genpkey -algorithm RSA -out private.key"

6. On your student machine, install the ftp service and connect from your kali machine.
    > No answer required

7. What is the default port for ftp? 
    > Your answer :

8. Is the ftp protocol secured?
    > Your answer :

9. On your student machine, install the telnet service and connect from your kali machine.
    > No answer required

10. What is the default port for telnet? 
    > Your answer :

11. Is the telnet protocol secured?
    > Your anbswer :
    
12. Create a share file with samba between your Kali machine and your student machine.
    > No answer required

# **Downloading files**

1. On your Kali machine, create a file named malware.php.
    ````
    echo "This is a malware file" > malware.php
    ````
    Then, in the same directory, ccreate a temporary server with python on port 5000.
    ````
    python3 -m http.server 5000
    ````
2. On your Student machine, download the malware.txt file with the wget command.
    > Your command :

3. On your Student machine, download the malware.txt file with the cURL command.
    > Your command :

4. On the student machine, create a file named password.txt and transfer it to your student machine with netcat
    > Your commands

5. On the student machine,  transfer ``/etc/passwd`` file to your kali machine with tftp
    > your commands:
