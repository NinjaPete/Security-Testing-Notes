https://github.com/darkoperator/dnsrecon
`dnsrecon -d megacorpone.com -t std` - `-d` specifies domain `-t` specifies the type of enumeration to perform
`dnsrecon -d megacorpone.com -D ~/list.txt -t brt` - `-D` specifies a file list. The enumeration type is set to bruteforce so this will bruteforce subdomains off of that list.