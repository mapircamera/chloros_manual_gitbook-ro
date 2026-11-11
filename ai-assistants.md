# Utilizarea Chloros cu asistenții AI

Acest manual este destinat două categorii de public: oamenii și asistenții AI cu ajutorul cărora oamenii lucrează din ce în ce mai mult. Fiecare pagină prezintă valori exacte, setări implicite și comenzi care pot fi copiate și lipite, astfel încât un asistent (Claude, ChatGPT, Copilot, un agent de programare etc.) să poată scrie automatizări Chloros-funcționale încă de la prima încercare.

Versiunea Chloros: **

1.3.0**. Platforme CLI/SDK: Windows 10/11 x64 și Linux (x86\_64 / Jetson aarch64).

## Ce să îi oferiți asistentului

| Resursă | URL | La ce servește |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Index lizibil de către mașini pentru fiecare pagină din acest manual. |
| **Referință CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Setul complet de comenzi `chloros-cli`: fiecare comandă, indicator, valoare implicită, cod de ieșire și regulă privind folderul de ieșire. Scris pentru utilizarea de către LLM. |
| **Referință SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | API-ul complet Python al `chloros_sdk`: clase, semnături, excepții și exemple practice. Scris pentru utilizarea de către LLM. |
| **Orice pagină în format Markdown brut** | adăugați `.md` la URL-ul paginii | de ex., `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` returnează pagina în format Markdown brut — ideal pentru a fi lipită într-o fereastră de context sau preluată de la un agent. |

Linkuri din manual: [Referință CLI](reference/cli-reference.md) · [Referință SDK](reference/sdk-reference.md).

{% hint style="info" %}
Cele două pagini de referință sunt autonome: un asistent care a citit una dintre ele nu are nevoie de restul manualului pentru a scrie un script corect.
{% endhint %}

## Permiteți asistentului să ruleze direct Chloros: `chloros-mcp`

Tot ce a fost menționat mai sus presupune ca asistentul dvs. să *scrie* un script sau o comandă. Cu `chloros-mcp`, un asistent care utilizează [Model Context Protocol](https://modelcontextprotocol.io) (MCP), precum Claude Desktop, Claude Code sau Cursor, controlează direct Chloros. Tu soliciți în limbaj simplu, iar acesta apelează instrumentele serverului:

* **Procesarea imaginilor:** creează un proiect, importă un folder cu capturi, alege setările, procesează și raportează ce a exportat execuția.
* **Operarea echipamentelor hardware:** conectează camerele LATTICE, matricile și senzorii de lumină DAQ, afișează previzualizarea, setează expunerea și culorile, capturează, înregistrează și actualizează firmware-ul.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Instrumentele serverului acoperă funcționalitățile oferite de [Python SDK](api-python-sdk.md). Acesta se instalează odată cu SDK-ul, așa că, de obicei, nu mai este nevoie să instalați nimic altceva: pe Windows, programul de instalare Chloros plasează `chloros-mcp.exe` în folderul `Scripts` al Python, iar cel pentru Linux, `.deb`, plasează fișierul `chloros-mcp` în `/usr/bin`.

**Cod Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**adăugați acest lucru în `claude_desktop_config.json` (se deschide din**Setări > Dezvoltator > Editare configurație**), apoi reporniți Claude Desktop.**Cursorul** preia aceeași intrare din `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Dacă clientul indică faptul că serverul nu a reușit să pornească, probabil că `chloros-mcp` nu se află în `PATH` pe care îl vede: introduceți în schimb calea completă (`where chloros-mcp` pe Windows, `which chloros-mcp` pe Linux).

Trebuie să aveți Chloros instalat pe același computer și să vă autentificați acolo, în aplicație sau cu `chloros-cli login`, la orice nivel plătit Chloros+ (Copper sau superior). Serverul nu se poate autentifica în locul dvs.

{% hint style="info" %}
Configurarea clientului pentru fiecare aplicație, opțiuni, fiecare instrument și modul în care se desfășoară o execuție: [Server MCP (Asistenți AI)](reference/sdk-reference.md#mcp-server-ai-assistants) în Referința SDK.
{% endhint %}

## Rețete de prompturi

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

### 2. Monitorizare în lot a unui director de capturi

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Conectarea unei matrice LATTICE și captură

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Înregistrarea spectrelor senzorilor de lumină DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Scripturile DAQ din linia de comandă trec întotdeauna prin familia `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Alte subcomenzi de tip `daq` pe care asistentul dvs. le-ar putea inventa nu sunt disponibile în versiunile livrate și generează o eroare la ieșire.
{% endhint %}

## De ce scripturile scrise de IA funcționează bine cu Chloros

Fiecare dintre acestea reprezintă un comportament real și verificat al Chloros 1.3.0 — ele elimină modurile clasice de eșec ale automatizării scrise de mașini:

* **Fără proceduri complicate de configurare.**Asistenții de conectare inteligentă ai SDK-ului (`connect_camera`, `connect_array`, `connect_daq_sensor`) și punctele de intrare de procesare (`ChlorosLocal`, `process_folder`)**pornesc automat backend-ul local**. Un script generat nu are nevoie de interfața grafică deschisă sau de un server pornit manual — are nevoie doar de pachetul pentru desktop/interfața CLI instalat.
* **Întregul flux de procesare se realizează printr-o singură comandă.** `chloros_sdk.process_folder("path", indices=["NDVI"])` execută de la cap la coadă: import → calibrare → reflectanță → export indice. Cu cât suprafața este mai mică, cu atât sunt mai puține locuri în care un script generat poate da erori.
* **Rulările fără ieșire se autodiagnostică.** După `process()`, rezumatul rulării este atașat la rezultat, iar fiecare indiciu de procesare (de ex. *motivul* pentru care o execuție nu a generat nicio ieșire) este, de asemenea, retransmisă ca un script Python `UserWarning` — astfel încât chiar și un script care nu verifică niciodată dicționarul de rezultate afișează diagnosticul.
* **CLI-ul eșuează în mod evident.**O execuție `chloros-cli process` care a solicitat produse, dar nu a scris niciunul, afișează `Processing finished but wrote no image products.` și**se închide cu un cod de ieșire diferit de zero**, astfel încât scripturile shell și CI o detectează printr-o simplă verificare a codului de ieșire. Execuțiile reușite raportează `Image products written: N`, plus `(plus M clamp masks)` atunci când au fost scrise măști. La fel, comenzile de captură menționează fiecare fișier pe care nu l-au putut scrie și se închid cu `1`.
* **Căile relative au semnificația pe care o așteaptă un script.** Cu un backend local, o cale relativă de ieșire (`-o out/` în CLI, `output_dir="out/"` în SDK) este rezolvată în raport cu folderul din care se execută comanda sau scriptul.

O asimetrie pe care un asistent ar trebui să o cunoască: `process()` din SDK **nu** generează în mod deliberat o excepție în cazul unei rulări cu produs zero — în schimb, raportează acest lucru prin rezumat/sugestii. Dacă un pipeline Python trebuie să se oprească în cazul unei rulări fără rezultate, verificați rezumatul (rețeta 2 face acest lucru).

## Avertismente

* **Este necesară autentificarea Chloros+.**CLI-ul și SDK-ul necesită un nivel**plătit** de Chloros+, impus la nivel de server: cererile eșuează cu eroarea `401 AUTH_REQUIRED` dacă nu sunteți autentificat și cu eroarea `403 PLAN_UPGRADE_REQUIRED` în cazul nivelului gratuit. Rulați `chloros-cli login` o dată pe fiecare mașină înainte de a rula scripturile generate. Consultați [Autentificarea Chloros+](chloros+-login.md).
* **Comenzile de captură controlează hardware-ul real.** Comenzile `lattice` / `daq` / `project`, obiectele de sesiune SDK și instrumentele `chloros-mcp` se conectează, transmit în flux și declanșează camere și senzori fizici. Verificați un script generat înainte de prima sa execuție și rulați-l în prezența echipamentului hardware.
* **Verificați aleatoriu rezultatele.** Verificați folderele produsului și câteva valori ale pixelilor înainte de a publica rezultatele. În special, fișierele TIFF de reflectanță sunt scalate în funcție de sursă — citiți eticheta XMP `Chloros:PixelScale` (fișierele LATTICE o conțin: 32768 = 1,0 reflectanță; fișierele Survey3 nu o conțin și folosesc 65535 = 1,0) în loc să presupuneți un divizor. Ambele pagini de referință documentează acest lucru la secțiunea „Citirea pixelilor de reflectanță”.
* **Mici capcane care pot afecta codul generat:**`pool-record` scrie în sistemul de fișiere al**gazdei backend** (implicit `~/Documents/DAQ Live View/`); pe mașinile cu mai multe interfețe de rețea, preferați `daq pool-connect --eth-host <ip-or-hostname>` în locul detectării automate; utilizați `http://127.0.0.1:5000` (niciodată `localhost`) oriunde apare un URL de backend; un script care parcurge arborele de ieșire al unui proiect ar trebui să omită folderele `clamp_masks/` (fișierele `*_clampmask.tif` din acestea sunt măști, nu imagini — a se vedea [Formate de imagine de ieșire](output-image-formats.md#clamp-masks)); iar pe Windows, cu opțiunea pentru căi lungi dezactivată, păstrați folderele de proiect scurte — un folder de proiect cu o lungime mai mare de aproximativ 130 de caractere este respins înainte de începerea procesării.
