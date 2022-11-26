# Laboratorium numer 1

Uruchomienie przykładowego projektu

CI/CD - Ciągła integracja oraz ciągłe wdrażanie

## Przed laboratorium - utworzenie środowiska deweloperskiego

Ćwiczenie można wykonywać na dwa sposoby:

1. Używając swojego prywatnego komputera - sugerowane rozwiązanie
2. Używając VDI (Sugeruje użycie Ubuntu 20.04)

Opcja 1:
Upewnij się, ze masz zainstalowane następujące aplikacje:

- [git](https://git-scm.com/downloads) oraz konto na github - może być to konto z e-mailem uczelni
- python 3 [python 3.10](https://www.python.org/downloads/)
- terminal do wykonywania poleceń CLI (może być z IDE)
- IDE:
  - [PyCharm](https://www.jetbrains.com/pycharm/download/)
  - [VSCode](https://code.visualstudio.com/Download)
- [poetry](https://python-poetry.org/docs/#installing-with-the-official-installer)

Opcjonalnie:
[Ustawienia prywatności dla adresu email w GitHub](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address)

Opcja 2:
VDI - sprawdź czy wszystkie aplikacje (git, python3, IDE, poetry) sa dostępne w systemie do użycia

## Zadanie 1 - przygotowanie repozytorium do laboratorium

Wykonaj [fork](https://docs.github.com/en/get-started/quickstart/fork-a-repo)
repozytorium do swojego przestrzeń użytkownika na GitHubie

Czym jest fork repozytorium?

## Zadanie 2 - Uruchomienie projektu

Uruchomienie aplikacji lokalnie w celach deweloperskich

- Sklonuj repozytorium na swój komputer
- Przejdź do katalogu `example-app`
- Upewnij się, ze poetry działa prawidłowo `poetry debug info`
- Wykonaj polecenie `poetry install` w celu instalacji zależności projektowych
- Uruchom projekt poleceniem `poetry run task server`
- Przejdź do adresu `http://localhost:5000`
- Czy projekt się uruchomił? Wnioski?
- Przetestuj pozostałe możliwe zadania (sekcja `tasks`) do uruchomienia dostępne w pliku [pyproject.toml](../example-app/pyproject.toml)

## Zadanie 3 - Tworzenie środowiska ciągłej integracji

Zadanie ma na celu stworzenie środowiska wewnątrz GitHub Actions, które będzie weryfikowało poprawność kodu pod względem:

- statycznej analizy kodu
- możliwych problemów bezpieczeństwa
- testów jednostkowych

- Wejdź w fork (wykonany w pkt1) na GitHubie
- Przejdź do zakładki "actions"

![Actions](/assets/github-actions.png)

- Wyszukaj ze zbioru gotowych rozwiązań workflow "python application" i stwórz go
Pytanie 1: W jakim katalogu github sugeruje tworzenie konfiguracji do uruchamiania CI/CD?
Pytanie 2: Czy jest to ta sama metoda instalowania zależności?

- Testowana aplikacja wymaga instalacji `poetry` do kontekstu Github-Actions zatem:
W zakładce Marketplace po prawej stronie edytora wyszukaj snippet (fragment gotowego kodu)

![Poetry Action](/assets/poetry.png)

Zastosuj się do podanej dokumentacji

- Spróbuj uzyskać podobny efekt do wykonywania tych samych kroków manualnie na swoim komputerze

```yaml

jobs:
  some-name:

    runs-on: some-os

    defaults:
      run:
        working-directory: ./example-app

    steps:
      ...
```

Wykorzystaj podany wycinek w swoim rozwiązaniu
Alternatywnie: Za każdym razem twoje rozwiązanie musi przejść do katalogu `example-app`

Pytania na koniec:

- Ile czasu github-actions runnera sumarycznie zużyłeś podczas tworzenia środowiska CI?
- Jaki jest limit miesięczny minut przydzielony w planie darmowym dla dewelopera?

## Zadanie 4 - tworzenie matrycy/ciągów zadań ciągłej integracji

Modyfikacja środowiska CI by stworzyć ciąg zdarzeń

Wykorzystując poprzednio stworzony pipeline zmodyfikuj go tak, by pewne zadania mogły wykonywać sie równolegle:

- osobno budowanie zależności: instalacja poetry, instalacja zależności projektowych
- osobno testy statyczne kodu: linter, formatter, security
- osobno testy jednostkowe

Przykładowy docelowy wygląd środowiska

![Pipeline](/assets/pipe.png)

Efekt końcowy powinien wykorzystywać:

- mechanizm grupowania:

```yaml
    concurrency:
      group: some-group-name
```

- mechanizm pamięci podręcznej:

```yaml
    - name: Cache build venv
      id: cached-poetry-dependencies
      uses: actions/cache@v3
      with:
        path: /home/runner/work/<path>/example-app/.venv
        key: ${{ runner.os }}-venv
```

- element ustalania kolejności

```yaml
needs: some-other-job
```

Pytania:

- Jaka jest różnica między `job` a `stage`?
- Dlaczego wykorzystujemy mechanizm `cache` zamiast duplikować cały kod?

## Zadanie 5 - Testowanie różnych systemów operacyjnych

Dodaj testowanie w środowisku bazującym na systemach windows dla poprzednio zdefiniowanych zadań

Do celu tego ćwiczenia będzie potrzebne polecenie `matrix`

```yaml
matrix:
        os: [ "ubuntu-latest", "windows-latest" ]
```

Przykładowy wygląd środowiska ciągłej integracji po modyfikacji:

![Martix](/assets/martix.png)

W jakim celu budowanie kodu jest przydatne?

Zanotuj koszty wykorzystania innych systemów operacyjnych do testowania

## Zadanie 6 - Wdrażanie kodu w sposób ciągły (CD)

Uruchamianie aplikacji w kontekście PaaS

- Wejdź na stronę dostawcy platformy jako serwis (PaaS): [render.com](https://render.com/)
- Załóż konto z wykorzystaniem konta GitHuba
- Połącz swoje repozytorium do tworzonego stosu w kontekście PaaS
- Uruchom wdrożenie aplikacji z planu znajdującego się w pliku: [render.yaml](../render.yaml)

Pytanie: Czy operacja zakończyła się powodzeniem?

### Dodawanie bazy danych w wersji produkcyjnej

- Dodaj bazę danych [PostgreSQL](https://dashboard.render.com/new/database) do serwisu
- Otrzymane informacje o utworzonej bazie zastosuj dla środowiska aplikacji webowej utworzonej na portalu
- Wykonaj ponowne wdrożenie kodu, by wprowadzone zmiany zostały rozpropagowane po środowisku

Pytanie: Czy modyfikacje github actions są potrzebne, by uzyskać ciągle wdrażanie w tym przypadku?
