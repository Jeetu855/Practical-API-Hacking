
Bookstore form THM

`http://10.10.5.29:5000/api/v2/resources/books?published=1993`

- first try to fuzz v2 with things like v0 v1 v3 v4 ...
- next try to fuzz endpoints, in this case 'books'
- then fuzz the parameters
- and finally the values of parameter

v1 and v2 both work 
status code and content length dosent change

fuzz books -w /usr/share/wordlists/dirb/small.txt
no other endpoints

fuzz query published
nothing there

change v2 to v1 and fuzz the parameter again

we get status code 500 for parameter show
this status code different from all other ones
![[ffuf1.png]]

we get an error saying filename is not defined so maybe parameter causing issue

![[fileNameError.png]]


change query to `show` and fuzz parameter value

since it said file not found, try different file extensions

![[ffuf2.png]]

gave 200 OK to value .bash_history

***when fuzzing a parameter value put the url in quotes***

can also use wfuzz

![[wfuzz2.png]]

```sh
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt --sc 200  'http://10.10.107.124:5000/api/v1/resources/books?show=FUZZ'
```
--sc 200:show only status code 200

![[wfuzz1.png]]


---

### Discovery via source code 

Signup and send generate traffic to send to burp suite

Go in developer tools --> Debugger --> Look through js files for sensitive information

Look for config files

found few endpoints
export const crapienv = {
    JAVA_MICRO_SERVICES: "identity/",
    PYTHON_MICRO_SERVICES: "workshop/",
    GO_MICRO_SERVICES: "community/",
};
