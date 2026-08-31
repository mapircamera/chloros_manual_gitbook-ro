# Utilizarea Chloros cu asistenții AI

Acest manual se adresează două categorii de public: oamenii și asistenții AI cu ajutorul cărora oamenii lucrează din ce în ce mai mult. Fiecare pagină prezintă valori exacte, setări implicite și comenzi care pot fi copiate și lipite, astfel încât un asistent (Claude, ChatGPT, Copilot, un agent de programare etc.) să poată scrie un program de automatizare Chloros funcțional încă de la prima încercare.

Versiunea Chloros: **

1.2.0**. Platforme CLI/SDK: Windows 10/11 x64 și Linux (x86_64 / Jetson aarch64).

## Ce să îi dai asistentului tău

| Resursă | URL | La ce servește |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Index lizibil de către mașină pentru fiecare pagină din acest manual. |
| **Referință CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Suprafața completă de comenzi `chloros-cli`: fiecare comandă, indicator, valoare implicită, cod de ieșire și regulă privind folderul de ieșire. Scris pentru utilizare de către LLM. |
| **Referință SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | `chloros_sdk` Python API complet: clase, semnături, excepții și exemple practice. Scris pentru studenții programului LLM. |
| **Orice pagină în format Markdown brut** | adăugați `.md` la pagina URL | de ex. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` returnează pagina în format Markdown brut — ideal pentru a fi lipită într-o fereastră de context sau preluată de la un agent. |

Linkuri din manual: [Referință CLI](reference/cli-reference.md) · [Referință SDK](reference/sdk-reference.md).

{% hint style="info" %}
Cele două pagini de referință sunt autonome: un asistent care a citit una dintre ele nu are nevoie de restul manualului pentru a scrie un script corect.
{% endhint %}

## Rețete de prompt

Copiați, completați `<placeholders>` și lipiți în asistentul dvs.

### 1. Prelucrați un dosar de zbor în NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Monitorizarea în lot a unui director de capturi

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Conectarea unei matrice LATTICE și captarea

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Înregistrați spectrele senzorilor de lumină DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Scripturile DAQ din linia de comandă trec întotdeauna prin familia `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Alte subcomenzi `daq` pe care asistentul dvs. le-ar putea inventa nu sunt disponibile în versiunile livrate și generează o eroare.
{% endhint %}

## De ce scripturile scrise de IA funcționează bine cu Chloros

Fiecare dintre acestea reprezintă un comportament real și verificat al Chloros 1.2.0 — ele elimină modurile clasice de eșec ale automatizării scrise de mașini:

* **Fără proceduri complicate de configurare.**Asistenții de conectare inteligentă ai SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) și punctele de intrare de procesare (`ChlorosLocal`, `process_folder`)**pornesc automat backend-ul local**. Un script generat nu necesită ca interfața grafică să fie deschisă sau ca serverul să fie pornit manual — are nevoie doar de pachetul desktop/CLI instalat.
* **Întregul flux de procesare se realizează printr-o singură comandă.** `chloros_sdk.process_folder("path", indices=["NDVI"])` execută de la început până la sfârșit: import → calibrare → reflectanță → export indice. Cu cât suprafața este mai mică, cu atât sunt mai puține locuri în care un script generat poate da erori.
* **Rulările fără ieșire se autodiagnostică.** După `process()`, rezumatul rulării este atașat la rezultat, iar fiecare indiciu de procesare (de ex. *motivul* pentru care o execuție nu a generat nicio ieșire) este, de asemenea, redifuzat ca un Python `UserWarning` — astfel încât chiar și un script care nu inspectează niciodată dicționarul de rezultate afișează diagnosticul.
* **CLI eșuează în mod evident.**O execuție `chloros-cli process` care a solicitat produse, dar nu a scris niciunul, afișează `Processing finished but wrote no image products.` și**se închide cu un cod de ieșire diferit de zero**, astfel încât scripturile shell și CI o detectează printr-o simplă verificare a codului de ieșire. Execuțiile reușite raportează `Image products written: N`.

O asimetrie pe care un asistent ar trebui să o cunoască: `process()` al SDK **nu** generează în mod deliberat o eroare la o execuție cu zero produse — în schimb, raportează prin rezumat/sugestii. Dacă un pipeline Python trebuie să se oprească în cazul unei rulări goale, verificați rezumatul (rețeta 2 face acest lucru).

## Avertismente

* **Este necesară autentificarea cu Chloros+.**CLI și SDK necesită un nivel**plătit** Chloros+, impus la nivel de server: cererile eșuează cu codul `401 AUTH_REQUIRED` dacă nu sunteți autentificat și cu codul `403 PLAN_UPGRADE_REQUIRED` pe planul gratuit. Rulați `chloros-cli login` o dată pe fiecare mașină înainte de a rula scripturile generate. Consultați [Chloros+ Autentificare](chloros+-login.md).
* **Comenzile de captură controlează hardware-ul real.** Comenzile `lattice` / `daq` / `project` și obiectele de sesiune SDK se conectează, transmit în flux și declanșează camere și senzori fizici. Verificați un script generat înainte de prima sa execuție și rulați-l sub supravegherea personalului de la echipamentele hardware.
* **Verificați aleatoriu rezultatele.** Verificați folderele cu produse și câteva valori ale pixelilor înainte de a publica rezultatele. În special, fișierele TIFF de reflectanță sunt scalate în funcție de sursă — citiți eticheta XMP `Chloros:PixelScale` (LATTICE: 32768 = reflectanță 1,0; Survey3: 65535) în loc să presupuneți un divizor. Ambele pagini de referință documentează acest lucru la secțiunea „Citirea pixelilor de reflectanță”.
* **Mici capcane care pot afecta codul generat:**`pool-record` scrie în sistemul de fișiere al**gazdei backend** (implicit `~/Documents/DAQ Live View/`); pe mașinile cu mai multe interfețe de rețea, preferați `daq pool-connect --eth-host <ip-or-hostname>` în locul detectării automate; și utilizați `http://127.0.0.1:5000` (niciodată `localhost`) oriunde apare un backend URL.
