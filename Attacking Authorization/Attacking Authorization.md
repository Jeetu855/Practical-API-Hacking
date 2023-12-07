
Authentication : Who we are, our ID

Authorization : What we are allowed to do

BOLA : Broken Object Level Authorization
BFLA : Broken Function Level Authorization

If we are given an ID on sign in , if we can we other people's profile by changing the ID then it is BOLA, Access to objects refrenced by their ID

If we can carry out actions like making a post on behalf of other users then this is BFLA
Access to functionality that we shouldnt have


---

### BOLA Lab

Signup on webpage at port 8888 (\a@b.com,abcd,Qwerty#1234)
Check on port 8025, we get VIN and pincode, add the vehicle using that
Generate traffic 

Each vehicle has a unique ID

Looking at traffic in burp, in /community endpoint, the response sent back reveals
vehicle ID of other users as well

![[BOLA LAb.png]]

Try to access details of other vehicles
We can do that but in tehcnicality that should not be allowed and we should be retuned a frbidden acces but we are able to view it so we have BOLA

**This is also excessive data exposure**

***Always look at object ID's***

---

### BFLA Lab

Postman --> Import --> vAPI Postman both json files

Look through documnetation

We  are able to we other users sensitive infomation when we list all usernames, this should be administative privilege but since we can view them, we have BFLA


---

### Lab Challenge

BOLA : Send a ticket to the mechanic and check traffic in Burp Suite

![[Chall1.png]]

This is the data we send with HTTP POST method

![[Chall.png]]

This is the response we get and it contains report ID, try changing it to

/workshop/api/mechanic/mechanic_report?report_id=4

Here we can change the id paramter to view other peoples ticket and the access is not blocked so we have BOLA

**We may also have SSRF**


BFLA Challenge

Go to profile --> Click on the menu icon and we get an upload video feature

![[chall3.png]]

After uploading, we get an id for the uploaded video

Posrman --> Signup --> Login --> Upload a video for new user

![[chall4.png]]

We can fetch the video info 

\http://localhost:8888/identity/api/v2/user/videos/{{video_id}}

Trying to delete a video we get error saying it is an admin function

![[chall5.png]]

We also have a API request that allows us to delte video as admin

\http://localhost:8888/identity/api/v2/admin/videos/{{video_id}}

We get another error saying

![[chall6.png]]

Invalid token

It dosent say invalid access but rather Invalid token

Change the original allowed request at 

\http://localhost:8888/identity/api/v2/user/videos/{{video_id}}

Change user to admin since we know that user endpoint dosent ask for token

\http://localhost:8888/identity/api/v2/admin/videos/{{video_id}}

and we are able to delete other users video

![[chall7.png]]

Now try to check the video, and we get


![[chall8.png]]

404 video not found

We can also delete our user video

So we have BFLA