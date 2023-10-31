# Quest CTFroom

# Quest Web Challenge

## Initial Access

I launched the instance and  navigated to the target. 
On visiting the target i  came across a login page. Since we did not have any credentials to access the app, I decided to do more recon.  
I  decided to look for any hidden directories.  I started with directory busting using ffuf

![ ffuf Scan](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/ffuf_scan.png)


I had left a ffuf running in the background to identify and hidden directories or files, but it  never  got any thing interesting.  
Output of the ffuf scan.  

![ ffuf Scan Output](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/ffufScan_output.png)



Visiting the files required us  to login in. 


I had forgetten to check for common files e.g  readme.md which are usually found in web applications and went down a rabbit hole, which I never got anything useful.

But when visiting README.md  i got the README.md file. On  looking at its contents, i came across support email and how  the emails were generated for login page. We had the support email so all that we needed to login was the password for jameson@quest-dev.com user.  


![ ffuf Scan Output](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/readme_file.png)



With this  in mind, i decided to bruteforce the login page, since we  now had  one piece of information needed to access the app, the email.

login parameters

email=jameson%40quest-dev.com&password=1234

email: jameson@quest-dev.com

I used rockyou.txt wordlist to bruteforce the login page  using hydra and the command was as follows: 

![ ffuf Scan Output](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/hydra.png)


After sometime of bruteforcing, I was able to get the  password for  jameson@quest-dev.com.  

The password was basketball. 

I used it login,  to the application and we got the first flag. 


# Horizontal Privilege Escalation 


On login  I decided to intercept the login. I noticed the server was  issuing userid and AccessLevel cookies after logging to the application. 

cookies used: PHPSESSID=njhcahtpk1nus9fe1ju5l8v1br; AccessLevel=dXNlcg%3D%3D; userid=MTI%3D

accesslevel is base64 encoded which we could decode and understand how the application was utilising the accesslevel  cookie. 

Also the userid was encoded in base64 when decoding it, it was the userid.

![ ffuf Scan Output](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/user_id.png)

With the look at the cookies, we could be able to  modify the userid cookie to access other users details and documents. Also we could also modify the AccessLevel cookie and increase our privileges. 

On looking at the profile page, we were able to see  a message from bob. It  leaked bob userid. So i noted  down the userid.  Which was 71. If we could edit this value and be able to access another user details, the application could be vulnerable to IDOR.  
 
IDOR vulnerabilities occur  when a web application exposes a direct reference to an object, like a file or database  resource, which the end-user can directly control to obtain access to other similar objects
 
With this in mind, i decided to edit the value with bob userid and see if we could be able to access bob dashboard. 

Bob  uid was 71 as identified previously. So base64encoded it as below.  

![ ffuf Scan Output](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/encoded_reference.png)


I later url encoded the value using cyberchef. 

![ ffuf Scan Output](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/cyberchef_b64.png)


I then edited the values and replaced it with bob userid in the uid parameter and base64 encoded uid in  userid cookie value. 
I had intercepted the  request and sent to repeater for more testing. I sent the request  after  updating  it with our bob  uid and we were able to access bob  dashboard  after exploiting the IDOR vulnerability.  

![secon flag](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/secondFlag.png)


And we able to get  access to bob dashboard and got the second flag.

# Vertical Privilege Escalation. 

On bob dashboard, there was a files menu, I decided to visit the files menu. I came across a csv file. 
I download the csv file and it contained names and  default credentials and it was  indicating if there were changed or not.

![ csv file](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/quest_access_log_csv.png)

 
Because we  know how the  login email are created, i decided to come up with a list of emails, using the names found the csv file. The emails were as below: 

![ email list](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/emailList.png)


I started  by trying to test the password to  users who had not changed it yet.  

email: stephen@quest-dev.com  
pass: R6XnGs@!9P&34

Stephen had not changed his password.  We were able to succesfully login as Stephen user. 

And we got access  as the webmaster. On the dashboard we could  find the third flag. 

 ![ third flag](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/webMaster_dashboard.png)



# Injection 

There was an input to input a command, So intercepted the request and sent it to repeater and tested for command injection. 

![ injection ](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/ls.png)


I  used the ls command  as input  and it was succesfully executed and we  had the  command output in the response.
With this we confirmed  that the application  was vulnerable to command injection.  There was a hint on the  last challenge description part  that the flag contained "adios" in english. 

I  used the find command to  identify the flag. 

find / -name \*bye\* 2> /dev/null 

![find command](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/find_command.png)



I successfully got the  final flag.   


 ![Final Flag](https://github.com/Tenkph/EK4-Cybersecurity-Barcamp-CTF/blob/main/Quest/screenshots/last_flag.png)


It was a very fun web challenge and learned more about recon. 

Happy Hacking. 
