
In lab
If want to clear the database, just go to /clear endpoint

Create a new user

![[Pasted image 20231213203936.png]]

We can see all the users and their roles which is the privilege parameter

We have to send a POST request to /register to create a user

***Basically we have to find a hidden parameter that sets the user privileges and add that parameter to the request and send it with appropriate value***

If the application is open source, we can just look at the source code to find what the paramters are by checking database schema

Here we know that the parameter that sets user roles is called 'privileges'


![[Pasted image 20231213204425.png]]

This is a normal request on registration

![[Pasted image 20231213204524.png]]

In this request we added the parameter and assigned it the value admin
privileges =admin

and sent the request

Now if we check all the present users

![[Pasted image 20231213204621.png]]

We can see that the newly created user is an admin

***In our index.js, the vulnerable line of code is ...req.body which uses the spread operator and passes all the paramters and their values to the schema, so if the attribute exists in schema and we know about it and the code uses spread operator, we can send a value to that paramter and it will pass it to the database***

More secure way of writing it would be req.body.username, req.body.password to only pass the values of username and password to the database


---

### crAPI challenge

service running on port 8888

```sh
ffuf -c -w /usr/share/wordlists/seclists/SecLists-master/Fuzzing/http-request-methods.txt:FUZZ -u http://localhost:8888/shop/products -X FUZZ
```

Fuzzing for diffrent allowed HTTP methods

Change the method to POST and send the request

Instead of getting 403 forbidden, we get 400 Bad Request with the response

![[Pasted image 20231213210926.png]]

So we need to send these field values along with the POST request

Remember to send the type of data as JSON

Burp --> Extension(right click)  --> Content type converter --> Convert to JSON


![[Pasted image 20231213211545.png]]


{"name":"cheeseckae",

"price":"-500",

"image_url":"\http://localhost:8888/images/seat.svg"

}

Send the request with these values and we get 200 OK

***We make its value as -500 so when we purchase it, instead of debiting money from our account, it is going to do -(-500) and then add it to our current balance.***


