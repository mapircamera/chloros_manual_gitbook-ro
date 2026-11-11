---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Descărcare

Descărcați cea mai recentă versiune a programului Chloros
pentru a începe prelucrarea imaginilor multispectrale.

### Cerințe de sistem

#### Windows


| Cerință          | Minim                                              | Recomandat                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistem de operare** | Windows
10 (64 de biți)                                  | Windows
11 (64 de biți)                                  |
| **Procesor**        | Intel Core i5 sau echivalent                          | Intel Core i7 sau superior                              |
| **Memorie (RAM)**     | 8 GB                                                  | 16 GB sau mai mult                                         |
| **Placă grafică**    | Compatibilă cu DirectX 11                                | Placă grafică NVIDIA cu 4 GB+ VRAM                            |
| **Spațiu de stocare**          | 6 GB spațiu liber                                       | SSD cu 10 GB+ spațiu liber                            |
| **Ecran**          | 1920x1080                                            | 2560x1440 sau mai mare                                  |
| **Internet**         | Necesar pentru activarea licenței [opționale] Chloros
+ | Necesar pentru activarea licenței [opționale] Chloros
+ |

#### Linux
amd64 (x86_64)

| Cerințe       | Minime                    | Recomandate               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuție**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 sau superior   |
| **Memorie (RAM)**  | 8 GB                        | 16 GB sau mai mult              |
| **Placă grafică** | Niciuna (procesare CPU)      | GPU NVIDIA cu 4 GB+ VRAM |
| **Spațiu de stocare**       | 2 GB spațiu liber             | SSD cu 10 GB+ spațiu liber       |
| **Python**        | Python
3.7+ (pentru SDK
)      | Python
3.10+              |

#### Linux
arm64 (NVIDIA Jetson)

| Cerințe      | Minime                      | Recomandate                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platformă**     | NVIDIA Jetson cu JetPack 6 | Jetson Orin NX 16 GB sau AGX Orin |
| **Memorie (RAM)** | 8 GB (partajată GPU/CPU)         | 16 GB+ partajată                    |
| **Spațiu de stocare**      | 2 GB spațiu liber               | SSD NVMe cu 10 GB+ spațiu liber        |
| **Python**       | Python
3.7+ (pentru SDK
)        | Python
3.10+                    |

{% hint style="info" %}
**Accelerare GPU**: Utilizatorii Chloros
+ cu GPU-uri NVIDIA pot folosi accelerarea CUDA pentru o procesare semnificativ mai rapidă. Aceasta funcționează cu GPU-urile NVIDIA pentru desktop pe Windows
și Linux
amd64, precum și pe NVIDIA Jetson cu JetPack 6. Utilizatorii Chloros
+ beneficiază, de asemenea, de procesare multithread pentru viteză maximă.
{% endhint %}

***

## Descărcați Chloros


### Cea mai recentă versiune stabilă: Versiunea 1.3.0 (11 noiembrie 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Descărcați Chloros pentru Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Descărcați Chloros pentru Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Descărcați Chloros pentru Linux arm64 / Jetson (.deb)</a>


#### Program de instalare pentru Windows
(GUI + CLI
+ Backend)

* **Tip de fișier**: .exe (Program de instalare pentru Windows
)

**Pași de instalare:**

1. Descărcați fișierul .exe de mai sus
2. Faceți dublu clic pe programul de instalare pentru a începe instalarea
3. Urmați instrucțiunile expertului de instalare
4. Alegeți directorul de instalare (implicit: `C:\Program Files\Chloros\`)
5. În pagina **Selectare sarcini suplimentare**, lăsați bifate ambele opțiuni**Setări de sistem** (recomandat — vedeți mai jos)
6. Finalizați instalarea și lansați Chloros
sau Chloros
CLI
7. Autentificați-vă cu [contul MAPIR
Cloud Chloros
+](https://cloud.mapir.camera/pricing) (sau continuați cu versiunea gratuită)

**Opțiunile de setări de sistem** (ambele bifate implicit):

| Opțiune, așa cum este formulată în programul de instalare | Ce face | Dezactivare instalare silențioasă |
| --- | --- | --- |
| **Activare căi lungi în Windows** (permite ca folderele de proiect să aibă nume mai lungi de 260 de caractere; recomandat) | Setează la 1 setarea `LongPathsEnabled` a Windows
la nivel de sistem. Windows
vine cu această setare dezactivată, iar în acest caz nicio cale de fișier nu poate depăși 260 de caractere; produsele Chloros
se află cu până la aproximativ 130 de caractere sub folderul de proiect, astfel încât, cu opțiunea pentru căi lungi dezactivată, un folder de proiect situat mai adânc este respins. Se aplică programelor pornite după instalare. Rămâne activată după dezinstalare, deoarece alte programe pot depinde de această setare. | `/MERGETASKS="!longpaths"` |
| **Excludeți backend-ul Chloros
din scanarea Microsoft Defender** (captură mult mai rapidă cu camera; recomandat) | Adaugă o excludere a procesului Microsoft Defender pentru `chloros-backend.exe` din folderul de instalare, astfel încât protecția în timp real să nu scaneze fiecare fișier pe care îl scrie backend-ul, indiferent unde îl scrie. La rate de captură ridicate, scanarea consumă aproximativ 30% din rata de captură (77,7 față de 52,7 capturi pe secundă într-o captură continuă cu 9 camere). Niciun alt program nu este exclus, iar excluderea este eliminată la dezinstalare. În cazul în care Defender lipsește sau este blocat de o politică, acest pas eșuează în mod silențios, iar instalarea continuă. | `/MERGETASKS="!defenderexclusion"` |

O instalare silențioasă (`/SILENT` sau `/VERYSILENT`) aplică ambele opțiuni, cu excepția cazului în care renunțați la acestea folosind valoarea `/MERGETASKS` afișată.

{% hint style="success" %}
Programul de instalare adaugă folderul CLI
(`C:\Program Files\Chloros\cli\`) în sistemul dvs. `PATH` o singură dată — reinstalarea sau actualizarea nu mai adaugă o intrare duplicată și elimină duplicatele adăugate de programele de instalare anterioare. Deschideți un terminal nou după instalare pentru a utiliza `chloros-cli`. Programul de instalare adaugă, de asemenea, două reguli pentru Paravanul de protecție Windows
pentru backend, în fiecare profil de rețea: **Chloros
Backend (UDP-In)**, pentru fluxurile de la camere, PTP, mDNS și fluxul de date DAQ-E, și**Chloros
Backend (TCP-In)**, limitat la subrețeaua locală, pentru o actualizare a firmware-ului DAQ-E, în cadrul căreia senzorul se reconectează la acest computer pentru a colecta imaginea. Ambele sunt eliminate la dezinstalare.
{% endhint %}

#### Linux
amd64 (pachet .deb — CLI
+ backend)

* **Tip fișier**: .deb (pachet Debian/Ubuntu)
* **Arhitectură**: x86\_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux
arm64 — NVIDIA Jetson (pachet .deb — CLI
+ Backend)

* **Tip fișier**: .deb (JetPack 6)
* **Arhitectură**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Folosiți numele fișierului pe care l-ați descărcat efectiv și păstrați `./`, astfel încât `apt` să instaleze acel fișier în loc să caute un pachet după nume. Instalați cu `apt install`, nu cu `dpkg -i`: apt instalează și cerințele Python
SDK
, pe care pachetul le recomandă, nu le impune, iar `dpkg -i` le omite. Începând cu versiunea 1.3.0, backend-ul și interfața CLI
sunt instalate deja dezarhivate sub `/usr/lib/chloros/`, astfel încât acestea pornesc fără a se mai dezarhiva mai întâi.

Pachetul Jetson utilizează versiunea PyTorch pentru CUDA din JetPack 6, astfel încât procesarea, inclusiv debayerul Texture Aware, rulează pe GPU-ul Jetson. Acesta utilizează bibliotecile CUDA și cuDNN instalate de JetPack 6 (`sudo apt install nvidia-jetpack` pe o imagine care nu le conține).

Consultați [Instalarea pe Linux
](linux/linux-installation.md) pentru instrucțiuni detaliate de configurare și [Ghidul NVIDIA Jetson](linux/nvidia-jetson-guide.md) pentru îndrumări specifice Jetson.

#### Python
SDK
(Toate platformele)

Fiecare program de instalare include un pachet wheel `chloros_sdk` corespunzător, astfel încât versiunea SDK
să corespundă întotdeauna cu GUI/CLI
/backend-ul instalat. Pe Windows
, programul de instalare îl instalează automat în Python
-ul sistemului. Pe Linux
, `.deb` instalează SDK
-ul pentru sistemul `python3` însuși (în `/usr/lib/python3/dist-packages`), astfel încât `import chloros_sdk` funcționează imediat după instalare, fără pip și fără conexiune la rețea. Doar un virtualenv, care nu vede pachetele sistemului, are nevoie ca roata inclusă în pachet să fie instalată în acesta:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pentru gazdele care utilizează doar pip (fără pachetul Chloros
instalat; SDK
-ul controlează atunci un backend pe o altă mașină), SDK
-ul se află și pe PyPI:

```bash
pip install chloros-sdk
```

Consultați [API
: Python
SDK
](api-python-sdk.md) și [Referința SDK
](reference/sdk-reference.md) pentru documentație.

{% hint style="info" %}
**Utilizatori Linux**: Pachetul `.deb` instalează interfața CLI
, backend-ul și SDK
-ul Python
. Nu există o interfață grafică (GUI) pentru Linux
— toată interacțiunea se realizează prin CLI
sau SDK
.
{% endhint %}

***

## Resurse suplimentare

### SDK
Python
pentru Chloros


Pentru dezvoltatori și fluxuri de lucru de automatizare, Chloros
Python
SDK
este inclus în fiecare program de instalare: programul de instalare pentru Windows
îl instalează în Python
, iar `.deb` pentru Linux
îl instalează pentru sistemul `python3`. Pe un computer fără pachetul Chloros
(unul care controlează un backend pe un alt computer), instalați-l de pe PyPI:

```bash
pip install chloros-sdk
```

**Documentație**: [API
: Python
SDK
](api-python-sdk.md)

**Cerințe**: un backend Chloros
pe care să-l controlați (programul de instalare pentru Windows
sau pachetul `.deb` pentru Linux
instalează unul) și un cont Chloros
+ cu abonament plătit

***

## Ce este inclus

### Program de instalare pentru Windows


* ✅ **Interfața grafică Chloros** - Interfață grafică cu funcționalități complete
* ✅ **Chloros
CLI
** - Interfață de linie de comandă (necesită licență Chloros
+)
* ✅ **Backend Chloros** - Motor de procesare
* ✅ **Python
SDK
** - instalat în Python
, dacă acesta este prezent (necesită licență Chloros
+)
* ✅ **Profiluri de cameră** - șabloane de cameră MAPIR
preconfigurate

### Pachet .deb pentru Linux


* ✅ **Chloros
CLI
** - Interfață de linie de comandă (necesită licență Chloros
+)
* ✅ **Chloros
Backend** - Motor de procesare
* ✅ **Python
SDK
** - instalat pentru sistemul `python3` (necesită licență Chloros
+)
* ✅ **Profiluri de cameră** - Șabloane de cameră MAPIR
preconfigurate
* ❌ Fără GUI — Linux
este doar CLI
/SDK
fără interfață grafică

### Python
SDK
(PyPI, pentru mașini fără pachet Chloros
)

* ✅ **Chloros
SDK
** - Python
API
(necesită licență Chloros
+)

***

## Trecere la Chloros+

Deblocați funcții avansate cu un abonament Chloros
+:

* 🚀 **Procesare multithread** — Procesați imagini în paralel
* ⚡ **Accelerare GPU (CUDA)** - Profitați de puterea GPU-urilor NVIDIA
* 💻 **Acces CLI** - Automatizați cu instrumente de linie de comandă
* 🐍 **Python
SDK
** - Acces programatic la API

* 📱 **Dispozitive multiple** - Utilizați pe 2-10+ dispozitive (în funcție de plan)
* **🐻 Metodă avansată de debayerizare cu recunoaștere a texturii** - o metodă de debayerizare de înaltă calitate, cu recunoaștere a marginilor, combinată cu un model de eliminare a zgomotului bazat pe AI/ML, care elimină aproape tot zgomotul generat de debayerizare.
* 🧮 **Formule personalizate** - Creați indici multispectrali personalizați

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Vedeți planurile și prețurile Chloros+</a></p>***

## Ajutor pentru instalare

### Depanare

**Instalarea eșuează cu următorul mesaj de eroare:**

* Asigurați-vă că aveți drepturi de administrator
* Dezactivați temporar programul antivirus
* Verificați dacă îndepliniți cerințele minime de sistem

**Aplicația nu pornește (Windows

):**

* Verificați dacă Windows

10/11 (64 de biți) este instalat
* Actualizați driverele grafice
* Verificați Vizualizatorul de evenimente Windows

pentru detalii despre eroare
* Contactați serviciul de asistență cu jurnalele de erori

**CLI

nu pornește (Linux

):**

* Verificați dacă pachetul `.deb` este instalat corect: `dpkg -l | grep chloros`
* Verificați permisiunile: `sudo chmod +x /usr/bin/chloros-cli`
* Rulați diagnosticarea: `chloros-cli selftest`
* Verificați dacă lipsesc biblioteci: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (începând cu versiunea 1.3.0, `/usr/lib/chloros/chloros-backend` este un mic script de lansare; programul propriu-zis se află în `/usr/lib/chloros/backend/`)

**Probleme cu activarea licenței:**

* Asigurați-vă că aveți conexiune activă la internet
* Verificați datele de autentificare la [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verificați dacă firewall-ul nu blochează Chloros

* Consultați [Chloros

+ Autentificare](chloros+-login.md) pentru instrucțiuni detaliate

### Asistență

Aveți nevoie de ajutor cu instalarea sau configurarea?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Site web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentație**: [Noțiuni introductive](./)
* ❓ **Întrebări frecvente**: [Întrebări frecvente](faq.md)***

## Actualizări software

Chloros

verifică dacă există actualizări, anunță când este disponibilă o nouă versiune și oferă un link către această pagină de descărcare — actualizarea se face prin rularea noului program de instalare semnat. Setările, proiectele și datele de autentificare rămân intacte după actualizare. Pe Linux

și Jetson, `chloros-cli update` verifică dacă există o versiune mai nouă și vă oferă posibilitatea de a descărca și instala versiunea corespunzătoare `.deb` (această comandă este disponibilă doar pe Linux

).

***

## Jurnal de modificări**Versiunea 1.3.0 (cea mai recentă, 11 noiembrie 2026)**— consultați**Noutăți în Chloros

1.3.0** pe pagina [Noțiuni introductive](./) pentru lista completă.**Versiunea 1.2.0**— consultați**Noutăți în Chloros

1.2.0** pe pagina [Noțiuni introductive](./).

<details>

<summary>Versiunea 1.0.5</summary>

**Data lansării: 10 februarie 2026**

**Funcții noi*** **Metoda de debayering cu recunoaștere a texturii \[Numai Chloros

+] -** Texture Aware utilizează un proces de debayering de înaltă calitate, sensibil la margini, combinat cu un model de eliminare a zgomotului bazat pe AI/ML, care elimină aproape tot zgomotul generat de debayering.
* **Suport pentru ținte de calibrare T4P*** **Procesare GPU Chloros

+ mai rapidă, gestionare îmbunătățită a memoriei**

**Remedierea erorilor*** Interfață grafică (GUI) complet nouă, care ar trebui să funcționeze acum pe toate computerele cu Windows

.

</details>

<details>

<summary>Versiunea 1.0.4</summary>

**Data lansării: 5 ianuarie 2026**

**Funcții noi*** **Comutator imagine/metadate**: S-a adăugat un comutator în File Browser pentru a vizualiza metadatele imaginii selectate într-un tabel, în loc de grila de imagini
* **Glisor de zoom pentru grila de imagini**: Un nou glisor în interfața utilizatorului pentru a regla dimensiunea miniaturilor (acceptă și combinația CTRL + rotița mouse-ului)
* **Butoane de export pentru grila de imagini**: butoane în rândul de sus pentru a comuta miniaturile de la format JPG la exporturi procesate (Targets, Reflectance, Index, LUT)
* **Fila Hartă**: nouă hartă 2D interactivă care afișează marcajele de localizare GPS ale imaginilor
  * Suportă Google Maps și secțiuni de hartă ESRI (selectează automat cel mai bun serviciu de secțiuni de hartă în funcție de disponibilitatea nivelului de zoom)
  * Previzualizare miniatură la trecerea mouse-ului peste marcajele de pe hartă

**Remedierea erorilor*** Suport îmbunătățit pentru instalarea Chloros pe calculatoare cu limba de sistem diferită de engleză

</details>

<details>

<summary>Versiunea 1.0.3</summary>

**Data lansării: 20 decembrie 2025**

**Funcții noi*** Lansare inițială

**Îmbunătățiri*** Lansare inițială

**Remedierea erorilor*** Lansare inițială

**Probleme cunoscute*** Lansare inițială

</details>

***

## Acord de licență**Software proprietar** - Copyright (c) 2026 MAPIR Inc.

Utilizarea, distribuirea sau modificarea neautorizată este interzisă.

**Versiune gratuită**: Disponibilă pentru uz personal și comercial, cu funcționalități limitate**Chloros+**: Licență pe bază de abonament pentru funcționalități avansate și implementări comerciale
