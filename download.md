---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Descărcare

Descărcați cea mai recentă versiune a Chloros pentru a începe prelucrarea imaginilor multispectrale.

### Cerințe de sistem

#### Windows

| Cerință          | Minim                                              | Recomandat                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistem de operare** | Windows 10 (64 de biți)                                  | Windows 11 (64 de biți)                                  |
| **Procesor**        | Intel Core i5 sau echivalent                          | Intel Core i7 sau superior                              |
| **Memorie (RAM)**     | 8 GB                                                  | 16 GB sau mai mult                                         |
| **Placă grafică**    | Compatibilă cu DirectX 11                                | GPU NVIDIA cu 4 GB+ VRAM                            |
| **Spațiu de stocare**          | 6 GB spațiu liber                                       | SSD cu 10 GB sau mai mult spațiu liber                            |
| **Ecran**          | 1920x1080                                            | 2560x1440 sau mai mare                                  |
| **Internet**         | Necesar pentru activarea licenței \[opțională] Chloros+ | Necesar pentru activarea licenței \[opțională] Chloros+ |

#### Linux amd64 (x86\_64)

| Cerințe           | Minime                    | Recomandate               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuție**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 sau superior   |
| **Memorie (RAM)**  | 8 GB                        | 16 GB sau mai mult              |
| **Placă grafică** | Niciuna (procesare CPU)      | Placă grafică NVIDIA cu 4 GB+ VRAM |
| **Spațiu de stocare**       | 2 GB spațiu liber             | SSD cu 10 GB+ spațiu liber       |
| **Python**        | Python 3.7+ (pentru SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Cerințe      | Minim                      | Recomandat                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platformă**     | NVIDIA Jetson cu JetPack 6 | Jetson Orin NX 16 GB sau AGX Orin |
| **Memorie (RAM)** | 8 GB (partajată GPU/CPU)         | 16 GB+ partajată                    |
| **Spațiu de stocare** | 2 GB spațiu liber               | SSD NVMe cu 10 GB+ spațiu liber        |
| **Python**       | Python 3.7+ (pentru SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Accelerare GPU**: utilizatorii Chloros+ cu GPU-uri NVIDIA pot folosi accelerarea CUDA pentru o procesare semnificativ mai rapidă. Aceasta funcționează atât pe Windows (GPU-uri pentru desktop), cât și pe Linux (GPU-uri pentru desktop și NVIDIA Jetson). Utilizatorii versiunii Chloros+ beneficiază, de asemenea, de procesare multi-threaded pentru viteză maximă.
{% endhint %}

***

## Descărcați Chloros

### Cea mai recentă versiune stabilă: Versiunea 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Descărcați Chloros pentru Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Descărcați Chloros pentru Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Descărcați Chloros pentru Linux arm64 / Jetson (.deb)</a>

#### Programul de instalare Windows (GUI + CLI + Backend)

* **Tip fișier**: .exe (program de instalare Windows)**Pași de instalare:**

1. Descărcați fișierul .exe de mai sus
2. Faceți dublu clic pe programul de instalare pentru a începe instalarea
3. Urmați instrucțiunile expertului de instalare
4. Alegeți directorul de instalare (implicit: `C:\Program Files\MAPIR\Chloros\`)
5. Finalizați instalarea și lansați Chloros sau Chloros CLI
6. Autentificați-vă cu contul dvs. [MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (sau continuați cu versiunea gratuită)

{% hint style="success" %}
Programul de instalare adaugă automat `chloros-cli` la variabila PATH a sistemului dvs. pentru accesul prin linia de comandă.
{% endhint %}

#### Linux amd64 (pachet .deb — CLI + Backend)

* **Tip de fișier**: .deb (pachet Debian/Ubuntu)
* **Arhitectură**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (pachet .deb — CLI + Backend)

* **Tip fișier**: .deb (JetPack 6)
* **Arhitectură**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Consultați [Instalarea Linux](linux/linux-installation.md) pentru instrucțiuni detaliate de configurare și [Ghidul NVIDIA Jetson](linux/nvidia-jetson-guide.md) pentru îndrumări specifice pentru Jetson.

#### Python SDK (Toate platformele)

Fiecare program de instalare include un pachet corespunzător de roți `chloros_sdk`, astfel încât versiunea SDK se potrivește întotdeauna cu GUI-ul instalat/CLI/backend-ul. Pe Windows, programul de instalare îl instalează automat în sistemul dvs. Python; pe Linux, `.deb` plasează pachetul „wheel” în `/usr/lib/chloros/sdk/` și afișează comanda de instalare:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pentru gazdele care utilizează doar pip (fără pachetul Chloros instalat), SDK se găsește și pe PyPI:

```bash
pip install chloros-sdk
```

Consultați [API : Python SDK](api-python-sdk.md) și [Referința SDK](reference/sdk-reference.md) pentru documentație.

{% hint style="info" %}
**Utilizatorii Linux**: Pachetul `.deb` instalează CLI și backend-ul. Nu există o interfață grafică pentru Linux — toate interacțiunile se realizează prin CLI sau SDK.
{% endhint %}

***

## Resurse suplimentare

### Python SDK

Pentru dezvoltatori și fluxuri de lucru de automatizare, instalați Chloros, Python și SDK:

```bash
pip install chloros-sdk
```

**Documentație**: [API: Python SDK](api-python-sdk.md)**Cerințe**: Chloros trebuie să fie instalat (programul de instalare Windows sau pachetul Linux `.deb`), este necesară autentificarea cu licența Chloros+***

## Ce este inclus

### Programul de instalare Windows

* ✅ **Chloros GUI** - Interfață grafică cu funcționalități complete
* ✅ **Chloros CLI** - Interfață de linie de comandă (necesită licență Chloros+)
* ✅ **Chloros Backend** - Motor de procesare
* ✅ **Profiluri de cameră** - Șabloane de cameră MAPIR preconfigurate

### Pachet .deb Linux

* ✅ **Chloros CLI** - Interfață de linie de comandă (necesită licență Chloros+)
* ✅ **Chloros Backend** - Motor de procesare
* ✅ **Profiluri de cameră** - Șabloane de cameră MAPIR preconfigurate
* ❌ Fără interfață grafică — Linux este doar CLI/SDK fără interfață grafică

### Python SDK (PIP, toate platformele)

* ✅ **Chloros SDK** - Python API (necesită licență Chloros+)***

## Trecere la Chloros+

Deblocați funcții avansate cu un abonament Chloros+:

* 🚀 **Procesare multithread** - Procesează imagini în paralel
* ⚡ **Accelerare GPU (CUDA)** - Folosește puterea GPU-urilor NVIDIA
* 💻 **Acces la CLI** - Automatizați cu instrumente de linie de comandă
* 🐍 **Python SDK** - Acces programatic la API
* 📱 **Dispozitive multiple** - Utilizare pe 2-10+ dispozitive (în funcție de plan)
* **🐻 Metodă avansată de debayerizare cu recunoaștere a texturii** - o metodă de debayerizare de înaltă calitate, sensibilă la margini, combinată cu un model de eliminare a zgomotului bazat pe AI/ML, care elimină aproape tot zgomotul generat de debayerizare.
* 🧮 **Formule personalizate** - Creați indici multispectrali personalizați

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Vedeți planurile și prețurile Chloros+</a></p>***

## Ajutor pentru instalare

### Rezolvarea problemelor

**Instalarea eșuează cu următorul mesaj de eroare:**

* Asigurați-vă că aveți drepturi de administrator
* Dezactivați temporar programul antivirus
* Verificați dacă îndepliniți cerințele minime de sistem

**Aplicația nu pornește (Windows):**

* Verificați dacă Windows 10/11 (64-bit) este instalat
* Actualizați driverele grafice
* Verificați Vizualizatorul de evenimente Windows pentru detalii despre eroare
* Contactați serviciul de asistență cu jurnalele de erori

**CLI nu pornește (Linux):**

* Verificați dacă pachetul `.deb` este instalat corect: `dpkg -l | grep chloros`
* Verificați permisiunile: `sudo chmod +x /usr/bin/chloros-cli`
* Rulați diagnosticarea: `chloros-cli selftest`
* Verificați dacă lipsesc biblioteci: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Probleme cu activarea licenței:**

* Asigurați-vă că conexiunea la internet este activă
* Verificați datele de autentificare la [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verificați dacă firewall-ul nu blochează Chloros
* Consultați [Chloros+ Autentificare](chloros+-login.md) pentru instrucțiuni detaliate

### Asistență

Aveți nevoie de ajutor cu instalarea sau configurarea?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Site web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentație**: [Noțiuni introductive](./)
* ❓ **Întrebări frecvente**: [Întrebări frecvente](faq.md)***

## Actualizări software

Chloros verifică dacă există actualizări, anunță când este disponibilă o nouă versiune și oferă un link către această pagină de descărcare — actualizarea se efectuează prin rularea noului program de instalare semnat. Setările și proiectele dvs. rămân intacte după actualizare. Pe Linux și Jetson, `chloros-cli update` verifică dacă există o versiune mai nouă și vă oferă posibilitatea de a descărca și instala `.deb` corespunzător (această comandă este disponibilă doar pe Linux).

***

## Jurnal de modificări**Versiunea 1.2.0 (cea mai recentă)**— consultați**Noutăți în Chloros 1.2.0** pe pagina [Noțiuni introductive](./) pentru lista completă a funcționalităților.

<details>

<summary>Versiunea 1.0.5</summary>

**Data lansării: 10 februarie 2026**

**Funcționalități noi*** **Metoda de debayering cu recunoaștere a texturii \[Numai pentru Chloros+] -** Texture Aware utilizează o metodă de debayering de înaltă calitate, sensibilă la margini, combinată cu un model de eliminare a zgomotului bazat pe AI/ML, care îndepărtează aproape tot zgomotul generat de procesul de debayering.
* **Suport pentru ținte de calibrare T4P*** **Procesare GPU mai rapidă pentru Chloros+, gestionare îmbunătățită a memoriei**

**Remedierea erorilor*** Interfață grafică (GUI) complet nouă, care ar trebui să funcționeze acum pe toate computerele Windows.

</details>

<details>

<summary>Versiunea 1.0.4</summary>

**Data lansării: 5 ianuarie 2026**

**Funcții noi*** **Comutator imagine/metadate**: S-a adăugat un comutator în File Browser pentru a vizualiza metadatele imaginii selectate într-un tabel în loc de grila de imagini
* **Glisor de zoom pentru grila de imagini**: Un nou glisor în interfața utilizatorului pentru a regla dimensiunea miniaturilor (acceptă și combinația CTRL + rotița mouse-ului)
* **Butoane de export pentru grila de imagini**: butoane în rândul de sus pentru a comuta miniaturile de la format JPG la exporturi procesate (Ținte, Reflectanță, Index, LUT)
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

</details>***

## Acord de licență**Software proprietar** - Drepturi de autor (c) 2026 MAPIR Inc.

Utilizarea, distribuirea sau modificarea neautorizată este interzisă.

**Versiunea gratuită**: Disponibilă pentru uz personal și comercial, cu funcționalități limitate**Chloros+**: Licență pe bază de abonament pentru funcționalități avansate și implementări comerciale
