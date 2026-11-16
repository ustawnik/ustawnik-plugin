---
name: "weryfikacja-faktury-ksef"
description: "Sprawdza fakturę sprzedaży względem schemy FA(3) i art. 106e ustawy o VAT i podaje podstawę prawną każdego ustalenia. Użyj, gdy użytkownik prosi o sprawdzenie, weryfikację albo kontrolę faktury, pyta, czy faktura jest poprawna pod KSeF, albo podaje dane faktury (NIP, daty, stawki, kwoty)."
metadata:
  pltax-requires-tools: "check_invoice_against_ksef_rules get_provision search_regulation"
---

# Weryfikacja faktury pod KSeF

Ta instrukcja mówi, jak sprawdzić fakturę sprzedaży narzędziami tego serwera i jak pokazać wynik. Nie zawiera treści prawa.
Brzmienie przepisów, kwoty, stawki i terminy bierz tylko z wyników narzędzi.

## Kroki

1. **Zbuduj wejście.** Narzędzie `check_invoice_against_ksef_rules` przyjmuje pole `invoice`: uproszczony JSON w nazewnictwie FA(3),
   na przykład `RodzajFaktury`, `P_1`, `P_2`, `P_6`, `KodWaluty`, `Podmiot1`, `Podmiot2`, `FaWiersz`, `P_15`. Pełną listę pól podaje
   schemat wejścia narzędzia. Przepisz dane, które podał użytkownik. Nie uzupełniaj brakujących pól własnymi wartościami: brak pola ma zostać
   ustaleniem narzędzia. Narzędzie nie przyjmuje pliku XML.
2. **Ustal datę.** Bez `as_of_date` narzędzie bierze datę wystawienia z `P_1`. Podaj `as_of_date` tylko wtedy, gdy użytkownik chce
   ocenić fakturę według stanu prawnego z innego dnia.
3. **Wywołaj `check_invoice_against_ksef_rules`** — jedno wywołanie na jedną fakturę.
4. **Przeczytaj werdykt.** Pole `verdict` ma wartość `errors`, `warnings` albo `ok`. Pole `summary` podaje liczby ustaleń.
   Każdy element `findings` ma `severity`, `field` i `message`. Ustalenie ma też podstawę: `legal_basis` (przepis) albo
   `schema_citation` (element schemy FA(3); jego opis jest w `schema_doc`). Wyjątek: ostrzeżenie o polu nieobsługiwanym nie ma
   żadnej podstawy.
5. **Pokaż podstawę ustalenia, gdy ją ma.** Podstawą ustalenia jest `legal_basis` albo `schema_citation`. Pole `context` pokazuje
   brzmienie przepisów, które cytuje `legal_basis`: są to jednostki redakcyjne pobrane po kluczu, a nie wynik wyszukiwania. Pole `for`
   mówi, do której podstawy należy element. `context` nie ocenia, czy faktura jest zgodna z przepisem.
6. **Punkt w ustępie.** Gdy podstawa cytuje punkt, element `context` ma `match` o wartości `exact_citation_parent`: `citation`
   i `unit_key` wskazują ustęp, `requested` — cytowany punkt, a `excerpt` to tylko ten punkt. Pole `lead_in` to wprowadzenie
   wyliczenia: pokaż je razem z punktem, bo bez niego punkt może zmienić sens. Podstawa z listą punktów daje osobny element na każdy
   punkt, z tym samym `for`. Podpisz tekst punktu przez `citation_markdown` i `requested`.
7. **Pełny tekst.** Gdy `excerpt_truncated` albo `lead_in_truncated` ma wartość `true`, wywołaj `get_provision`: w polu `unit` podaj
   `requested` elementu (punkt; wtedy `text` to punkt, a `lead_in` to pełne wprowadzenie) albo `unit_key` (cała jednostka), a w polu
   `act` akt z cytatu (na przykład `VAT`).
8. **Szerszy kontekst.** Gdy użytkownik pyta o coś więcej niż ustalenie (na przykład o wyjątki od przepisu), użyj
   `search_regulation`. Nie odpowiadaj na takie pytanie z pamięci.

## Jak czytać ustalenia

- Ustalenie `error` albo `warning` z podstawą w ustawie ma element w `context`, chyba że `notices` ma notę typu `basis_context`:
  `context` nie pokazuje wtedy części podstaw. Ustalenie, które ma tylko `schema_citation`, nie ma elementu w `context`. Ustalenia
  `info` też go nie mają.
- Ostrzeżenie „Pole nieobsługiwane przez narzędzie” dotyczy nazwy pola na wejściu. Powiedz użytkownikowi, że narzędzie tego pola
  nie sprawdziło. To nie jest stwierdzenie błędu na fakturze.
- Ustalenia `info` nie zmieniają pola `verdict`.
- Wartość `ok` w polu `verdict` znaczy tylko, że wykonane kontrole nie znalazły problemu. Nie znaczy, że KSeF przyjmie fakturę.
  Napisz to użytkownikowi.

## Czego narzędzie nie sprawdza

Wymień w odpowiedzi te pozycje, które dotyczą faktury użytkownika:

- faktury korygujące i zaliczkowe (poza samą wartością pola `RodzajFaktury`),
- dostawy wewnątrzwspólnotowe i eksport,
- mechanizm podzielonej płatności,
- waluty obce i kursy,
- procedura marży i załączniki,
- plik XML i schema XSD (narzędzie nie zastępuje walidacji KSeF),
- to, czy NIP istnieje i czy podmiot jest czynnym podatnikiem (narzędzie liczy tylko sumę kontrolną).

## Noty tego zadania

Noty są elementami tablicy `notices`. Element ma `type`, `severity` i `text`; gdy dotyczy elementu `context`, ma też `applies_to`
z jego pozycją w `context`.

- Nota typu `version` przy elementach `context`: korpus nie ma wersji aktu na datę faktury i narzędzie zwróciło najstarszą wersję.
  Pokaż notę dosłownie. Napisz, że tekst w `excerpt` może być nowszy niż stan prawny w dniu wystawienia faktury.
- Nota typu `act_text` przy elementach `context`: tekst aktu może nie uwzględniać części zmian. Pokaż każdą.
- Nota typu `point_ambiguous` przy elemencie `context`: serwer nie umiał wyznaczyć punktu bez zgadywania i `excerpt` zaczyna się od
  początku jednostki. Pokaż notę z jej powodem. Nie wskazuj punktu w tekście samodzielnie.
- Nota typu `enumeration_closing` przy elemencie `context`: wyliczenie kończy się częścią wspólną, która dotyczy także tego punktu,
  a `excerpt` jej nie zawiera, albo serwer nie umie ustalić, czy tekst po ostatnim punkcie jest częścią wspólną. Pokaż notę i wywołaj
  `get_provision` z brzmieniem, które podaje nota: dla ustalonej części wspólnej wynik ma ją w polu `closing`, a w drugim przypadku
  nota każe pobrać całą jednostkę.
- Nota typu `basis_context`: części podstaw prawnych nie udało się pokazać albo jednostek jest więcej, niż mieści `context`. Nota
  mówi, czego brakuje i dlaczego. Pokaż ją dosłownie. Gdy nota mówi, że jednostki pominięto z powodu limitu, podaje tylko ich liczbę:
  brakujące są pola `legal_basis` z `findings`, które nie występują w żadnym polu `for`, oraz punkty z listy podstawy (na przykład
  „pkt 3 i 5”), dla których nie ma elementu z takim punktem w `requested`. Pobierz je przez `get_provision`.
  Gdy nota podaje inny powód (na przykład akt spoza korpusu), napisz użytkownikowi, że tekstu tej podstawy serwer nie ma.
- `note`: zastrzeżenie o zakresie kontroli razem z tekstem `disclaimer`. To nie jest element `notices`.
- Wyniki `get_provision` i `search_regulation` też mają `notices`. Przy błędzie tych narzędzi pokaż użytkownikowi `hint` błędu.

## Jak pokazać wynik

- Przy każdym twierdzeniu o prawie podaj `citation` i link `url` z wyniku narzędzia. Gdy wynik ma `citation_markdown`, użyj go.
- Pokaż użytkownikowi każdy element `notices` — także taki, którego ta instrukcja nie wymienia. Nie streszczaj noty tak, żeby
  zmienić jej sens.
- Nie cytuj przepisu, którego narzędzie nie zwróciło. Nie podawaj z pamięci kwot, stawek ani terminów.
- Przekaż użytkownikowi `disclaimer`. W części narzędzi stoi on na końcu pola `note`.
- Gdy narzędzie nie dało odpowiedzi albo zwróciło błąd, napisz to wprost. Nie zastępuj wyniku własną wiedzą.
- Wynik narzędzi jest pomocą. Decyzję podejmuje księgowy albo doradca podatkowy.
