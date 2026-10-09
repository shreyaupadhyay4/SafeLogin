# SafeLogin

A login API that doesn't just roll over when someone keeps guessing passwords.

I'm learning backend development and I'm really into cybersecurity, so I wanted a first project that mixed both. Every login tutorial I found stopped at "check the password, hand back a token." But I kept wondering what happens if someone just sits there and tries 10,000 passwords. So I built something that answers that.

It's small on purpose. I wanted to actually finish it and understand every line, instead of getting lost in a huge project.

## What it does

You can sign up, log in, and get a token. That part is normal.

The part I care about is what happens when things go wrong. If someone enters the wrong password 5 times within 5 minutes, that account gets locked. Even typing the right password afterwards won't work until the time window passes. Every failed attempt is saved with the email, the IP, and the time, and you can see them all at `/failed-logins`.

There's also a `/me` route that only works if you send a valid token, which is how I tested that logging in really gives you access to something.

## How a login goes

1. You send an email and password.
2. The server checks whether that account already has 5 recent failures. If so, it says no right away.
3. Otherwise it looks up the user and checks the password against the stored hash.
4. Either way, it writes the attempt to the database.
5. Wrong password gets a 401. Right password gets a token that expires in an hour.

## Built with

Python, FastAPI, SQLite with SQLAlchemy, passlib for bcrypt hashing, and python-jose for the tokens.

## Running it

```bash
git clone [your-repo-url]
cd safelogin
python3 -m venv venv
source venv/bin/activate      # on Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload
```

Open `http://localhost:8000/docs` and you'll get a page where you can try every route without writing any code. I use it constantly.

## Trying to break it

I wrote a little script, `attack_test.py`, that attacks my own server. It signs up a test user, tries 7 wrong passwords, then tries the correct one. This is what I get:

```
Attempt 1: 401
Attempt 2: 401
Attempt 3: 401
Attempt 4: 401
Attempt 5: 401
Attempt 6: 403
Attempt 7: 403
Correct password: 403
```

[Put a screenshot of your own output here]

The first five get rejected, the sixth gets locked out, and the correct password at the end still fails. Seeing that actually happen was the best moment of the whole project. Please only run it against your own local server.

## Choices I made on purpose

Passwords are hashed, so a leaked database doesn't leak real passwords. The error for a wrong email and a wrong password is identical, so nobody can use the login form to find out which emails are registered. Tokens expire, so a stolen one stops working. And everything gets logged, because you can't investigate an attack you didn't record.

