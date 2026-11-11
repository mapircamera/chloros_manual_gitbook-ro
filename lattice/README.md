# Camere LATTICE

LATTICE este sistemul modular de camere multispectrale al MAPIR destinat imagisticii agricole și științifice. Fiecare cameră LATTICE este construită pe baza senzorului Sony IMX265 cu obturator global (**3,1 MP, pixeli de 3,45 µm**) și se conectează prin Ethernet ca dispozitiv**GigE Vision**.

Chloros 1.3.0 controlează camerele LATTICE în timp real — descoperire, previzualizare live, captură și matrice sincronizate cu mai multe camere — de pe trei interfețe:

| Interfață    | Unde                                                          | Platforme                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Fila **Camere** din bara laterală Chloros                         | Windows 10/11 x64                                        |
| CLI        | Familia de comenzi `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Căutați componentele hardware?**Modulele de cameră, obiectivele, filtrele și benzile, ramele și sistemele de montare, cablurile, PoE și cablajul de declanșare sunt documentate în [**manualul de utilizare LATTICE**](https://mapir.gitbook.io/lattice-camera). Acest capitol tratează controlul camerelor din Chloros.

Capturile LATTICE sunt fișiere standard de tip `.tif`/`.tiff`, iar Chloros le procesează întotdeauna pornind de la captura brută. Consultați [Referința CLI](../reference/cli-reference.md) și [Referința SDK](../reference/sdk-reference.md) pentru lista completă de comenzi și interfața API.

## Două configurații ale senzorilor

| Configurație | Senzor       | Filtru                                | Ce oferă o cameră                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Culoare Bayer | filtru triplu de bandă trecătoare                |**Trei benzi calibrate dintr-o singură expunere**                 |
| **M3M**| Monocrom   | un singur filtru de interferență cu bandă îngustă |**O bandă calibrată**; combinați mai multe camere M3M pentru indici |

Deoarece o cameră M3M este monocromă în spatele unui singur filtru, fiecare bandă beneficiază de propria expunere. O cameră M3C acoperă toate cele trei benzi ale sale cu o singură expunere a senzorului.

## Șiruri de model și denumire

Fiecare cameră își stochează identitatea în GenICam `DeviceUserID` sub forma unui șir de model:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros o afișează cu prefixul `LATT-` (de exemplu, `LATT-M3M-L87-F450`). Același șir `LATT-…` este scris în eticheta EXIF `Model` a fiecărui export și este utilizat ca nume al folderului de ieșire al camerei în proiectele procesate.

| Componentă | Valori                                                   | Semnificație                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Senzor    | `M3C` / `M3M`                                            | Culoare Bayer / monocrom                                                                          |
| Obiectiv      | `L41` / `L87`                                            | Numărul reprezintă **câmpul vizual orizontal în grade**: L41 = îngust (41°), L87 = larg (87°)    |
| Filtru    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) sau `F<nm>` (M3M) | A se vedea [Filtre și benzi spectrale](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Șirul de caractere al modelului determină tot ce urmează: Chloros determină profilul senzorului, dispunerea benzilor și calibrarea din fabrică pe baza valorilor `DeviceUserID` + `DeviceSerialNumber`. Nu este necesară nicio configurare pentru fiecare cameră în parte — consultați [Conectarea camerelor](connecting.md).

## Filtre și benzi

Centrele benzilor, marginile FWHM și catalogul complet M3M cu 23 de SKU-uri sunt specificații ale produsului, așa că se regăsesc în manualul hardware: [**Filtre și benzi spectrale**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Ce contează din punct de vedere software: codul filtrului din șirul modelului determină ce produse poate construi Chloros. Camerele cu filtru RGB (`FRGB`) emit numai produse debayered și de previzualizare — radianța și reflectanța pe bandă nu sunt relevante pentru un senzor de bandă largă, așa că Chloros le omite și menționează acest lucru. Orice alt filtru generează lanțul complet radianță → reflectanță → indice.

## Calibrarea radiometrică pe scurt

Fiecare cameră LATTICE este calibrată individual în fabrică în raport cu un lanț trasabil conform NIST și este livrată împreună cu un certificat specific pentru fiecare cameră. Ce acoperă aceasta, cum se măsoară și precizia pe care o puteți indica se regăsesc în manualul hardware: [**Calibrarea radiometrică din fabrică**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Din punct de vedere software, ceea ce contează este faptul că Chloros determină calibrarea corectă atunci când o cameră se conectează și fixează coeficienții aplicați în fiecare export — consultați [Conectarea camerelor](connecting.md).

## În acest capitol

* [Conectarea camerelor](connecting.md) — detectarea automată, fereastra de dialog de conectare din interfața grafică, echivalentele CLI/SDK și modul în care se stabilește calibrarea din fabrică (pachet pe cameră vs. cloud) atunci când o cameră se conectează.
* [Setări cameră](camera-settings.md) — fiecare element de control din fila „Camere”: lista de camere, panoul de setări pentru fiecare cameră și panoul de setări pentru matrice (inclusiv editorul ROI și alinierea).
* [Setări și moduri de captură](capture.md) — Captură totală, captură unică / continuă / la intervale, tipuri de export și structura folderului de capturi.
* [Matrice cu mai multe camere](arrays.md) — expertul de conectare a matricei, sonda cablului de sincronizare, lățimea de bandă, PTP și vizualizarea în timp real a matricei.
* [Camere mono și indici de vegetație](mono-indices.md) — camere M3M și fluxul de lucru aliniere → stivuire → indexare.

Interfața completă de comenzi se regăsește în [Referința CLI](../reference/cli-reference.md) și [Referința SDK](../reference/sdk-reference.md).
