

### Capstone 1

Service running on port 80

Create 2 accounts
We get session token that is a JWT
Place order, and then track order

When we place an order, the request sent is like

GET /v1/order.php?email=asdf&coffee=2&q=2 HTTP/1.1

We only specify the coffee type and its roast level, so it looks like the email is being taken from the JWT since in the decoded JWT we can see that it only has our email in the payload

```sh
jwt_tool eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFzZGYifQ==.++u3Rlxwm0DtWRP6UVDEVCmIsLox7MDpnBQ4BTZ6ndc= -C -d /usr/share/wordlists/rockyou.txt
```

-C : crack the key
-d : use a dictionary attack

coffee is the CORRECT key!

So the key used for signing is coffee

Using this we can tamper with the payload and inject our claims

We can forge token for other user that we created and place an order on their behalf
We can place an order on behalf of other user if we just change the username in the Request made, so the backend is not checking if token signature even exists or not
We can just remove the signature part and place an order and the request goes through

So BFLA

Fuzz for other endpoints as well

```sh
ffuf -c -w /usr/share/wordlists/dirb/big.txt:FUZZ -u http://localhost/v1/FUZZ -e .php
```

- * FUZZ: admin
* * FUZZ: coffee.php
- * FUZZ: track.php

We found a hidden endpoint, that is admin

/v1/admin  ---> Status 403 forbidden

FUZZ again  for /admin

```sh
ffuf -c -w /usr/share/wordlists/dirb/big.txt:FUZZ -u http://localhost/v1/admin/FUZZ -e .php
```

- * FUZZ: orders.php

![[capstone1_1.png]]

And it reveals all the orders placed, so broken access control leading to excessive data exposure

grab one of the revealed order ID and put it in track order

We are able to check orders placed by other users as well so we have BOLA

