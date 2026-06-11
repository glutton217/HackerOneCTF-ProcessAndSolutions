Upon entering the level I see that things have changed. I can no longer edit unless I have an admin account and I tried using idor but nothing took since it requires and admin account. It's time to try exploiting the admin account now.

Because I don't know what the admin accounts are called we are going to assume there is no hashing done on the password and hope we can get past that by using a Union to force a new admin account creation.

Username: `' UNION SELECT 'password123' AS password FROM admins WHERE '1'='1`
Password: `password123`

we logged in successfully!

## Flag 0:
1) upon logging in immediately we are greeted with a private page!
2) upon clicking in we get a flag: `^FLAG^850f8ccc0c1ef2b62964a0c2e1e167fa167f9ba16b4b4f2b790d4a235a044e43$FLAG$`

I tried the button event listener alert like the previous level's button exploit and that failed. what else can I try?

No IDOR I can find
