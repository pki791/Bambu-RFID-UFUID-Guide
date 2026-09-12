# Programowanie tagów RFID Bambu Lab na monetach UFUID

Uproszczona instrukcja programowania **zapisywalnych tagów RFID (UFUID)** danymi z oryginalnych szpul Bambu Lab, tak aby drukarka/AMS rozpoznawały je jako oryginalny filament.

To jest skrócona wersja dwóch oryginalnych projektów — jeśli czegoś tu zabraknie, szukaj tam:

- **Zrzuty tagów (dane do zapisu)** — [Bambu-Lab-RFID-Library](https://github.com/queengooborg/Bambu-Lab-RFID-Library) — baza zeskanowanych tagów z oryginalnych szpul, posegregowana wg materiału → wariantu → koloru.
- **Pełna instrukcja zapisu** — [Bambu-Lab-RFID-Tag-Guide → WriteTags.md](https://github.com/queengooborg/Bambu-Lab-RFID-Tag-Guide/blob/main/docs/WriteTags.md) — ta instrukcja jest tego skrótem, ograniczonym tylko do tagów typu **UFUID**.

## Czego potrzebujesz

- Czytnik/programator **Proxmark3** (najlepiej z firmware Iceman).
- Puste, zapisywalne tagi **Gen4 UFUID** z warstwą anti-metal. Testowane egzemplarze: monety UFUID 25 mm / 30 mm z [neven7.eu](https://www.neven7.eu/p/30mm-ufuid-rfid-coin).
- Zrzut (`.bin`) tagu filamentu, który chcesz podrobić — pobrany z [Bambu-Lab-RFID-Library](https://github.com/queengooborg/Bambu-Lab-RFID-Library).

> **Dlaczego akurat UFUID, a nie zwykłe tagi "magic"?**
> Tagi Gen1/Gen2 nie działają z AMS. Tagi **UFUID** zachowują się jak Gen1 (w pełni zapisywalne) *dopóki ich nie "zapieczętujesz"* — a po zapieczętowaniu twardo naśladują oryginalny, jednorazowy tag Bambu, co jest wymagane, żeby AMS je zaakceptował.

## Krok 1 — sprawdź tag

```
hf mf info
```

Upewnij się, że Proxmark3 widzi tag i rozpoznaje go jako magic Gen4 (UFUID).

## Krok 2 — zapisz dane na tag

```
hf mf cload -f /sciezka/do/dump.bin
```

`dump.bin` to plik pobrany z Bambu-Lab-RFID-Library dla konkretnego koloru/wariantu filamentu.

## Krok 3 — zweryfikuj zapis

```
hf mf info
hf mf dump --ns
```

Porównaj wynik z oryginalnym zrzutem — dane powinny się zgadzać.

## Krok 4 — ZAPIECZĘTUJ tag przed włożeniem do AMS (obowiązkowe!)

```
hf 14a raw -a -k -b 7 40
hf 14a raw -k 43
hf 14a raw -k -c e100
hf 14a raw -c 85000000000000000000000000000008
```

Na koniec zweryfikuj:

```
hf mf info
```

Tag nie powinien już odpowiadać na komendy trybu "magic".

## ⚠️ Najważniejsze — dlaczego pieczętowanie jest krytyczne

Z własnych testów: tagi, które **zostają niezapieczętowane** (czyli teoretycznie wciąż w pełni zapisywalne, do wielokrotnego przeprogramowania) są **trwale niszczone przez AMS**. AMS traktuje je jak kartę magic, próbuje coś na nich zapisać/zablokować przy pierwszym użyciu i tag przestaje reagować na cokolwiek — nie da się go już ani odczytać, ani ponownie zaprogramować.

Innymi słowy: **zaleta "dowolnej liczby przeprogramowań" znika w praktyce**, bo AMS i tak bricku­je taki tag przy pierwszym kontakcie, jeśli nie zostanie wcześniej zapieczętowany komendami z Kroku 4. Zapieczętowany tag działa już tylko jak zwykły, jednorazowy tag Bambu (ale za to niezawodnie działa w AMS) — i to jest jedyna sprawdzona, bezpieczna droga.

**Zasada: zapisz → od razu zapieczętuj → dopiero wtedy włóż do AMS.** Nie testuj niezapieczętowanego tagu w AMS "na próbę".

## Źródła i podziękowania

Cała ciężka robota (skanowanie tagów, reverse-engineering formatu, pełna dokumentacja Proxmark3) to zasługa projektu [queengooborg/Bambu-Lab-RFID-Tag-Guide](https://github.com/queengooborg/Bambu-Lab-RFID-Tag-Guide) i [queengooborg/Bambu-Lab-RFID-Library](https://github.com/queengooborg/Bambu-Lab-RFID-Library). Ten dokument to tylko skrót "do roboty" dla konkretnego, przetestowanego typu tagów (UFUID).

Tagi testowane w praktyce: [25 mm i 30 mm UFUID RFID coin, anti-metal — neven7.eu](https://www.neven7.eu/p/30mm-ufuid-rfid-coin).
