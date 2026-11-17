# Restful-booker API tests

![API tests](https://github.com/Drahomira-H/Restful-booker-tests/actions/workflows/api-tests.yml/badge.svg)

> This README is written in English and Czech. The English version comes first, the Czech version follows below ([jump to Czech](#česky)).
>
> Tento README je psaný anglicky a česky. Nejdřív je anglická verze, pod ní česká ([přejít na češtinu](#česky)).

# English

Practice project: automated tests for the [Restful-booker](https://restful-booker.herokuapp.com/apidoc/index.html) REST API in Postman, run with the Postman CLI and GitHub Actions.

The goal is to practise API testing (positive and negative scenarios, authentication, chaining requests), working with Git, and the basics of CI/CD.

## Tools used

- **Postman** (Local View, the collection is stored as files in the repository)
- **Postman CLI** (runs the collection from the terminal and in CI)
- **Git and GitHub** (branches, pull requests, SSH)
- **GitHub Actions** (runs the tests automatically)

## Repository structure

```
.github/workflows/api-tests.yml     GitHub Actions workflow
postman/collections/Restful-booker  collection (folders 01 Auth, 02 Booking CRUD, 03 Negative scenarios)
postman/environments/               environment (shared value baseUrl)
.gitignore                          files that Git does not track
```

## Tested scenarios

| Folder | What is verified | Status |
|---|---|---|
| 01 Auth | login, saving the token to a variable | done |
| 02 Booking CRUD | create, read, update, delete, check that the deleted booking returns 404 | done |
| 03 Negative scenarios | missing fields, wrong types, missing token, wrong password, non-existent ID | in progress |

The requests depend on each other: the token and `bookingId` are saved to variables and the next request uses them. The order must stay: Auth, Create, Read, Update, Delete, Read after delete.

## Inconsistencies found in the API

| Operation | What the API returns | What I would expect |
|---|---|---|
| POST /booking (create) | 200 | 201 |
| DELETE /booking/{id} (delete) | 201 | 204 |

More findings from the negative scenarios (missing required field, wrong password in `/auth`, wrong field type) will be added once they are finished.

## How to run the tests

### In Postman
1. Open the project folder (Files, Open Folder) and switch to **Local View**.
2. Select the `restful-booker` environment in the top right corner.
3. Collection, three dots, **Run**, select the folders and start the run.

### In the terminal (Postman CLI)
In the project root:
```powershell
postman collection run postman/collections/Restful-booker --environment postman/environments/restful-booker.environment.yaml
```
The exact syntax can be checked with `postman collection run --help`. The message "No authorization data found" is only a warning. Logging in is not needed for a local run.

**Possible future improvement:** running the tests with Newman as well. Newman can only read the collection in the v2.1 JSON format, so the collection would first have to be exported. The `newman/` folder (for Newman reports) is already listed in `.gitignore`.

### Automatically on GitHub
The `API tests` workflow runs on every pull request and after merging into `main`. The result is in the **Actions** tab, and on the pull request in the checks box.

## Postman notes

- A request must be **saved (Ctrl+S)**, otherwise the change does not reach the file or the commit.
- `baseUrl` must be a **shared value** in the environment, otherwise the CLI cannot read it from the file (error `getaddrinfo ENOTFOUND auth`).
- `token` stays empty and is filled in at runtime by the script in `/auth`. No secrets belong in the repository.
- Restful-booker accepts authentication as the header `Cookie: token={{token}}`.
- The response time check is flaky (the public API is sometimes slow), so the limit has a margin (3000 ms).
- After a booking is deleted, `PUT` and `DELETE` on the same ID no longer work (405), so the scenario is always run from the beginning.
- Checks: `git status` and `git diff` before every commit, mainly because of tokens and passwords.

## Git cheat sheet (my own reference)

### Three places where changes move

```
working directory --git add--> staging --git commit--> local repository --git push--> GitHub
```

- **Working directory:** files on disk as I edit them.
- **Staging:** changes prepared for the next commit.
- **Repository:** the saved history of commits.
- **GitHub (`origin`):** the remote copy of the repository.

### Basic commands

| Command | What it does |
|---|---|
| `git status` | what is changed, what is staged, what is untracked (run it often) |
| `git add file` | adds a file to staging |
| `git add .` | adds all changes in the current folder (check `git status` first) |
| `git commit -m "message"` | saves the staging area as a commit (message: what and why) |
| `git push` | uploads commits to GitHub |
| `git pull` | downloads changes from GitHub (fetch + merge) |
| `git fetch` | downloads changes without merging them |
| `git diff` | what I changed and have not added yet |
| `git diff --stat` | short overview of changed files |
| `git log --oneline` | list of commits |
| `git ls-files` | all files that Git tracks |

**Pager:** `git diff` and `git log` open the output in a pager. Close it with **`q`**, scroll with the arrow keys or the space bar.

### Setup (once)

```powershell
git config --global user.name "Name"
git config --global user.email "email@example.com"
git init                         # new repository in a folder
git remote add origin URL        # connect to GitHub
git remote -v                    # check addresses
git branch -M main               # rename the branch to main
git push -u origin main          # first push (-u remembers the link)
```

### Working with branches

```powershell
git switch main                  # switch to main
git pull                         # download the current state
git switch -c dh/branch-name     # create a new branch and switch to it
git push -u origin dh/branch-name
git branch -d dh/branch-name     # delete the branch locally after merging
git branch --show-current        # which branch I am on
```

### Making a change through a pull request

1. `git switch main`, `git pull`
2. `git switch -c dh/branch-name`
3. Make changes, then `git status`, `git diff`
4. `git add ...`, `git commit -m "..."`
5. `git push -u origin dh/branch-name`
6. On GitHub open a **pull request** (Compare & pull request, or Pull requests, New pull request). A pull request is not created automatically.
7. Wait for the **API tests** check. A green tick means it is safe to merge.
8. **Merge pull request**, delete the branch.
9. Locally: `git switch main`, `git pull`, `git branch -d dh/branch-name`

Every further push to a branch with an open pull request runs the checks again. Do not merge when the checks are red.

### Undoing changes

| Situation | Command |
|---|---|
| discard uncommitted changes in a file | `git restore file` |
| remove a file from staging (it stays on disk) | `git restore --staged file` |
| change the last commit (local only, before push) | `git commit --amend` |
| undo an already pushed commit without rewriting history | `git revert <hash>` or `git revert HEAD --no-edit` |
| discard everything uncommitted (irreversible) | `git reset --hard` (careful) |

### Merge and rebase

- **Merge** joins branches with a new merge commit, the history stays as it happened.
- **Rebase** replays my commits on top of the latest target branch, so the history is linear, but it gets rewritten. Do not use it on shared branches.
- Conflicts happen when the same place in a file was changed. After resolving: `git add file`, then `git rebase --continue` (for rebase) or finish the merge with a commit. `git rebase --abort` returns to the state before the start.

### Pull request, push and pull (so they do not get mixed up)

- `git push` sends changes from my computer to GitHub.
- `git pull` brings changes from GitHub to me.
- A **pull request** is a request to merge a branch into the main branch, where a review and tests happen.

### Checking what is on GitHub

```powershell
git ls-files                          # files that Git tracks
git log --name-status --oneline       # files in each commit
git show --stat                       # last commit
git log origin/main..HEAD --oneline   # what has not been pushed yet (empty = everything is up)
git check-ignore -v file              # why Git ignores a file
```

### .gitignore

Tells Git which files not to track (for example `node_modules/`, `newman/`, `.env`). It is committed to the repository so the rules apply to everyone. Ignoring affects only new files, files that are already tracked are left alone.

### SSH login to GitHub

```powershell
dir ~/.ssh                                        # keys (.pub is public, the other file is private and never shared)
ssh-keygen -t ed12345 -C "email@example.com"      # create a key
Get-Content ~/.ssh/id_ed12345.pub | Set-Clipboard # copy the public key
ssh-keygen -lf ~/.ssh/id_ed12345.pub              # key fingerprint (compare with GitHub)
ssh -T git@github.com                             # test the login
git remote set-url origin git@github.com:Drahomira-H/Restful-booker-tests.git
```
The public key is added on GitHub in **Settings, SSH and GPG keys**. When connecting to GitHub for the first time, confirm the server fingerprint by typing `yes`.

### If I push a token or a password

1. **Revoke it immediately** in the given service.
2. Create a new one and store it safely (password manager, GitHub Secrets).
3. Remove it from the code. Deleting it in the next commit is not enough, it stays in the history.
4. Optionally clean the history (`git filter-repo`, BFG) if it makes sense.
5. Check whether it was misused, and report it at work.

### Common errors and fixes

| Error | Cause and fix |
|---|---|
| `npm ... running scripts is disabled` | PowerShell blocks scripts. Use `npm.cmd`, or `cmd`, or `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` |
| `Permission denied (publickey)` | the public key is not on GitHub, or it differs from the one on disk (compare fingerprints) |
| `Permission ... denied to ...` when pushing over HTTPS | the token has no write permission, use SSH |
| `Author identity unknown` | `user.name` and `user.email` are not set |
| `remote origin already exists` | check with `git remote -v` |
| Push was rejected | GitHub has something I do not have locally: `git pull` and try again |
| `nothing to commit` | nothing is staged or nothing changed, check `git status` |
| `.gitignore` is not on GitHub | it is not added or committed: `git add .gitignore`, `git commit`, `git push` |

### Related concepts

- **Authentication** (who I am) and **authorization** (what I may do). In HTTP: 401 not authenticated, 403 not permitted.
- **REST:** resources with URLs, methods GET, POST, PUT, PATCH, DELETE, statelessness, status codes (2xx success, 4xx client error, 5xx server error).
- **JWT, OAuth 2.0, API key, Basic auth:** ways of authentication. OAuth is a framework, JWT is a token format.
- **On-prem and cloud:** running on the company's own infrastructure, or at a provider.

---

# Česky

Cvičný projekt: automatizované testy REST API [Restful-booker](https://restful-booker.herokuapp.com/apidoc/index.html) v Postmanu, spouštěné přes Postman CLI a GitHub Actions.

Cílem je procvičit testování API (pozitivní i negativní scénáře, autentizace, řetězení požadavků), práci s Gitem a základy CI/CD.

## Použité nástroje

- **Postman** (Local View, kolekce uložená jako soubory v repozitáři)
- **Postman CLI** (spuštění kolekce z terminálu a v CI)
- **Git a GitHub** (větve, pull requesty, SSH)
- **GitHub Actions** (automatické spuštění testů)

## Struktura repozitáře

```
.github/workflows/api-tests.yml     workflow pro GitHub Actions
postman/collections/Restful-booker  kolekce (složky 01 Auth, 02 Booking CRUD, 03 Negativní scénáře)
postman/environments/               environment (sdílená hodnota baseUrl)
.gitignore                          soubory, které Git nesleduje
```

## Testované scénáře

| Složka | Co se ověřuje | Stav |
|---|---|---|
| 01 Auth | přihlášení, uložení tokenu do proměnné | hotovo |
| 02 Booking CRUD | vytvoření, načtení, úprava, smazání, ověření smazání (404) | hotovo |
| 03 Negativní scénáře | chybějící pole, špatné typy, chybějící token, špatné heslo, neexistující ID | rozpracováno |

Požadavky na sebe navazují: token a `bookingId` se ukládají do proměnných a další požadavek je použije. Pořadí proto musí zůstat Auth, Vytvoření, Načtení, Úprava, Smazání, Načtení po smazání.

## Nalezené nesrovnalosti v API

| Operace | Co API vrací | Co by se čekalo |
|---|---|---|
| POST /booking (vytvoření) | 200 | 201 |
| DELETE /booking/{id} (smazání) | 201 | 204 |

Další nálezy z negativních scénářů se doplní po jejich dokončení (chybějící povinné pole, špatné heslo u `/auth`, špatný typ pole).

## Jak testy spustit

### V Postmanu
1. Otevři složku projektu (Files, Open Folder) a přepni na **Local View**.
2. Vpravo nahoře vyber environment `restful-booker`.
3. Kolekce, tři tečky, **Run**, vyber složky a spusť.

### V terminálu (Postman CLI)
V kořenu projektu:
```powershell
postman collection run postman/collections/Restful-booker --environment postman/environments/restful-booker.environment.yaml
```
Přesné zadání lze ověřit přes `postman collection run --help`. Hláška "No authorization data found" je jen upozornění, pro lokální běh přihlášení nepotřebuju.

**Možné budoucí vylepšení:** spouštění testů i přes Newman. Newman umí číst kolekci jen ve formátu v2.1 JSON, takže by se kolekce musela nejdřív exportovat. Složka `newman/` (pro reporty z Newmanu) je už v `.gitignore` uvedená.

### Automaticky na GitHubu
Workflow `API tests` se spustí u každého pull requestu a po sloučení do `main`. Výsledek je v záložce **Actions** a u pull requestu v rámečku s kontrolami.

## Poznámky k Postmanu

- Požadavek musí být **uložený (Ctrl+S)**, jinak se změna nepromítne do souboru ani do commitu.
- `baseUrl` musí být ve **sdílené hodnotě** environmentu, jinak ji CLI ze souboru nenačte (chyba `getaddrinfo ENOTFOUND auth`).
- `token` zůstává prázdný, plní se za běhu skriptem u `/auth`. Do repozitáře nepatří žádné tajné údaje.
- Restful-booker přijímá autentizaci jako hlavičku `Cookie: token={{token}}`.
- Kontrola doby odezvy je nestabilní (veřejné API je občas pomalé), proto je limit nastavený s rezervou (3000 ms).
- Po smazání rezervace už `PUT` ani `DELETE` na stejné ID nefungují (405), proto se scénář spouští vždy celý od začátku.
- Kontroly: `git status` a `git diff` před každým commitem, hlavně kvůli tokenům a heslům.

## Přehled příkazů Gitu (můj tahák)

### Tři místa, mezi kterými se změny přesouvají

```
pracovní složka --git add--> staging --git commit--> lokální repozitář --git push--> GitHub
```

- **Pracovní složka:** soubory na disku, tak jak je upravuju.
- **Staging:** změny připravené do dalšího commitu.
- **Repozitář:** uložená historie commitů.
- **GitHub (`origin`):** vzdálená kopie repozitáře.

### Základní příkazy

| Příkaz | Co dělá |
|---|---|
| `git status` | co je změněné, co je ve stagingu, co je neznámé (spouštět často) |
| `git add soubor` | přidá soubor do stagingu |
| `git add .` | přidá všechny změny v aktuální složce (nejdřív zkontrolovat `git status`) |
| `git commit -m "zpráva"` | uloží staging jako commit (zpráva: co a proč) |
| `git push` | nahraje commity na GitHub |
| `git pull` | stáhne změny z GitHubu (fetch + merge) |
| `git fetch` | stáhne změny, ale nic nesloučí |
| `git diff` | co jsem změnila a ještě nepřidala |
| `git diff --stat` | stručný přehled změněných souborů |
| `git log --oneline` | seznam commitů |
| `git ls-files` | všechny soubory, které Git sleduje |

**Pager:** `git diff` a `git log` otevřou výstup v pageru. Zavře se klávesou **`q`**, posun šipkami nebo mezerníkem.

### Nastavení (jednou)

```powershell
git config --global user.name "Jméno"
git config --global user.email "email@example.com"
git init                         # nový repozitář ve složce
git remote add origin URL        # propojení s GitHubem
git remote -v                    # kontrola adres
git branch -M main               # přejmenování větve na main
git push -u origin main          # první push (-u zapamatuje vazbu)
```

### Práce s větvemi

```powershell
git switch main                  # přepnout na main
git pull                         # stáhnout aktuální stav
git switch -c dh/nazev-vetve     # vytvořit novou větev a přepnout se na ni
git push -u origin dh/nazev-vetve
git branch -d dh/nazev-vetve     # po sloučení smazat větev lokálně
git branch --show-current        # na které větvi jsem
```

### Postup změny přes pull request

1. `git switch main`, `git pull`
2. `git switch -c dh/nazev-vetve`
3. Úpravy, pak `git status`, `git diff`
4. `git add ...`, `git commit -m "..."`
5. `git push -u origin dh/nazev-vetve`
6. Na GitHubu otevřít **pull request** (Compare & pull request, nebo Pull requests, New pull request). Pull request se nevytváří automaticky.
7. Počkat na kontrolu **API tests**. Zelená fajfka znamená, že lze sloučit.
8. **Merge pull request**, smazat větev.
9. Lokálně: `git switch main`, `git pull`, `git branch -d dh/nazev-vetve`

Každý další push do větve s otevřeným pull requestem spustí kontroly znovu. Při červených kontrolách se nesloučuje.

### Vrácení změn

| Situace | Příkaz |
|---|---|
| zahodit necommitnuté změny v souboru | `git restore soubor` |
| vrátit soubor ze stagingu (zůstane na disku) | `git restore --staged soubor` |
| upravit poslední commit (jen lokálně, před pushem) | `git commit --amend` |
| vrátit už pushnutý commit bez přepisování historie | `git revert <hash>` nebo `git revert HEAD --no-edit` |
| zahodit vše necommitnuté (nevratné) | `git reset --hard` (opatrně) |

### Merge a rebase

- **Merge** spojí větve novým merge commitem, historie zůstane tak, jak vznikla.
- **Rebase** přehraje moje commity na nejnovější špičku cílové větve a historie je lineární, ale přepisuje se. Nepoužívat na sdílených větvích.
- Konflikty vznikají, když se změní stejné místo v souboru. Po vyřešení: `git add soubor`, pak `git rebase --continue` (u rebase) nebo dokončit merge commitem. `git rebase --abort` vrátí stav před začátkem.

### Pull request, push a pull (ať se nepletou)

- `git push` posílá změny z mého počítače na GitHub.
- `git pull` stahuje změny z GitHubu ke mně.
- **Pull request** je žádost o sloučení větve do hlavní větve, kde proběhne kontrola (review a testy).

### Kontrola, co je na GitHubu

```powershell
git ls-files                          # soubory, které Git sleduje
git log --name-status --oneline       # soubory v jednotlivých commitech
git show --stat                       # poslední commit
git log origin/main..HEAD --oneline   # co ještě nebylo pushnuté (prázdné = vše je nahoře)
git check-ignore -v soubor            # proč Git soubor ignoruje
```

### .gitignore

Říká Gitu, které soubory nesledovat (např. `node_modules/`, `newman/`, `.env`). Commituje se do repozitáře, aby pravidla platila pro všechny. Ignorování se týká jen nových souborů, ty už sledované nechá být.

### SSH přihlášení k GitHubu

```powershell
dir ~/.ssh                                        # klíče (.pub je veřejný, druhý soubor je soukromý a nesdílí se)
ssh-keygen -t ed12345 -C "email@example.com"      # vytvoření klíče
Get-Content ~/.ssh/id_ed12345.pub | Set-Clipboard # zkopírovat veřejný klíč
ssh-keygen -lf ~/.ssh/id_ed12345.pub              # otisk klíče (porovnat s GitHubem)
ssh -T git@github.com                             # test přihlášení
git remote set-url origin git@github.com:Drahomira-H/Restful-booker-tests.git
```
Veřejný klíč se přidává na GitHubu v **Settings, SSH and GPG keys**. Při první komunikaci s GitHubem je třeba potvrdit otisk serveru slovem `yes`.

### Když pushnu token nebo heslo

1. **Okamžitě zneplatnit** (revoke) token nebo klíč v dané službě.
2. Vytvořit nový a uložit ho bezpečně (správce hesel, GitHub Secrets).
3. Odstranit ho z kódu. Samotné smazání v dalším commitu nestačí, v historii zůstává.
4. Případně vyčistit historii (`git filter-repo`, BFG), pokud to dává smysl.
5. Zkontrolovat, jestli nedošlo ke zneužití, a v práci to nahlásit.

### Časté chyby a řešení

| Chyba | Příčina a řešení |
|---|---|
| `npm ... running scripts is disabled` | PowerShell blokuje skripty. Použít `npm.cmd`, nebo `cmd`, případně `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` |
| `Permission denied (publickey)` | veřejný klíč není na GitHubu, nebo je jiný než ten na disku (porovnat otisky) |
| `Permission ... denied to ...` při pushi přes HTTPS | token nemá oprávnění zapisovat, použít SSH |
| `Author identity unknown` | nenastavené `user.name` a `user.email` |
| `remote origin already exists` | kontrola přes `git remote -v` |
| Push se odmítl | na GitHubu je něco, co nemám lokálně: `git pull` a znovu |
| `nothing to commit` | nic není ve stagingu nebo se nic nezměnilo, zkontrolovat `git status` |
| `.gitignore` není na GitHubu | není přidaný nebo commitnutý: `git add .gitignore`, `git commit`, `git push` |

### Související pojmy

- **Autentizace** (kdo jsem) a **autorizace** (co smím). V HTTP: 401 neověřená, 403 nemá oprávnění.
- **REST:** zdroje s URL, metody GET, POST, PUT, PATCH, DELETE, bezstavovost, status kódy (2xx úspěch, 4xx chyba klienta, 5xx chyba serveru).
- **JWT, OAuth 2.0, API key, Basic auth:** způsoby autentizace, OAuth je rámec, JWT je formát tokenu.
- **On-prem a cloud:** provoz na vlastní infrastruktuře firmy, nebo u poskytovatele.
