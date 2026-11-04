# NASK (hack.cert.pl) – recon, edycja 11/2026

Program: Bug Bounty CERT Polska / NASK · nagrody 300–20 000 zł · 03–30.11.2026
Zasady: testy bez logowania lub tylko na kontach własnych/testowych; ≤ 5 req/s; zakaz DoS/spamu; PoC wymagany.
Stan na: 2026-11-04. Całość rozpoznania pasywna / nieinwazyjna (tylko GET, bez modyfikacji).

## Fingerprint celów Tier 2

| Host | Stan | Stack | Uwagi |
|---|---|---|---|
| **dataannotation.pllum.org.pl** (HIVE) | 200 | SPA (TanStack Router) + **FastAPI** `/api/v1` | **najlepszy trop** – patrz niżej |
| pllum.org.pl | 200 | Azure App Service, React/Vue | CSP z `script-src 'unsafe-inline'`; główny serwis PLLuM |
| science.nask.pl | 200 | Next.js (buildId `cx2h7afHZiCXQTnulJSjK`) | CSP obecny; serwis treściowy |
| 116sos.pl | 200 | Next.js (buildId `eY_q-tnDts2YvuKlalLWP`) | bardzo szczegółowy CSP; LiveChat/Genesys |
| bezpiecznewybory.pl | 403 | Akamai WAF | blokuje automaty – **nie obchodzimy WAF** |
| secure.edu.pl | brak odpowiedzi | — | nie odpowiada z sesji |

## Najlepszy trop: HIVE (dataannotation.pllum.org.pl)

Wewnętrzne narzędzie do anotacji danych dla projektu PLLuM. Czemu najlepszy cel:
bespoke, małe, z wyraźnym podziałem ról i zasobami adresowanymi po ID – typowe miejsce na
błędy kontroli dostępu (wysoko płatne).

**Potwierdzone fakty (pasywnie):**
- API: `https://dataannotation.pllum.org.pl/api/v1`, backend **FastAPI**.
- Uwierzytelnianie: **JWT Bearer** + nagłówek CSRF (`X-CSRF...` z ciasteczka `csrf_token`), `credentials: include`, refresh-token w localStorage.
- Role w kodzie frontendu: **`ADMIN`** i **`ANNOTATOR`** → rozdział uprawnień.
- Zasoby adresowane po ID: `projects/$projectShortName`, `.../conversation/$recordId`, `.../edit/$editedItem`.
- Obszar administracyjny: `/admin`, `/admin/logs`.
- Przepływy kont: `/login`, `/register`, `/forgot-password`, `/reset-password`, `/account`.
- `GET /api/v1/projects/` i `/users/` → **401** bez tokenu (autoryzacja wymuszona – brak wycieku bez logowania).
- `/docs`, `/openapi.json` → zwracają tylko index.html SPA → **dokumentacja OpenAPI NIE jest wystawiona**.
- Drobiazg: API robi `307` z `/projects` → `http://.../projects/` (downgrade https→http w Location). Z HSTS niewykorzystywalne; samo w sobie prawdopodobnie nienagradzane.

**Hipotezy do weryfikacji (wymagają kont testowych):**
1. **Horizontal IDOR / BAC** – czy `ANNOTATOR` widzi/edytuje projekty i konwersacje (`$projectShortName`, `$recordId`), do których nie jest przypisany.
2. **Vertical escalation** – czy `ANNOTATOR` dosięga endpointów `admin` (`/admin/logs`, zarządzanie projektami/userami).
3. **Mass assignment** przy tworzeniu/edycji konta lub projektu (np. podniesienie `role` do `ADMIN`).
4. **Reset hasła** – logika `/forgot-password` / `/reset-password` (token, brak powiązania z ofiarą).

## Status i następny krok

Faza bez logowania wyczerpana – API jest szczelne, brak łatwych wycieków.
Dalej potrzeba **uwierzytelnienia**, a najlepsze bugi (IDOR/BAC) wymagają **dwóch kont ANNOTATOR**
(do testów poziomych). Ograniczenie: rejestracja HIVE może być zamknięta/moderowana (to narzędzie wewnętrzne);
recaptcha/weryfikacja mailowa może blokować zakładanie konta z sesji chmurowej.

Kont nie zakładam samodzielnie bez decyzji – zgodnie z zasadami muszą to być konta własne/testowe.
