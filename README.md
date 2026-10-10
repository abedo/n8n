  ---

🚀 1. Szybki start i dostęp do panelu

1. Otwórz w przeglądarce adres: http://localhost:5678 (http://localhost:5678)
2. Przy pierwszym uruchomieniu n8n poprosi Cię o założenie lokalnego konta administratora (e-mail + hasło) oraz ustawienie podstawowych preferencji. Dane te są zapisywane w lokalnym wolumenie Dockera
   (n8n_data).

  ---

🧠 2. Kluczowe pojęcia w n8n

* Nodes (Węzły): Podstawowe klocki budulcowe. Każdy węzeł wykonuje określoną akcję (pobranie danych, przekształcenie, wysyłka wiadomości, integracja z API).
* Triggery (Wyzwalacze): Węzły rozpoczynające przepływ – np. kliknięcie przycisku (Manual Trigger), harmonogram czasowy (Schedule Trigger), lub zapytanie przychodzące (Webhook).
* Data Flow (Przepływ danych): Dane przekazywane są pomiędzy węzłami w formacie obiektów JSON:
  1   [ { "json": { "klucz": "wartość" } } ]
* Executions (Uruchomienia): Historia wszystkich historii uruchomień wraz z widokiem podglądu danych wejściowych/wyjściowych na każdym etapie.

  ---

🛠️ 3. Przykład praktyczny: Pobieranie i formatowanie kursów walut NBP

Przygotowałem dla Ciebie prosty, ale bardzo praktyczny workflow, który:
1. Uruchamia się na żądanie (ręcznie).
2. Pobiera aktualne kursy walut z darmowego API Narodowego Banku Polskiego.
3. Za pomocą węzła Code (JavaScript) filtruje najpopularniejsze waluty (USD, EUR, GBP, CHF) i generuje sformatowany raport.

Opcja A: Szybki import gotowego pliku JSON

W katalogu projektu został utworzony plik z gotowym przepływem:  
📂 /home/wojkaz/Sites/abedo/n8n/example-workflow.json

Aby go zaimportować:
1. W n8n w prawym górnym rogu kliknij ikony menu ... (Trzy kropki) → Import from File.
2. Wybierz plik example-workflow.json z dysku (lub otwórz go i wklej zawartość bezpośrednio na płótno w n8n za pomocą Ctrl+V).
3. Kliknij przycisk Execute Workflow na dole ekranu.

  ---

Opcja B: Wyklikanie workflow krok po kroku (Najlepsze do nauki)

Jeśli chcesz przejść krok po kroku własnoręcznie:

1. Stwórz nowy workflow:
    * Kliknij Create Workflow (lub + w lewym menu).

2. Krok 1: Dodaj Wyzwalacz (Trigger):
    * Kliknij Add first step...
    * Wyszukaj i wybierz On clicking 'Test workflow' (węzeł Manual Trigger).

3. Krok 2: Pobranie danych z API (HTTP Request):
    * Najedź na prawą krawędź węzła wyzwalacza i kliknij + aby połączyć kolejny krok.
    * Wyszukaj węzeł HTTP Request.
    * Skonfiguruj parametry:
        * Method: GET
        * URL: https://api.nbp.pl/api/exchangerates/tables/A/?format=json
    * Kliknij Test step, aby upewnić się, że dane z NBP pobierają się w formacie JSON.

4. Krok 3: Przetworzenie i filtrowanie danych (Code):
    * Dodaj kolejny węzeł klikając + za węzłem HTTP Request.
    * Wybierz węzeł Code (JavaScript).
    * Wklej poniższy kod w polu edytora kodu:

    1    // Pobieramy dane z poprzedniego węzła (API NBP)
    2    const tableData = $input.first().json;
    3    const rates = tableData.rates || [];
    4
    5    // Interesujące nas waluty
    6    const targets = ['USD', 'EUR', 'GBP', 'CHF'];
    7    const selectedRates = rates.filter(r => targets.includes(r.code));
    8
    9    // Formatujemy czytelny raport
10    const date = tableData.effectiveDate;
11    let report = `📊 **RAPORT KURSU WALUT NBP (${date})**\n\n`;
12
13    selectedRates.forEach(r => {
14      report += `• **${r.code}** (${r.currency}): ${r.mid.toFixed(4)} PLN\n`;
15    });
16
17    return {
18      data_publikacji: date,
19      pobranych_walut: rates.length,
20      wybrane_waluty: selectedRates,
21      raport_tekstowy: report
22    };

5. Krok 4: Przeprowadzenie testu:
    * Kliknij Execute Workflow na dole ekranu.
    * Po zakończeniu kliknij na węzeł Code i sprawdź zakładkę Output (zobaczysz wygenerowany raport oraz sformatowane dane JSON).

  ---

💡 Propozycje kolejnych kroków do poćwiczenia:

1. Automatyzacja czasowa: Zastąp węzeł Manual Trigger węzłem Schedule Trigger (np. uruchamianie w każdy dzień roboczy o 9:00).
2. Powiadomienie: Dodaj na końcu węzeł integracji, np. Telegram, Discord, Slack lub Email (Send Email), aby wygenerowany raport wysyłał się automatycznie.
3. Zapis do pliku/bazy: Dodaj węzeł Write Binary File lub połączenie do bazy danych (np. MySQL / PostgreSQL / SQLite).

  ---

🔧 Komendy do zarządzania Dockerem:

* Zatrzymanie n8n:  
  docker compose -f /home/wojkaz/Sites/abedo/n8n/docker-compose.yml stop
* Uruchomienie ponowne:  
  docker compose -f /home/wojkaz/Sites/abedo/n8n/docker-compose.yml up -d
* Podgląd logów:  
  docker compose -f /home/wojkaz/Sites/abedo/n8n/docker-compose.yml logs -f
