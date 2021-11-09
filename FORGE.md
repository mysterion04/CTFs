# FORGE #
## Info ##
- accessible on HTB
- Difficulty: `medium`
## Enumeration ##
### namp ###
> sudo nmap -sS -sV -sC -oN ~/Desktop/nmapHTB 10.10.11.111

-    ![namp]()

- Discovery
    - found open ports 80, 22, 21
- On opening the site there was a file upload functionality
- and ofcourse i tried all possible ways of shell upload and they didn't work

    ![site]()
### FFUF ###
- Couldn't find any files anywhere
- I tried subdomain brute forcing and found `admin`

    ![ffuf results]()
- On openening the site it wasn't accessible

    ![inaccessible]()

## FLAG ##

- I discovered from forum that it has a ssrf vuln
- in the file upload by url tried going on the admin url but was stopped by the filter

    ![filter]()
- to bypass the file i tried case brute `ADMIN.FORGE.HTB`
- got the file upload url
- tried to go to url and inspect the page
- found a `announcement` folder

    ![anncounemt]()
- repeated the same with this folder and file upload page `ADMIN.FORGE.HTB/annoucements`
- and found the ftp credentials

    ![ftp creds]()
- and again following the same url upload method i got in the ftp server and found some files
- traversed to the ssh_id_rsa and found the key ;)

    ![ssh]()
- Logged in the ssh and found a file `user.txt`
    > cat user.txt
- flag
    > 3e047ef970091d009jh4