---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Ținte de calibrare

MAPIR oferă diverse ținte de calibrare pentru o gamă largă de aplicații. Modelul compact T4-R50 prezentat mai jos conține 4 panouri a căror reflectanță a luminii a fost măsurată în intervalul 250 - 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Țintele de referință difuze T4 au următoarele curbe de reflectanță, [descărcați datele aici](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>Reflectanța MAPIR T4 :: 250-2.500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>Reflectanța MAPIR T4 :: 400-1.000 nm</p></figcaption></figure>Țintele de referință difuze T4P au următoarele curbe de reflectanță, [descărcați datele aici](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>Reflectanța MAPIR T4P :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>Reflectanța MAPIR T4P :: 400-1000 nm</p></figcaption></figure>Analizând graficul de reflectanță, se poate observa că valorile reprezintă lungimea de undă (axa x) în funcție de procentul de reflectanță (axa y). Când capturăm o imagine a țintei de calibrare, creăm o relație între valoarea pixelului și procentul de reflectanță, în cadrul spectrului la care fiecare bandă a senzorului camerei este sensibilă.

Aceasta înseamnă că, pentru fiecare imagine capturată cu camerele noastre, puteți utiliza o fotografie a țintelor noastre de reflectanță, cum ar fi [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) sau [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), pentru a calibra imaginile în funcție de reflectanță. Odată calibrate, fiecare pixel din imagine corespunde unui procent de reflectanță.

Pentru rezultatele din **Survey3**, dacă exportați imaginile calibrate din Chloros

în formatele obișnuite JPG sau TIFF

, procentul de reflectanță se calculează împărțind valoarea pixelului la adâncimea de biți a formatului de imagine. Așadar, pentru JPG se împarte la 255, iar pentru TIFF

se împarte la 65.535. De asemenea, puteți alege formatul de ieșire PERCENT în Chloros

, iar atunci fiecare pixel va avea o valoare procentuală cuprinsă între 0,0 și 1,0 (0% până la 100% reflectanță). Rețineți însă că unele aplicații de imagine nu acceptă imaginile exprimate în procente (cu virgulă mobilă), iar acestea ocupă mult spațiu de stocare.

{% hint style="info" %}
**Reflectanța LATTICE utilizează o scală de pixeli diferită.** Reflectanța LATTICE este stocată cu DN 32768 = 100% reflectanță (nu 65535), iar fiecare fișier conține o etichetă XMP `Chloros:PixelScale` care specifică scala sa. Citiți eticheta și împărțiți la valoarea acesteia, în loc să presupuneți o constantă — consultați [Formate de imagine de ieșire](output-image-formats.md).
{% endhint %}

## Ținte de calibrare cu camerele LATTICE

În cazul camerelor LATTICE, o țintă de calibrare este **opțională** pentru reflectanță: Chloros

poate, în schimb, să raporteze reflectanța la iradianța descendentă măsurată de un senzor de lumină DAQ (ρ = π·L/E). Referința se alege prin setarea sursei de reflectanță a proiectului, configurată cu `chloros-cli process --reflectance-source` (interfața grafică nu are niciun control asupra acesteia și utilizează valoarea stocată a proiectului, `auto` în mod implicit — a se vedea [Setările proiectului](project-settings/project-settings.md)):

| Valoare | Comportament |
| --- | --- |
| `auto` *(implicit)* | O țintă din cadru care trece testul de control al calității (QA) este **referința absolută**; când nu există nicio țintă sau când testul de control al calității eșuează, Chloros

revine la diviziunea de curgere descendentă a DAQ. O țintă care trece de controlul calității cu un indicator (un panou eliminat sau o bandă defectă) nu este preferată automat: lanțurile țintă, țintă+DAQ și numai DAQ sunt încercate în ordinea scorului lor măsurat pe panourile de testare. |
| `target` | Strict numai țintă — fără substituire DAQ. |
| `daq` | DAQ autoritar — măsurarea descendentă este întotdeauna referința. |

Comportament suplimentar al țintelor pentru LATTICE:

* **Geometrii ale țintelor** — sunt acceptate panourile marcate cu ArUco, panourile cu ROI fix și țintele de tip bandă; geometria provine din configurația țintelor proiectului.
* **Date privind țintele măsurate pe unitate** — `--target-reflectance-dir DIR` indică un director cu scanări ale reflectanței țintelor măsurate pe unitate (`<serial>.csv`, căutate după numărul de serie/codul QR al unității țintei). În cazul unei ratări, Chloros

revine la spectrele nominale T3/T4P.
* **Ancorare temporală** — o țintă detectată calibrează cadrele din jurul său și este menținută între observările țintei.

Semantica completă a indicatorilor și exemple se găsesc în [Referința CLI

](reference/cli-reference.md) (vezi „Comutatoare de export pe produs”).

### F988

„Reflectanța F988 este calibrată folosind un panou de reflectanță din scenă: banda se află dincolo de intervalul calibrat al senzorului de lumină DAQ, așa că Chloros

aplică cea mai recentă captură a panoului și o menține între observările panoului.”

Dacă F988 este rulat cu calibrare exclusiv prin DAQ, Chloros

respinge reflectanța bazată pe DAQ pentru banda respectivă și explică motivul (motivul omisiunii `dls-uncalibrated-band-988`); fluxul de lucru cu panoul este calea acceptată.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
