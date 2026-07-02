Upon entering the level I see that things have changed. I can no longer edit unless I have an admin account and I tried using idor but nothing took since it requires and admin account. It's time to try exploiting the admin account now.

## SQLI (SQL Injection) into admin
Because I don't know what the admin accounts are called we are going to assume there is no hashing done on the password and hope we can get past that by using a Union to force a new admin account creation.

Username: `' UNION SELECT 'password123' AS password FROM admins WHERE '1'='1`
Password: `password123`

we logged in successfully!

### SQLI - why it works
so let's go over the why this works as a refresher (for myself, but I guess for anyone else that ever reads this). Most text fields in input forms for sites are expecting data, but if not properly sanitized hackers can exploit the fact that the input can be treated as executable code instead of just data. Here's how that happens.

#### Background Knowledge:
Say the backend login logic runs 
`SELECT * FROM users WHERE username = 'admin' AND password = 'password123';`
when taking your input from the form in order to query the database for the requested account verification. The database will execute this command, look for a match, and return the user's row if it exists.

The vulnerability exists when the input is taken and concatenated into the command directly without being isolated and sanitized first.

#### Constructing the payload
payload used : `' UNION SELECT 'password123' AS password FROM admins WHERE '1'='1`

resulting database query:
`SELECT password FROM admins WHERE username = '' UNION SELECT 'password123' AS password FROM admins WHERE '1'='1' AND password = '...';`

##### Breaking Down the Syntax: Why it Works

Dissecting the injected string piece by piece to see how the database parsed it:

- **`'` (The Single Quote):** This is the most critical character. It tells the database, _"Close the text string for the username right here."_ Everything after this quote is now treated as SQL command syntax, not data. (essentially it escapes the data part; alternatively if you want to set a username and not just a password for future backdoor entrance you can fill in the username then use the quote)
    
- **`UNION`:** This is an SQL operator used to combine the results of two separate `SELECT` queries into a single result sheet.
    
- **`SELECT 'password123'`:** Because the first query (`WHERE username = ''`) returns zero rows (since no user has a blank username), the database moves to the second query. It generates a fake, virtual row out of thin air containing the text string `'password123'`.
    
- **`AS password`:** This is called an **alias**. The backend application code is likely programmed to look for a column specifically named `password`. By using `AS password`, you forced the database to name your fake column exactly what the application was looking for.
    
- `FROM admins`: In many SQL databases (like MySQL), you can run a query like `SELECT 'abc';` without naming a table, and the database will happily spit back a virtual row. However, some databases (like PostgreSQL, SQLite under certain configurations, or Oracle) or specific application setups require every `SELECT` statement to explicitly declare where the data is coming from using a `FROM` clause. By adding `FROM admins`, we satisfied the database engine's requirement to target an existing table, preventing a fatal database syntax error.
	
- `WHERE '1'='1'`: This is a classic SQL injection logic trick. `1=1` is a mathematical absolute—it is **always true**.
  Why did we need it here instead of a comment character (like `--`; *to be explained later*)?
	- If the application code takes the original query and appends text to the end of it later in the script, a comment character might break the developer's code and cause a server crash (500 Error).
	- By using `WHERE '1'='1'`, we effectively hijacked the logic. Even if the application appends extra rules to your query afterward, the database evaluates your `UNION` side of the query as "True" and successfully outputs your fake password row.
	
- ***Note:*** ****`--` (The Comment):** This tells the database, _"Ignore everything after this line."_ It effectively deletes the rest of the developer's original query (the trailing quote, the `AND password = '...'` check, etc.). Sometimes this can ensure the database doesn't throw a syntax error. Other times it doesn't serve a purpose at all.

##### The Data-Matching Requirement

There is one more hidden rule of `UNION` statements that this payload perfectly solved: **Column Matching**.

A `UNION` operator behaves like gluing two spreadsheets together vertically. For the glue to work, both spreadsheets **must have the exact same number of columns**.

```
[ Original Query Result ]  <- Has 1 column (password)
          |
       (UNION)
          |
[ Your Injected Result ]  <- Must have 1 column ('password123')
```

Because the original query was only looking for one thing (`SELECT password`), your injected query could only look for one thing (`SELECT 'password123'`). If the original query had been looking for `id, username, password` (3 columns), your payload would have failed unless you supplied three items (e.g., `' UNION SELECT 1, 'admin', 'password123'`).

The fact that `' UNION SELECT 'password123'` worked proves that the developer's original backend query is only selecting a **single column** from the database.

##### How the Backend Processed the Payload

1. The database looked for a user with a blank username (`''`) and found nothing.
    
2. The database moved to your `UNION` statement. It went to the `admins` table, saw that `'1'='1'` is true, and generated a row where the column named `password` contained the literal text `password123`.
    
3. The database handed this single-column result back to the web application.
    
4. The web application compared the database result (`password123`) to what you typed in the password input box (`password123`). They matched, and the door unlocked.

## Flag 0:
1) upon logging in immediately we are greeted with a private page!
2) upon clicking in we get a flag: `^FLAG^850f8ccc0c1ef2b62964a0c2e1e167fa167f9ba16b4b4f2b790d4a235a044e43$FLAG$`

I tried the button event listener alert like the previous level's button exploit and that failed. what else can I try?

No IDOR I can find

## Deductions:
So far I have 2 remaining deductions I can think of:
1) **The Admin Login Page:** We successfully exploited a SQL injection here (`' UNION SELECT 'password123' AS password FROM admins WHERE '1'='1`). This let us bypass authentication, but did we actually look around the _database table_ itself?
    
2) **The Edit / POST Logic:** We found that a user must be logged in to make a `POST` request, and that HTML characters are encoded in the page body (`<script>` doesn't display because it is encoded as `&lt`).
## Exploring more SQLI (deduction 1)
What else can I find? let's try another SQLI attack, exploiting the address this time. the address is a page query, so lets try using inline math this time as well as the `'` trick from last level. Neither work so we know that is secure.

Let's go back to the login page and see if we can get more information, possibly a look at all the tables that exist! It's time for reconnaissance!

### Data Exfiltration
I'm going to try inputting other sql commands into the login fields to see if it will print anymore information for me. When I log in, normally there will be error messages like "Unknown User" or "Invalid Password". Let's see if there's any way we can mess with the user error to see if we can get it to display information, as this will tell us a lot of information on how the application handles, retrieves, and displays information from the database and if we can exploit that.

#### Creating a new account
First we need to create a new account. There's no way to do this normally through the app, so I'm gonna leverage what I did earlier with creating my own backdoor admin account but this time I will set a username as well. 

Therefore, if I set my username as `AliceBob` my new payload should look like this:
`' UNION SELECT 'AliceBob', 'wrongpassword' AS password FROM admins WHERE '1'='1`

upon inputting that into the username, we got an Internal Server Error. This means the number of columns in the original developer's query is only 1 not 2, so username reflection isn't going to work. Maybe we can try password reflection, however the only error I've gotten so far has been "Invalid Password". I got "Invalid Password" by using the original payload and typing a different password from what we set it to.

#### Blind SQLI using Binary Logic Validation
Let's come back to this later since I think I'm beginning to lose the plot, get confused, and this is going to be tedious so if I decide to revisit this later it'll be the only thing I focus on.

## Exploring Editing (deduction 2)
Let's look for hidden values, dropdowns, and input fields!
- I see nothing out of the ordinary or nothing that looks exploitable. only title and body fields in the body, no hidden parameters immediately accessible through the ui
However, we know there needs to be a way for the backend of the CMS to track public and private pages because we saw that some pages were restricted through our IDOR crawl earlier and also by logging in and discovering a hidden page. Why don't we try editing a page and targeting that parameter manually through curl rather than visually through the app's ui?
#### Parameter Pollution/Mass Assignment Attack
Let's see if we can exploit anything using a manual POST request to the server using the curl command:
```
# Test 1: Guessing 'public'
curl -X POST -d "title=Test&body=MyBody&public=0" http://localhost/page/edit/YOUR_PAGE_ID --cookie "session=YOUR_ADMIN_COOKIE"

# Test 2: Guessing 'published'
curl -X POST -d "title=Test&body=MyBody&published=0" http://localhost/page/edit/YOUR_PAGE_ID --cookie "session=YOUR_ADMIN_COOKIE"
```
1) for the first curl command I got the next flag! **Note:** the url must be from the edit page for it to work
## Flag 1: 
Using Parameter Pollution/Mass Assignment I got the next flag which was:
`^FLAG^c2e9163fb2e95721081976ed18f99ff0eeaf7ea0da6150b3521e74d8b9779e5c$FLAG$`

## Quick Recap:
So far, we've found 3 vulnerabilities we have been able to exploit:
- **Vulnerability A:** SQL Injection on the **Login Page** (Functional and highly exploitable).
- **Vulnerability B:** Mass Assignment on the **Edit Page** (Just exploited to get Flag 2).
- **Vulnerability C:** Cross-Site Scripting (XSS) — We noticed the application encodes `<` to `&lt;` on standard inputs, but we haven't checked if a SQL Injection could be used to smuggle a raw script tag out of the database onto a page that doesn't sanitize _database outputs_.

**Vulnerability A** allowed me to access flag 0 as I was able to use sqli to log into the CMS and view the private page with the first flag. **Vulnerability B** allowed me to find flag 1. **Vulnerability C** was discovered while exploring but we haven't tried anything with it just yet.