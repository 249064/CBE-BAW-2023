# Damn Vulnerable Web Application
DVWA (Damn Vulnerable Web Application) to aplikacja internetowa stworzona specjalnie w celu symulowania podatności i słabości typowych dla aplikacji internetowych. DVWA służy jako platforma do nauki i testowania umiejętności oraz narzędzi związanych z penetrowaniem, audytem bezpieczeństwa i zarządzaniem ryzykiem w środowisku webowym. Dzięki swojej podatności na różne ataki, takie jak wstrzykiwanie SQL, File inclusion, czy ataki XSS, DVWA umożliwia użytkownikom praktyczne zrozumienie zagrożeń związanych z bezpieczeństwem aplikacji webowych. Dostęp do tych podatności jest kontrolowany, co pozwala użytkownikom na rozwijanie umiejętności w zakresie ochrony aplikacji i skutecznego testowania bezpieczeństwa aplikacji webowych.

## Cel i przeznaczenie projektu
Celem projektu jest znalezienie oraz exploitacja jak największej liczby podatności w aplikacji webowej "Damn Vulnerable Web Application". Stworzone zostały opisy znalezionych podatności wraz z przykładowymi technikami ich eksploitacji. Część opisów zawiera również uniwersalne skrypty umożliwiające automatyczną eksploitację.

## Wymagania systemowe i sposób uruchamiania
Przedstawione przykłady zostały zrealizowane w środowisku systemu Linux i wykorzystanie ich będzie w nim najłatwiejsze. Wymagana jest dostępność aplikacji DVWA, uruchomionej lokalnie bądź zdalnie (na przykład z użyciem instancji udostępnianej przez TryHackMe).
Przykładowe skrypty są uruchamiane z linii poleceń i mogą wymagać:
- curl
- wget
- python, wraz z określonymi w kodzie bibliotekami
- pliki ze słownikami nazw użytkownika i haseł

Aktualny rzecznik: [@249064](https://www.github.com/249064)

# Index

## Lista opracowanych podatności

1. [Brute Force](https://github.com/249064/CBE-BAW-2023/blob/main/brute_force.md)
2. [CSRF](https://github.com/249064/CBE-BAW-2023/blob/main/csrf.md)
3. [File Upload](https://github.com/249064/CBE-BAW-2023/blob/main/file_upload.md)
4. [Insecure CAPTCHA](https://github.com/249064/CBE-BAW-2023/blob/main/insecure_captcha.md)
5. [LFI - Local File Inclusion](https://github.com/249064/CBE-BAW-2023/blob/main/local_file_inclusion.md)
6. [RFI - Remote File Inclusion](https://github.com/249064/CBE-BAW-2023/blob/main/remote_file_inclusion.md)
7. [CSP Bypass](https://github.com/249064/CBE-BAW-2023/blob/main/CSP.md)
8. [Weak JavaScript](https://github.com/249064/CBE-BAW-2023/blob/main/JavaScript.md)
9. [SQL Injection](https://github.com/249064/CBE-BAW-2023/blob/main/SQLi.md)
10. [SQL Injection - Blind](https://github.com/249064/CBE-BAW-2023/blob/main/SQLi_blind.md)
11. [Command Injection](https://github.com/249064/CBE-BAW-2023/blob/main/Command_Injection.md)
12. [Insecure Session IDs](https://github.com/249064/CBE-BAW-2023/blob/main/Insecure_IDs.md)
13. [XSS - DOM](https://github.com/249064/CBE-BAW-2023/blob/main/XSS_DOM.md)
14. [XSS - Reflected](https://github.com/249064/CBE-BAW-2023/blob/main/XSS_Reflected.md)
15. [XSS - Stored](https://github.com/249064/CBE-BAW-2023/blob/main/XSS_Stored.md)
