# AGENTS.md

## Cel

Ten plik zawiera minimalne guardrails dla Codexa pracującego nad repozytorium `woolkan/konsultacje`.

Szczegółowa wiedza produktowa i architektoniczna jest utrzymywana w Confluence, a praca wykonawcza w Jira. Nie kopiuj pełnej dokumentacji do repozytorium.

## Źródła prawdy

- Confluence — wiedza produktowa, UX, reguły biznesowe, architektura wysokiego poziomu, compliance i ADR.
- Jira — Epiki, SMART Stories, atomowe Zadania, zależności i status realizacji.
- GitHub — kod, testy, konfiguracja, dokumentacja techniczna repo, branche, PR i CI.

Główne linki:

- Confluence: https://bluproject-team.atlassian.net/wiki/spaces/LK/overview
- Sposób pracy: https://bluproject-team.atlassian.net/wiki/spaces/LK/pages/12058838/Spos+b+pracy+Product+Jira+Confluence+i+GitHub
- Standard Jira: https://bluproject-team.atlassian.net/wiki/spaces/LK/pages/12615682/Standard+opisu+zg+osze+Jira
- ADR: https://bluproject-team.atlassian.net/wiki/spaces/LK/pages/12419074/Decyzje+architektoniczne+ADR
- Jira: https://bluproject-team.atlassian.net/jira/software/c/projects/LK/list?jql=project%20%3D%20LK

## Zasady pracy

1. Nie rozpoczynaj implementacji bez konkretnego Zadania Jira spełniającego Definition of Ready.
2. Przed zmianą przeczytaj issue oraz wszystkie podlinkowane strony Confluence / ADR.
3. Nie zgaduj nowych decyzji produktowych ani przekrojowych decyzji architektonicznych.
4. Jeśli źródła prawdy są sprzeczne, zatrzymaj pracę i eskaluj konflikt do Product Ownera.
5. Dla pracy implementacyjnej obowiązuje domyślnie:

```text
1 atomowe Zadanie Jira
= 1 branch
= 1 logiczny Pull Request
```

6. Branch i PR powinny zawierać klucz Jira, np. `lk-12-booking-hold`.
7. PR musi linkować do właściwego issue Jira.
8. Nie rozszerzaj zakresu PR o niepowiązane refaktoryzacje lub funkcjonalności.
9. Dodawaj / aktualizuj testy odpowiednie do zmiany.
10. Aktualizuj dokumentację techniczną repo, jeśli zmienia się kontrakt, konfiguracja lub sposób uruchomienia.
11. Jeśli implementacja ujawnia nową decyzję przekrojową, nie utrwalaj jej przypadkowo w kodzie — zgłoś potrzebę ADR.

## Pull Request

PR powinien zawierać:

- link do Jira,
- rezultat,
- zakres zmiany,
- wykonane testy,
- ryzyka / kompromisy,
- informacje o ewentualnych zmianach dokumentacji.

Merge do `main` następuje dopiero po:

- spełnieniu kryteriów odbioru Jira,
- zielonym CI,
- wymaganym review,
- rozwiązaniu istotnych uwag review.

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