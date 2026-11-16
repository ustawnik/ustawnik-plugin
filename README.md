# Ustawnik

[![mcptry](https://mcptry.com/badge/com.ustawnik/ustawnik)](https://mcptry.com/servers/com.ustawnik/ustawnik)

Podatki, VAT, KSeF i księgowość: polskie prawo podatkowe i rachunkowe w oryginalnym brzmieniu, także w wersji obowiązującej
w wybranym dniu. Każda odpowiedź ma cytat i link do źródła.

Z Ustawnikiem Claude pracuje na tekście polskich przepisów podatkowych i rachunkowych. Odpowiedzi na pytania o podatek VAT,
podatki dochodowe CIT i PIT, rachunkowość, Ordynację podatkową i KSeF zawierają dokładny przepis, datę stanu prawnego i link
do oficjalnego źródła.

Zakres: ustawa o VAT, ustawy o CIT i PIT, ustawa o rachunkowości, Ordynacja podatkowa, rozporządzenia w sprawie KSeF,
dokumentacja KSeF 2.0 ze strukturą faktury FA(3) i interpretacje podatkowe KIS.

Co Claude może zrobić z Ustawnikiem:

- przytoczyć przepis słowo w słowo, w brzmieniu obowiązującym w wybranym dniu;
- przeszukać ustawy podatkowe, dokumentację KSeF 2.0, strukturę faktury FA(3) i interpretacje podatkowe KIS (EUREKA);
- sprawdzić, czy faktura jest zgodna ze strukturą FA(3) i art. 106e ustawy o VAT, zanim trafi do KSeF;
- policzyć terminy ustawowe, na przykład na złożenie JPK_V7 (JPK VAT) i zapłatę VAT, z uwzględnieniem weekendów i świąt.

Ustawnik powstał dla biur rachunkowych, które prowadzą księgowość klientów. Nie udziela porad podatkowych: każda odpowiedź ma źródło i datę, a decyzję podejmuje
księgowy. Wszystkie narzędzia tylko odczytują dane. Korzystanie jest bezpłatne, z dziennym limitem wywołań na konto.

## Co zawiera plugin

- **Konektor** do serwera Ustawnika pod adresem `https://mcp.ustawnik.com/mcp`.
- **Trzy skille**, czyli instrukcje dla Claude: jak korzystać z narzędzi Ustawnika i jak pokazać wynik.
  - `przepis-na-dzien`: dosłowne brzmienie przepisu i stan prawny, z którego pochodzi tekst;
  - `weryfikacja-faktury-ksef`: kontrola faktury sprzedaży względem struktury FA(3) i art. 106e ustawy o VAT, z podstawą prawną
    każdego ustalenia;
  - `interpretacje-kis`: interpretacje KIS podobne do opisanej sprawy, z sygnaturą, datą i statusem.

Skille nie zawierają treści prawa. Brzmienie przepisów, kwoty, stawki i terminy pochodzą tylko z odpowiedzi serwera.

## Jak zacząć

1. Zainstaluj plugin i połącz konektor Ustawnik. Na claude.ai zrobisz to na karcie **Connectors** pluginu, a w Claude Code
   poleceniem `/mcp`.
2. Zaloguj się kontem Google. Konto w Ustawniku powstaje przy pierwszym logowaniu.
3. Zadaj pytanie, na przykład: „Przytocz art. 113 ust. 1 ustawy o VAT w aktualnym brzmieniu.” albo „Do kiedy muszę złożyć JPK_V7M
   i zapłacić VAT za marzec 2027?”

## Jakie dane wysyła plugin

Plugin nie uruchamia żadnego programu na Twoim komputerze. Łączy się tylko z serwerem `mcp.ustawnik.com`. Claude wysyła do serwera
argumenty narzędzia: pytanie, numer przepisu, datę, okres albo pola faktury. Nie wpisuj w pytania danych osobowych swoich klientów:
do kontroli faktury wystarczą kwoty, daty i stawki. Logowanie prowadzi w naszym imieniu serwis Auth0 (firma Okta). Serwer Ustawnika
nie dostaje Twojego adresu e-mail. Co serwer zapisuje i jak długo, opisuje
[polityka prywatności](https://ustawnik.com/polityka-prywatnosci).

## Licencja i warunki

Pliki tego repozytorium są na licencji MIT (plik `LICENSE`). Korzystanie z serwera Ustawnika reguluje
[regulamin](https://ustawnik.com/regulamin). Strona: https://ustawnik.com. Kontakt: info@ustawnik.com.

## In English

Ustawnik gives Claude the text of Polish tax and accounting law, quoted word for word with the legal-state date and a link to the
official source. It is built for accounting and bookkeeping offices in Poland and does not give tax advice. The plugin bundles the Ustawnik remote MCP
server (`https://mcp.ustawnik.com/mcp`, OAuth sign-in with a Google account) and three skills written in Polish: the wording of
a provision on a given date, an invoice check against the FA(3) structure and Art. 106e of the VAT Act, and KIS tax rulings. The
plugin runs nothing on your computer. Claude sends the server only tool arguments: a question, a provision number, a date,
a period or invoice fields. All tools are read-only. Free, with a daily limit of calls per account. Privacy policy:
https://ustawnik.com/polityka-prywatnosci.
