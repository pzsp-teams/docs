# Dokumentacja dla eksperta od oprogramowania PZSP2 – Zespół 1 – teams

## Repozytorium kodu z dostępem online

Utworzona została organizacja w serwisie GitHub, w skład której wchodzą 3 repozytoria:

- biblioteka ułatwiająca komunikację z API MS Teams [lib](https://github.com/pzsp-teams/lib)
- binding biblioteki w pythonie [lib-python](https://github.com/pzsp-teams/lib-python)
- aplikacja terminalowa prezentująca użycie biblioteki [cli](https://github.com/pzsp-teams/cli)

## Pipeline'y CI/CD

**CI na lib**
- **Lint** - statyczna analiza kodu przy pomocy golangci-lint
- **Go Mod Tidy** - sprawdzenie, czy z plikach obsługujących pakiety nie ma nadmiarowych dependencji
- **Test** - kiedy przejdą poprzednie sekcje, uruchamiane są testy jednostkowe
- **Vulnerability Check** - analiza pod kątem znanych podatności

![CI pipeline](./CI.png)

**Generowanie i publikowanie dokumentacji na lib oraz lib-python**
- **Build Docs** - wygenerowanie dokumentacji przez MkDocs
- **Publish** - deployment strony z dokumentacją

## Narzędzia formatowania i analizy statycznej

Używane narzędzie to **golangci-lint**. Jego konfiguracja znajduje się w pliku *.golangci.yml*. CI uruchamia wiele różnych, zewnętrznych linterów (np. gocritic, gocyclo etc.) i na podstawie ich właściwości ocenia poprawności kodu. Do formatowania wykorzystane zostały wykorzystane narzędzia domyślna języka go - **gofmt** i **goimports**.

## Metodyka tworzenia kodu

Na githubie tworzone są issues, które są przydzielane jako zadania do konkretnych osób. Do każdego feature tworzony jest osobny branch. Po wstępnym zakończeniu pracy nad funkcjonalnością tworzony jest Pull Request. Aby sfinalizować Pull Request wymagane zaakceptowanie przez przynajmniej jednego innego maintainera. Jeśli reviewer negatywnie oceni jakość powstałego kodu, zostawia komentarze, do których musi odnieść się autor kodu.

## Wybór języka

Do wykonania projektu został wybrany język **Go**. Go posiada bardzo bogaty ekosystem do aplikacji typu CLI np. *cobra*, *bubbletea*. W języku tym można również korzystać gotowych rozwiązań Microsoftu takich jak MSAL (biblioteka do autoryzacji) oraz MSGraphSDK. System do zarządzania dependencjami jest bardzo dobry i lepszy niż w językach typu Python. Innym kryterium wyboru była chęć nauczenia się nowego języka, który ma dobrą reputację.

## Propozycje testów akceptacyjnych

### Test Akceptacyjny: Wysyłanie wiadomości na kanały Teams

**Scenariusz:** Wysłanie spersonalizowanej wiadomości na wybrane kanały zespołu

---

#### Pliki testowe

**Plik: message.txt**

```
Drogi ${zespol},

Przypominam o spotkaniu ${data} o godzinie ${godzina}.

Temat spotkania: ${temat}

Prosimy o przygotowanie:
${zadanie}

Pozdrawiam,
Kierownik Projektu
```

**Plik: variables.json**

```json
{
  "Ogólny": {
    "zespol": "Zespole",
    "data": "1 grudnia 2025",
    "godzina": "10:00",
    "temat": "Spotkanie ogólne Q4",
    "zadanie": "Raport z postępów"
  },
  "Projekty": {
    "zespol": "Zespole projektowy",
    "data": "1 grudnia 2025",
    "godzina": "11:00",
    "temat": "Review projektów",
    "zadanie": "Prezentacja projektu"
  },
  "Kampanie": {
    "zespol": "Zespole marketingowy",
    "data": "1 grudnia 2025",
    "godzina": "14:00",
    "temat": "Podsumowanie kampanii",
    "zadanie": "Analiza wyników"
  }
}
```

---

#### Kroki testowe

##### Krok 1: Przygotowanie plików

**Akcja użytkownika:**

- Użytkownik tworzy plik `message.txt` z szablonem wiadomości zawierającym placeholdery
- Użytkownik tworzy plik `variables.json` z wartościami dla każdego kanału

##### Krok 2: Wykonanie komendy

**Akcja użytkownika:**

```bash
teams-automation send --team "Marketing Team" --template message.txt --variables variables.json
```

##### Krok 3: Przetwarzanie przez system

**Akcja systemu:**

- System odczytuje szablon z pliku `message.txt`
- System odczytuje zmienne dla każdego kanału z pliku `variables.json`
- System podstawia odpowiednie zmienne dla każdego kanału
- System wysyła wiadomości do kanałów podanych w pliku `variables.json`

##### Krok 4: Otrzymanie potwierdzenia

**Wyświetlony wynik:**

```
Zespół: Marketing Team
Szablon: message.txt
Kanały: 3

Wysłano: 3/3
```

---

#### Kryteria akceptacji

- System podstawia różne wartości zmiennych w wiadomościach dla każdego kanału
- Każdy kanał wymieniony w pliku `variables.json` otrzymuje spersonalizowaną wiadomość
- Komunikat końcowy informuje o liczbie pomyślnie wysłanych wiadomości

### Test Akceptacyjny: Pobieranie nieodczytanych wiadomości Teams

**Scenariusz:** Pobranie nieodczytanych wiadomości użytkownika

---

#### Kroki testowe

##### Krok 1: Wykonanie komendy

**Akcja użytkownika:**

```bash
teams-automation unread
```

##### Krok 2: Przetwarzanie przez system

**Akcja systemu:**

- System pobiera listę nieodczytanych wiadomości użytkownika
- System wyświetla listę nieodczytanych wiadomości

##### Krok 3: Otrzymanie potwierdzenia

**Wyświetlony wynik:**

```
Nieodczytane wiadomości:

  Marketing Team > Ogólny
  Od: Jan Kowalski
  Data: 2025-11-27 09:30
  Treść: Przypominam o spotkaniu dzisiaj o 14:00. Proszę o przygotowanie raportów.
  
  Development Team > Projekty
  Od: Anna Nowak
  Data: 2025-11-27 10:15
  Treść: Merge request #234 czeka na review. Czy możesz sprawdzić?

Pobrano: 2 nieodczytane wiadomości
```

---

#### Kryteria akceptacji

- System pobiera wszystkie nieodczytane wiadomości użytkownika
- System wyświetla szczegóły każdej wiadomości
- System wyświetla treść wiadomości w formie
- Komunikat końcowy potwierdza poprawne pobranie listy wiadomości

### Test Akceptacyjny: Automatyczne tworzenie kanałów dla grup projektowych

**Scenariusz:** Automatyczne generowanie numerowanych kanałów i przypisywanie do nich zdefiniowanych grup użytkowników

---

#### Pliki testowe

**Plik: groups_config.json**

```json
[
  {
    "id": 1,
    "members": [
      "jan.kowalski@firma.com",
      "anna.nowak@firma.com",
      "piotr.wisniewski@firma.com",
      "maria.wojcik@firma.com"
    ]
  },
  {
    "id": 2,
    "members": [
      "krzysztof.krawczyk@firma.com",
      "agnieszka.kaminska@firma.com",
      "tomasz.lewandowski@firma.com",
      "ewa.zielinska@firma.com"
    ]
  },
  {
    "id": 3,
    "members": [
      "marcin.szymanski@firma.com",
      "monika.wozniak@firma.com",
      "adam.dabrowski@firma.com",
      "natalia.kozlowska@firma.com"
    ]
  }
]
```

---

#### Kroki testowe

##### Krok 1: Przygotowanie plików

**Akcja użytkownika:**

- Użytkownik tworzy plik `groups_config.json` definiujący listę obiektów (grup)
- Każdy obiekt zawiera listę adresów e-mail użytkowników Teams, którzy mają znaleźć się w jednej grupie

##### Krok 2: Wykonanie komendy

**Akcja użytkownika:**

```bash
teams-automation create-groups --team "Hackathon 2025" --config groups_config.json --base-name "Zespół Projektowy"
```

##### Krok 3: Przetwarzanie przez system

**Akcja systemu:**

- System odczytuje konfigurację z pliku `groups_config.json`
- System generuje nazwę kanału łącząc podaną nazwę bazową (--base-name) z kolejnym numerem (np. "Zespół Projektowy 1")
- System tworzy nowy kanał dla każdej grupy zdefiniowanej w pliku
- System dodaje użytkowników z listy members do odpowiadającego im, nowo utworzonego kanału

##### Krok 4: Otrzymanie potwierdzenia

**Wyświetlony wynik:**

```
Zespół docelowy: Hackathon 2025
Nazwa bazowa: "Zespół Projektowy"
Zdefiniowane grupy: 3

[OK] Utworzono kanał: "Zespół Projektowy 1"
     Dodano członków: 4

[OK] Utworzono kanał: "Zespół Projektowy 2"
     Dodano członków: 4

[OK] Utworzono kanał: "Zespół Projektowy 3"
     Dodano członków: 4

Status: Operacja zakończona pomyślnie
```

---

#### Kryteria akceptacji

- System automatycznie numeruje kanały dodając licznik do nazwy bazowej (np. #1, #2)
- Do każdego numerowanego kanału trafiają wyłącznie użytkownicy zdefiniowani dla tej konkretnej grupy w pliku JSON
- Liczba utworzonych kanałów odpowiada liczbie obiektów w pliku konfiguracyjnym
- Komunikat końcowy potwierdza pomyślne utworzenie wszystkich kanałów
