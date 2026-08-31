# Fluxuri de lucru pentru reflectanță

Un senzor de lumină DAQ transformă imaginile radiometrice în valori de reflectanță. Există două fluxuri de lucru distincte:

1. **Senzor unic** — un singur DAQ măsoară iradianța descendentă în timp ce o cameră foto capturează imaginea, iar Chloros împarte radianța camerei pe acea valoare de referință.
2. **Cu două senzori** — doi senzori DAQ, unul orientat spre cer și unul orientat spre un obiect, generează o curbă spectrală de reflectanță în timp real, fără a fi necesară utilizarea unei camere.

## Un singur senzor + cameră (referință descendentă)

DAQ-ul funcționează ca senzor de lumină descendentă (DLS): camera măsoară radianța ascendentă **L**(W/m²/sr/nm), DAQ măsoară iradianța descendentă**E** (W/m²/nm), iar Chloros calculează reflectanța pe bandă după formula:

> ρ = π · L / E

Citirea DAQ este întotdeauna **sincronizată temporal cu expunerea** — de aceea DAQ-ul și camerele împărtășesc un ceas sincronizat prin PTP (vezi [Rețea DAQ-E și sincronizarea timpului](ethernet-ptp.md)). Puneți șapca Sunshine cu cosin pentru lucrul în aer liber și declarați-o corect; declararea șapcii scalează direct valoarea E (a se vedea [Cap Profiles &amp; Calibrated Range](caps-and-range.md)). Pentru lucrări cantitative, rețineți caracteristica instrumentului: iradianța cantitativă provine dintr-o medie a citirilor efectuate pe cel puțin 15 secunde.

### Captură în timp real

Conectați DAQ-ul la o cameră în fila „Cameras”: panoul de setări al fiecărei camere are un meniu derulant **Light Sensor** care listează toate DAQ-urile conectate (DAQ-U/M/E) din fila „Light Sensors”; pentru o matrice sincronizată, o selecție a senzorului de lumină la nivel de matrice se propagă către fiecare element (camerele individuale pot totuși să suprascrie această setare). Odată asociat, spectrele senzorului alimentează slotul DLS al camerei, iar valorile de reflectanță exportate sunt împărțite la valoarea citirii corespunzătoare.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Două comportamente demne de reținut:

* **Nu este asociat niciun DAQ → reflectanța este respinsă, nu simulată.** Chloros respinge produsul de reflectanță și înregistrează motivul omisiunii, în loc să returneze în tăcere un produs inferior.
* **Valoarea măsurată utilizată este păstrată.** Pentru fiecare cadru de reflectanță, valoarea DAQ aplicată efectiv este scrisă ca un fișier auxiliar `.daq` alături de imagini, astfel încât captura să poată fi reprocesată ulterior ([Înregistrarea și formatul .daq](recording.md)).

### Prelucrarea imaginilor înregistrate

Pentru prelucrarea post-zbor, înregistrați un fișier `.daq` în timpul sesiunii și păstrați-l alături de imagini — fluxul de lucru rezolvă automat datele de zbor corelate cu marcajul temporal, preluând orice calibrare din fabrică lipsă din cloud-ul MAPIR la prima utilizare. Înregistrările GUI sunt adăugate automat la proiectul deschis atunci când se opresc.

Referința de reflectanță poate fi selectată în momentul procesării — `--reflectance-source` pe `chloros-cli process` sau setarea sursei de reflectanță din Setările proiectului din interfața grafică:

| Valoare | Comportament |
| --- | --- |
| `auto` (implicit) | O țintă de calibrare în cadru care a trecut testul de asigurare a calității (QA) reprezintă referința absolută; fluxul descendent DAQ (ρ = π·L/E) este opțiunea de rezervă |
| `daq` | DAQ cu autoritate |
| `target` | Țintă strictă în cadru; fără substituire DAQ |

Consultați [Ținte de calibrare](../calibration-targets.md) pentru fluxurile de lucru ale țintelor și [capitolul LATTICE](../lattice/README.md), precum și [Referința CLI](../reference/cli-reference.md) pentru fluxul complet de procesare. La citirea pixelilor de reflectanță exportați, utilizați scara marcată (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — consultați [Formate de imagine de ieșire](../output-image-formats.md).

### Benzi în afara intervalului calibrat al DAQ-ului

Intervalul calibrat radiometric al DAQ-ului este de ~374–974 nm. Chloros refuză reflectanța bazată pe DAQ pentru orice bandă a camerei cu mai puțin de jumătate din ponderea spectrală în interiorul acestui interval, raportând motivul de omitere `dls-uncalibrated-band-<nm>`. Dintre codurile de produs (SKU) disponibile, acest lucru afectează doar modelul F988: reflectanța camerei F988 este calibrată folosind un panou de reflectanță integrat în scenă; banda se află în afara intervalului calibrat al senzorului de lumină al DAQ-ului, astfel încât Chloros aplică cea mai recentă captură a panoului și o menține între observările panoului. Dacă o cameră F988 funcționează numai în modul DAQ, Chloros respinge reflectanța bazată pe DAQ pentru banda respectivă, cu motivul de omitere `dls-uncalibrated-band-988` — fluxul de lucru cu panoul este calea acceptată.

## Senzor dublu (ambiant + obiect)

Două senzori DAQ — orice pereche, pe orice mijloc de transport — oferă un spectru de reflectanță în timp real fără cameră: un senzor este orientat spre cer (**Sursă de lumină ambientală**), iar celălalt spre subiect (**Scaner de obiect**), iar Chloros calculează pentru fiecare lungime de undă:

> R(λ) = obiect(λ) / ambiental(λ)

(zero acolo unde ambiental ≤ 0).

### În interfața grafică

Cu ambii senzori conectați în fila „Senzori de lumină”, deschideți fereastra de adăugare a senzorilor (butonul „+” de pe o casetă a graficului în vizualizarea tip grilă) și alegeți **Combinare lumină ambientală + obiect**. Selectați cei doi senzori din meniurile derulante Sursă de lumină ambientală și Scaner de obiecte și faceți clic pe Creare. Grupul apare ca un grafic separat și ca un rând în bara laterală, cu o insignă verde**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Sub graficul de reflectanță (vizualizare listă), un **tabel al indicilor de vegetație** în timp real calculează indicii pe baza curbei, folosind centrele de bandă la 450 nm (albastru) / 550 nm (verde) / 670 nm (roșu) / 800 nm (NDVI). Indicii bazați pe raporturi care anulează scara absolută (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) sunt afișați întotdeauna; indicii care necesită reflectanță absolută (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) apar numai atunci când ambii senzori sunt modele calibrate din punct de vedere al puterii.

### Aparent vs. relativ — regula de etichetare

Chloros etichetează ieșirea senzorului dual în funcție de ceea ce perechea de senzori poate revendica efectiv:

| Pereche de senzori | Etichetă |
| --- | --- |
| Ambii senzori calibrați — pachet din fabrică încărcat | **Reflectanță aparentă** |
| Unul dintre senzori necalibrat | **Reflectanță relativă** |

Toate cele trei modele sunt radiometrice: odată ce pachetul de calibrare din fabrică al unui senzor este încărcat, spectrele sale sunt exprimate în W/m²/nm absolut, astfel încât o pereche de senzori calibrați corespunde unei reflectanțe aparente absolute — transportul nu o determină. Un senzor care încă transmite date brute (fără pachet accesibil) reduce rezultatul la o curbă relativă (forma spectrală rămâne valabilă). Ambii senzori trebuie să aibă limite declarate corect ([Profiluri de limită și interval calibrat](caps-and-range.md)).

### Din Python

Nu există o comandă dedicată pentru senzori dubli în suprafața comună SDK: deschideți două sesiuni cu `chloros_sdk.connect_daq_sensor()` și comparați voi înșivă spectrele lor `latest()`, aplicând aceeași convenție de etichetare. (Există, de asemenea, un instrument de înregistrare cu senzor dublu pe suprafața internă directă a hardware-ului MAPIR, listat în [Referința CLI](../reference/cli-reference.md) pentru exhaustivitate — acesta nu face parte din CLI livrat; fluxul de lucru al interfeței grafice de mai sus este calea activă acceptată.)
