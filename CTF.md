> # CONTENTS #

- ## [Info](#info) ##

- ## [Enumeration](#enumeration) ##

- ## [In Wonderland](#in) ##

- ## [Flag](#flag) ##

- - -
> ## Info ##

- ### Wonderland ###

 ![](https://i.ibb.co/yfg9285/Screenshot-2021-11-02-at-2-00-31-PM.png)

- ### The machine is availabe and deployed on [Tryhackme](https://tryhackme.com/room/wonderland). ###

- ### Difficutly: `Medium` ###

- - -

> ## Enumneration ##

- ### Now, we don't know anything about the machine except it's `IP address` ###

- ### We all know the best tool for network enumeration is `nmap`, so as always

![nmap results](https://i.ibb.co/yp5zCBL/Screenshot-2021-11-02-at-2-38-21-PM.png)

- ### We have gathered: ###
    - Ports 22 and 80 are open
    - It's a linux-Ubuntu machine
    - - -
- ### Let's try finding if there are some directories left out open for us, I'm gonna use `ffuf`, cause it's just the fastest one out there

- ### I'm just gonna use the web-content discovery worldlist from the seclists ###

![ffuf resulsts](https://i.ibb.co/3s7qnpq/Screenshot-2021-11-02-at-3-14-03-PM.png)

- ### We have gathered: ###
    - `img, r, poem` named directories
    - - -
- ### Now, we know port 80 is a http webserver ###
    - When we visit the webserver it fetches the following site
    ![](https://i.ibb.co/sJnGSqD/Screenshot-2021-11-02-at-2-51-08-PM.png)
    - We don't see much here on the website, except for a image
    - - -
- ### But, previously from our ffuf results we know that img is folder, so let's go to the folder `http://10.10.56.104/img/` ###
    ![](https://i.ibb.co/y5BZjRt/Screenshot-2021-11-02-at-3-17-58-PM.png)
    - On our first web page we encountered a photo of rabbit and the title said `"Fall down the rabbit hole and enter wonderland."` that might be a hint.
    - So I'm gonna download the image `white_rabbit_1.jpg`
    - Downloaded the image and opened it, nothing intresting there, but... it might be a steganography
    - Let's try looking for other files in this image file
    - Here we will be using a tool called `steghide`, which is popularly used for looking embedded files in a image file
    ![](https://i.ibb.co/JytXLMq/Screenshot-2021-11-02-at-3-32-12-PM.png)
    - I asked for a paraphrase since I didn't have any I just hit ENTER and hopefully it was not set got fetched a file called `hint.txt`
    - Now let's extract the file we found and see what hint we have in here
    ![](https://i.ibb.co/4sBw4bP/Screenshot-2021-11-02-at-3-34-57-PM.png)
    - The hint says
        > `follow the r a b b i t`
    - Now, after stracting my head for a while I got went back to [Enumeration](#enumeration)
    - We had gathered a director namely `r` from the ffuf scan
    - So here in the hint the `r` is directory, and now following the directory `r/a/b/b/i/t`
    - We follow it- <http://10.10.56.104/r/a/b/b/i/t/>
    - Here we get yet another web page
    ![](https://i.ibb.co/pdX9Mc3/Screenshot-2021-11-02-at-3-46-35-PM.png)
    - Just like the previous web page there's nothing to do here, but... when we view the view the page source
    ![](https://i.ibb.co/RbnHPGs/Screenshot-2021-11-02-at-3-58-12-PM.png)
    - In the last paragraph tag we see the `display: none` parameter
    - Seems intresting that they don't want us to see it
    - Alice sounds like a name and if you remember from [Enumeration -> nmap scan results](#enumeration) there was a SSH port open on the server; so these might be the credentials to the SSH login

        > alice

        > HowDothTheLittleCrocodileImproveHisShiningTail
    - We are in the machine
    - - -
- - -
> ## In Wonderland ##
![](https://i.ibb.co/gPNcVhs/Screenshot-2021-11-02-at-4-12-23-PM.png)
- Let's wonder around here n see if we can find something intresting
- We found two files `root.txt and walrus_and_the_carpenter.py`
- When I tried `cat walrus_and_the_carpenter.py`, I found some random poem written in it
- And when executed the python file the same poem was the output
- But, since the moment I saw the `root.txt` I knew there was something in there
- But when I tried to `cat root.txt`
- I got
    > cat: root.txt: Permission denied
- So I tried to list of user in the machine and hopefuuly I was able to and
- I found that there was anther user called rabbit
- Here we can do some previlage escalation and get the `root.txt` but first let's get the `user.txt` before that
![](https://i.ibb.co/0F3F5M1/Screenshot-2021-11-02-at-5-35-16-PM.png)
- So now as asked in the flag on the THM page there is user.txt in the machine which holds the flag
- Then we move around in the machine seaching for `user.txt`using the command
    > find user.txt
- And in the hint we were told "everything is upside down"
- We have root flag in the user folder so maybe the user flag is in the root folder
- Finally I found it in the root folder, so now
    > cat /root/user.txt

  ![](https://i.ibb.co/sPDPrnZ/Screenshot-2021-11-02-at-5-40-41-PM.png)
- - -

> ## Flag in user.txt ##
    thm{"Curiouser and curiouser!"}
