---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formate de imagine de ieșire

Chloros exportă produsele prelucrate în patru formate de fișiere. Selectați formatul în Setările proiectului (GUI), cu `--format` (CLI) sau cu `export_format` (SDK). CLI și SDK acceptă exact șirurile de caractere de mai jos.

| Șir de format | Extensie | Tip de pixel | Interval de pixeli | Note |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(implicit)* | `.tif` | număr digital uint16 | 0 – 65535 | Recomandat pentru fotogrammetrie / GIS. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = reflectanță 100%. Unele aplicații nu pot citi fișiere TIFF cu virgulă mobilă; fișierele sunt mai mari. |
| `PNG (8-bit)` | `.png` | număr digital uint8 | 0 – 255 | Compresie fără pierderi, potrivită pentru vizualizare pe web și vizualizare. |
| `JPG (8-bit)` | `.jpg` | număr digital uint8 | 0 – 255 | Compresie cu pierderi, fișiere de dimensiuni minime. |

## Unde se salvează fișierele de ieșire

Fișierele sunt salvate în folderul proiectului, grupate pe cameră și apoi pe format de fișier:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Folderul camerei este `LATT-<sensor>-<lens>-F<filter>` pentru LATTICE și `<model>_<filter>` (de exemplu, `Survey3N_RGN`) pentru Survey3. **Fiecare produs exportat păstrează numele fișierului sursă — folderul identifică produsul, nu sufixul numelui fișierului.** Consultați [Unde se salvează rezultatele](reference/cli-reference.md) în Ghidul de referință CLI pentru regulile complete.

## Produse LATTICE (niveluri de captură și export)

Un cadru brut LATTICE se ramifică în fiecare produs solicitat într-o singură trecere. Fiecare tip de produs are propriul comutator (casete de selectare din interfața grafică sau CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, toate activate implicit):

| Nivel | Conținut | Tip de date |
| --- | --- | --- |
| `raw` | Date Bayer direct de la senzor (camere monocrome: banda unică). Prelucrarea începe întotdeauna de la date brute. | Așa cum au fost capturate |
| `debayered` | Demosaițare liniară — 3 canale pentru M3C, 1 canal în tonuri de gri pentru M3M. | DN liniar |
| `radiance` | Radianța spectrală absolută din lanțul radiometric complet, în **W/m²/sr/nm**. Întotdeauna scris ca TIFF pe 32 de biți (`tiff32/Radiance_Images/`), indiferent de formatul de export selectat. | float32 |
| `reflectance` | Reflectanța ρ, unde **DN 32768 = ρ 1,0 (100%)**, cu marjă de manevră până la ρ 2,0. Compatibil cu Pix4D. | uint16 |
| `preview` | Randare gata de afișare: RGB = balans de alb + gamma; multispectral = extindere în culori false. | afișaj pe 8 biți |

## Citirea valorilor pixelilor de reflectanță

Reflectanța este stocată ca un număr digital întreg, iar **DN-ul care corespunde lui ρ = 1,0 (reflectanță de 100%) depinde de camera sursă**:

| Camera sursă | ρ = 1,0 este DN | Cum se determină |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (marjă de până la ρ 2,0) | Eticheta XMP `Chloros:PixelScale=32768` este înscrisă în fișier. |
| Survey3 | `65535` (tăiat la ρ 1,0) | Nu există etichete XMP `Chloros:*` — această absență este semnalul. |

**Citiți eticheta XMP `Chloros:PixelScale` și împărțiți la valoarea acesteia**, în loc să presupuneți o constantă. Eticheta este definită în domeniul uint16, deci rămâne `32768` în toate formatele de ieșire care rescalează — normalizați mai întâi tipul de date stocat înapoi la uint16 (×257 de la 8 biți, ×65535 de la float32).

{% hint style="warning" %}
**Un caz nu are scalare, conform proiectării.** Când o captură din sursă de 8 biți (BayerRG8) este scrisă ca TIFF de 8 biți, pipeline-ul o limitează la intervalul 0–255 în loc să o rescalizeze, astfel încât fișierul nu are nicio scală — Chloros omite în mod deliberat `Chloros:PixelScale` în acest caz. Dacă eticheta lipsește dintr-un fișier de reflectanță LATTICE, nu presupuneți existența unei scări; reexportați în schimb la 16 biți sau 32 de biți.
{% endhint %}

Pentru regulile complete (inclusiv etichetele compatibile cu MicaSense), consultați **„Citirea pixelilor de reflectanță”** în [Referința CLI](reference/cli-reference.md).
