---
title: Kolska Leaks - ECSC2022
author: k1k9
categories: [ECSC2022, web]
tags: [ctf, ecsc2022]
---
Punkty: 150
Rozwiązań: 30

Finally. The notorious hacking group got what had been coming to them. Somebody hacked them and released a bunch of their personal files in a info dump called "Kolska Leaks".
Check if they have published anything interesting besides the exfiltrated files:
https://kolska-leaks.ecsc22.hack.cert.pl/

## Rozeznanie
Sama strona prezentuje nam w następujący sposób:
![wyglad_strony](/assets/posts/kolska-leaks/page.png)

Mamy kilka obrazków, gdy otworzymy jakiś obrazek mamy tam taki url ```https://kolska-leaks.ecsc22.hack.cert.pl/download?filename=files/cloud-migration.png```, pierwsze co przyszło mi to Local File Inclusion. Po spróbowaniu ```https://kolska-leaks.ecsc22.hack.cert.pl/download?filename=/etc/passwd``` i o ile udało się potwierdzić LFI to sam plik nie jest interesujący.

## Local File Inclusion
Skoro wiemy, że strona jest podatna na atak LFI to pozostaje dowiedzieć się czego szukać. Po kilku próbach zgadnięcia różnych form flag flag.txt etc. wróciłem do czytania strony.

1. Po wejściu w zakładkę ```Get the flag``` otrzymujemy komunikat ```You're not an admin :(```.
2. Na stronie nie ma żadnego panelu logowania
3. Po wejściu w tę zakładkę jedyne co mogłoby być sprawdzane to ```cookie```.

Na samym dole strony głównej mamy taką informację: ```except Exception as e.``` co może świadczyć o tym że strona postawiona jest na jakimś frameworku pythonowym. Sprawdźmy czy mam rację szybki url ```https://kolska-leaks.ecsc22.hack.cert.pl/download?filename=requirements.txt``` i widzę, że mamy apkę napisaną w flask:
```txt
Flask==2.0.1
gunicorn==20.1.0
```

## Flask
A więc przejdźmy do analizy kodu aplikacji:
```python
from flask import Flask, render_template, request, abort, send_file, session
from flask.sessions import SecureCookieSessionInterface
from pathlib import Path
import json
import os

app = Flask(__name__)
app.config['SECRET_KEY'] = "p5VAmUfaP71Zpy1g"
session_serializer = SecureCookieSessionInterface().get_signing_serializer(app)

ROOT = Path(__file__).parent
FILES = json.loads((ROOT / "files.json").read_text())

@app.route("/")
def get_leaks():
    if "is_admin" not in session:
        session["is_admin"] = 0

    return render_template("leaks.html", files=FILES)

@app.route("/flag")
def get_flag():
    if session.get("is_admin") == 1:
        return "Welcome to our family. " + os.environ["FLAG"]
    else:
        return "You're not an admin :("

@app.route("/download")
def download_file():
    filename = request.args.get("filename")
    if not filename:
        abort(400)
    
    if ".." in filename:
        abort(400)

    try:
        if not (ROOT / filename).is_file():
            abort(404)
    except ValueError:
        abort(400)

    return send_file(filename, attachment_filename=os.path.basename(filename))
```

Widzimy, że korzystamy tutaj z pakietu session, który przechowuje wartość ```is_admin```. Oryginalne ciasteczko wygląda w sposób taki:
![Alt text](/assets/posts/kolska-leaks/org_cookie.png)

Więc pozostaje je zmienić na takie aby wartość ```is_admin``` wynosiła 1 i wysłać request do ```/flag```:
![Alt text](/assets/posts/kolska-leaks/final.png)
