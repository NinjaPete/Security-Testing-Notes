https://github.com/NinjaPete/tools/blob/main/smtp-user-enum.py
Basic python script to check if a username is valid against the SMTP service:

`python3 smtp.py username $target-ip`
e.g.

```
kali@kali:~/Desktop$ python3 smtp.py root 192.168.50.8
b'220 mail ESMTP Postfix (Ubuntu)\r\n'
b'252 2.0.0 root\r\n'


kali@kali:~/Desktop$ python3 smtp.py johndoe 192.168.50.8
b'220 mail ESMTP Postfix (Ubuntu)\r\n'
b'550 5.1.1 <johndoe>: Recipient address rejected: User unknown in local recipient table\r\n'
```

Loop it with:

`while read -r username; do python3 smtp.py $username $target-ip; done < userlist.txt`