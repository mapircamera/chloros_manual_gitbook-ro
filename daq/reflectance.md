# Fluxuri de lucru pentru reflectanță

Un senzor de lumină DAQ transformă imaginile radiometrice în valori de reflectanță. Există două fluxuri de lucru distincte:

1. **Senzor unic** — un senzor DAQ măsoară iradianța descendentă în timp ce o cameră foto realizează captarea imaginii, iar Chloros
împarte radianța înregistrată de cameră la această valoare de referință.
2. **Cu două senzori** — doi senzori DAQ, unul orientat spre cer și unul orientat spre un obiect, generează o curbă spectrală de reflectanță în timp real, fără a fi necesară utilizarea unei camere.

## Un singur senzor + cameră (referință descendentă)

DAQ-ul funcționează ca senzor de lumină descendentă (DLS): camera măsoară radianța ascendentă **L**(W/m²/sr/nm), DAQ-ul măsoară iradianța descendentă**E** (W/m²/nm), iar Chloros
calculează reflectanța pe bandă după formula:

> ρ = π · L / E

Valorile citite de DAQ sunt întotdeauna **sincronizate temporal cu expunerea** — de aceea DAQ-ul și camerele utilizează un ceas sincronizat prin PTP (vezi [Rețea și sincronizare temporală DAQ-E](ethernet-ptp.md)). Puneți capacul Sunshine cu cosin pentru lucrările în aer liber și declarați-l corect; declararea capacului scalează direct valoarea E (a se vedea [Cap Profiles &amp; Calibrated Range](caps-and-range.md)). Pentru lucrări cantitative, rețineți caracteristica instrumentului: iradianța cantitativă provine dintr-o medie a citirilor efectuate pe o perioadă de cel puțin 15 s.

### Captură în timp real

Conectați DAQ-ul la o cameră în fila „Cameras”: panoul de setări al fiecărei camere are un meniu derulant **Light Sensor** care listează toate DAQ-urile conectate (DAQ-U/M/E) din fila „Light Sensors”; pentru o matrice sincronizată, o selecție a senzorului de lumină la nivel de matrice se propagă către fiecare element (camerele individuale pot totuși să suprascrie această setare). Odată asociat, spectrele senzorului alimentează slotul DLS al camerei, iar exporturile de reflectanță se împart la valoarea de citire corespunzătoare. Într-o matrice, un element cu senzor propriu utilizează propria valoare măsurată — pentru benzile sale din compoziția în timp real, exporturile sale și expunerea automată — iar celelalte elemente utilizează valorile matricei.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Două comportamente demne de reținut:

* **Fără DAQ asociat → reflectanța este respinsă, nu falsificată.** Chloros
e respinge produsul de reflectanță și înregistrează motivul omiterii, în loc să returneze în tăcere un produs inferior.
* **Valoarea măsurată utilizată este păstrată.** Pentru fiecare cadru de reflectanță, valoarea de citire DAQ aplicată efectiv este scrisă ca fișier `.daq` în folderul `daq/` al capturilor, astfel încât captura să poată fi reprocesată ulterior ([Înregistrarea și formatul .daq](recording.md)). Un fișier `.daq.cameras.json` situat lângă acesta identifică camerele pe care le descrie, iar reprocesarea îl asociază exclusiv cu imaginile acelor camere.

### Procesarea imaginilor înregistrate

Pentru procesarea post-zbor, înregistrați un fișier `.daq` în timpul sesiunii și păstrați-l alături de imagini — fluxul de lucru rezolvă automat datele de înregistrare corelate cu fluxul descendent, preluând orice calibrare de fabrică lipsă din cloud-ul MAPIR
la prima utilizare. Înregistrările GUI sunt adăugate automat la proiectul deschis atunci când se opresc.

Referința de reflectanță poate fi selectată în momentul procesării — `--reflectance-source` pe `chloros-cli process` sau setarea sursei de reflectanță din Setările proiectului din interfața grafică:

| Valoare | Comportament |
| --- | --- |
| `auto` (implicit) | O țintă de calibrare în cadru care a trecut de controlul calității (QA) reprezintă referința absolută; fluxul descendent DAQ (ρ = π·L/E) este soluția de rezervă. O țintă care trece controlul de calitate cu un indicator de eroare (un panou eliminat sau o bandă defectă) nu este preferată automat: lanțurile țintă, țintă+DAQ și numai DAQ sunt încercate în ordinea scorului măsurat pe panourile de testare. |
| `daq` | DAQ cu autoritate |
| `target` | Țintă strictă în cadru; fără substituire DAQ |

Consultați [Țintele de calibrare](../calibration-targets.md) pentru fluxurile de lucru ale țintelor și [capitolul LATTICE](../lattice/README.md), precum și [Referința CLI
](../reference/cli-reference.md) pentru fluxul complet de procesare. La citirea pixelilor de reflectanță exportați, utilizați scala marcată (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3
: 65535) — consultați [Formate de imagine de ieșire](../output-image-formats.md).

### Benzi în afara intervalului calibrat al DAQ-ului

Intervalul calibrat radiometric al DAQ-ului este de ~374–974 nm. Chloros
e respinge datele de reflectanță bazate pe DAQ pentru orice bandă a camerei cu mai puțin de jumătate din ponderea sa spectrală în interiorul acestui interval, raportând motivul de omitere `dls-uncalibrated-band-<nm>`. Dintre modelele disponibile în stoc, acest lucru afectează doar modelul F988: reflectanța camerei F988 este calibrată folosind un panou de reflectanță integrat în scenă; banda se află în afara intervalului calibrat al senzorului de lumină al DAQ-ului, astfel încât Chloros
aplică cea mai recentă captură a panoului și o menține între observările panoului. Dacă o cameră F988 funcționează doar cu DAQ, Chloros
refuză reflectanța bazată pe DAQ pentru banda respectivă, cu motivul de omitere `dls-uncalibrated-band-988` — fluxul de lucru cu panoul este calea acceptată.

## Senzor dublu (lumină ambientală + obiect)

Două senzori DAQ — orice pereche, pe orice platformă de transport — oferă un spectru de reflectanță în timp real fără cameră: un senzor este orientat spre cer (**Sursă de lumină ambientală**), iar celălalt spre subiect (**Scaner de obiect**), iar Chloros
calculează pentru fiecare lungime de undă:

> R(λ) = obiect(λ) / ambiental(λ)

(zero acolo unde ambiental ≤ 0).

### În interfața grafică

Cu ambii senzori conectați în fila Senzori de lumină, deschideți fereastra de adăugare a senzorilor (butonul „+” de pe o casetă a graficului în vizualizarea tip grilă) și alegeți **Combinare lumină ambientală + obiect**. Selectați cei doi senzori din meniurile derulante**Sursă de lumină ambientală**și**Scaner de obiecte**și faceți clic pe**Creați**. Grupul apare ca un grafic separat și ca un rând în bara laterală, cu o insignă verde**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Sub graficul de reflectanță (vizualizare listă), un **tabel al indicilor de vegetație** în timp real calculează indicii pe baza curbei, utilizând centrele de bandă la 450 nm (albastru), 550 nm (verde), 670 nm (roșu) și 800 nm (NIR
). Indicii bazați pe raporturi care anulează scara absolută (NDVI
, GNDVI
, ENDVI, WDRVI
, GRVI
, CVI
, GCI
, MSR) sunt afișați întotdeauna; indicii care necesită reflectanță absolută (EVI
, SAVI
, OSAVI
, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI
, NLI, MNLI, FCI, GEMI
) apar numai atunci când ambii senzori sunt modele calibrate din punct de vedere al puterii.

### Aparentă vs. relativă — regula de etichetare

Chloros
etichetează rezultatul senzorilor duali în funcție de ceea ce perechea de senzori poate revendica efectiv:

| Pereche de senzori | Etichetă |
| --- | --- |
| Ambii senzori calibrați — pachet din fabrică încărcat | **Reflectanță aparentă** |
| Unul dintre senzori necalibrat | **Reflectanță relativă** |

Toate cele trei modele sunt radiometrice: odată ce pachetul de calibrare din fabrică al unui senzor este încărcat, spectrele sale sunt exprimate în W/m²/nm absolut, astfel încât o pereche de senzori calibrați corespunde unei reflectanțe aparente absolute — transportul nu o determină. Un senzor care încă transmite date brute (fără pachet accesibil) reduce rezultatul la o curbă relativă (forma spectrală rămâne valabilă). Ambii senzori trebuie să aibă limite declarate corect ([Profiluri de limită și interval calibrat](caps-and-range.md)).

### Din Python


Nu există o apelare dedicată pentru senzori duali în interfața SDK
: deschideți două sesiuni cu `chloros_sdk.connect_daq_sensor()` și comparați manual spectrele acestora cu `latest()`, aplicând aceeași convenție de etichetare. (Există, de asemenea, un instrument de înregistrare cu două senzori în interfața internă directă cu hardware-ul a MAPIR
, menționat în [Referința CLI
](../reference/cli-reference.md) pentru exhaustivitate — acesta nu face parte din CLI
-ul livrat; fluxul de lucru GUI de mai sus este calea activă suportată.)
