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
| **Sistem de operare** | Windows 10 (64-bit)                                  | Windows 11 (64-bit)                                  |
| **Procesor**        | Intel Core i5 sau echivalent                          | Intel Core i7 sau superior                              |
| **Memorie (RAM)**     | 8 GB                                                  | 16 GB sau mai mult                                         |
| **Placă grafică**    | Compatibilă cu DirectX 11                                | GPU NVIDIA cu 4 GB+ VRAM                            |
| **Spațiu de stocare**          | 6 GB spațiu liber                                       | SSD cu 10 GB+ spațiu liber                            |
| **Afișaj**          | 1920x1080                                            | 2560x1440 sau mai mare                                  |
| **Internet**         | Necesar pentru activarea licenței [opțional] Chloros+ | Necesar pentru activarea licenței [opțional] Chloros+ |

#### Linux amd64 (x86\_64)

| Cerințe       | Minime                    | Recomandate               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuție**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 sau superior   |
| **Memorie (RAM)**  | 8 GB                        | 16 GB sau mai mult              |
| **Placă grafică** | Niciuna (procesare CPU)      | GPU NVIDIA cu 4 GB+ VRAM |
| **Spațiu de stocare** | 2 GB spațiu liber             | SSD cu 10 GB+ liber       |
| **Python**        | Python 3.7+ (pentru SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Cerință      | Minim                      | Recomandat                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platformă**     | NVIDIA Jetson cu JetPack 6 | Jetson Orin NX 16 GB sau AGX Orin |
| **Memorie (RAM)** | 8 GB (GPU/CPU partajată)         | 16 GB+ partajată                    |
| **Spațiu de stocare**      | 2 GB spațiu liber               | SSD NVMe cu 10 GB+ liber        |
| **Python**       | Python 3.7+ (pentru SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Accelerare GPU**: Utilizatorii Chloros+ cu GPU-uri NVIDIA pot folosi accelerarea CUDA pentru o procesare semnificativ mai rapidă. Aceasta funcționează atât pe Windows (GPU-uri desktop), cât și pe Linux (GPU-uri desktop și NVIDIA Jetson). Utilizatorii Chloros+ beneficiază, de asemenea, de procesare multi-threaded pentru viteză maximă.
{% endhint %}

***

## Descărcați Chloros

### Ultima versiune stabilă (23 martie 2026): Versiunea 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Descărcați Chloros pentru Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Descărcați Chloros pentru Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Descărcați Chloros pentru Linux arm64 / Jetson (.deb)</a>

#### Program de instalare Windows (GUI + CLI + Backend)

* **Tip fișier**: .exe (Program de instalare Windows)**Pași de instalare:**

1. Descărcați fișierul .exe de mai sus
2. Faceți dublu clic pe programul de instalare pentru a începe instalarea
3. Urmați instrucțiunile expertului de instalare
4. Alegeți directorul de instalare (implicit: `C:\Program Files\[USER]\Chloros\`)
5. Finalizați instalarea și lansați Chloros sau Chloros CLI
6. Conectați-vă cu contul dvs. [MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (sau continuați cu versiunea gratuită)

{% hint style="success" %}
Programul de instalare adaugă automat `chloros-cli` la PATH-ul sistemului dvs. pentru accesul la linia de comandă.
{% endhint %}

#### Linux amd64 (Pachet .deb — CLI + Backend)

* **Tip fișier**: .deb (pachet Debian/Ubuntu)
* **Arhitectură**: x86\_64 (amd64)

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

```bash
pip install chloros-sdk
```

Consultați [API : Python SDK](api-python-sdk.md) pentru documentație.

{% hint style="info" %}
**Utilizatori Linux**: Pachetul `.deb` instalează CLI și backend-ul. Python SDK se instalează separat prin pip. Nu există o interfață grafică pentru Linux — toate interacțiunile se realizează prin CLI sau SDK.
{% endhint %}

***

## Resurse suplimentare

### Python SDK

Pentru dezvoltatori și fluxuri de lucru de automatizare, instalați Chloros Python SDK:

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
* ❌ Fără GUI — Linux este doar CLI/SDK fără interfață grafică

### Python SDK (pip, toate platformele)

* ✅ **Chloros SDK** - Python API (necesită licență Chloros+)***

## Actualizați la Chloros+

Deblocați funcții avansate cu un abonament Chloros+:

* 🚀 **Procesare multi-thread** - Procesați imagini în paralel
* ⚡ **Accelerare GPU (CUDA)** - Folosiți puterea GPU-ului NVIDIA
* 💻 **Acces CLI** - Automatizați cu instrumente de linie de comandă
* 🐍 **Python SDK** - Acces programatic API
* 📱 **Dispozitive multiple** - Utilizare pe 2-10+ dispozitive (în funcție de plan)
* **🐻 Metodă avansată de debayer cu recunoaștere a texturii** - un debayer de înaltă calitate, sensibil la margini, combinat cu un model de denoizare AI/ML care elimină aproape tot zgomotul de debayer.
* 🧮 **Formule personalizate** - Creați indici multispectrali personalizați

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Vedeți planurile și prețurile Chloros+</a></p>***

## Ajutor pentru instalare

### Depanare

**Instalarea eșuează cu mesajul de eroare:**

* Asigurați-vă că aveți drepturi de administrator
* Dezactivați temporar software-ul antivirus
* Verificați dacă îndepliniți cerințele minime de sistem

**Aplicația nu pornește (Windows):**

* Verificați dacă Windows 10/11 (64-bit) este instalat
* Actualizați driverele grafice
* Verificați Windows Event Viewer pentru detalii despre eroare
* Contactați asistența tehnică cu jurnalele de erori

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

### Obținerea de asistență

Aveți nevoie de ajutor cu instalarea sau configurarea?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Site web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentație**: [Noțiuni introductive](./)
* ❓ **Întrebări frecvente**: [Întrebări frecvente](faq.md)***

## Jurnal de modificări

<details>

<summary>Versiunea 1.1.0 (Cea mai recentă)</summary>

**Data lansării: martie 2026**

**Funcții noi*** **Suport Linux** — CLI și SDK native pentru Linux amd64 (x86\_64) și arm64 (NVIDIA Jetson JetPack 6). Instalare prin pachetele `.deb`.
* **Suport NVIDIA Jetson** — Procesare optimizată pentru dispozitivele de margine Jetson Nano, Orin Nano, Orin NX și AGX Orin.
* **Adaptare dinamică a calculului** — Detectare automată a hardware-ului și optimizare a strategiei de procesare. Chloros se adaptează la hardware-ul dvs., de la un Jetson Nano la o stație de lucru multi-GPU.
* **Pipeline de procesare cu 4 thread-uri** — Thread-uri concurente de detectare, calibrare, procesare și export cu alocare dinamică a memoriei GPU.
* **Noi comenzi CLI** — `selftest` (diagnosticare sistem) și `update` (gestionarea actualizărilor Linux).
* **Noi indicatori de proces CLI** — `--debayer` (standard/sensibil la textură), `--indices` (specifică indici), `--target` (caută mai întâi în subfolderul țintă pentru o detectare mai rapidă).
* **Noi elemente de meniu GUI** — Adăugare fișiere, Adăugare folder și Pornire/Oprire procesare sunt acum accesibile din meniul derulant principal.**Îmbunătățiri**

* Detectare automată a backend-ului multiplataformă (căile Windows și Linux)
* Îmbunătățiri SDK `get_status()` cu urmărirea progresului pe fiecare thread
* Noi excepții SDK: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Gestionarea termică și limitarea adaptivă pentru NVIDIA Jetson
* Gestionarea automată a memoriei cu fallback OOM către procesarea GPU în blocuri

</details>

<details>

<summary>Versiunea 1.0.5</summary>

**Data lansării: 10 februarie 2026**

**Funcții noi*** **Metoda de debayerizare Texture Aware \[Numai Chloros+] -** Texture Aware utilizează un debayer de înaltă calitate, sensibil la margini, combinat cu un model de denoizare AI/ML care elimină aproape tot zgomotul de debayerizare.
* **Suport pentru ținte de calibrare T4P*** **Procesare GPU Chloros+ mai rapidă, gestionare mai bună a memoriei**

**Remedierea erorilor*** Interfață complet nouă (GUI), ar trebui să funcționeze acum pe toate computerele Windows.

</details>

<details>

<summary>Versiunea 1.0.4</summary>

**Data lansării: 5 ianuarie 2026**

**Funcții noi*** **Comutare imagine/metadate**: S-a adăugat o opțiune de comutare în File Browser pentru a vizualiza metadatele imaginii selectate într-un tabel în loc de grila de imagini
* **Cursor de zoom pentru grila de imagini**: Noul cursor UI pentru ajustarea dimensiunii miniaturilor (acceptă și CTRL + rotița mouse-ului)
* **Butoane de export pentru grila de imagini**: Butoane în rândul de sus pentru a comuta miniaturile de la JPG la exporturi procesate (Ținte, Reflectanță, Index, LUT)
* **Fila Hartă**: Nouă hartă 2D interactivă care afișează marcajele de localizare GPS ale imaginilor
  * Suportă Google Maps și plăci de hartă ESRI (selectează automat cel mai bun serviciu de plăci în funcție de disponibilitatea nivelului de zoom)
  * Previzualizare miniatură la trecerea mouse-ului peste marcajele de pe hartă

**Remedierea erorilor*** Suport îmbunătățit pentru instalarea Chloros pe computere cu limbă de utilizare diferită de engleză

</details>

<details>

<summary>Versiunea 1.0.3</summary>

**Data lansării: 20 decembrie 2025**

**Funcții noi*** Lansare inițială

**Îmbunătățiri*** Lansare inițială

**Remedierea erorilor*** Lansare inițială

**Probleme cunoscute*** Lansare inițială

</details>***

## Acord de licență**Software proprietar** - Copyright (c) 2026 MAPIR Inc.

Utilizarea, distribuirea sau modificarea neautorizată este interzisă.

**Versiune gratuită**: Disponibilă pentru uz personal și comercial, cu funcționalități limitate**Chloros+**: Licență pe bază de abonament pentru funcționalități avansate și implementări comerciale
