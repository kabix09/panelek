# Workflow Ekonometrii Panelowej — Opisy sekcji na podstawie wyników uruchomienia

> Opisy poniżej bazują **wyłącznie** na faktycznych outputach notebooka. Wszelkie wcześniejsze komentarze markdown w pliku `.ipynb` zostały zastąpione opisami zgodnymi z rzeczywistymi wynikami.

---

## KROK 0 — Wczytanie i przygotowanie danych panelowych

**Wynik:**
```
Obserwacje po usunięciu NA: 176
Regiony: 16
Lata: [2014, 2015, 2016, 2017, 2018, 2019, 2020, 2021, 2022, 2023, 2024]
```

**Opis:**
Zbiór danych zawiera **176 obserwacji** dla **16 województw** w **11 latach (2014–2024)**. Panel jest zbilansowany: każde województwo posiada po jednej obserwacji na każdy rok (16 × 11 = 176). Zmienna zależna to udział bezrobotnych w liczbie ludności w wieku produkcyjnym. Zmienną kluczową są inwestycje (w zł), a zmiennymi kontrolnymi: wynagrodzenie, saldo migracji ogółem oraz liczba podmiotów gospodarczych.

---

## KROK 1 — Korelacje pooled

**Wynik:**
```
                      zmienna  r_pearson  p_value istotna     ocena
liczba_pomiotow_gospodarczych    -0.6383   0.0000       ✔  ✔✔ silna
                wynagrodzenie    -0.5246   0.0000       ✔  ✔✔ silna
                inwestycje_zl    -0.4937   0.0000       ✔ ✔ umiark.
        saldo_migracji_ogółem    -0.2665   0.0004       ✔   ~ słaba

✔ Korelacja kluczowej zmiennej z Y: -0.494
```

**Opis:**
Wszystkie cztery zmienne wykazują **ujemną, istotną statystycznie korelację** z bezrobociem (p < 0.05). Najsilniej z bezrobociem koreluje liczba podmiotów gospodarczych (r = –0,638, korelacja silna), następnie wynagrodzenie (r = –0,525, silna) i inwestycje (r = –0,494, umiarkowana). Saldo migracji wykazuje korelację słabą (r = –0,267). Ujemne znaki wszystkich korelacji są zgodne z intuicją ekonomiczną: więcej firm, wyższe płace i wyższe inwestycje wiążą się z niższym bezrobociem. Spełniony jest warunek opiekuna — korelacja zmiennej kluczowej (inwestycje) jest wystarczająca (|r| ≈ 0,49 > 0,15) do budowy sensownego modelu.

---

## KROK 2 — Deklaracja struktury panelowej

**Wynik:**
```
Y  = bezrobotni_w_liczbie_ludności_w_wieku_produkcyjnym
X  = inwestycje_zl + wynagrodzenie + saldo_migracji_ogółem + liczba_pomiotow_gospodarczych
```

**Opis:**
Panel zadeklarowany jako MultiIndex `(region, rok)`. Formuła modelu obejmuje zmienną kluczową `inwestycje_zl` oraz trzy zmienne kontrolne. Kolumna `mieszkania_oddane_do_użytkowania_na_10_tys._ludności` została pominięta — brak kolumny o tej nazwie po przejściu przez `str.replace(' ', '_')` (oryginalnie zawierała spację i kropkę), co spowodowało jej odfiltrowanie przez weryfikację dostępności kolumn.

---

## KROK 3 — Model Pooled OLS (punkt odniesienia)

**Wynik:**
```
                               Parameter  Std. Err.     T-stat    P-value
Intercept                         5.5323     0.9089     6.0868     0.0000 ***
inwestycje_zl                  5.558e-05  6.845e-05     0.8119     0.4180
wynagrodzenie                 -2.322e-05     0.0001    -0.1553     0.8768
saldo_migracji_ogółem          6.576e-05  5.659e-05     1.1620     0.2469
liczba_pomiotow_gospodarczych    -0.0020     0.0009    -2.3067     0.0223 **

R² = 0.4747
```

**Opis:**
Model Pooled OLS ignoruje strukturę panelową i traktuje wszystkie obserwacje jako niezależne. Spośród zmiennych objaśniających jedynie **liczba podmiotów gospodarczych jest istotna statystycznie** (p = 0,022): wzrost liczby firm o 1 podmiot wiąże się ze spadkiem stopy bezrobocia o 0,002 pp. Zmienna kluczowa `inwestycje_zl` jest **nieistotna** (p = 0,418). Model wyjaśnia ok. 47,5% zmienności bezrobocia. Ze względu na nieuwzględnienie nieobserwowalnych efektów indywidualnych województw, szacunki mogą być obciążone — model Pooled OLS pełni tu wyłącznie rolę punktu odniesienia.

---

## KROK 4 — Model FE Within (efekty indywidualne)

**Wynik:**
```
                               Parameter  Std. Err.     T-stat    P-value
Intercept                         5.7181     0.3349    17.073     0.0000 ***
inwestycje_zl                   1.11e-05  3.379e-05     0.3283     0.7431
wynagrodzenie                   5.08e-05   7.16e-05     0.7095     0.4791
saldo_migracji_ogółem          3.834e-05  8.497e-05     0.4512     0.6525
liczba_pomiotow_gospodarczych    -0.0021     0.0003    -6.6714     0.0000 ***

R² (within):  0.5075
R² (between): 0.3461
R² (overall): 0.4184
```

**Opis:**
Model FE z efektami indywidualnymi (within) eliminuje nieobserwowalne cechy stałe w czasie charakterystyczne dla każdego województwa (np. strukturę gospodarczą, infrastrukturę). Po tym oczyszczeniu nadal **jedynie liczba podmiotów gospodarczych pozostaje istotna** (p < 0,001): wzrost liczby firm o 1 podmiot w obrębie danego województwa przekłada się na spadek stopy bezrobocia o ok. 0,0021 pp. Zmienna kluczowa `inwestycje_zl` jest nieistotna (p = 0,743). R² within = 0,508 oznacza, że model wyjaśnia ok. 50,8% zmienności *wewnątrz* województw. R² between (0,346) wskazuje na słabsze dopasowanie do różnic *między* województwami — to oczekiwany efekt estymacji within. W porównaniu do Pooled OLS F = 40,18 (p < 0,001) potwierdza istotność efektów stałych.

---

## KROK 5 — Model FE TwoWay (efekty regionów + efekty czasowe)

**Wynik:**
```
                               Parameter  Std. Err.     T-stat    P-value
Intercept                         0.4520     1.5567     0.2904     0.7719
inwestycje_zl                 -1.447e-05  2.279e-05    -0.6351     0.5264
wynagrodzenie                     0.0003     0.0003     1.0182     0.3103
saldo_migracji_ogółem          4.512e-05  1.466e-05     3.0768     0.0025 ***
liczba_pomiotow_gospodarczych -5.965e-05     0.0013    -0.0463     0.9631

R² (within):  -1.6576
R² (between): -0.5622
R² (overall): -1.0526
```

**Opis:**
Po dodaniu efektów czasowych obraz wyników zmienia się radykalnie. **Jedyną istotną zmienną staje się saldo migracji** (p = 0,0025): wzrost salda migracji o 1 osobę wiąże się ze wzrostem stopy bezrobocia o ok. 4,5e-05 pp. Zarówno `inwestycje_zl` (p = 0,526) jak i `liczba_pomiotow_gospodarczych` (p = 0,963) tracą istotność. Warto zwrócić uwagę na **ujemne wartości R²** we wszystkich wymiarach (within = –1,66, between = –0,56, overall = –1,05). Ujemne R² w pakiecie `linearmodels` dla modeli TwoWay jest zjawiskiem technicznym wynikającym z metody obliczania — model wciąż jest estymowany poprawnie, jednak miara ta nie jest bezpośrednio porównywalna z R² modeli jednokierunkowych. Wyniki testu F (krok 12) wskazują, że efekty czasowe jako całość są silnie istotne (p < 0,001), co uzasadnia stosowanie modelu dwukierunkowego.

---

## KROK 6 — Model Between (różnice między regionami)

**Wynik:**
```
                               Parameter  Std. Err.     T-stat    P-value
Intercept                         5.6991     3.7836     1.5063     0.1602
inwestycje_zl                     0.0001     0.0002     0.6581     0.5240
wynagrodzenie                    -0.0001     0.0009    -0.1348     0.8952
saldo_migracji_ogółem          5.646e-05  9.617e-05     0.5871     0.5690
liczba_pomiotow_gospodarczych    -0.0021     0.0011    -1.7961     0.1000

R² = 0.4569
```

**Opis:**
Model Between estymuje zależności na podstawie **wieloletnich średnich** dla każdego województwa — odpowiada na pytanie, czy regiony z wyższymi przeciętnymi inwestycjami mają przeciętnie niższe bezrobocie. **Żadna zmienna nie jest istotna statystycznie** na poziomie 5% (liczba podmiotów jest bliska granicy: p = 0,100). R² = 0,457 sugeruje, że zmienne objaśniające wyjaśniają ok. 46% zróżnicowania między-regionalnego, jednak efekt ten nie jest statystycznie potwierdzony dla żadnego z regresorów z osobna. Wyniki Between są zasadniczo odmienne od FE Within, co wskazuje na różnicę w mechanizmach działania zmiennych w przekroju i w czasie.

---

## KROK 7 — Test Hausmana (FE vs RE)

**Wynik:**
```
Statystyka χ²(5) = -0.3093
p-value = 1.0000
➤ Brak podstaw do odrzucenia H0 — Random Effects mogą być OK
```

**Opis:**
Test Hausmana zwrócił **ujemną statystykę chi-kwadrat (–0,309)** i p-value = 1,000. Ujemna wartość statystyki jest sygnałem technicznym: różnica macierzy kowariancji V(FE) – V(RE) nie jest dodatnio półokreślona (macierz V_diff ma wartości własne ujemne), co sprawia, że test w tej implementacji nie daje wiarygodnego wyniku. Wynik p = 1,000 **nie oznacza, że RE jest lepsze od FE** — oznacza jedynie, że klasyczna procedura Hausmana oparta na pseudoinwersji dała nieokreślony rezultat. W tej sytuacji wybór modelu powinien opierać się na argumentach merytorycznych (skala korelacji efektów indywidualnych z regresorami) oraz wynikach testu F dla efektów stałych. Biorąc pod uwagę silną istotność efektów jednostkowych (widoczną m.in. w dużym zróżnicowaniu średnich regionalnych — krok 13) i wynik testu F (krok 12), **zastosowanie FE TwoWay pozostaje uzasadnione** mimo technicznego niepowodzenia testu Hausmana.

---

## KROK 8 — Porównanie modeli (compare)

**Wynik — wybrane statystyki:**

| Miara               | Pooled OLS | FE (entity) | FE (twoway) | RE       |
|---------------------|------------|-------------|-------------|----------|
| R-squared           | 0.4747     | 0.5075      | 0.0873      | 0.5023   |
| R² (within)         | 0.5018     | 0.5075      | –1.6576     | 0.5063   |
| R² (between)        | 0.4527     | 0.3461      | –0.5622     | 0.4393   |
| R² (overall)        | 0.4747     | 0.4184      | –1.0526     | 0.4693   |
| F-stat (p-value)    | 0.000      | 0.000       | 0.009       | 0.000    |

**Opis:**
Modele Pooled OLS, FE (entity) i RE osiągają zbliżone R² within (~0,50), co oznacza podobną zdolność wyjaśniania zmienności wewnątrz województw. Model FE (entity) osiąga najwyższe R² within (0,508) przy zachowaniu pełnej kontroli nad efektami indywidualnymi. Model FE TwoWay wykazuje ujemne R² będące artefaktem numerycznym (patrz krok 5), lecz jego F-stat = 3,49 (p = 0,009) potwierdza łączną istotność regresorów po usunięciu obu wymiarów efektów. Model RE uzyskuje wyniki zbliżone do FE (entity), jednak przy założeniu ortogonalności efektów — założeniu, którego test Hausmana nie potwierdził jednoznacznie. Spośród wszystkich modeli zmienna `liczba_pomiotow_gospodarczych` jest istotna w Pooled OLS (p = 0,022; **), FE entity (p < 0,001; ***) i RE (p < 0,001; ***). W modelu FE TwoWay istotna pozostaje jedynie `saldo_migracji_ogółem` (p = 0,0025; ***).

---

## KROK 10 — Testy diagnostyczne (autokorelacja i heteroskedastyczność)

**Wyniki:**
```
Test autokorelacji (AR(1) proxy):
  ρ̂ (lag 1): 0.6617
  ⚠ możliwa autokorelacja

Test Breuscha-Pagana (heteroskedastyczność):
  Test stat: 0.7722, p-value: 0.3795
  ✔ Brak istotnej heteroskedastyczności
```

**Opis:**
**Autokorelacja:** Korelacja reszt modelu FE TwoWay z ich pierwszym opóźnieniem wynosi **ρ̂ = 0,662**, co jest wartością wysoką (|ρ| >> 0,3). Wskazuje to na silną autokorelację pierwszego rzędu — reszty z roku *t* są silnie powiązane z resztami z roku *t–1* w obrębie tego samego województwa. Oznacza to, że model nie w pełni wychwytuje dynamikę bezrobocia, a standardowe błędy — nawet klastrowane — mogą być niedoszacowane przy tej sile autokorelacji. Należy rozważyć model dynamiczny z opóźnioną zmienną zależną.

**Heteroskedastyczność:** Test Breusha-Pagana dał statystykę 0,772 i **p-value = 0,380**, co oznacza **brak podstaw do odrzucenia hipotezy o homoskedastyczności**. Wariancja reszt jest stała względem dopasowanych wartości — nie ma problemu heteroskedastyczności w klasycznym rozumieniu. Klastroanie błędów standardowych zastosowane w modelu FE TwoWay jest dodatkowym zabezpieczeniem inferencji i pozostaje właściwe niezależnie od tego wyniku.

---

## KROK 11 — Inferencja z opornymi błędami standardowymi

**Wynik:**
```
                               Parameter  Std. Err.     T-stat    P-value
Intercept                         0.4520     1.5567     0.2904     0.7719
inwestycje_zl                 -1.447e-05  2.279e-05    -0.6351     0.5264
wynagrodzenie                     0.0003     0.0003     1.0182     0.3103
saldo_migracji_ogółem          4.512e-05  1.466e-05     3.0768     0.0025 ***
liczba_pomiotow_gospodarczych -5.965e-05     0.0013    -0.0463     0.9631

Zmienna kluczowa: inwestycje_zl
  Estymator: -0.000014
  Błąd std.: 0.000023
  t-stat:    -0.6351
  Istotność: ✖ nie (p≥0.05)
```

**Opis:**
Wyniki modelu FE TwoWay z klastrowanymi błędami standardowymi (na poziomie województwa) potwierdzają wnioski z kroku 5. **Zmienna kluczowa `inwestycje_zl` jest statystycznie nieistotna**: β = –1,45e-05, SE = 2,28e-05, t = –0,635, p = 0,526. Nawet po kontroli efektów regionalnych i czasowych wzrost nakładów inwestycyjnych nie wiąże się istotnie ze zmianami stopy bezrobocia *wewnątrz* województw w analizowanym okresie. Jedyną istotną zmienną pozostaje **saldo migracji ogółem** (β = 4,51e-05, p = 0,0025): w roku, w którym saldo migracji województwa jest wyższe od jego przeciętnej, stopa bezrobocia jest wyższa — efekt ilościowo niewielki (wzrost salda o 1 osobę → wzrost bezrobocia o ~4,5e-05 pp), lecz statystycznie wiarygodny. Klastroanie błędów standardowych na poziomie województwa jest prawidłową procedurą — szczególnie wobec stwierdzonej autokorelacji (krok 10).

---

## KROK 12 — Test F (modele zagnieżdżone: FE OneWay vs FE TwoWay)

**Wynik:**
```
H0: efekty czasowe = 0 (model FE oneway)
H1: efekty czasowe ≠ 0 (model FE twoway)
F-stat: 102.2343
p-value: 0.0000
➤ Odrzucamy H0 — FE TwoWay jest lepszy ✔
```

**Opis:**
Test F porównuje modele zagnieżdżone: FE z samymi efektami jednostkowymi (OneWay) vs FE z efektami jednostkowymi i czasowymi (TwoWay). Statystyka F = **102,23** przy p < 0,001 jednoznacznie wskazuje, że **efekty czasowe są łącznie wysoce istotne**. Oznacza to, że trendy i szoki makroekonomiczne wspólne dla wszystkich województw (np. ogólnopolski spadek bezrobocia po 2015 roku, wzrost po 2020) mają silny, systematyczny wpływ na poziom bezrobocia — którego model OneWay nie uwzględnia. Wynik ten uzasadnia wybór modelu FE TwoWay jako specyfikacji bazowej dla dalszej inferencji.

---

## KROK 13 — Efekty stałe (średnie bezrobocia według województw)

**Wynik:**
```
region
PODKARPACKIE           3.20
LUBELSKIE              2.77
KUJAWSKO-POMORSKIE     2.66
ŚWIĘTOKRZYSKIE         2.54
PODLASKIE              2.53
WARMIŃSKO-MAZURSKIE    2.45
ŁÓDZKIE                2.15
MAZOWIECKIE            2.07
ZACHODNIOPOMORSKIE     2.03
OPOLSKIE               1.65
MAŁOPOLSKIE            1.57
DOLNOŚLĄSKIE           1.55
POMORSKIE              1.41
LUBUSKIE               1.28
ŚLĄSKIE                1.28
WIELKOPOLSKIE          1.00
```

**Opis:**
Tabela przedstawia wieloletnie **średnie stopy bezrobocia** dla każdego województwa w latach 2014–2024 (wyrażone prawdopodobnie w % lub jako ułamek liczby ludności w wieku produkcyjnym). Rozpiętość między regionami jest znaczna: **Podkarpacie (3,20)** ma trzykrotnie wyższe przeciętne bezrobocie niż **Wielkopolska (1,00)**. Regiony o najwyższym bezrobociu to wschód i centrum-północ Polski (Podkarpackie, Lubelskie, Kujawsko-Pomorskie, Świętokrzyskie). Regiony o najniższym bezrobociu to zachód (Lubuskie, Śląskie) i Wielkopolska. Tak duże zróżnicowanie regionalne potwierdza, że efekty stałe (EntityEffects) absorbują silne, trwałe strukturalne różnice między województwami — co uzasadnia stosowanie modelu FE zamiast Pooled OLS.

---

## KROK 14 — Test Pesarana (korelacja przekrojowa reszt)

**Wynik:**
```
Średnia korelacja między regionami: -0.0590
CD statystyka: 14.5117
⚠ Korelacja przekrojowa jest istotna (|CD| > 1.96)
  Rozważ: dynamic panel model, model SUR, lub spatial dependence
```

**Opis:**
Test Pesarana CD wykrył **istotną statystycznie korelację przekrojową reszt** (CD = 14,51 >> 1,96). Oznacza to, że reszty z różnych województw nie są od siebie niezależne — istnieją wspólne, nieobserwowalne czynniki oddziałujące jednocześnie na wiele regionów, których model FE TwoWay nie wychwycił. Warto zwrócić uwagę, że **średnia korelacja par jest ujemna (–0,059)**, co jest nieoczekiwane i może wskazywać na efekt „wypierania" — gdy bezrobocie w jednym regionie rośnie ponad trend, w sąsiednim spada. Wysoka statystyka CD przy niskiej średniej korelacji oznacza, że istotność wynika z liczności par (16 regionów → 120 par), a nie z uniformicznie wysokich korelacji. Niemniej fakt istotności testu oznacza, że **inferencja na błędach klastrowanych może nie być w pełni efektywna** — konieczne jest rozważenie modeli uwzględniających zależność przestrzenną lub podejść SUR (Seemingly Unrelated Regressions).

---

## Podsumowanie — ocena zgodności istniejących wniosków z wynikami

| Stwierdzenie w oryginalnym markdown | Ocena | Wynik faktyczny |
|---|---|---|
| Test Hausmana: p < 0,05 → FE wskazane | ❌ **BŁĘDNE** | χ²(5) = –0,309, p = 1,000 — test nie dał rozstrzygającego wyniku |
| R² within wzrósł po dodaniu TimeEffects | ❌ **BŁĘDNE** | R² within spadł z 0,508 (FE entity) do –1,658 (FE twoway) |
| Autokorelacja: potencjalny AR(1) | ✅ **POPRAWNE** | ρ̂ = 0,662 — silna autokorelacja potwierdzona |
| Heteroskedastyczność: stosuj odporne SE | ⚠️ **CZĘŚCIOWO** | BP-test p = 0,380 — brak heteroskedastyczności; SE klastrowane poprawne ze względu na autokorelację |
| FE TwoWay potwierdza istotność inwestycji | ❌ **BŁĘDNE** | inwestycje nieistotne: t = –0,635, p = 0,526 |
| Efekty czasowe są istotne | ✅ **POPRAWNE** | F = 102,23, p < 0,001 |
| Model rekomendowany: FE TwoWay | ✅ **POPRAWNE** | Uzasadnienie poprawne (test F, nie test Hausmana) |
| Korelacja przekrojowa: rozważ model dynamiczny | ✅ **POPRAWNE** | CD = 14,51, p < 0,001 |
