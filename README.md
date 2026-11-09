# SSH Brute-Force Lab: atak i zabezpieczenie

Projekt z przedmiotu Atak i zabezpieczenie sieci (Uniwersytet Komisji Edukacji Narodowej w Krakowie). W izolowanym laboratorium zademonstrowano podatność źle skonfigurowanej usługi SSH, przeanalizowano ślady w logach, a następnie wdrożono i zweryfikowano wielowarstwowe zabezpieczenia.

Autorki: Amelia Skoczylas, Zuzanna Lech

Pełne sprawozdanie ze zrzutami ekranu: [docs/atak-brute-force-ssh.pdf](docs/atak-brute-force-ssh.pdf)

> Wszystkie działania wykonano wyłącznie w odizolowanej sieci wirtualnej, na maszynach należących do autorek, w celach edukacyjnych.

## Środowisko

GNS3 (router Cisco 3725, switch) połączony z maszynami wirtualnymi VirtualBox.

| Maszyna | Adres IP | Rola |
|---|---|---|
| Kali Linux | 192.168.1.10 | Stacja audytowa |
| Ubuntu Server (OpenSSH 9.6p1) | 192.168.1.20 | Serwer SSH |
| Router Cisco | 192.168.1.1 | Brama domyślna |

## Zakres pracy

- **Rekonesans** – identyfikacja usługi i jej wersji narzędziem Nmap.
- **Demonstracja podatności** – test odporności konta `tester` (celowo słabe hasło) na atak słownikowy; w domyślnej konfiguracji serwer nie ograniczał liczby prób.
- **Analiza logów** – przegląd `/var/log/auth.log` i rozpoznanie wzorca pozostawianego przez zautomatyzowane narzędzie (seria wpisów `Failed password` kilka razy na sekundę).
- **Zabezpieczenie i weryfikacja** – wdrożenie zabezpieczeń i potwierdzenie ich skuteczności.

Dokładne polecenia i zrzuty ekranu znajdują się w [sprawozdaniu PDF](docs/atak-brute-force-ssh.pdf).

## Zabezpieczenia (defense in depth)

| Warstwa | Zmiana | Plik |
|---|---|---|
| IPS | Fail2Ban: blokada IP po 3 nieudanych próbach w ciągu 5 min, ban na 10 min | [config/jail.local](config/jail.local) |
| Konfiguracja usługi | `Port 2222`, `PermitRootLogin no`, `MaxAuthTries 3` | [config/sshd_config.hardening](config/sshd_config.hardening) |
| Uwierzytelnianie | klucze RSA 4096, `PasswordAuthentication no` | [config/sshd_config.hardening](config/sshd_config.hardening) |

## Weryfikacja skuteczności

| | Przed | Po |
|---|---|---|
| Port SSH | 22 | 2222 |
| Limit prób | brak | 3 próby, potem blokada IP (Fail2Ban) |
| Logowanie roota | dozwolone | zablokowane |
| Logowanie hasłem | włączone | wyłączone, tylko klucze |
| Wynik ponownego testu | dostęp uzyskany | połączenie odrzucone |

Test Fail2Ban przeprowadzono przed wyłączeniem logowania hasłem, aby zweryfikować samą blokadę. Po przejściu na klucze próba logowania bez klucza kończy się komunikatem `Permission denied (publickey)`.

## Wnioski

- Słabe hasło niweczy bezpieczeństwo nawet dobrze zaprojektowanego protokołu.
- Zmiana portu ogranicza jedynie szum od automatycznych skanerów i nie chroni przed celowanym atakiem.
- Najskuteczniejsze jest połączenie warstw: Fail2Ban reaguje na zachowanie, a uwierzytelnianie kluczami eliminuje podatność na zgadywanie haseł.

## Technologie

Linux (Ubuntu Server), OpenSSH, Fail2Ban, GNS3, VirtualBox, Nmap, Cisco IOS
