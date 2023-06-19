---
title: Loteria flagowa - PWNing2016
author: k1k9
categories: [PWNing2016, web]
tags: [ctf, pwning2016]
---

# Loteria flagowa
Punkty: 100
Rozwiązań: 44

Znaleźliśmy w internecie loterię flagową. W zgodzie z ustawą o grach hazardowych zostanie ona zaraz zablokowana, ale spróbuj przed zamknięciem wyciągnąć z niej flagę.


## Let's go
Na samym początku widzimy zakładkę ```SERVER SOURCE``` i mamy tam kod źródłowy naszej aplikacji:
```js
var express = require("express");
var app = express();
var expressWs = require('express-ws')(app);
var fs = require("fs");

var flag = fs.readFileSync("../flag").toString();

app.use(express.static('.'));

app.ws('/', function(ws, req) {
	var seed = new Date().valueOf() & 0xFFFFFFFF;
	var rnd = betterRand(seed)
    var userId = new Buffer(seed.toString()+","+rnd.next().value).toString("base64")
    
    var numbers = Array.from(Array(6)).map(() => Math.floor(rnd.next().value * 89 + 10))

    ws.on('message', function(msg) {
        try {
            var m = JSON.parse(msg.replace("'", '').replace("'", ''));
            var resp = {"numbers": numbers}
            
            if(JSON.stringify(resp.numbers) === JSON.stringify(m.numbers))
                resp.flag = flag;

            console.log(resp);
            ws.send(JSON.stringify(resp));
        } catch(err) { }

        ws.close()
    });

    console.log("[*] Peer connected!");
    ws.send(JSON.stringify({"userId": userId}))
});

console.log("[*] Listening on port 5555...")
app.listen(5555);

function* betterRand(seed) {
  var m = 25, a = 11, c = 17, z = seed || 3;
  for(;;) yield (z=(a*z+c)%m)/m;
}
```

Widzimy, że po nawiązaniu połączenia WebSocket jest nam losowany seed i przekazywany do funkcji ```betterRand```, z której następnie są losowane nasze szczęśliwe liczby i porównywane. 
Pozostaje przepisać kod i odpalić z wygenerowanym już przez program ziarnem.
Ziarno:
![Alt seed](/assets/posts/loteria-flagowa/seed.png)
```sh
$ echo "LTgzNTI4OTgyMywtMC40NA==" | base64 -d
> -835289823,-0.44
 ```
Pierwsza wartość to nasze ziarno a druga to "losowa liczba", teraz wystarczy podstawić nasz seed pod skrypt i otrzymamy liczby jakie wybrał sobie serwer:
```js
function* betterRand(seed) {
    var m = 25, a = 11, c = 17, z = seed || 3;
    for(;;) yield (z=(a*z+c)%m)/m;
  }

var rnd = betterRand(-835230646)
rnd.next().value // pominięcie pierwszej liczby, która jest dołączana do userID
console.log(Array.from(Array(6)).map(() => Math.floor(rnd.next().value * 89 + 10)))
```

Pozostało nam tylko wpisać liczby i przesłać. Niestety frontend apki nie puści nas z ujemnymi liczbami więc w tym celu wykorzystałem postmana. Nawiązałem połączenie, wygenerowałem liczby i przesłałem je do serwera:
![Alt text](/assets/posts/loteria-flagowa/result.png)