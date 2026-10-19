# Podsumowanie projektu

## Cel

Celem projektu było sprawdzenie, czy dane kliniczne pacjenta mogą pomóc w identyfikacji osób wymagających dalszej diagnostyki kardiologicznej. Pominięcie chorego pacjenta (fałszywie ujemna predykcja) traktujemy jako poważniejszy błąd niż skierowanie zdrowej osoby na dodatkowe badania. Model ma wspierać przesiew i decyzję o dalszej diagnostyce, a nie zastępować lekarza.

## Dane i metoda

Analizę wykonano na przygotowanym zbiorze `heart_disease_cleaned.csv`, obejmującym 919 obserwacji. Oryginalne wartości celu połączono do dwóch klas: brak choroby oraz choroba (`num > 0`). Zbiór podzielono warstwowo na część treningową, walidacyjną i testową w proporcji 60/20/20. Preprocessing, w tym skalowanie cech numerycznych i kodowanie One-Hot cech kategorycznych, jest częścią pipeline'u dopasowywanego wyłącznie podczas treningu.

Porównano regresję logistyczną, drzewo decyzyjne, Random Forest i Gradient Boosting. Hiperparametry strojono przez pięciokrotną walidację krzyżową z priorytetem Recall. Jako model końcowy wybrano Gradient Boosting: uzyskał najwyższy wynik Recall CV, a wśród modeli o zbliżonym Recall miał lepsze F1 CV i niższą zmienność niż regresja logistyczna. Jego parametry to 100 drzew, głębokość 3, `learning_rate = 0.1` i `min_samples_leaf = 1`.

## Wyniki

Na zbiorze testowym przy standardowym progu 0,50 model uzyskał Accuracy 0,875, Precision 0,891, Recall 0,882, F1 0,887, ROC-AUC 0,952 i PR-AUC 0,962.

Pierwszy wariant doboru progu maksymalizował F2 bez dodatkowego ograniczenia i wskazał 0,14. Na walidacji dawał Recall 0,970, ale Precision tylko 0,731, czyli zbyt niski odsetek trafnych wskazań wśród pacjentów kierowanych dalej. Dlatego zastosowano bardziej praktyczne kryterium: maksymalizację Recall przy Precision co najmniej 0,80 na walidacji. Wybrany według tej reguły próg wyniósł 0,30; uzyskał Precision 0,825 i Recall 0,931. Limit Precision jest jawnym założeniem operacyjnym, a nie uniwersalną normą medyczną.

Na teście próg 0,30 dał Accuracy 0,886, Precision 0,872, Recall 0,931, F1 0,900 i F2 0,919; ROC-AUC i PR-AUC pozostały odpowiednio 0,952 i 0,962. Model poprawnie wykrył 95 z 102 chorych i wskazał 14 zdrowych do dalszej diagnostyki. Przy progu 0,50 wykrył 90 chorych, z 12 fałszywie ujemnymi i 11 fałszywie dodatnimi. Zatem próg 0,30 zmniejszył liczbę przeoczonych przypadków o 5, kosztem 3 dodatkowych fałszywych alarmów.

## Rekomendacja i ograniczenia

Do dalszej oceny przyjęto próg 0,30, ponieważ spełnia ustalony na potrzeby projektu warunek Precision ≥ 80%, a spośród takich progów maksymalizuje Recall. Na teście zmniejszył liczbę przeoczonych chorych z 12 do 7 względem progu 0,50, przy wzroście fałszywych alarmów z 11 do 14. Przed praktycznym zastosowaniem limit Precision i akceptowalną liczbę skierowań należy uzgodnić z osobami odpowiedzialnymi za dalszą diagnostykę. Wyniki dotyczą jednego podziału testowego i przygotowanego zbioru; potrzebna jest niezależna walidacja na danych z rzeczywistej populacji oraz ocena kliniczna. Model nie powinien samodzielnie rozpoznawać ani wykluczać choroby.
