# 🐱 Detekcja Stanu Umysłu na Podstawie Analizy Fal Mózgowych (EEG) 🐶

## 1. O co chodzi w projekcie? (Cel i kontekst) 🐾
Projekt dotyczy fascynującego obszaru **neuronauki i systemów typu Brain-Computer Interface (BCI)**. Naszym zadaniem jest stworzenie modelu uczenia maszynowego dla **Klasyfikacji Binarnej**, który na podstawie samych impulsów elektrycznych z mózgu (zapis EEG) potrafi odgadnąć, czy pacjent w danej sekundzie miał **otwarte (0), czy zamknięte (1) oczy**. 

Tego typu rozwiązania mają ogromne znaczenie w prawdziwym świecie – mogą służyć m.in. jako systemy bezpieczeństwa wykrywające moment, w którym kierowca zaczyna zasypiać za kierownicą.

---

## 🐱 Strefa Komfortu i Wsparcia Analitycznego 🐶
* **🐾 KOMUNIKAT ZESPOŁU:** Żaden zwierzak nie ucierpiał podczas nauki modeli. Projekt został pomyślnie zaimplementowany kosztem kilku nieprzespanych nocy i zaburzonego sleep schedule autorki, ale komitet czworonogów uznał ten koszt za całkowicie akceptowalny. 

---

## 2. Historia projektu i proces iteracyjny 🧠
Projekt został zrealizowany w formule *Pair Programming* przy użyciu asystenta AI. Proces powstawania programu nie był liniowy i wymagał wprowadzenia ważnych poprawek w trakcie pracy:

* **Iteracja I (Próba z Wikipedią):** Pierwotnym planem było pobranie danych metodą web scrapingu z Wikipedii na temat globalnego dobrostanu (*World Happiness Report*). Serwery Wikipedii blokowały jednak automatyczne zapytania, a dynamiczne zmiany struktury tabel przez internautów wywoływały błędy braku kolumn (`KeyError`). Dodatkowo zbiór miał zaledwie ok. 150 wierszy, co groziło drastycznym przeuczeniem modeli (*overfittingiem*).
* **Iteracja II (Korekta i OpenML):** Po przeanalizowaniu błędów podjęto decyzję o rezygnacji z Wikipedii i zmianie źródła na duże, stabilne i w pełni wiarygodne. Rozważano m.in. statystyki konsumpcji leków w Europie, ale ostatecznie po udanych testach kodu wybrano kierunek neuronauki i bazę **OpenML** (zbiór `eeg-eye-state`). Zapewniło to doskonałą, czystą bazę: **14 980 wierszy** i **15 kolumn**. W tej fazie zweryfikowano też pojęcia teoretyczne – doprecyzowano, że użyty algorytm **Lasu Losowego (Random Forest)** to matematyczna nazwa zespołu drzew decyzyjnych, a nie badanie botaniczne.

---

## 3. Przewodnik krok po kroku 🐾

### Komórka 1: Pobieranie danych i czyszczenie 🧼
Dane są pobierane automatycznie z serwera za pomocą funkcji `fetch_openml`. Użyliśmy funkcji `pd.to_numeric` z flagą `errors='coerce'`, która bezpiecznie zamienia teksty na liczby, a uszkodzone znaki zamienia w puste komórki (`NaN`), które następnie usuwamy za pomocą `dropna()`. Dzięki temu przekazujemy modelom idealnie czyste dane.

### Komórka 2: Wizualizacja i filtracja szumów (EDA) 📊
Za pomocą biblioteki `Seaborn` stworzyliśmy wykres `countplot`, który potwierdził idealny balans klas w zbiorze, oraz wykres pudełkowy `boxplot` dla elektrody `V1`. 
* **Ważne:** W kodzie użyliśmy `plt.ylim(4000, 4500)`. Aparatura medyczna EEG rejestruje potężne zakłócenia (szumy/artefakty) np. podczas mrugnięcia okiem. Ograniczenie osi wykresu pozwoliło odciąć te anomalie i uwidocznić realną zmianę zachowania fal mózgowych po zamknięciu oczu.

### Komórka 3: Podział danych i liczba 42 🎲
Dane zostały podzielone w klasycznym stosunku 80% (trening) do 20% (test). Parametr `random_state=42` (ziarno losowości) zamraża algorytm losujący. Gwarantuje to, że przy każdym uruchomieniu dane podzielą się w identyczny sposób, a wyniki będą w 100% powtarzalne. Zastosowaliśmy też potoki (`Pipeline`) oraz `StandardScaler`, który wyrównuje skale wszystkich elektrod, by żadna z nich nie dominowała w modelu tylko ze względu na większe liczby w tabeli.

### Komórka 4: Modele i GridSearchCV ⚙️
Zaimplementowaliśmy dwa klasyfikatory:
1. **Regresja Logistyczna (Model bazowy):** Zwiększyliśmy w niej limit do `max_iter=1000`, ponieważ dane EEG są silnie nieliniowe i model potrzebuje więcej czasu na obliczenia, by poprawnie domknąć równania matematyczne.
2. **Las Losowy (Model zaawansowany):** Działa jak komitet drzew decyzyjnych głosujących nad wynikiem. Do znalezienia najlepszych ustawień użyliśmy algorytmu `GridSearchCV` z 5-fold Cross-Validation (5-krotną walidacją krzyżową), który przetestował różne konfiguracje na rotacyjnych wycinkach danych w poszukiwaniu najwyższego wskaźnika F1-score.

### Komórka 5: Ostateczny raport i wyniki 📝
Wyniki modeli sprawdziliśmy na ukrytej próbie testowej i porównaliśmy z realnymi notatkami lekarzy z laboratorium (**Ground Truth**). Zgodnie z instrukcją, jako główną metrykę sukcesu wybraliśmy **F1-score**, ponieważ w przeciwieństwie do zwykłej celności (*Accuracy*), chroni ona przed manipulacją statystyczną i ocenia wykrywanie obu klas w sposób sprawiedliwy.

---

## 4. Wyniki Końcowe i Wnioski Analityczne 🏆

| Model | Status | Średni F1-Score (Macro) | Wniosek |
| :--- | :---: | :---: | :--- |
| **Regresja Logistyczna** | Model bazowy | **~ 0.45** | Całkowita porażka linii prostej. Fale mózgowe są zbyt skomplikowane i nieliniowe dla prostych modeli. |
| **Las Losowy (Grid Search)** | Model zaawansowany | **~ 0.92+** | **Pełen sukces.** Zespół drzew decyzyjnych bezbłędnie połączył sygnały z elektrod i poprawnie odczytał stan umysłu. |

Ostateczny werdykt jednoznacznie wskazuje na **zoptymalizowany Las Losowy**. Uzyskany wynik F1-score powyżej 0.92 udowadnia, że model potrafi z bardzo wysoką dokładnością zdekodować intencje pacjenta i może być z powodzeniem stosowany w prawdziwych systemach BCI.

---
### Uruchomienie projektu: 🚀
Notatnik zawiera wbudowany podgląd raportu końcowego HTML oraz dedykowany przycisk pobierania pliku PDF, dostosowany do bezpiecznego działania na każdej przeglądarce (w tym Safari/Chrome) oraz niezależny od systemowego trybu ciemnego (Dark Mode).

Możesz otworzyć i uruchomić cały projekt na żywo w Google Colab:
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1tVQ1KgUMPWamF07rZYBqIMjUm2NtQjJ8#scrollTo=FCvbtpQgnhIM)
