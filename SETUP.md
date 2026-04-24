# vatownik-status — setup

Repo oparte o [Upptime](https://upptime.js.org/) — monitoring przez GitHub
Actions, status page na GitHub Pages, 0 zł infrastruktury.

Konfiguracja jest już gotowa. Do uruchomienia zostały 4 kroki operacyjne.

---

## 1. Push do GitHuba

```bash
cd /Users/monczga/Sites/Accounting/vatownik-status

git init
git add .
git commit -m "initial Upptime config for status.vatownik.pl"
git branch -M master

gh repo create vatownik-status \
  --public \
  --description "Status page for vatownik.pl" \
  --source=. \
  --push
```

Repo **musi być publiczne** — GitHub Actions są wtedy darmowe bez limitu minut.
Prywatne repo zjedzie plan w ~5 dni przy cronie co 5 min.

## 2. Personal Access Token

Upptime commituje raporty i otwiera Issues — potrzebuje tokenu z uprawnieniami
`repo` + `workflow`.

1. <https://github.com/settings/tokens/new?scopes=repo,workflow&description=vatownik-status>
2. Generate token → skopiuj wartość
3. W repo `vatownik-status`:
   - Settings → Secrets and variables → Actions → New repository secret
   - Name: `GH_PAT`
   - Value: *(wklej token)*

Bez tego Actions będą padać z `403: Resource not accessible by integration`.

## 3. GitHub Pages + custom domain

W repo `vatownik-status`:

1. Settings → Pages
2. Source: **Deploy from a branch**
3. Branch: `gh-pages` / `/ (root)` *(pojawi się po pierwszym successful run — patrz krok 4)*
4. Custom domain: `status.vatownik.pl`
5. Zaznacz **Enforce HTTPS** *(dostępne dopiero po propagacji DNS)*

## 4. DNS — Cloudflare

Dashboard Cloudflare → DNS → Records → Add record:

| Type  | Name   | Target              | Proxy status              | TTL  |
|-------|--------|---------------------|---------------------------|------|
| CNAME | status | `monczga.github.io` | **DNS only** (szara chmurka) | Auto |

> Proxy **WYŁĄCZONE**. GitHub Pages wystawia własny SSL przez Let's Encrypt
> i proxy Cloudflare wchodzi mu w paradę — HTTPS nie wstanie. Po zadziałaniu
> (~15 min) można ewentualnie włączyć proxy, ale nie jest to potrzebne.

## 5. Pierwszy run

Po pushu Actions nie odpalają się od razu — uruchom ręcznie pierwszy cykl:

```bash
gh workflow run setup.yml -R monczga/vatownik-status
gh workflow run uptime.yml -R monczga/vatownik-status
```

Po 2-3 minutach:

- Branch `gh-pages` powinien się pojawić
- `https://status.vatownik.pl` powinno wstać (DNS + SSL może zająć do 30 min)
- `history/` w repo wypełni się snapshotami

---

## Co jest monitorowane

Edytuj `sites:` w `.upptimerc.yml` żeby zmienić listę:

- `https://vatownik.pl` (GET, oczekuje 200)
- `https://vatownik.pl/blog` (GET)
- `https://vatownik.pl/blog/rss.xml` (GET)
- `https://vatownik.pl/api/lead` (POST — oczekuje 4xx od endpointu z walidacją)
- `https://vatownik.pl/api/contact` (POST — jw.)

Częstotliwość: co 5 minut (cron `*/5 * * * *` w `.github/workflows/uptime.yml`).

## Co się dzieje przy incydencie

1. Upptime wykrywa >1 failed check z rzędu
2. Otwiera GitHub Issue z tagiem service (np. `blog`)
3. Commituje update do `history/<service>/<slug>.md`
4. Aktualizuje `README.md` + odbudowuje status page
5. Po recovery — zamyka Issue z komentarzem

Dodatkowe powiadomienia (Telegram, Slack, email) — secrets w repo, pełna lista:
<https://upptime.js.org/docs/notifications>

## Zmiana tokenów / haseł

Nic nie trzeba robić w kodzie — `GH_PAT` może zostać zrotowany w Settings → Secrets.
Stary token przestaje działać po `Delete` w GitHub settings.

## Jeśli coś nie działa

| Symptom | Przyczyna | Fix |
|---------|-----------|-----|
| Actions failuje `403` | Brak `GH_PAT` secret | Krok 2 |
| `gh-pages` branch nie powstaje | Brak `GH_PAT` z `workflow` scope | Token musi mieć obie permissions |
| `status.vatownik.pl` → certificate error | Cloudflare proxy włączone | Wyłącz proxy (szara chmurka) |
| 404 na status page | GitHub Pages bez branch | Settings → Pages → ustaw `gh-pages` po pierwszym runie |
| Cron nie odpala | Repo prywatne + wyczerpane minuty | Zmień na publiczne |

---

## Linki

- Upptime docs: <https://upptime.js.org/>
- Konfiguracja: `.upptimerc.yml`
- Template: <https://github.com/upptime/upptime>
