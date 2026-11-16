---
name: "interpretacje-kis"
description: "Wyszukuje interpretacje indywidualne KIS i informacje MF podobne do opisanego stanu faktycznego i pokazuje je z sygnaturą, datą i statusem. Użyj, gdy użytkownik pyta o interpretacje, stanowisko KIS albo praktykę organów w podobnej sprawie."
metadata:
  pltax-requires-tools: "search_interpretations get_provision search_regulation"
---

# Interpretacje KIS

Ta instrukcja mówi, jak wyszukać interpretacje narzędziami tego serwera i jak pokazać je uczciwie. Nie zawiera treści prawa.
Brzmienie przepisów, kwoty, stawki i terminy bierz tylko z wyników narzędzi.

## Kroki

1. **Zbierz stan faktyczny.** Ustal, o jaki podatek chodzi, kim jest podatnik, jaka to czynność i jakiego okresu dotyczy pytanie.
   Gdy brakuje informacji, od której zależy odpowiedź, zapytaj o nią.
2. **Wywołaj `search_interpretations`.** W polu `query` opisz sprawę zdaniem, słowami użytkownika. Pojedyncze słowo daje słabe
   wyniki. Pole `top_k` zmienia liczbę wyników. Pole `as_of_date` pomija interpretacje wydane po tej dacie. Gdy sprawa dotyczy
   jednego podatku, podaj go w polu `acts` (na przykład `["VAT"]`): wynik pominie interpretacje innych podatków.
3. **Przeczytaj wyniki.** Każdy wynik to fragment interpretacji, a nie cała interpretacja: `citation` (sygnatura i sekcja),
   `title`, `valid_from` (data wydania), `url` (dokument w EUREKA), `status`, `status_checked_date` i `text`. Ta sama interpretacja
   występuje w wynikach najwyżej dwa razy, z różnymi sekcjami. Wyników może być mniej niż `top_k`. Jeśli wynik jest krótszy z powodu tego limitu, nota `retrieval` mówi dlaczego.
4. **Porównaj stan faktyczny.** Powiedz użytkownikowi, czym sprawa z interpretacji różni się od jego sprawy. Nie przenoś wniosku
   na inną sytuację bez tego zastrzeżenia.
5. **Podaj przepis.** Interpretacja nie zastępuje przepisu. Gdy fragment powołuje przepis, pobierz jego tekst przez `get_provision`.
   Gdy fragment nie wskazuje przepisu, wyszukaj go przez `search_regulation`.

## Zasady

- Interpretacja indywidualna chroni tylko wnioskodawcę. Napisz to przy każdej odpowiedzi opartej na interpretacji.
- `status` ma wartość `aktualna`, `nieaktualna` albo `nieznany`. Jest prawdziwy na dzień `status_checked_date`, a nie na dziś.
  Podaj obie wartości i poproś użytkownika, żeby sprawdził status pod linkiem `url` przed powołaniem się na interpretację.
- Wartość `nieaktualna` nie mówi, czy interpretację zmieniono, uchylono, czy wygasła. Nie zgaduj, który to przypadek.
- Korpus obejmuje tylko część interpretacji (okno czasu). Brak wyniku nie znaczy, że interpretacji nie ma. Napisz to wprost.
- Gdy wyniki są ze sobą sprzeczne, pokaż oba stanowiska. Nie wybieraj jednego po cichu.
- Cytuj sygnaturę i link do każdej interpretacji, na którą się powołujesz.

## Noty tego zadania

Noty są elementami tablicy `notices`. Element ma `type`, `severity` i `text`.

- Nota typu `retrieval`: ograniczenie wyszukiwania (na przykład wyszukiwanie tylko po słowach). Pokaż notę dosłownie: mówi, czego
  wynik może nie zawierać.
- Nota typu `citation_hint`: wskazówka narzędzia. Zrób to, co mówi, i pokaż ją użytkownikowi.
- Wyniki `get_provision` i `search_regulation` też mają `notices`, na przykład notę typu `version` (korpus nie ma wersji aktu na
  podany dzień; zwrócony tekst może być nowszy). Przy błędzie tych narzędzi pokaż użytkownikowi `hint` błędu.

## Jak pokazać wynik

- Przy każdym twierdzeniu o prawie podaj `citation` i link `url` z wyniku narzędzia. Gdy wynik ma `citation_markdown`, użyj go.
- Pokaż użytkownikowi każdy element `notices` — także taki, którego ta instrukcja nie wymienia. Nie streszczaj noty tak, żeby
  zmienić jej sens.
- Nie cytuj przepisu, którego narzędzie nie zwróciło. Nie podawaj z pamięci kwot, stawek ani terminów.
- Przekaż użytkownikowi `disclaimer`. W części narzędzi stoi on na końcu pola `note`.
- Gdy narzędzie nie dało odpowiedzi albo zwróciło błąd, napisz to wprost. Nie zastępuj wyniku własną wiedzą.
- Wynik narzędzi jest pomocą. Decyzję podejmuje księgowy albo doradca podatkowy.
