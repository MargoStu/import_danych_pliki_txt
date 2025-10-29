# import_danych_pliki_txt
import json
import os

PLIK_DANE = "dane.json"
PLIK_HISTORIA = "historia.txt"

def wczytaj_dane():
    if os.path.exists(PLIK_DANE):
        with open(PLIK_DANE, "r", encoding="utf-8") as f:
            dane = json.load(f)
            return dane["konto"], dane["magazyn"]
    else:
        return 0, {}  # domyślne wartości

def zapisz_dane(konto, magazyn):
    with open(PLIK_DANE, "w", encoding="utf-8") as f:
        json.dump({"konto": konto, "magazyn": magazyn}, f, ensure_ascii=False, indent=4)

def wczytaj_historie():
    if os.path.exists(PLIK_HISTORIA):
        with open(PLIK_HISTORIA, "r", encoding="utf-8") as f:
            return [linia.strip() for linia in f.readlines()]
    else:
        return []

def zapisz_historie(historia):
    with open(PLIK_HISTORIA, "a", encoding="utf-8") as f:
        for wpis in historia:
            f.write(wpis + "\n")

konto, magazyn = wczytaj_dane()
historia = wczytaj_historie()

print("=== Program zarządzania kontem i magazynem ===")
print("Dane zostały wczytane z plików.")
print("Saldo konta:", konto, "zł")
print()

while True:
    komenda = input("Podaj komendę: ")

    if komenda == "koniec":
        print("Zamykanie programu. Zapis danych...")

        zapisz_dane(konto, magazyn)
        zapisz_historie(historia)

        print("Dane zapisane do plików.")
        break

    elif komenda == "saldo":
        kwota = input("Podaj kwotę (może być ujemna): ")
        if kwota.startswith("-") and kwota[1:].isdigit():
            konto += int(kwota)
            historia.append("saldo " + kwota)
        elif kwota.isdigit():
            konto += int(kwota)
            historia.append("saldo " + kwota)
        else:
            print("Błąd: Podano nieprawidłową wartość.")

    elif komenda == "sprzedaż":
        produkt = input("Nazwa produktu: ")
        cena = input("Cena sprzedaży: ")
        ilosc = input("Liczba sztuk: ")

        if cena.isdigit() and ilosc.isdigit():
            cena = int(cena)
            ilosc = int(ilosc)

            if produkt in magazyn and magazyn[produkt]["ilosc"] >= ilosc:
                magazyn[produkt]["ilosc"] -= ilosc
                konto += cena * ilosc
                historia.append(f"sprzedaż {produkt} {cena} {ilosc}")
                print(f"Sprzedano {ilosc} szt. produktu {produkt} za {cena * ilosc} zł.")
            else:
                print("Błąd: Produkt nie istnieje lub zbyt mała ilość.")
        else:
            print("Błąd: Cena i ilość muszą być liczbami całkowitymi.")

    elif komenda == "zakup":
        produkt = input("Nazwa produktu: ")
        cena = input("Cena zakupu: ")
        ilosc = input("Liczba sztuk: ")

        if cena.isdigit() and ilosc.isdigit():
            cena = int(cena)
            ilosc = int(ilosc)
            koszt = cena * ilosc

            if konto >= koszt:
                konto -= koszt
                if produkt in magazyn:
                    magazyn[produkt]["ilosc"] += ilosc
                    magazyn[produkt]["cena"] = cena
                else:
                    magazyn[produkt] = {"cena": cena, "ilosc": ilosc}
                historia.append(f"zakup {produkt} {cena} {ilosc}")
                print(f"Zakupiono {ilosc} szt. produktu {produkt} za {koszt} zł.")
            else:
                print("Błąd: Niewystarczające środki na koncie.")
        else:
            print("Błąd: Cena i ilość muszą być liczbami całkowitymi.")

    elif komenda == "konto":
        print("Aktualne saldo konta:", konto, "zł.")

    elif komenda == "lista":
        print("Stan magazynu:")
        if len(magazyn) == 0:
            print("Magazyn jest pusty.")
        else:
            for produkt in magazyn:
                dane = magazyn[produkt]
                print(f"{produkt}: {dane['ilosc']} szt. po {dane['cena']} zł")

    elif komenda == "magazyn":
        produkt = input("Podaj nazwę produktu: ")
        if produkt in magazyn:
            dane = magazyn[produkt]
            print(f"{produkt}: {dane['ilosc']} szt. po {dane['cena']} zł")
        else:
            print("Produkt nie istnieje w magazynie.")

    elif komenda == "przegląd":
        od = input("Podaj indeks początkowy (Enter = od początku): ")
        do = input("Podaj indeks końcowy (Enter = do końca): ")

        if od == "":
            od_index = 0
        elif od.isdigit():
            od_index = int(od)
        else:
            print("Błąd: Nieprawidłowy indeks początkowy.")
            continue

        if do == "":
            do_index = len(historia)
        elif do.isdigit():
            do_index = int(do)
        else:
            print("Błąd: Nieprawidłowy indeks końcowy.")
            continue

        if od_index < 0 or do_index > len(historia) or od_index > do_index:
            print("Błąd zakresu. Liczba operacji:", len(historia))
        else:
            for i in range(od_index, do_index):
                print(f"{i}: {historia[i]}")

    else:
        print("Nieznana komenda.")

