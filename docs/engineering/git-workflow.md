# Workflow Git projektu konsultacje

## Zakres

Mechanikę `git worktree` definiują globalne instrukcje Codexa użytkownika w `~/.codex/AGENTS.md`.

Ten dokument opisuje projektowe rozszerzenia dla `woolkan/konsultacje` i musi być stosowany razem z lokalnym `AGENTS.md`.

## Gałąź integracyjna

- Gałęzią integracyjną projektu jest `main`.
- Nie używamy `develop`, dopóki nie zostanie świadomie wprowadzona.
- Stały checkout integracyjny znajduje się w katalogu `main/`.
- `main` ma pozostać zielony i potencjalnie wdrażalny.
- Nie prowadzimy długiej implementacji bezpośrednio w `main/`.

## Preferowana struktura lokalna

```text
konsultacje/
  main/
  feature/
    lk-12-booking-hold/
    lk-13-stripe-webhook/
```

Katalog `feature/` jest miejscem dla worktree zadań niezależnie od tego, czy branch ma prefiks `feature/`, `fix/` czy `chore/`.

## Powiązanie z Jira

Dla pracy wynikającej z atomowego Zadania Jira obowiązuje:

```text
1 atomowe Zadanie Jira
= 1 branch
= 1 worktree
= 1 logiczny PR
```

Nazwa brancha i tytuł PR muszą zawierać rzeczywisty klucz Jira.

Przykłady:

```text
feature/lk-12-booking-hold
fix/lk-28-stripe-webhook
chore/lk-7-ci
```

Nie twórz pseudoidentyfikatorów, których nie ma w Jira.

## Rozpoczęcie zadania

Nowe Zadanie zaczynamy od aktualizacji stałego checkoutu integracyjnego.

Przykład:

```bash
cd main
git fetch --prune origin
git pull --ff-only origin main

git worktree add ../feature/lk-12-booking-hold \
  -b feature/lk-12-booking-hold origin/main
```

Jeżeli worktree już istnieje, najpierw upewnij się, że `main/` jest aktualny, a następnie bezpiecznie odśwież branch zadania zgodnie z jego stanem.

Nie stosuj destrukcyjnych operacji Git do usuwania niepowiązanych zmian użytkownika.

## Atomizacja

Story nie jest bezpośrednią jednostką implementacji Codexa.

Hierarchia:

```text
Epik
  ↓
SMART Story
  ↓
atomowe Zadania
  ↓
branch + worktree
  ↓
PR
  ↓
CI + review
  ↓
merge
```

Atomowe Zadanie:

- zmienia jedną spójną rzecz,
- ma jeden rezultat,
- ma jasny zakres i poza zakresem,
- posiada testowalne kryteria odbioru,
- wskazuje testy i zależności,
- nie wymaga nowej decyzji produktowej ani przekrojowej architektonicznej,
- mieści się w jednym logicznym PR.

Jeżeli nie da się spełnić tych warunków, Zadanie wymaga dalszego refinementu przed implementacją.

## Kolejność zadań

Jeżeli Zadanie B wymaga kodu z Zadania A:

1. A jest implementowane i przechodzi review,
2. A zostaje scalone do `main`,
3. checkout `main/` jest aktualizowany,
4. dopiero wtedy rozpoczynamy lub odświeżamy worktree B.

Zależnych zadań nie realizujemy równolegle tylko dlatego, że `git worktree` technicznie na to pozwala.

Równoległa praca jest dopuszczalna dla rzeczywiście niezależnych zakresów z możliwie rozłącznymi plikami i kontraktami.

## Commity

- Commity są małe i logiczne.
- Nie zawierają niepowiązanych refaktoryzacji „przy okazji”.
- Dla pracy z Jiry klucz Zadania powinien znaleźć się w komunikacie commita, gdy jest to praktyczne.

## Pull Request

Preferowany tytuł:

```text
LK-12: Dodać tymczasową blokadę terminu
```

PR powinien:

- odpowiadać jednemu atomowemu Zadaniu Jira,
- linkować do Zadania,
- opisywać wykonany rezultat i zakres,
- wskazywać sposób weryfikacji i testy,
- wymieniać istotne ryzyka / kompromisy,
- nie zawierać niepowiązanych zmian,
- mieć zielone kontrole adekwatne do zmiany.

Preferowany sposób scalania: **squash merge**.

Codex nie scala PR samodzielnie bez jawnej zgody użytkownika.

## Merge

Merge do `main` wykonujemy z kontekstu stałego checkoutu `main/` po:

- spełnieniu kryteriów odbioru Jira,
- zielonym CI,
- review Product Ownera,
- rozwiązaniu istotnych uwag review.

Po merge przed rozpoczęciem kolejnego zależnego Zadania aktualizujemy `main/`.

## Prace bootstrapowe i wyjątki

Bezpośrednia zmiana na `main` jest wyjątkiem i wymaga jawnej zgody użytkownika, np. przy inicjalizacji całkowicie pustego repozytorium lub pilnej korekcie zasad repo.

Po ustanowieniu workflow kolejne zmiany implementacyjne powinny przechodzić przez osobny branch/worktree i PR.
