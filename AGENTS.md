# AGENTS.md — konsultacje

## Cel

Ten plik zawiera projektowe uzupełnienia do globalnych instrukcji Codexa użytkownika.

Globalny `~/.codex/AGENTS.md` definiuje mechanikę pracy z `git worktree`. Ten plik jej nie zastępuje — doprecyzowuje ją dla projektu `woolkan/konsultacje`.

Szczegółowa wiedza produktowa i architektoniczna jest utrzymywana w Confluence, a praca wykonawcza w Jira. Nie kopiuj pełnej dokumentacji do repozytorium.

Jeżeli Jira, Confluence, dokumentacja repo i kod są ze sobą sprzeczne, nie zgaduj — eskaluj konflikt do Product Ownera.

## Źródła prawdy

- **Confluence** — wiedza produktowa, UX, reguły biznesowe, architektura wysokiego poziomu, compliance i ADR.
- **Jira** — Epiki, SMART Stories, atomowe Zadania, zależności, kryteria odbioru i status realizacji.
- **GitHub** — kod, testy, konfiguracja, dokumentacja techniczna repo, branche, PR i CI.

Główne linki:

- Confluence: https://bluproject-team.atlassian.net/wiki/spaces/LK/overview
- Założenia produktu i MVP: https://bluproject-team.atlassian.net/wiki/spaces/LK/pages/12451841/Za+o+enia+produktu+i+MVP
- Sposób pracy: https://bluproject-team.atlassian.net/wiki/spaces/LK/pages/12058838/Spos+b+pracy+Product+Jira+Confluence+i+GitHub
- Standard Jira: https://bluproject-team.atlassian.net/wiki/spaces/LK/pages/12615682/Standard+opisu+zg+osze+Jira
- ADR: https://bluproject-team.atlassian.net/wiki/spaces/LK/pages/12419074/Decyzje+architektoniczne+ADR
- Jira: https://bluproject-team.atlassian.net/jira/software/c/projects/LK/list?jql=project%20%3D%20LK

## Git worktree — zasady projektu

Stosuj globalne zasady `git worktree` użytkownika oraz poniższe doprecyzowania projektu.

### Gałąź integracyjna

- Gałęzią integracyjną projektu jest `main`.
- Nie używamy `develop`, dopóki nie zostanie świadomie wprowadzona decyzją projektową.
- Stały checkout integracyjny znajduje się w katalogu `main/`.
- Nie prowadź długiej implementacji bezpośrednio w `main/`.
- Merge do gałęzi integracyjnej wykonuj z katalogu `main/`.

### Struktura worktree

Preferowana lokalna struktura:

```text
konsultacje/
  main/
  feature/
    lk-12-booking-hold/
    lk-13-stripe-webhook/
```

Każde atomowe Zadanie Jira realizuj w osobnym worktree pod `feature/` i na osobnym branchu.

Przykładowe nazwy branchy:

```text
feature/lk-12-booking-hold
fix/lk-28-stripe-webhook
chore/lk-7-ci
```

Branch i tytuł PR dla pracy z Jiry muszą zawierać rzeczywisty klucz Jira. Nie twórz pseudoidentyfikatorów.

### Aktualizacja i tworzenie worktree

Przed rozpoczęciem nowego zadania:

1. przejdź do stałego checkoutu `main/`,
2. zaktualizuj informacje o zdalnym repo,
3. zaktualizuj lokalny `main`,
4. dopiero potem utwórz lub odśwież worktree zadania.

Bazowy schemat:

```bash
cd main
git fetch --prune origin
git pull --ff-only origin main

git worktree add ../feature/lk-12-booking-hold \
  -b feature/lk-12-booking-hold origin/main
```

Jeżeli lokalna sytuacja Git wymaga innego bezpiecznego kroku, zachowaj nadrzędną zasadę: najpierw aktualny checkout integracyjny, potem worktree zadania.

## Atomizacja pracy

Dla pracy implementacyjnej obowiązuje:

```text
Epik
  ↓
SMART Story
  ↓
atomowe Zadanie
  ↓
1 branch
  ↓
1 worktree
  ↓
1 logiczny Pull Request
```

### SMART Story

Story opisuje jedną spójną zdolność produktową lub techniczną. Nie jest bezpośrednim poleceniem implementacyjnym dla Codexa.

Story musi być zrefinementowana i możliwa do rozbicia na atomowe Zadania.

### Atomowe Zadanie

Atomowe Zadanie powinno:

- zmieniać jedną spójną rzecz,
- mieć jeden jednoznaczny rezultat,
- posiadać jasny `Zakres` i `Poza zakresem`,
- mieć testowalne kryteria odbioru,
- określać wymagane testy / bramki jakościowe,
- wskazywać twarde zależności,
- linkować do wymaganej dokumentacji Confluence / ADR,
- nie wymagać od Codexa nowej decyzji produktowej ani przekrojowej decyzji architektonicznej,
- nadawać się do realizacji jako jeden logiczny PR.

Jeżeli Zadanie nie spełnia tych warunków, nie rozpoczynaj implementacji — wymaga dalszego refinementu.

## Zależności i równoległość

- Jeżeli Zadanie B wymaga kodu z Zadania A, B rozpoczynamy dopiero po merge A do `main` i aktualizacji checkoutu `main/`.
- Zadań zależnych nie realizujemy równolegle tylko dlatego, że worktree technicznie to umożliwia.
- Równoległa praca jest dopuszczalna wyłącznie dla rzeczywiście niezależnych zakresów o możliwie rozłącznych plikach i kontraktach.

## Zasady pracy z Jira

1. Nie rozpoczynaj implementacji bez konkretnego atomowego Zadania Jira spełniającego Definition of Ready.
2. Przed pierwszą zmianą kodu przeczytaj issue oraz wszystkie podlinkowane strony Confluence / ADR.
3. Nie zmieniaj samodzielnie zakresu, kryteriów akceptacji, priorytetu ani zależności issue.
4. Nie zgaduj nowych decyzji produktowych ani przekrojowych decyzji architektonicznych.
5. Jeśli źródła prawdy są sprzeczne albo brakuje istotnej decyzji, eskaluj do Product Ownera.
6. PR musi linkować do właściwego issue Jira.
7. Nie rozszerzaj zakresu PR o niepowiązane refaktoryzacje lub funkcjonalności.

## Commity

- Commity mają być małe i logiczne.
- Nie dodawaj niepowiązanych zmian „przy okazji”.
- Dla pracy z Jiry umieszczaj klucz Zadania w komunikacie commita, gdy jest to praktyczne.

## Pull Request

Preferowany tytuł:

```text
LK-12: Dodać tymczasową blokadę terminu
```

PR powinien zawierać:

- link do Jira,
- rezultat,
- zakres zmiany,
- wykonane testy,
- ryzyka / kompromisy,
- informacje o ewentualnych zmianach dokumentacji.

PR powinien odpowiadać jednemu atomowemu Zadaniu Jira i nie zawierać zmian spoza zakresu.

Preferowany sposób scalania: **squash merge**.

Codex nie scala PR samodzielnie bez jawnej zgody użytkownika.

Merge do `main` następuje dopiero po:

- spełnieniu kryteriów odbioru Jira,
- zielonym CI,
- wymaganym review Product Ownera,
- rozwiązaniu istotnych uwag review.

## Testy i jakość

- Każda zmiana zachowania wymaga adekwatnego testu automatycznego, jeśli jest realistycznie testowalna.
- Nie obniżaj wymagań lintingu, analizy statycznej ani testów tylko po to, aby CI było zielone.
- Aktualizuj dokumentację techniczną repo, jeśli zmienia się kontrakt, konfiguracja lub sposób uruchomienia.

## Autonomia Codexa

Codex może samodzielnie podejmować lokalne, odwracalne decyzje implementacyjne zgodne z istniejącymi ustaleniami.

Codex nie może samodzielnie rozstrzygać:

- nowego zachowania produktowego,
- istotnej zmiany UX,
- przekrojowej zmiany architektury,
- zmiany zaakceptowanego ADR,
- rozszerzenia zakresu Story / Zadania,
- destrukcyjnej decyzji niewynikającej z issue,
- konfliktu Jira ↔ Confluence ↔ GitHub.

W tych przypadkach eskaluj do Product Ownera.

## Szczegóły workflow Git

Projektowy opis workflow: `docs/engineering/git-workflow.md`.
