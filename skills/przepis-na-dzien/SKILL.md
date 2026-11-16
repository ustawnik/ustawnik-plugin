---
name: "przepis-na-dzien"
description: "Podaje dosłowne brzmienie przepisu polskiego prawa podatkowego albo rachunkowego i mówi, z jakiego stanu prawnego pochodzi zwrócony tekst. Użyj, gdy użytkownik cytuje przepis (art., ust., §), pyta o brzmienie przepisu, albo pyta, co obowiązywało w dniu zdarzenia (data faktury, data dostawy, okres rozliczeniowy)."
metadata:
  pltax-requires-tools: "get_provision search_regulation"
---

# Przepis na dzień

Ta instrukcja mówi, jak pobrać dosłowny tekst przepisu narzędziami tego serwera i jak uczciwie powiedzieć, z jakiego stanu prawnego
on pochodzi. Nie zawiera treści prawa. Brzmienie przepisów, kwoty, stawki i terminy bierz tylko z wyników narzędzi.

## Kroki

1. **Ustal datę zdarzenia.** Gdy pytanie dotyczy konkretnego zdarzenia (data faktury, data dostawy, okres rozliczeniowy), podaj tę
   datę jako `as_of_date` w postaci `RRRR-MM-DD`. Gdy odpowiedź zależy od daty, a użytkownik jej nie podał, zapytaj o nią. Gdy
   użytkownik pyta o obecne brzmienie, pomiń `as_of_date`: narzędzie zwróci najnowszą wersję.
2. **Pytanie cytuje przepis → `get_provision`.** Pole `act` to kod aktu (na przykład `VAT`, `UoR`, `OP`, `PIT`, `CIT`), jego nazwa
   albo ELI. Pole `unit` to jedna jednostka w zapisie ludzkim (`art. 113 ust. 1`, `art. 12 § 5`) albo jej klucz (`art:113/ust:1`).
   Zakres albo lista jednostek to osobne wywołania, po jednym na jednostkę. Punkt podaj w `unit` („art. 106e ust. 1 pkt 5”); jeden
   ciągły zakres punktów („pkt 7–12”) też. Lista punktów („pkt 3 i 5”) to osobne wywołania, po jednym na punkt.
3. **Pytanie tematyczne → `search_regulation`.** Podaj `query`, w polu `sources` wartość `akt` (same akty prawne), a gdy pytanie
   dotyczy jednego aktu, także `acts`. Podaj ten sam `as_of_date`. Potem wywołaj `get_provision`, żeby dostać pełny, dosłowny tekst
   jednostki: w polu `unit` podaj `unit_key` wybranego wyniku, a w polu `act` nazwę aktu z jego `citation`.
4. **Sprawdź, co wróciło.** Porównaj `citation` wyniku z tym, o co pytał użytkownik. Podaj `legal_status_date`: to jest stan prawny
   zwróconego tekstu.

## Jak czytać wynik

- `text` w `get_provision` to pełny tekst jednostki albo — gdy wynik ma `lead_in` — tylko cytowany punkt. W `search_regulation` zwykły wynik rankingu może być tylko fragmentem
  jednostki, a wynik dokładny bywa ucięty i ma wtedy `text_truncated`. Pełny tekst zawsze zwraca `get_provision`.
- Cytat z punktem albo literą: `text` to sam punkt (z literą — cały punkt z jej literami), `lead_in` to wprowadzenie wyliczenia,
  a `closing` — gdy jest — część wspólna po ostatnim punkcie, która dotyczy także tego punktu. Pokaż punkt razem z `lead_in`
  i `closing`: bez nich punkt może zmienić sens. `citation` i `unit_key` wskazują ustęp, a `requested` mówi, o który punkt proszono:
  podpisz tekst przez `citation_markdown` i `requested`.
- Punktu albo litery, których nie ma w jednostce, serwer nie zastępuje ustępem: odpowiedzią jest błąd `UNIT_NOT_FOUND` (zobacz niżej).
- `unit_status` o wartości `uchylony` albo `pominięty` znaczy, że jednostka nie jest obowiązującym przepisem. Napisz to wprost.
  Nie podstawiaj innego przepisu.
- W `search_regulation` pole `match` mówi, skąd jest wynik: `exact_citation` (dokładnie cytowana jednostka), `exact_citation_parent`
  (jednostka nadrzędna, z polem `requested`) albo `search` (zwykły wynik rankingu). Wynik `search` nie jest potwierdzeniem, że to
  ten przepis, o który pytał użytkownik.
- `children` w `get_provision` to lista jednostek podrzędnych. Użyj jej, gdy użytkownik chce konkretny ustęp artykułu.

## Reguła uczciwości o stanie prawnym

Korpus może nie mieć wersji aktu, która obowiązywała w dniu zdarzenia. Wynik ma wtedy w `notices` notę typu `version`, a zwrócony
tekst to najstarsza wersja w korpusie. Taki tekst może być nowszy niż stan prawny w dniu zdarzenia.

- Pokaż notę `version` dosłownie i podaj `legal_status_date`.
- Napisz wprost, że nie potwierdzasz brzmienia przepisu na dzień zdarzenia.
- Nie pisz, że w dniu zdarzenia przepis miał zwrócone brzmienie, gdy wynik ma notę `version`.
- Zakres wersji w korpusie bierz z wyniku (nota `version`, `legal_status_date`) albo z instrukcji serwera i zasobu `pltax://korpus`.
  Nie podawaj go z pamięci: zmienia się po każdej aktualizacji korpusu.

## Błędy `get_provision`

Błąd ma `code`, `message`, `hint` i `retryable`. Pokaż użytkownikowi `message` i `hint`.

- `ACT_UNKNOWN`: aktu nie ma w korpusie pod tą nazwą. `hint` tego błędu podaje dozwolone kody. Nie ponawiaj z tą samą wartością.
- `UNIT_NOT_FOUND`: serwer nie ma tej jednostki albo jednostka nie ma cytowanego punktu lub litery. `hint` tego błędu podaje powód
  i co zrobić (lista jednostek podrzędnych, lista punktów albo liter jednostki, brak wersji aktu w indeksie albo akt dostępny tylko
  przez wyszukiwanie). Nie zgaduj innej jednostki ani innego punktu: pokaż użytkownikowi `hint` błędu.
- `INVALID_INPUT`: pole `unit` nie jest jedną jednostką albo wskazuje listę punktów. Rozbij zakres albo listę na osobne wywołania;
  `hint` tego błędu podaje gotowe brzmienia.
- `DATE_OUT_OF_RANGE`: data nie istnieje w kalendarzu albo leży zbyt daleko w przyszłości. Popraw datę albo pomiń `as_of_date`.
- `INDEX_OUTDATED`: serwer nie może sprawdzić jednostek. Użyj `search_regulation` i powiedz użytkownikowi, że tekst pochodzi
  z wyszukiwania, a nie z dokładnego odczytu jednostki.
- `INTERNAL`: błąd serwera. Ponów tylko wtedy, gdy `retryable` ma wartość `true`. W innym przypadku przekaż użytkownikowi
  `correlation_id`.

## Noty tego zadania

Noty są elementami tablicy `notices`. Element ma `type`, `severity` i `text`; gdy dotyczy jednego wyniku `search_regulation`,
ma też `applies_to` z jego pozycją w `results`.

- Nota typu `version`: korpus nie ma wersji aktu na `as_of_date`. Zobacz „Reguła uczciwości o stanie prawnym”.
- Nota typu `act_text`: tekst aktu może nie uwzględniać części zmian albo akt wchodzi w życie etapami. Pokaż każdą.
- Nota typu `point_ambiguous` w `get_provision`: serwer nie umiał wyznaczyć cytowanego punktu bez zgadywania i zwrócił cały tekst
  jednostki. Pokaż notę z jej powodem. Nie wskazuj punktu w tekście samodzielnie: powiedz, że punkt trzeba odczytać w akcie.
- Nota typu `enumeration_closing` w `get_provision`: po ostatnim punkcie jest tekst, którego serwer nie umie przypisać — może
  dotyczyć także zwróconego punktu. Pokaż notę i zrób to, co mówi (pobierz całą jednostkę).
- Nota typu `citation_hint` w `search_regulation`: cytat z zapytania nie dał dopasowania dokładnego albo tekst jest ucięty. Zrób
  to, co mówi nota (na przykład dodaj nazwę aktu), i pokaż ją użytkownikowi.
- Pozostałe noty (na przykład o wyszukiwaniu tylko po słowach) pokaż dosłownie: mówią, czego wynik może nie zawierać.

## Jak pokazać wynik

- Przy każdym twierdzeniu o prawie podaj `citation` i link `url` z wyniku narzędzia. Gdy wynik ma `citation_markdown`, użyj go.
- Pokaż użytkownikowi każdy element `notices` — także taki, którego ta instrukcja nie wymienia. Nie streszczaj noty tak, żeby
  zmienić jej sens.
- Nie cytuj przepisu, którego narzędzie nie zwróciło. Nie podawaj z pamięci kwot, stawek ani terminów.
- Przekaż użytkownikowi `disclaimer`. W części narzędzi stoi on na końcu pola `note`.
- Gdy narzędzie nie dało odpowiedzi albo zwróciło błąd, napisz to wprost. Nie zastępuj wyniku własną wiedzą.
- Wynik narzędzi jest pomocą. Decyzję podejmuje księgowy albo doradca podatkowy.
