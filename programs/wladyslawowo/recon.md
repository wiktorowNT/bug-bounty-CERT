# Gmina Władysławowo (CCIOD) – recon, edycja 11/2026

Program: Bug Bounty Gmina Władysławowo · nagrody 100–4000 zł · 03–30.11.2026
Zasady: testy bez logowania lub na kontach własnych/testowych; ≤ 5 req/s; zakaz DoS/spamu; PoC wymagany.
Zakres: Tier1 `eobieg.wladyslawowo.pl` · Tier2 `wladyslawowo.pl`, `cciod.wladyslawowo.pl`. `bip.wladyslawowo.pl` POZA zakresem.
Stan na: 2026-11-04. Całość pasywna/nieinwazyjna (GET + jeden POST do wyszukiwarki z benign canary).

## Fingerprint
| Host | Stan | Stack |
|---|---|---|
| wladyslawowo.pl | 200 | openresty + **CMS edito (Ideo / idcom-jst.pl)**, jQuery UI |
| cciod.wladyslawowo.pl | 200 | **IdeaWebServer/6.4.2** (Ideo), strona informacyjna CCIOD |
| eobieg.wladyslawowo.pl | brak odpowiedzi na 443/8443/4443/8080 | regulamin: „nie zwraca zawartości na standardowych portach" |

## Testy bez logowania – wyniki

### wladyslawowo.pl (CMS edito)
- Wyszukiwarka: **POST `/wyszukiwarka`** (`opcja=szukaj`, `fraza=...`).
  - Reflected XSS: **NIE** – sanitizer **usuwa** `< > " '` (canary `...<b>a</b>"'` wrócił jako `babxy` w `value="..."`).
- Strona 404: soft-404 (HTTP 200), **ścieżka nie jest odbijana** → brak XSS z path.
- GET do `/wyszukiwarka`: parametr odbity **URL-encoded** w atrybucie → niewykonywalny.
- Nagłówki: ciasteczko `uid` bez `HttpOnly`/`Secure`/`SameSite`; brak CSP/X-Frame-Options/HSTS.
  → **wykluczone z nagród** (regulamin §8.2: brak nagłówków/flag nie jest nagradzany).

### cciod.wladyslawowo.pl (IdeaWebServer)
- Strona informacyjna (CCIOD), głównie statyczna + dane kontaktowe. Brak formularzy/parametrów wejściowych w widoku.

### eobieg.wladyslawowo.pl
- Niedostępny na typowych portach (zgodnie z notą w zakresie). Nie skanuję portów agresywnie (zasady DoS).

## Wniosek
Brak błędu nadającego się do zgłoszenia na dostępnej, nieuwierzytelnionej powierzchni.
Jedyne obserwacje (brak nagłówków, flagi ciasteczka) są wprost wykluczone z nagród.
