# Insecure Session IDs
## Opis techniki
<blockquote>
Podatność na słabe identyfikatory sesji odnosi się do użycia słabo chronionych lub przewidywalnych identyfikatorów sesji. Ta słabość może umożliwić przejęcie sesji lub podgląd boczny sesji, gdzie atakujący przechwytuje identyfikator sesji i może uzyskać dostęp do konta ofiary na danej stronie.

Oto kilka kluczowych cech i zagrożeń związanych z X:

* Słabe generowanie identyfikatorów sesji: Podatność na Insecure Session IDs często wynika z niewłaściwego generowania identyfikatorów sesji. Jeżeli identyfikatory sesji są generowane w sposób przewidywalny lub nie są dostatecznie losowe, atakujący może odgadnąć lub przewidzieć identyfikatory innych sesji.
* Przechowywanie i przesyłanie identyfikatorów sesji: Jeżeli identyfikatory sesji są przechowywane lub przesyłane w niesecure sposób, tak jak na przykład w czystym tekście, mogą być one przechwycone przez atakującego. To dotyczy również przesyłania identyfikatorów sesji przez niezabezpieczone połączenia lub ich przechowywania w niewłaściwie zabezpieczonych ciasteczkach.
* Brak unieważniania identyfikatorów sesji: Jeżeli identyfikatory sesji nie są unieważniane po zakończeniu sesji, na przykład po wylogowaniu użytkownika, mogą one zostać przechwycone i użyte do nieautoryzowanego dostępu do konta użytkownika.
* Zagrożenie dla prywatności i bezpieczeństwa użytkowników: Podatność na Insecure Session IDs stanowi poważne zagrożenie dla prywatności i bezpieczeństwa użytkowników. Może prowadzić do kradzieży sesji, co umożliwia atakującemu przejęcie sesji użytkownika, co z kolei pozwala na nieautoryzowany dostęp do konta użytkownika, kradzież danych lub wykonanie działania w imieniu użytkownika.
</blockquote>

## Zapobieganie
<blockquote>
W celu zapobiegania tej podatności, aplikacje powinny używać silnych mechanizmów generowania identyfikatorów sesji, które są trudne do odgadnięcia lub brutalnego przeszukania. Ważne jest także, aby aplikacja prawidłowo unieważniała identyfikatory sesji po wylogowaniu i przydzielała nowe identyfikatory sesji przy logowaniu.
</blockquote>

## DVWA labs
- [DVWA Insecure Session IDs #1 - Medium](#dvwa-Insecure-SessionIDs-1---medium)
- [DVWA Insecure Session IDs #2 - High](#dvwa-Insecure-SessionIDs-2---high)

<br/>

## DVWA Insecure Session IDs #1 - Medium
#### Request i odpowiedź serwera:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/Insecure_IDs_1_med.png "Kilkukrotna generacja ID sesji w celu zbadania mechanizmu.")
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/Insecure_IDs_2_med.png "Kilkukrotna generacja ID sesji w celu zbadania mechanizmu.")

Wybrane następujące po sobie requesty w odstępie około jednej sekundy spowodowały ustawienie identyfikatora sesji (dvwaSession) na wartości odpowiadające timestampowi w formacie epoch time. 
Mechanizm ten uniemożliwiłby działanie aplikacji webowej, gdzie 2 userów próbowałoby się zalogować w tej samej sekundzie. Ponadto – w zależności od ruchu, podmiana dvwaSession na dowolną wartość z przedziału czasu działania aplikacji spowodowałoby przejęcie sesji innego użytkownika. System DVWA nie przewidział stworzenia dodatkowych użytkowników, których sesję możnaby przejąć w ten sposób, dlatego też trudno zaprezentować wynik wykorzystania danej podatności na owej maszynie.

<br/>
<br/>

## DVWA Insecure Session IDs #2 - High
#### Request i odpowiedź serwera:
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/Insecure_IDs_1_high.png "Kilkukrotna generacja ID sesji w celu zbadania mechanizmu.")
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/Insecure_IDs_2_high.png "Kilkukrotna generacja ID sesji w celu zbadania mechanizmu.")
<br/>
W tym wypadku wartość dvwaSession zachowywała pewne właściwości – tą sam rozmiar (128 bitów), brak znaków specjalnych oraz litery a-f. Sugerowało to format hasha MD5. Powtórzono więc requesty i uzyskano 4 wartości, które później wyszukano w wyszukiwarce znanych hashy (https://crackstation.net/)
<br/>
![alt text](https://github.com/249064/CBE-BAW-2023/raw/main/res/Insecure_IDs_3_high.png "Wynik wyszukania ciągów IDs w wyszukiwarce znanych hashy.")
<br/>
Generacja nowych identyfikatory sesji polega na uzyskaniu hasha MD5 liczby inkrementowanej o 1, co pozwala na łatwe podszycie się pod użytkownika. System DVWA nie przewidział stworzenia dodatkowych użytkowników, których sesję możnaby przejąć w ten sposób, dlatego też trudno zaprezentować wynik wykorzystania danej podatności na owej maszynie.
