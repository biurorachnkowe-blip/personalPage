# podatki.swiebodzin.pl — strona statyczna

Natywna wersja strony (HTML/CSS/JS, bez frameworków i CMS) przygotowana do hostowania na GitHub Pages.

## Struktura

```
index.html          — strona główna (hero, usługi, sekcja kontaktowa #kontakt z formularzem i mapą)
o-mnie.html          — podstrona "O mnie"
uslugi.html          — podstrona "Usługi"
polityka-prywatnosci.html — polityka prywatności
assets/css/style.css — style
assets/js/main.js    — nawigacja mobilna + obsługa formularza kontaktowego
assets/img/          — zdjęcia i logotypy
```

## Uruchomienie lokalne

Wystarczy otworzyć `index.html` w przeglądarce albo uruchomić prosty serwer, np.:

```powershell
python -m http.server 8000
```

i wejść na `http://localhost:8000`.

## Publikacja (GitHub Pages + domena podatki.swiebodzin.pl)

Strona jest hostowana na GitHub Pages z repozytorium klientki `biurorachnkowe-blip/personalPage` (branch `main`, folder `/ (root)`). Każdy push na `main` publikuje nową wersję w ciągu 1–2 minut. Kopia deweloperska: `commitpanic/podatki.swiebodzin`.

- Plik `CNAME` zawiera domenę `podatki.swiebodzin.pl`. Nie usuwać, bo GitHub Pages straci domenę.
- Plik `.nojekyll` wyłącza przetwarzanie przez Jekylla.
- Domena i poczta `beata@podatki.swiebodzin.pl` zostają w home.pl (serwer `46.242.253.21`). Na GitHub przechodzi tylko WWW.

### Rekordy DNS w panelu home.pl (strefa podatki.swiebodzin.pl)

Stan sprzed zmiany (2026-10-18):

```
podatki.swiebodzin.pl      A      46.242.253.21           (serwer home.pl: WWW + poczta)
podatki.swiebodzin.pl      MX     podatki.swiebodzin.pl
podatki.swiebodzin.pl      TXT    "v=spf1 a mx ~all"
podatki.swiebodzin.pl      TXT    "google-site-verification=..."
www.podatki.swiebodzin.pl  CNAME  podatki.swiebodzin.pl
```

MX wskazuje na samą domenę, więc poczta korzysta z rekordu A. Przed przepięciem WWW pocztę trzeba przenieść na osobną nazwę. Inaczej po zmianie A poczta trafiałaby na serwery GitHub i przestałaby działać.

**Krok 1. Poczta (najpierw, potem odczekać co najmniej 1 h, TTL 3600):**

| Nazwa                        | Typ | Wartość                         | Akcja   |
|------------------------------|-----|---------------------------------|---------|
| `mail.podatki.swiebodzin.pl` | A   | `46.242.253.21`                 | dodać   |
| `podatki.swiebodzin.pl`      | MX  | `10 mail.podatki.swiebodzin.pl` | zmienić |
| `podatki.swiebodzin.pl`      | TXT | `v=spf1 mx ~all`                | zmienić (zamiast `v=spf1 a mx ~all`) |

Jeśli w programie pocztowym klientki jako serwer IMAP/SMTP wpisane jest `podatki.swiebodzin.pl`, trzeba je zmienić na adres serwera z panelu home.pl (np. `serwerXXXXXX.home.pl`) albo na `mail.podatki.swiebodzin.pl`.

**Krok 2. WWW na GitHub Pages:**

Domena jest przypisana do serwera WWW w home.pl (stary WordPress). Wtedy rekordy A i `www` są zarządzane przez to przypisanie i nie da się ich edytować w konfiguracji domeny. Zmienia się je w **Usługi WWW → [serwer] → Przypisane domeny → edycja domeny** albo po odpięciu domeny od serwera. Przed odpięciem upewnić się w home.pl, że skrzynka `beata@podatki.swiebodzin.pl` zostaje (zob. krok 1).

home.pl pozwala na **jeden** rekord A dla domeny. GitHub Pages działa z jednym adresem, cztery to tylko redundancja.

| Nazwa                       | Typ   | Wartość                         | Akcja   |
|-----------------------------|-------|---------------------------------|---------|
| `podatki.swiebodzin.pl`     | A     | `185.199.108.153`               | zmienić (zamiast `46.242.253.21`) |
| `podatki.swiebodzin.pl`     | AAAA  | `2606:50c0:8000::153`           | dodać, jeśli panel pozwala; nie może zostać AAAA na home.pl |
| `www.podatki.swiebodzin.pl` | CNAME | `biurorachnkowe-blip.github.io` | zmienić |

Dla samej nazwy `podatki.swiebodzin.pl` nie używamy CNAME, bo kolidowałby z MX/TXT. Rekordu `google-site-verification` nie ruszamy.

Plik `CNAME` wypychamy do repozytorium dopiero po kroku 2. Wcześniej adres github.io przekierowywałby na starą stronę w home.pl.

### Konfiguracja w repozytorium (robi właścicielka repo)

3. **Settings → Pages → Custom domain**: `podatki.swiebodzin.pl` (wypełni się samo z pliku `CNAME`). Po rozpropagowaniu DNS zaznaczyć **Enforce HTTPS**. Certyfikat Let's Encrypt wystawia GitHub, zwykle w ciągu kilkunastu minut do kilku godzin.
4. Zalecane: weryfikacja domeny na koncie GitHub (**Settings konta → Pages → Add a domain**). GitHub poda rekord TXT `_github-pages-challenge-biurorachnkowe-blip.podatki.swiebodzin.pl` do dodania w home.pl. Chroni to przed przejęciem domeny przez cudze repozytorium.

## Formularz kontaktowy

GitHub Pages nie obsługuje backendu, więc formularz na stronie głównej (sekcja `#kontakt`) wysyła wiadomości przez [EmailJS](https://www.emailjs.com). Identyfikatory usługi, szablonu i klucz publiczny są w `assets/js/main.js`. Klucz publiczny EmailJS jest z założenia jawny.

Formularz przekazuje do szablonu zmienne: `{{name}}`, `{{from_name}}`, `{{email}}`, `{{reply_to}}`, `{{phone}}`, `{{message}}`, `{{title}}`.

Ochrona przed spamem: ukryte pole-pułapka `website` w formularzu. Dodatkowo w panelu EmailJS (Account → Security) warto ograniczyć dozwolone domeny do adresu strony.
