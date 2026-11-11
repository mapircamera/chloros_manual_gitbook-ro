# Referință Chloros CLI

**Versiune:**

1.3.0**Generat:**29 iulie 2026, ora 19:19 ·**Revizuit:** 6 noiembrie 2026**Public țintă:** Optimizat pentru utilizarea de către LLM; ușor de citit de către oameni.**Domeniu de aplicare:** Fiecare subcomandă a `chloros-cli` destinată utilizatorilor, cu opțiuni și exemple care pot fi copiate și lipite.

Acest document reprezintă referința completă pentru instrumentul de linie de comandă `chloros-cli` livrat împreună cu MAPIR Chloros. Este intenționat exhaustiv, astfel încât un LLM (sau o persoană) să poată compune orice flux de lucru acceptat din listele de mai jos fără a inspecta codul sursă.

Dacă aveți nevoie doar de informațiile esențiale, accesați:
- [Ghid de pornire rapidă în cinci minute](#five-minute-quickstart)
- [Fluxul de lucru pentru prima conectare a camerei LATTICE](#lattice-camera-first-connect-workflow)
- [Fluxul de lucru pentru prima conectare a senzorului DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Moduri de captură, înregistratoare și reprocesare offline](#capture-modes-recorders--offline-reprocess)

> **Utilizați Chloros-ul printr-un asistent AI?** Claude Desktop, Claude Code, Cursor și alți clienți MCP pot rula procesarea, camerele și matricile LATTICE, precum și senzorii de lumină DAQ prin `chloros-mcp`, serverul MCP care se instalează odată cu Python SDK (și cu pachetul .deb pentru Linux, în `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Configurarea, opțiunile și instrumentele se găsesc în secțiunea [Server MCP (asistenți AI)](sdk-reference.md#mcp-server-ai-assistants). Un asistent cu un shell poate, de asemenea, să execute direct fiecare comandă de pe această pagină.

> **Modificări în versiunea 1.3.0** (ce observă un script sau un flux de lucru existent din versiunea 1.2.0).
>
> **Modificări pe care un script din versiunea 1.2.0 ar putea fi nevoit să le gestioneze:**
>
> - **Se aplică `--preset`.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` și `align-stream` au renunțat la aceasta în versiunea 1.2.0 și au funcționat pe baza setărilor proprii ale camerei. Acum se setează formatul presetării, expunerea, amplificarea și rata de cadre: `--preset high_speed` capturează 8 biți la 5 ms, 6 dB, 60 fps. `lattice cam-connect --preset` păstrează și formatul presetării (în versiunea 1.2.0, camera era setată la 12 biți după aplicarea presetării). Vezi [Opțiuni comune](#common-options-shared-by-most-subcommands).
> - **Un format de pixeli Bayer pe o cameră M3M (mono) reprezintă adâncimea acesteia.** Presetările denumesc formate Bayer, pe care un senzor mono nu le poate înregistra: în versiunea 1.2.0, camera a refuzat formatul și l-a păstrat pe cel pe care îl avea, așa că `--preset high_speed` nu era de 8 biți în acel caz. Acum este formatul Mono cu aceeași adâncime: `high_speed` și `--pixel-format BayerRG8` dau `Mono8`, `high_quality`, `triggered` și `BayerRG12` dau `Mono12`. Într-o matrice mixtă M3C + M3M, fiecare cameră preia formatul comun din propria familie, iar `network-analysis` / `analyze-array --pixel-format` proiectează fiecare cameră la acea adâncime. Același lucru se întâmplă și invers: o cameră M3C pentru care s-a solicitat un format Mono preia formatul Bayer al acelei adâncimi, astfel încât o matrice al cărei master este un M3M nu mai redeschide camerele sale M3C în format Mono. Vezi [Camere mono (M3M) cu o singură bandă](#mono-m3m-single-band-cameras).
> - **Exporturile `lattice array-capture` sunt aliniate în mod implicit, la fel ca cele ale aplicației.** Fiecare fișier `debayered` al membrilor, `display`, `radiance`, `reflectance` și fișierele index sunt aliniate la cadrul camerei principale și decupate la zona de suprapunere a membrilor (mai mică decât senzorul cu valorile de decalaj ale camerelor, `XMP-mapir:AlignmentJSON` `"applied": true`), astfel încât o captură CLI conține aceleași fișiere ca și captura aceleiași scene realizată de aplicația Chloros; `raw` rămâne în formatul nativ. Anterior, o captură fără `--aligned` salva fiecare fișier la dimensiunea nativă, fără aliniere. `--no-aligned` păstrează această caracteristică. Aceeași setare implicită se aplică pentru `--smart`, `--continuous` și `--interval`, precum și pentru SDK și `chloros-mcp`. Consultați [modurile `array-capture`](#array-capture-modes).
> - **`lattice array-capture --compression` are ca setare implicită `none`.** Fișierele sunt scrise necomprimate (aproximativ 6,3 MB per cadru la rezoluție maximă) pentru cea mai rapidă rată susținută; versiunea 1.2.0 scria `deflate` (aproximativ 4,1 MB). Utilizați `--compression deflate` pentru fișiere mai mici. Ambele sunt fără pierderi.
> - **Formatele PNG și JPEG acceptă doar `display` / `preview` și produsele indexate.** O captură `--format png` sau `jpg` cu un nivel pe care un fișier de 8 biți nu îl poate stoca este refuzată înainte de a se efectua vreo captură (400 HTTP, ieșire `1`), la fel și `--processing all` implicit cu `--format png`/`jpg`. Consultați [Niveluri de export al capturilor](#capture-export-levels-the-all-default).
> - **`lattice dls` necesită `--mac`.**Nu mai există un MAC implicit (era adresa unei unități); `daq pool-discover --only ble` găsește adresa unui senzor.**`lattice dls --camera SERIAL`** este nou.
> - **Alte erori determină ieșirea din `1`.** `process` apare atunci când exporturile unei rulări nu au putut fi verificate sau s-a pierdut o imagine, deși celelalte produse sunt scrise; o comandă de captură cu orice fișier pe care nu l-a putut scrie (celelalte fișiere se salvează în continuare); `lattice continuous` și `multi-stream` în cazul unui disc plin; `array-burst` / `array-record` atunci când înregistratorul raportează o eroare; `array-build-video` atunci când un produs solicitat a fost omis sau nu a fost creat conform cererii; `project burst` / `stream` atunci când o captură eșuează pe parcurs; `project sensor log` atunci când înregistrarea nu conține nicio citire; `lattice exposure --set`, `gain --set` și `configure --set` cu o valoare pe care camera o refuză. `lattice network-analysis --json` se închide odată cu rezumatul (`1` pentru `needs_force_slip`). În sens invers: `daq pool-disconnect` al unui senzor care nu mai face parte din grup se închide cu codul `0`. A se vedea [Coduri de ieșire](#exit-codes).
> - **Un `lattice array-connect` lent nu eșuează niciodată cu matricea conectată.** Așteaptă până la 600 s (180 s în versiunea 1.2.0; `project connect` 90 s), afișând pasul backend-ului pe măsură ce acesta avansează, iar o conexiune care nu este finalizată până atunci este anulată de backend — fiecare cameră fiind readusă la starea inițială — astfel încât un `Error:` (ieșire `1`) înseamnă că nu este conectat niciun ansamblu. În versiunea 1.2.0, o conectare efectuată imediat după pornirea camerelor putea depăși timpul de așteptare și putea conecta matricea după ce comanda eșuase. Acum, Ctrl+C anulează conectarea. A se vedea [Conexiuni persistente](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` păstrează senzorul complet.** În absența opțiunilor `--width` / `--height` / `--binning`, fiecare cameră este setată pe senzorul complet, iar rata de cadre scade la nivelul oferit de acest computer și de rețea; ieșirea de conectare listează opțiuni mai rapide (binning sau o zonă de interes mai mică) împreună cu ratele estimate. Versiunea 1.2.0 a redus sau a grupat cadrele pentru a menține rata ridicată. Consultați [Opțiunile `array-connect`](#array-connect-options).
> - **O rețea lentă nu mai blochează funcționarea unui array.** Fiecare cameră este sincronizată în funcție de lățimea de bandă disponibilă, iar rata de cadre scade în consecință. În cazul în care camerele nu pot fi sincronizate la o viteză atât de redusă, array-ul funcționează în modul de emisie serială (`slip-emit-and-capture`): fiecare cadru ajunge, dar camerele nu sunt expuse simultan. `--force-tier sim-capture-sim-emit` insistă asupra expunerii simultane. A se vedea [FPS matrice &amp; modelul de rafală](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` proiectează ce se aplică la „connect”**: fără `--pixel-format`, setările de conectare ale formatelor (Mono12 / BayerRG12, anterior BayerRG8) și decizia privind [transportul comprimat pe 12 biți](#packed-12-bit-transport---packed). Prin urmare, aceeași comandă poate indica o rată de cadre diferită față de versiunea 1.2.0.
> - **Căile relative se referă la locația în care se execută comanda.** `-o` și celelalte căi deschise de backend (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), `output` al unui pas de rețetă, iar `output/` implicit al `lattice index --live` sunt rezolvate în funcție de folderul din care rulați `chloros-cli`. În versiunea 1.2.0, backend-ul le rezolva în propriul său folder de instalare, ceea ce, pe o versiune instalată, genera eroarea „Accesul este refuzat”. Consultați [Convenții](#conventions).
> - **`lattice index` rulează pe backend în orice formă**, la fel ca celelalte comenzi `lattice`, așa că porniți mai întâi backend-ul. Indexul offline (`--input`) este calculat de backend, care citește fișierul TIFF și scrie rezultatele lângă acesta sau în `-o`, iar `--list-presets` / `--list-gradients` afișează tabelele backend-ului. Pe Windows, acestea funcționează acum: înainte, CLI-ul Windows răspundea la orice formă de `lattice index`, inclusiv `--live`, cu `Error: lattice_sdk not available`. Pe Linux, indexul offline rula în interiorul CLI fără backend și fără autentificare; acum are nevoie de ambele, la fel ca orice altă comandă CLI. Vezi [Index / Matematică privind vegetația](#index--vegetation-maths).
> - **Fișiere noi lângă produse.** Fiecare produs de radianță, reflectanță și indice în care separarea benzilor a limitat un pixel primește o mască într-un dosar `clamp_masks/` lângă dosarul produsului, iar capturile de ecran afișează o linie `clamp mask:`. Consultați [Masca de limitare a separării](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Orientarea camerei.** O captură cu o singură cameră salvează fișierele `raw` și `debayered` așa cum au fost citite de senzor și înregistrează răsturnarea/rotirea camerei în XMP; `process` aplică aceste informații produselor pe care le creează. Fișierele membre ale matricei rămân în formatul nativ, cu excepția cazului în care se transmite noul `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Consultați [Orientarea camerei](#cum-arată-un-dosar-de-capturi).
> - **Un nume de nivel de export care nu corespunde este respins.** `lattice capture --processing` și fiecare listă `--levels` acceptă `raw`, `debayered`, `radiance`, `reflectance` sau `display` (sau un alias precum `preview`; `--processing` și, de asemenea, `all`). Orice alt nume este respins înainte ca camera să fie atinsă (400 HTTP `UNKNOWN_LEVEL`, ieșire `1`); 1.2.0 a salvat `raw` pentru aceasta și a ieșit `0`.
> - **`daq pool-set-cap` are nevoie de `--cap-id`.** Fără acesta, comanda iese din `2`; 1.2.0 a eliminat corecția capacului, lăsând senzorul neacoperit. `--cap-id none` solicită senzorul neprocesat.
> - **`lattice resolution --binning-mode` a dispărut.** Binning-ul este întotdeauna „Average”; indicatorul este încă acceptat, afișează un avertisment și nu face nimic. Consultați [Reglarea senzorului](#sensor-tuning).
> - **`process --debayer texture-aware` este refuzat pentru capturile LATTICE.** Pentru un folder care conține capturi LATTICE, `process` iese din `1` după import, indicând numărul acestora, înainte de a procesa ceva. Capturile LATTICE beneficiază întotdeauna de demosaic standard, așa că versiunea 1.2.0 a acceptat opțiunea și a ignorat-o; nici aplicația Chloros nu o oferă pentru acestea. Vezi [`chloros-cli process`](#chloros-cli-process).
> - **Camerele revin la starea în care au fost găsite — cu excepția cazului în care se utilizează `--keep-settings`.** `lattice array-disconnect`, fiecare comandă tranzitorie (`multi-capture` fără matrice, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) și `lattice continuous` readuc fiecare cameră la starea inițială: cablajul declanșatorului și al liniei de sincronizare, PTP-ul acesteia, precum și toate setările camerei modificate în timpul sesiunii (expunere și amplificare, ținta AE, limite și amortizare, balansul de alb, nivelul de negru, formatul pixelilor, binning și ROI, răsturnări, rata de cadre). `--keep-settings`, în schimb, salvează setările fiecărei camereîn memoria proprie a camerei, astfel încât aceasta să pornească cu ele; `--clear-settings` resetează memoria respectivă la setările din fabrică. În ambele cazuri, camerele sunt lăsate să funcționeze în mod liber. Un proiect își configurează camerele pornind de la propriile setări salvate atunci când le conectează. Consultați [Memoria camerei](#camera-memory----keep-settings----clear-settings).
> - **Fiecare mod salvează un proiect în același mod.** Orice modificare adusă unei camere, unui array sau unui senzor de lumină conectat la un proiect — din aplicație, din CLI, din SDK sau de pe serverul MCP — este salvată în fișierul `cameras.json` / `sensors.json` al respectivului proiect, iar deschiderea proiectului în orice mod configurează dispozitivele pe baza acestuia. Anterior, doar aplicația salva proiectul, iar următoarea salvare a acesteia suprascria orice altceva. Consultați [Un singur înregistrare de proiect, pentru fiecare mod](#one-project-record-every-mode).
> - **Fișierele create din mai multe camere menționează fiecare dintre ele.** O stivă multibandă (`align-apply`, `align-stream`, `index --live --save-multiband`), un index live creat din benzile mai multor camere și imaginea `composite/` dintr-o captură de tip matrice conțin seria, modelul și pachetul de calibrare ale fiecărei camere sursă sub formă de liste (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), iar listele EXIF `SerialNumber` conțin numerele de serie separate prin virgulă. Anterior, acestea nu conțineau nicio cameră sau conțineau doar numărul de serie al primei camere. A se vedea [Fișiere create din mai multe camere](#files-made-from-several-cameras).
> - **Doar un fișier `raw` conține etichetele de calibrare ale camerei.** Setul compatibil cu MicaSense (`BlackLevel`, `RadiometricCalibration`, fișierul pe canal `Band{R,G,B}RadiometricCalibration`, seturile `MicaSenseVignetting*`) și încărcătura utilă `CalibrationJSON` descriu valorile brute ale senzorului, deci se află doar în fișierul de captură `raw`. Toate celelalte fișiere — fișierele de captură `debayered`, `display`, `radiance`, `reflectance` și fișierele index, cadrele `lattice continuous` și `multi-stream`, și fiecare produs `process` — conține identitatea camerei (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) și, la o captură, înregistrarea specifică cadrului (`XMP-mapir:FrameJSON`: expunere, amplificare, temperatură, format de pixeli, ora capturii). În versiunea 1.2.0, acestea conțineau și setul de date brute, pe care un instrument care îl aplică (Pix4D, Metashape) îl aplica pixelilor care nu sunt valori brute. `process` identifică calibrarea unui astfel de fișier după numărul de serie și `CalBundleSha256`.
>
> **Alte modificări:**
>
> - **Rata automată de cadre a unui array se stabilește în funcție de ceea ce furnizează array-ul.** O rată care pierde mai mult de 1 % din declanșări este redusă, evaluarea fiind făcută pe baza mai multor perioade lungi de monitorizare, nu doar una singură, iar rata de la care a fost redusă este reîncercată din când în când. O rată setată manual rămâne neschimbată.
> - **Expunerea automată este mai stabilă.** Se menține constantă în condiții de lumină constantă, se stabilizează în câteva secunde după o schimbare, ignoră zonele foarte luminoase și restabilește funcționarea camerei care nu mai răspunde în lumina soarelui puternică. Vezi [Cum funcționează expunerea automată](#how-auto-exposure-behaves).
> - **`process` numără doar imaginile și înregistrările tale.** Dacă rulați din nou pe un folder care conține fișierul `camera_cal/` al unui proiect, acesta nu mai numără acele fișiere de calibrare („S-au găsit 96 de imagini” într-un set de 90 de imagini), iar un proiect care le-a preluat deja le elimină.
> - **`lattice multi-test` pornește de la starea de pornire a camerelor.** Activează PTP-ul camerelor pentru execuție, așteaptă ca ceasurile acestora să se sincronizeze și să se stabilizeze înainte de primul declanșator — fiecare cameră rămâne sincronizată timp de 120 s, aproximativ 2 minute de la pornire (`--ptp-settle SECONDS` îl setează), deoarece un ceas abia sincronizat prezintă o abatere de până la ~200 us timp de aproximativ 2 minute — și readuce PTP-ul fiecărei camere la starea inițială. O cameră care este încă sincronizată de la un test anterior nu mai este așteptată din nou. Anterior, lăsa PTP dezactivat într-o rețea cu un singur comutator în care niciun DAQ-E nu răspundea și citea ceasul liber al fiecărei camere ca pe un ceas PTP care nu se sincroniza (`PTP is not locked`, la fiecare rulare). A se vedea [Multi-Cameră (Sesiuni tranzitorii)](#multi-camera-transient-sessions).
> - **O înregistrare `.daq` indică dacă capacul a fost ales sau presupus.** DAQ-U, DAQ-M și DAQ-E sunt livrate fără capac, iar Chloros nu poate detecta dacă este montat unul, astfel încât un senzor conectat fără un capac specificat primește implicit `sunshine_cosine`, o *presupunere*. Fiecare înregistrare Chloros (schema 1.30) stochează acum `cap_id_source` alături de `cap_id`: `operator` atunci când ați denumit un capac (inclusiv `none`), `auto_default` atunci când acesta a fost cel implicit. La procesarea unei înregistrări al cărei capac a fost presupus, cu capacul DAQ al proiectului lăsat pe „Auto”, afișează un mesaj `[daq-dls] WARNING` care menționează fișierul: un DAQ-U fără cap, sub corecția de lumina soarelui, înregistrează valori de aproximativ 30 de ori mai mari. **Denumiți capul pe care l-ați montat** la conectare sau setați capul DAQ al proiectului, iar avertismentul va dispărea.
> - **Nu mai apar ferestre de consolă „pip” pe Windows.** Pe un PC cu Python instalat, backend-ul și fiecare dintre procesatoarele sale de imagini rulau `pip` o dată la pornire, iar fiecare a deschis o fereastră de consolă. Acum nu mai fac asta.
> - **Înregistrările live `.daq` supraviețuiesc unei întreruperi de curent.**O înregistrare este scrisă cu un jurnal și o sincronizare completă a discului la fiecare confirmare, în spatele senzorului, astfel încât un disc lent întârzie fișierul, dar niciodată citirile. O înregistrare oprită în mijlocul unei confirmări (întrerupere de curent, oprire forțată) poate lăsa `<name>.daq-journal` lângă ea:**copiați cele două împreună.**Chlorosul revine la jurnalul anterior la prima citire a fișierului, astfel încât înregistrarea se deschide cu tot ce a fost confirmat înainte de oprire (se pierd aproximativ ultimele 3 secunde); în versiunea 1.2.0, o întrerupere a alimentării cu energie electrică la momentul nepotrivit putea face ca întregul fișier să devină ilizibil („imaginea discului bazei de date este defectă”).
> - **O înregistrare pe un disc ocupat nu mai blochează senzorul.** `daq pool-record` pune citirile în coadă și le scrie pe un thread propriu; o citire care nu a putut fi scrisă este contabilizată (`daq pool-list` afișează `N write error(s)` cu ultima eroare) în loc să blocheze senzorul (o scriere a durat 8,4 s lângă patru camere care salvau fișiere TIFF).
> - **`lattice multi-capture` fără matrice nu mai pierde cadrele salvate atunci când un declanșator depășește timpul de așteptare.** Un declanșator al unei camere care depășește timpul de așteptare (`SC_ERR_TIMEOUT -1011`, observat cu patru camere pe un singur switch de 1 GbE) încheie rularea la acel grup de cadre: grupurile deja salvate sunt listate cu `Saved:`, apoi eroarea, ieșire `1`. În versiunea 1.2.0 era un cod de eroare HTTP500 fără nicio listă, deși fișierele se aflau pe disc.
> - **Camerele declanșate prin software nu mai omit fiecare al doilea declanșator.** O cameră în modul de declanșare prin software pune acum în coadă un declanșator care sosește în timpul citirii sale (sau Chloros așteaptă ca aceasta să fie gata), așa cum făceau deja camerele sincronizate prin cablu.
> - **`daq pool-discover` denumește un DAQ-E-S `DAQ-E-S`** în coloana sa `MODEL` (iar `--json` în coloana `model`). Se afișa `DAQ-E` pentru fiecare senzor Ethernet; cel care nu răspunde la verificarea de identitate în aproximativ 2 s este încă listat, ca `DAQ-E`.
> - **Produse îmbunătățite ale indicilor LATTICE** (NDVI și alți indici) începând cu `process`, în toate modurile de procesare.
> - **Camere OCN: produse îmbunătățite privind radianța, reflectanța și indicii în lumina soarelui.** Se aplică o corecție de flotă după separarea benzilor; camerele RGN rămân neschimbate, iar valorile OCN diferă față de cele din versiunea 1.2.0. Aceasta se aplică și unei capturi OCN realizate cu o versiune anterioară, atunci când este procesată pe un computer care deține calibrarea cu care a fost realizată captura (programul Chloros păstrează calibrarea camerei odată ce aceasta a fost conectată).
> - **`lattice continuous` înregistrează cadrul brut.** O execuție a programului `tiff` a generat cadre demosaicate în 3, pe care `process` nu le poate utiliza ca punct de plecare. Cadrele sale sunt acum fișierul `raw` pe care îl scrie o captură de tip matrice: un plan mozaicat la valorile proprii ale senzorului, necomprimat, împreună cu pachetul de calibrare. Un script care citește trei planuri din aceste fișiere citește doar unul.
>
> - **Fiecare captură salvează citirea DAQ pe care a utilizat-o, la fel cum o face interfața grafică.** `lattice capture`, `lattice continuous` (câte un `.daq` per cadru), `array-capture` și `multi-capture`, cu o matrice conectată (și SDK), scriu fișierul `.daq` și `.match.json` ori de câte ori un DAQ este asociat sau inclus într-un pool, inclusiv în cazul unei capturi exclusiv în format brut; în versiunea 1.2.0, o captură brută sau debayered se salva doar cu `--force-daq`. `--no-daq` o dezactivează, `--daq-tolerance-ms MS` setează toleranța de potrivire pentru acea captură, iar noul `daq pool-bind` leagă o cameră sau o matrice la un DAQ din CLI. O captură salvată cu `.daq` este potrivită în `process` numai cu camerele pe care le numește. O captură așteaptă acum următoarea citire a DAQ-ului, astfel încât să poată combina citirile de pe ambele părți ale expunerii. Consultați [Citirea DAQ-ului pe care o salvează o captură](#the-daq-reading-a-capture-saves) și [O citire DAQ departe de expunere](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** adaugă indexul la proiect pentru benzile fiecărei camere, inclusiv un proiect nou. În versiunea 1.2.0, un nume pe care proiectul nu îl lista încă era ignorat cu `Unknown index name(s) ignored`.
> - **`process` lângă o matrice activă:**nu mai repornește un backend care conține camere sau DAQ-uri și așteaptă până la 15 s pentru o execuție care se apropie de final.**Ctrl+C** oprește execuția în backend, nu doar în CLI. Consultați [Notele privind `process`](#notes).
> - **Un dosar de proiect prea adânc pentru limita de 260 de caractere a căii de acces din Windows** este respins din start, alegându-se cel mai lung dosar care se încadrează, în loc să se execute o rulare care nu scrie nimic. Programul de instalare pentru Windows oferă acum opțiunea „Activați căile lungi Windows” și o excludere Microsoft Defender pentru backend, ambele bifate implicit. Consultați [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** execută propria captură continuă a backend-ului: o linie de progres la fiecare 2 s, un rezumat în loc de o listare per fișier și o ieșire cu cod diferit de zero în cazul oricărui fișier nescris sau al unui disc plin. Consultați [modurile `array-capture`](#array-capture-modes).
> - **Primul `array-capture` după `array-connect`** așteaptă până la 20 s pentru primul grup de cadre al matricei, în loc să genereze eroarea HTTP 503 „Matricea încă se încălzește”.
> - **Expunerea matricei sub o zonă de interes (ROI):** o zonă de interes mică nu mai limitează expunerea unui element al matricei la perioada de funcționare liberă a senzorului (4,9 ms la 768 × 304). Consultați [Expunerea unui element al matricei](#exposure-on-an-array-member).
> - **Expunerea automată nu mai reduce rata automată de cadre a matricei.** O matrice sincronizată prin cablu funcționează la rata permisă de rețea și de citirea camerelor; expunerea automată umple perioada oferită de această rată, apoi crește amplificarea. O expunere manuală mai lungă decât perioada (`lattice exposure --set`, `configure --set`) încetinește declanșarea pentru a se adapta la aceasta, în loc să fie refuzată, iar rata revine la normal când elementul revine la modul automat. A se vedea [Expunerea pe un element al matricei](#exposure-on-an-array-member).
> - **Matricea generează mai puține grupuri de cadre incomplete** și mai puține erori 503 de captură („Nu există un grup de cadre sincronizat recent”): semnalul de declanșare al fiecărei camere slave este acum stabilizat.
> - **Transport comprimat pe 12 biți** (`--packed`, implicit `auto`) crește rata de cadre a matricei cu senzor complet cu aproximativ o treime. `array-connect --packet-size` este acum aplicat (versiunea 1.2.0 îl accepta, dar îl ignora). A se vedea [Transport comprimat pe 12 biți](#packed-12-bit-transport---packed).
> - **O matrice ale cărei camere se restabilesc automat după repornirea alimentării.** După repornirea comutatorului de alimentare, matricea `array-connect`-deschide camerele atunci când acestea răspund și reia transmisia cu PTP reactivat; în versiunea 1.2.0 se oprea definitiv și era necesar un nou `array-connect`. `array-status` raportează `ptp_enabled: False` (și `ptp_off:`) în timp ce o cameră citește PTP ca fiind dezactivat, în timp ce înainte raporta setarea cu care era conectat sistemul. A se vedea [Conexiuni persistente](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` înregistrează fără ca interfața grafică să fie deschisă.** Cu vizualizarea combinată a interfeței grafice închisă, backend-ul redă vizualizarea în sine:NDVI al matricei sau indexul pe care îl transmiteți cu noul `--expr`. O înregistrare care nu scrie niciun cadru eșuează acum (`no-frames: …`, ieșire `1`). Anterior, înregistrarea avea loc doar cât timp vizualizarea combinată a GUI-ului era deschisă, iar o înregistrare care nu scria nimic ieșea totuși din `0`. Vezi [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **O cameră pe care Chloros o deține deja nu este deschisă a doua oară.** O comandă care ar încerca să o deschidă primește în schimb un răspuns HTTP 409 `CAMERA_HELD`, indicând ce o deține și comanda care o eliberează. Aceasta include o cameră pe care o altă comandă o utilizează în acest moment (o comandă de tip `multi-test` sau `multi-capture` care își deschide camerele, o comandă de tip `lattice capture`): a doua comandă este refuzată până când prima se finalizează. Deschiderea acesteia din nou ar putea opri fluxul de date al deținătorului sau ar putea bloca backend-ul. A se vedea [Conexiuni persistente](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Comenzile tranzitorii pentru mai multe camere transmit toate camerele sau eșuează.** `lattice multi-capture` fără matrice conectată, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` și `index --live` ies din `1`, indicând o cameră care nu a transmis niciun cadru. Fluxurile sar peste un grup de cadre în care lipsește o cameră și se opresc după trei cadre consecutive; `multi-test` afișează un rând pentru fiecare declanșare, `(no frame)` indicând unde o cameră a ratat-o; `align-stream` iese, de asemenea, din `1` la o eroare de execuție. Fiecare execuție readuce apoi fiecare cameră la starea în care a găsit-o: cablarea declanșatorului și a liniei de sincronizare, PTP-ul (`PtpEnable`) și toate setările camerei modificate de execuție — expunere și amplificare, țintă AE, limite și amortizare, balans de alb, nivel de negru, format de pixeli, binning și ROI, răsturnări, rată de cadre — cu excepția cazului în care `--keep-settings` / `--clear-settings` (vezi [Memoria camerei](#camera-memory----keep-settings----clear-settings)). Anterior, aceste comenzi păstrau orice cadre primeau și ieșeau din `0` (`align-stream` afișa o eroare de sarcină ca `Note:`), și lăsau fiecare cameră în `TriggerMode` `On`. De asemenea, setau expunerea fiecărei camere pe baza scenei anterioare înainte de a păstra un cadru (cel mult câteva secunde), pe țintă, o matrice conectată pornește fiecare cameră la: 40 multispectral, 60 RGB (`--brightness N` o setează) și o mențin pe durata rulării; la fel procedează și un singur `lattice capture` al unei camere pe care Chloros nu o deține. Anterior, primul cadru al fiecărei camere era capturat la limita de expunere proprie a camerei, aproximativ 100 ms, indiferent de lumină, și era adesea decupat. A se vedea [Multi-Camera (Sesiuni tranzitorii)](#multi-camera-transient-sessions).
> - **Comenzile tranzitorii multi-cameră partajează rețeaua la fel cum o face `array-connect` și deschid doar `--serials`.** Fiecare cameră este sincronizată cu cota sa din legătura gazdă, astfel încât patru camere funcționează pe o legătură de 1 Gbps; înainte, cele patru solicitau 125 MB/s între ele, toate pe 1 GbE, iar fiecare cadru pierdea pachete. O matrice pe care rețeaua nu o poate suporta este refuzată înainte ca vreo cameră să transmită. `--serials` alege acum camerele, prima dintre ele declanșând comanda; înainte, fiecare comandă deschidea toate camerele din rețea, indiferent de ce indica `--serials`. O eroare care reprezintă pierderea de pachete GVSP indică acum acest lucru, în loc să indice cablul de sincronizare M8. A se vedea [Multi-Camera (Sesiuni tranzitorii)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` lasă camerele să funcționeze liber, așa cum au fost găsite.** `TriggerMode` `Off` și toate celelalte setări așa cum le-a găsit conexiunea — setările de declanșare și ale liniei de sincronizare (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP și setările camerei modificate în timpul sesiunii — astfel încât o singurăcomandă „-camera” adresată unui fost membru preia toate cadrele dintr-o dată, iar masterul nu mai rămâne pe `TriggerSource` `Software`. `--keep-settings` salvează setările sesiunii în memoria fiecărei camere, iar `--clear-settings` resetează memoria respectivă (vezi [Memoria camerei](#camera-memory----keep-settings----clear-settings)). `lattice continuous` setează, de asemenea, o cameră pe care o găsește activată în modul „free-run” pentru cursă (și o readuce la starea inițială după aceea), iar o cursă fără ieșiri din cadru `1`.
> - **`lattice configure --get` / `lattice exposure`** afișează un `note:` cuexpunerea reală sau amplificarea camerei în timp ce modul automat este activ. Un `lattice exposure --set` / `configure --set` refuzat indică intervalul acceptat de cameră și restabilește modul automat în loc să dezactiveze expunerea automată. Consultați [Reglarea senzorului](#sensor-tuning).
> - **Noi comenzi pentru cameră:**`lattice highlights` (reduce luminozitatea suprafețelor albe în redarea afișajului RGB) și `lattice exposure --brightness-auto`.**`lattice multi-detect`** detectează camerele conectate prin cablu într-un singur ciclu de scanare și setează implicit camera RGB ca master.
> - **Noi comenzi DAQ:** `daq pool-bind`; `pool-enable-ptp`; și pentru setările proprii ale unui DAQ-E: `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` și `pool-update-firmware`. Consultați [Setările dispozitivului DAQ-E](#daq-e-device-settings).
> - **Un DAQ-E care încetează să mai transmită date este reconectat de către backend.** Când fluxul unui DAQ-E din grup încetează (o repornire a comutatorului care îl alimentează, o întrerupere a legăturii cu camera), backend-ul îl reconectează cu setările pe care le avea, încercând din nou la intervale din ce în ce mai mari pe durata absenței sale. O înregistrare pe acesta continuă în același `.daq`, cu intervalul de întrerupere marcat în fișier, iar `pool-record --stop` listează fiecare întrerupere. În versiunea 1.2.0, senzorul a rămas inactiv până la `pool-disconnect` + `pool-connect`, ceea ce a dus și la încheierea înregistrării. A se vedea [Un DAQ-E care încetează să mai transmită](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** raportează dacă pachetele de la grandmasterul PTP ajung în rețea (`health`), afișând un mesaj `WARNING:` pe stderr atunci când acestea nu ajung. Consultați [`time-sync`](#chloros-cli-time-sync).
> - **Pachet .deb pentru Linux:** mărește dimensiunea bufferelor de recepție ale socket-urilor din kernel necesare pentru fluxurile de date ale camerelor (la setările implicite ale kernelului, se pierd cadre și se reduce rata de cadre), adaugă un serviciu backend care rulează sub drepturile utilizatorului dvs. și `lattice network` verifică bufferele. CLI-ul nu oprește niciodată un backend pe care rulează un serviciu, iar o actualizare repornește serviciile backend activate. Vezi [Rularea backend-ului ca serviciu](#running-the-backend-as-a-service-headless) și [Gazde Linux — buffere de recepție pentru socket](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` indică situațiile în care nu s-a putut realiza alinierea:**fiecare motiv pentru care o captură a fost salvată nealiniată (lipsa unui profil de aliniere, elemente cu rezoluții diferite, un tablou nesincronizat prin cablu, un cadru pe care profilul nu poate fi mapat) este afișat ca o linie `Alignment:`.**`chloros-cli daq help`** este `daq --help` (a ieșit din `2`), iar pe Linux, ajutorul listează comenzile directe de hardware `daq` pe care le include compilarea.
> - **Jetson:** pachetul este `chloros_1.3.0_arm64_jp6.deb`, pentru JetPack 6, iar procesarea sa pe GPU rulează pe Orin cu CUDA PyTorch de la JetPack. Vezi [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Convenții

- Toate comenzile au prefixul `chloros-cli`. Pe Windows, fișierul binar este `chloros-cli.exe`; pe Linux/Jetson este `chloros-cli`.
- Argumentele opționale sunt afișate ca `--flag`. Argumentele poziționale obligatorii sunt afișate fără paranteze.
- În cazul în care este specificată o valoare implicită, omiterea indicatorului duce la utilizarea acelei valori.
- Interfața CLI este un client HTTP simplu care utilizează backend-ul Chloros (serverul de pe `127.0.0.1:5000` pe care îl folosește și aplicația desktop). **Ce comenzi o pornesc:**
  - Comenzile principale — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` și `selftest` — utilizează un backend care rulează deja sau pornesc unul atunci când nu rulează niciunul și îl opresc din nou la finalizarea operațiunilor.
  - **Comenzile `lattice`, `project` și `daq` nu pornesc niciodată unul.** Acestea au nevoie de un backend care rulează deja: cel al aplicației desktop pentru Windows, [serviciul backend](#running-the-backend-as-a-service-headless) pentru Linux sau unul pe care îl porniți voi înșivă (consultați [Pornirea backend-ului](#start-the-backend)). În absența unui backend, acestea afișează mesajul `Error: Chloros backend not reachable at http://127.0.0.1:5000`, împreună cu comanda care îl pornește, și se închid cu mesajul `1`.
- `CHLOROS_BACKEND_URL=<url>` direcționează familiile de comenzi **`lattice`**,**`project`**și**`daq pool-*`** către un alt backend (`lattice network-analysis` îl direcționează în schimb către `--backend-url`). Comenzile principale fixează în mod deliberat `http://127.0.0.1:<port>` și îl ignoră (literalul IPv4 evită penalizarea de ~2 s pe cerere din Windows pentru `localhost`→`::1`). A se vedea [Variabile de mediu](#environment-variables).
- **Căile relative sunt rezolvate în raport cu folderul din care rulați `chloros-cli`.** Backend-ul este procesul care deschide folderele de ieșire și de intrare, așa că, înainte de a trimite o cale, CLI-ul o transformă în cale absolută: `-o` / `--output` (inclusiv valoarea sa implicită, `output`), `--input`, `--burst-dir`, `--profile`, `--save` și `--mask`, o etapă [rețetă](#recipe-dsl) `output` (implicit `output`), precum și folderul implicit `output` al `lattice index --live`. Prin urmare, `-o captures/` scrie în `<current folder>/captures/`, iar căile afișate sunt absolute. `~` este expandat. În cazul unui backend la distanță (`CHLOROS_BACKEND_URL` care denumește o altă mașină), căile sunt trimise așa cum au fost tastate și sunt rezolvate pe acea mașină.
- Este necesară autentificarea cu un cont Chloros+ pentru toate apelurile SDK/CLI (rulați `chloros-cli login` o dată pe fiecare mașină; stocat în cache în `~/.chloros/`).
- Exemplele utilizează căi de acces Linux; pe Windows, înlocuiți `/home/user/...` cu `C:/Users/.../...`. Portul serial al unui DAQ-Ueste de tipul `COM3` pe Windows și `/dev/ttyUSB0` / `/dev/ttyACM0` pe Linux.
- Sinopsisurile scriu indicatori reciproc exclusivi ca `{--a | --b}`, iar pe cele opționale ca `[...]`.

---

## Sinteză de nivel superior

```
chloros-cli [global options] COMMAND [command options]
```

### Opțiuni globale

| Indicator | Descriere |
| --- | --- |
| `--backend-exe PATH` | Suprascrie executabilul backend detectat automat. |
| `--port N` | Portul HTTP al backend-ului (implicit: `5000`). Utilizat numai de comenzile de bază (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` și `daq` utilizează `CHLOROS_BACKEND_URL` (implicit `http://127.0.0.1:5000`). |
| `-v, --verbose` | Activează ieșirea detaliată. |
| `--restart` | Forțează repornirea backend-ului (închide orice proces `backend_server.py` care rulează). Un backend rulat de un serviciu systemd (.deb Linux) nu este niciodată oprit: comanda afișează în schimb `sudo systemctl restart <unit>` și se închide `1`. |
| `--version` | Afișează versiunea (`Chloros CLI 1.3.0`). |
| `--help` | Afișează ajutorul de nivel superior. |

### Indexul comenzilor

| Comandă | Scop |
| --- | --- |
| [`process`](#chloros-cli-process) | Procesează de la cap la coadă un folder cu capturi Survey3 sau LATTICE. |
| [`login`](#chloros-cli-login) | Autentifică acest computer cu un cont Chloros+. |
| [`logout`](#chloros-cli-logout) | Șterge datele de autentificare stocate în cache. |
| [`status`](#chloros-cli-status) | Afișează starea curentă a licenței / autentificării. |
| [`export-status`](#chloros-cli-export-status) | Afișează progresul în timp real al exportului Thread-4 în timpul unei execuții `process`. |
| [`language`](#chloros-cli-language) | Setează sau afișează lista limbilor de afișare CLI (38 de limbi acceptate). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Dosarul implicit al proiectului (partajat cu GUI). |
| [`update`](#chloros-cli-update) | Verifică și instalează actualizările Chloros (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnosticarea sistemului + teste de funcționare. |
| [`time-sync`](#chloros-cli-time-sync) | Stare/control grandmaster PTP. |
| [`lattice`](#chloros-cli-lattice) | Controlul și captarea camerei LATTICE (peste 45 de subcomenzi). |
| [`daq`](#chloros-cli-daq) | Controlul senzorilor spectrali DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Deschiderea și rularea unui proiect Chloros salvat (camere + DAQ-uri). |

---

## Instalare

`chloros-cli` este inclus în programul de instalare Chloros pentru desktop pe toate platformele suportate — nu există o descărcare separată pentru CLI. Instalarea pachetului de platformă adaugă `chloros-cli` la `PATH`, alături de aplicația desktop și de binarul backend pe care îl controlează.

Cele mai recente descărcări: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Programul de instalare include, de asemenea, scripturi de lansare utile (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) care deschid un shell CLI gata de utilizare.

### Windows (.exe)

1. Descărcați programul de instalare pentru Windows de pe pagina de descărcare.
2. Rulați `Chloros Installer 1.3.0.exe` și urmați instrucțiunile expertului de instalare. Calea de instalare implicită este `C:\Program Files\Chloros\` (CLI-ul se află în `C:\Program Files\Chloros\cli\`, pe care programul de instalare îl adaugă la variabila PATH).
3. Deschideți un terminal nou (`cmd.exe`, PowerShell sau Terminalul Windows) pentru ca versiunea actualizată `PATH` să fie detectată.

```powershell
chloros-cli --version
```

Programul de instalare adaugă automat `chloros-cli.exe` la `PATH`-ul sistemului dvs. și include driverele camerei LATTICE. Backend-ul este `C:\Program Files\Chloros\backend\chloros-backend.exe` (folderul `backend` din folderul de instalare), care nu se află pe `PATH`.

**Setări de sistem.** Pagina *Selectare sarcini suplimentare* a programului de instalare conține două opțiuni sub **Setări de sistem**, ambele**bifate implicit**:

| Opțiune (așa cum este formulată în programul de instalare) | Ce face | Dezactivare instalare silențioasă |
| --- | --- | --- |
| **Activare căi lungi Windows** (permite ca folderele de proiect să aibă mai mult de 260 de caractere; recomandat) | Setează valoarea `LongPathsEnabled` la nivel de sistem în Windows la 1. Windows vine cu această opțiune dezactivată, iar în acest caz nicio cale nu poate depăși 260 de caractere; produsele Chloros se află cu până la aproximativ 130 de caractere sub folderul de proiect, astfel încât un folder de proiect mai adânc este refuzat (vezi [Depanare](#sfaturi-pentru-rezolvarea-problemelor)). Se aplică programelor pornite după instalare. Rămâne activat la dezinstalare, deoarece alte programe pot depinde de această setare. | `/MERGETASKS="!longpaths"` |
| **Excludeți backend-ul Chloros din scanarea Microsoft Defender**(captură mult mai rapidă cu camera; recomandat) | Adaugă o excludere a**procesului** Defender pentru `chloros-backend.exe` în folderul de instalare, astfel încât protecția în timp real să nu scaneze fiecare fișier pe care îl scrie backend-ul, indiferent unde îl scrie. Niciun alt program nu este exclus; excluderea este eliminată la dezinstalare. În cazul în care Defender lipsește sau este blocat de o politică, pasul eșuează în mod silențios, iar instalarea continuă. | `/MERGETASKS="!defenderexclusion"` |**De ce opțiunea Defender este importantă:** la rate de captură ridicate, scanarea în timp real a fișierelor capturate consumă aproximativ 30 % din rata de captură (măsurată pe o captură continuă cu 9 camere: 77,7 capturi/s excluse, 52,7 scanate). `lattice array-capture --continuous` afișează o notă atunci când detectează că Defender utilizează un nucleu de procesor sau mai multe pentru captură; rulați din nou programul de instalare cu opțiunea bifată.

O instalare silențioasă (`/SILENT` sau `/VERYSILENT`) aplică ambele opțiuni, cu excepția cazului în care se renunță la ele, de exemplu `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Paravanul de protecție Windows.** Programul de instalare adaugă întotdeauna două reguli de intrare pentru `chloros-backend.exe`, în fiecare profil de rețea, iar programul de dezinstalare le elimină: `Chloros Backend (UDP-In)`, pentru fluxurile de la camere, PTP, mDNS și fluxul de date DAQ-E, și `Chloros Backend (TCP-In)`, limitat la subrețeaua locală, pentru o actualizare a firmware-ului DAQ-E, în cadrul căreia senzorul se reconectează la acest computer pentru a colecta imaginea (a se vedea [Setări dispozitiv DAQ-E](#daq-e-device-settings)). API-ul HTTP al backend-ului rămâne legat de `127.0.0.1`.

### Linux amd64 (.deb)

Pentru stații de lucru x86_64 cu Ubuntu 22.04 LTS sau o versiune mai recentă / bazate pe Debian.

> **Ubuntu 20.04 nu este acceptat.** Lista de dependențe a pachetului este derivată din
> ceea ce leagă efectiv backend-ul, iar aceasta include `libc6 (>= 2.35)`;
> focal distribuie glibc 2.31. `apt` refuză instalarea, în loc să permită eșuarea acesteia în
> timpul rulării. Lista menționează, de asemenea, `libgl1`, pe care OpenCV-ul inclus îl încarcă și care poate lipsi dintr-o
> imagine minimă de server; `apt install` îl include.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

Fișierul .deb instalează:
- de la `chloros-cli` la `/usr/bin/chloros-cli` și `chloros-backend` (pornește backend-ul) până la `/usr/bin/chloros-backend`
- Backend-ul compilat și CLI-ul, despachetate, în `/usr/lib/chloros/backend/` și `/usr/lib/chloros/cli/`
- Driverele camerei LATTICE
- Modelele de denoizare, pachetele de calibrare și configurația canalului de actualizare
- SDK-ul Python pentru sistemul `python3` (`import chloros_sdk`). `apt install` include, de asemenea, cerințele sale, `python3-numpy` și `python3-requests` (recomandate, nu obligatorii: programele de mai sus sunt autonome; `dpkg -i` le omite, iar ieșirea instalării indică ce lipsește). Într-un virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`, care mărește bufferele de recepție ale socket-urilor din kernel pentru fluxurile camerei la 32 MB și le aplică imediat (o valoare deja setată la un nivel mai mare rămâne neschimbată). Dezinstalarea pachetului elimină fișierul. Consultați [Gazde Linux — buffere de recepție pentru socket](#linux-hosts--socket-receive-buffers).
- Două servicii systemd opționale pentru backend, niciuna dintre ele nefiind activată (secțiunea următoare)

#### Rularea backend-ului ca serviciu (fără interfață grafică)

Fără un serviciu, backend-ul (și, odată cu acesta, grandmaster-ul PTP) rulează doar atâta timp cât este pornit de ceva: `chloros-backend` într-un terminal, un script SDK sau o comandă de bază `chloros-cli` pe durata execuției sale. Pentru un backend mereu activ pe o mașină fără interfață grafică, activați serviciul **ca utilizator al dvs.**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` rulează backend-ul ca acel utilizator. Capturile pe care le scrie sunt ale acestuia, jurnalul său detaliat se află în fișierul `~/.cache/chloros/logs` al acestuia, și utilizează fișierul `chloros-cli login` (`~/.chloros`) al acestuia, așa că autentificați-vă mai întâi o dată ca acel utilizator. PTP nu necesită drepturi de root. La fel ca orice backend instalat, acesta ascultă numai pe portul `127.0.0.1:5000`.

`chloros-backend.service` rulează backend-ul ca **root**. Acesta este încă instalat, astfel încât o mașină care l-a activat să continue să funcționeze, dar tot ce scrie un backend root aparține utilizatorului root, așa că un utilizator nu își poate șterge propriile capturi. Pentru a comuta la contul tău de utilizator:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Jurnalul detaliat al serviciului root se află în `/var/log/chloros`, care poate fi citit de membrii grupului `adm` (serviciul setează `CHLOROS_LOG_DIR`; vezi [Variabile de mediu](#environment-variables)). Instalarea pachetului afișează comanda `enable` cu numele dvs. de utilizator completat, precum și comanda de comutare atunci când serviciul root este activat.

**Activați o unitate backend.** Se execută doar o singură unitate backend la un moment dat. O a doua unitate activată (`chloros-backend.service` alături de `chloros-backend@<user>.service` sau două unități `@` ale unor utilizatori) refuză să pornească și rămâne `failed` cu starea de ieșire 98; systemd nu o repornește. Unitatea sa `systemctl status` explică motivul: `Another instance is already running. Exiting.`, apoi ce reține backend-ul (portul 5000, sau fișierul de blocare și utilizatorul căruia îi aparține), apoi `systemd will not restart <unit> (exit status 98)`. După o repornire, oricare dintre unități poate fi cea care rulează. Pentru a comuta, dezactivați-o pe cea pe care nu o utilizați, așa cum s-a arătat mai sus.**CLI-ul nu oprește niciodată backend-ul unui serviciu.** `chloros-cli` îl folosește așa cum este, la fel ca GUI-ul și SDK-ul: `process` rulează pe acesta, încărcând propriul proiect în locul celui anterior. În cazul în care CLI-ul ar reporni altfel backend-ul (`--restart` sau un `process` al cărui import nu a găsit nimic), denumește unitatea, afișează `sudo systemctl restart <unit>` și iese din `1`, deoarece un backend oprit în acest mod ar rămâne oprit. `GET /api/status` denumește unitatea ca `service` (`null` pentru un backend pe care nu rulează niciun serviciu).**Actualizare și eliminare.** Actualizarea pachetului oprește toate serviciile backend, apoi le pornește din nou pe fiecare dintre cele activate, începând cu unitățile `@<user>` (una pornită manual, dar care nu a fost niciodată activată, rămâne oprită, la fel ca după o repornire). Dacă sunt activate mai multe, doar una rulează, iar celelalte rămân în stare de eroare, așa cum s-a menționat mai sus; actualizarea afișează mesajele `Chloros: N backend units are enabled: …` și `Keep one enabled: sudo systemctl disable --now <unit>`. Dezinstalarea le oprește și le dezactivează. Ștergerea definitivă a pachetului elimină și `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Aceeași structură ca și fișierul .deb pentru amd64, incluzând bufferele de recepție și [serviciile backend](#running-the-backend-as-a-service-headless), pentru Jetson Orin / Orin NX / Orin Nano pe **JetPack 6** (L4T R36). PyTorch-ul său este versiunea CUDA a JetPack 6 pentru Orin, astfel încât procesarea GPU, inclusiv debayerul sensibil la textură, rulează pe GPU-ul Jetson. Pachetul nu conține CUDA în sine: utilizează bibliotecile CUDA și cuDNN instalate de JetPack 6 (`sudo apt install nvidia-jetpack` pe o imagine care nu le conține). În cazul în care nu este disponibil un GPU CUDA, procesarea se execută pe CPU, iar funcția de debayer sensibilă la textură utilizează debayerul standard (vezi [`process`](#chloros-cli-process)).

### Autentificare unică pe fiecare mașină

Fiecare platformă necesită o autentificare Chloros+ unică înainte ca apelurile SDK/CLI să funcționeze:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Datele de autentificare sunt stocate în cache în `~/.chloros/user_session.json`.

### Pornirea backend-ului

Comenzile `lattice`, `project` și `daq` comunică cu un backend care rulează deja; ele nu pornesc niciodată unul (vezi [Convenții](#conventions)). Mențineți unul în funcțiune în timp ce le utilizați:

| Platformă | Cum |
| --- | --- |
| Windows | Deschideți aplicația desktop Chloros (backend-ul acesteia deservește și CLI), sau rulați `"C:\Program Files\Chloros\backend\chloros-backend.exe"` într-un terminal separat (din folderul `backend` al folderului de instalare) și lăsați-l deschis. |
| Linux/Jetson | Rulați `chloros-backend` într-un terminal separat sau mențineți unul mereu în funcțiune cu [serviciul backend](#running-the-backend-as-a-service-headless). |

Se execută doar un singur backend la un moment dat; fiecare client (interfața grafică, CLI, scripturile SDK) îl utilizează în comun. Comenzile de bază (`process`, `login`, `status`, …) îl utilizează, de asemenea, și pornesc unul atunci când nu rulează niciunul.

**Un backend continuă să ruleze între comenzi**, astfel încât următoarea comandă nu mai plătește din nou costul de pornire. Se oprește când vă deconectați (`chloros-cli logout`, cu excepția cazului în care aplicația Chloros este deschisă sau este în curs o execuție sau o compilare offline), când rulați `chloros-cli backend stop` sau se oprește automat după 30 de minute dacă nimic nu îl utilizează: nici o comandă, script sau aplicație, nici o execuție sau compilare offline, nici o cameră conectată sau senzor de lumină. `backend stop` refuză în timp ce o execuție sau o compilare offline este în curs, specificând numele acesteia; `--force` o oprește imediat, iar acea execuție sau compilare se pierde (o serie de imagini este păstrată, iar `array-build-video` o compilează din nou). Setați `CHLOROS_BACKEND_IDLE_MIN` înainte ca backend-ul să înceapă să modifice intervalul de 30 de minute (`0`: niciodată); [serviciul backend](#running-the-backend-as-a-service-headless) nu se oprește niciodată de la sine. `chloros-cli backend status` arată dacă unul rulează, ce îl utilizează și orice compilare offline în curs de execuție. `process` își deschide proiectul în backend-ul care rulează, cu excepția cazului în care aplicația Chloros este deschisă cu un proiect pe care a deschis: în acest caz, operațiunea este refuzată și trebuie să procesați din aplicație sau să închideți mai întâi proiectul respectiv. Un proiect lăsat deschis de o versiune anterioară a `process` sau de un script SDK este pur și simplu înlocuit.

### Verificarea instalării

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` se închide (`0`) numai după ce toate cele șapte verificări sunt trecute cu succes. Ultima verificare necesită un GPU CUDA (un GPU NVIDIA, sau unul Jetson Orin) și modelele de denoizare; astfel, pe un computer fără un astfel de GPU, afișează `Failed: CUDA + Denoiser` și iese din `1`; totul, cu excepția algoritmului de debayer sensibil la textură, care funcționează doar pe GPU, funcționează în continuare acolo.

> **Este necesar un abonament Chloros+.**Interfața CLI necesită un plan Chloros+ activ.**Copper**este nivelul de bază Chloros+ — fiecare nivel Chloros+ plătit are acces la CLI/SDK; doar nivelul gratuit**Iron** nu are. (Corelare ID-plan: `0`=Iron/gratuit, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Efectuați upgrade-ul la [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Această limită minimă este impusă de backend, nu doar de CLI: o solicitare marcată prin SDK fără un plan plătit este respinsă cu codul `403 PLAN_UPGRADE_REQUIRED`, indiferent dacă provine de la `chloros-cli`, Python SDK sau un client HTTP personalizat. Un apelant deconectat primește în schimb codul `401 AUTH_REQUIRED`. Accesul funcționează offline pe durata perioadei de grație a planului (30 de zile pentru planurile lunare, până la expirare pentru cele anuale) și se întrerupe la expirarea acesteia; `chloros-cli status` continuă să funcționeze, astfel încât motivul să fie vizibil (este singura rută SDK/CLI scutită de limitarea de nivel — `GET /api/license-status`).

---

## Ghid de pornire rapidă în cinci minute

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (pasul 2) pornește un backend pentru propria sa execuție atunci când nu rulează niciunul și îl oprește la finalizare, așa că porniți unul începând cu pasul 4. Consultați [Pornirea backend-ului](#start-the-backend).

---

## `chloros-cli process`

Procesați un folder de imagini prin întregul flux de lucru Chloros (detectarea țintei → calibrare → vignetă → reflectanță → export index).

### Sinteză

```
chloros-cli process INPUT [OPTIONS]
```

### Argumente de poziție

| Argument | Descriere |
| --- | --- |
| `INPUT` | Calea către folderul de intrare care conține fișiere `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) sau `.dng`. |

### Opțiuni comune

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `-o, --output PATH` | un folder nou denumit după `-n` în folderul implicit al proiectului | Folderul proiectului. Proiectul poartă numele acestuia, iar un folder care conține deja un fișier `project.json` primește un fișier `_1` / `_2`, în loc să fie suprascris. Dosarul implicit al proiectului este cel ales de `set-project-folder` (sau de interfața grafică), altfel este `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | automat (marcă temporală) | Numele proiectului, utilizat numai **fără** `-o` (cu `-o`, proiectul preia numele folderului). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` utilizează un debayer neuronal Chloros+ pentru imagini Survey3; mai lent, dar de calitate superioară. Este destinat exclusiv Survey3: Capturile LATTICE utilizează întotdeauna demosaic-ul standard, astfel încât, pentru un folder care conține capturi LATTICE, `process` respinge `texture-aware` după import, afișând o eroare care precizează numărul acestora, și iese din `1` înainte de a procesa ceva (nici aplicația Chloros nu oferă opțiunea „Texture Aware” pentru acestea). Este necesar un GPU CUDA (un GPU NVIDIA pe Windows/Linux x64, GPU-ul Orin pe un Jetson): fără acesta, procesul utilizează `standard`, iar backend-ul înregistrează o linie `[DEBAYER] … needs a CUDA GPU …` care explică motivul (prezentată cu `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Corecția vignetării. |
| `--reflectance / --no-reflectance` | `--reflectance` | Calibrarea reflectanței (utilizează ținta de panou dacă este găsită, calibrare NIST pe serie pentru LATTICE). Pentru LATTICE multispectral, aceasta servește și ca **produs** de reflectanță — consultați [Comutatoare de export pe produs](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | dezactivat | Aplică corecțiile PPK GNSS din fișierele sidecar. |
| `--exposure-pin-1 CAMERA_MODEL` | niciunul | Modelul camerei conectat la pinul de expunere 1, pentru sincronizarea temporală PPK. |
| `--exposure-pin-2 CAMERA_MODEL` | none | Modelul camerei conectat la pinul de expunere 2, pentru sincronizarea temporală PPK. |
| `--recal-interval SECONDS` | `0` | Intervalul minim de recalibrare, în secunde (*Intervalul minim de recalibrare* al proiectului). |
| `--timezone-offset HOURS` | `0` | Decalajul fusului orar al senzorului de lumină față de UTC, în ore (*Decalajul fusului orar al senzorului de lumină* al proiectului). |
| `--format FORMAT` | `TIFF (16-bit)` | Una dintre valorile `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros nu mărește niciodată adâncimea de biți a unei imagini: în cazul imaginilor de 8 biți, un format cu adâncime mai mare este scris ca `TIFF (8-bit)`, iar procesul menționează acest lucru o singură dată. Într-un proiect care combină capturi de 8 biți și de 12/16 biți, fiecare captură se exportă la propria adâncime de culoare sau la una mai mică. Radianța este întotdeauna de 32 de biți în format float. |
| `--indices NAME [NAME ...]` | niciunul | Indici de vegetație după numele presetării (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), fără diferențiere între majuscule și minuscule. Fiecare este adăugat la proiect pentru benzile camerei respective, fiind inclus un proiect nou. Un nume care nu este o presetare sau pe care benzile camerei nu îl pot genera (de exemplu, „`NDRE`” necesită RE și NIR, pe care nicio cameră nu le oferă), este menționat într-un mesaj de avertizare și nu primește nicio imagine index. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forțează punctul de intrare în fluxul de procesare pentru fișierele TIFF LATTICE (fișierele .raw din Survey3 nu sunt afectate). De asemenea, este o soluție de rezervă care permite procesarea unei capturi **fără fișier raw** — a se vedea [Cum arată un folder de capturi](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | activat | Emite produsul debayerat liniar (`Debayered_Images`). A se vedea [Comutatoare de export per produs](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | activat | Emite previzualizarea afișajului (`Preview_Images`): RGB = balans de alb (iluminant DAQ, dacă este disponibil, altfel „gray-world”) + gamma; multispec = extindere în culori false. |
| `--radiance / --no-radiance` | activat | Emite radianța float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referință pentru produsul de reflectanță LATTICE: `auto` = ținta din cadru care a trecut de controlul calității (QA) este referința absolută (o țintă cu un indicator QA concurează cu DAQ în ceea ce privește scorul măsurat), soluție de rezervă DAQ pentru radiația descendentă (ρ = π·L/E); `target` = strict (fără substituire DAQ); `daq` = DAQ-autoritar. A se vedea [Comutatoare de export pe produs](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | niciunul | Directorul scanărilor de reflectanță țintă **măsurate** pe unitate (`<serial>.csv`); în caz de eșec, se revine la spectrele nominale T3/T4P. |
| `--array-alignment / --no-array-alignment` | activat | Matrice LATTICE: se aplică alinierea modul-la-modul înregistrată în fișierul XMP `Chloros:Alignment*` al fiecărei capturi la fiecare produs procesat (debayering / previzualizare / radianță / reflectanță / indice). Nu se aplică pentru imaginile fără etichete. |
| `--array-alignment-crop / --no-array-alignment-crop` | decupare | Decupează exporturile aliniate la regiunea de suprapunere comună a matricei, astfel încât toate modulele să împărtășească aceeași amprentă; `--no-…` păstrează întreaga suprafață a senzorului (umplere neagră în afara sursei). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Reeșantionare pentru deformarea de aliniere. `nearest` păstrează valorile DN exacte ale sursei (fără amestecarea valorilor radiometrice între pixeli). |

### Opțiuni de detectare a țintelor

| Indicator | Descriere |
| --- | --- |
| `--min-target-size PIXELS` | Dimensiunea minimă a țintei de calibrare în pixeli pentru detector. |
| `--target-clustering 0-100` | Procentul de grupare a țintelor (implicit `60`). |
| `--target / --targets` | Căutarea țintelor de calibrare numai într-un subfolder `target` sau `targets` al fișierului de intrare (mai rapid). |

### Exemple

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Comutatoare de export pentru fiecare produs (LATTICE multispectral)

Procesarea LATTICE se ramifică în **fiecare produs aplicabil într-o singură trecere**. Cele patru comutatoare pe tip — `--debayered`, `--preview`, `--radiance`, `--reflectance` — sunt toate**activate implicit**; utilizați formularul `--no-<type>` pentru a dezactiva una dintre ele. Camerele master RGB emit întotdeauna doar imagini debayered + previzualizare (fără radianță/reflectanță pe bandă), așa că `--radiance`/`--reflectance` nu au niciun efect pentru acestea. Comutatoarele sunt ignorate pentru `.raw` din Survey3 (care urmează calea standard de reflectanță/țintă). *(Vechiul indicator `--radiometric-output {reflectance,radiance,sensor-response}` a fost **eliminat** și înlocuit cu aceste comutatoare; nu mai există nivelul `sensor-response`.)*

| Produs | Ieșire | Este necesară captarea datelor de tip downwelling? |
| --- | --- | --- |
| `--debayered` | Demosaic liniar (`Debayered_Images`): același fișier pe care o captură îl salvează la nivelul său `debayered` — compensarea întunecimii (DSNU) din pachetul de calibrare aplicată asupra datelor brute, urmată de demosaicarea sensibilă la margini a OpenCV; fără LUT de liniaritate, câmp plat sau separare de culori. | Nu. |
| `--preview` | Previzualizare pe ecran (`Preview_Images`): RGB = WB + gamma; multispectral = extindere în culori false. | Nr. |
| `--radiance` | float32 W/m²/sr/nm din lanțul radiometric complet (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 reflectanță ρ (`32768` = 1,0), Compatibil cu Pix4D. | **Da**, cu excepția cazului în care este ancorată de o țintă din cadru care a trecut testul de control al calității (vezi mai jos). |

`--reflectance-source` alege referința de reflectanță:**`auto`**(implicit) face ca o țintă din cadru care a trecut testul de control al calității să fie**referința absolută**— lanțurile de linii empirice ancorate la țintă sunt evaluate încrucișat pe panouri reținute, iar valoarea câștigătoare măsurată este aplicată, bandă cu bandă: o bandă pe care DAQ-ul nu o poate calibra sau a cărei potrivire a pantei DAQ eșuează preia propria linie a țintei (`ReflectanceElBandMethods` specifică care anume) — o țintă care trece de controlul calității cu un indicator de eroare (un panou omis, o bandă defectă) concurează, de asemenea, cu pragul de separare al DAQ pe baza acelui scor măsurat, iar cadrul în fața căruia pierde este marcat cu `ReflectanceChain=daq` — se revine la pragul de descendent al DAQ (ρ = π·L/E) atunci când nu este prezentă nicio țintă sau când controlul calității eșuează;**`target`**este strict (fără substituție DAQ);**`daq`**optează pentru comportamentul autoritar al DAQ. Geometria țintei (ArUco / ROI fix / bandă) provine din configurația țintei proiectului; `--target-reflectance-dir DIR` păstrează scanările**măsurate** scanări (`<serial>.csv`) căutate după numărul de serie/codul QR al unității țintă, cu spectrele nominale T3/T4P ca soluție de rezervă.

Calea de reflectanță DAQ rezolvă automat **raza descendentă cu timestamp-ul corespunzător**dintr-un**`.daq`**(DAQ-U/M/E) înregistrat**sau dintr-un fișier `.csv` nativ DAQ-M**găsit alături de imagini. Dacă un pachet de calibrare per cameră sau DAQ nu este stocat local în cache, pipeline-ul**îl preia automat de pe AWS** la prima utilizare (necesită conexiune la internet o singură dată; stocat în cache sub `~/.chloros/`).**A cui este valoarea măsurată.**O valoare `.daq` salvată de o captură reprezintă valoarea măsurată de camera (camerele) pe care o (le) descrie — denumită într-un fișier `<file>.daq.cameras.json` atașat alături de aceasta (`lattice capture`, `multi-capture`, `array-capture` și interfața grafică; o serie brută menționează citirea fiecărui element în manifestul său) — iar `process` o asociază**numai cu imaginile provenite de la acele camere**. Prin urmare, o cameră autonomă fără citire proprie nu mai împrumută DAQ-ul altei camere; asociați-i un DAQ sau înregistrați unul. Înregistrarea proprie a unui DAQ (`daq pool-record` sau o înregistrare inițiată în GUI) sau orice `.daq` fără acel fișier auxiliar (inclusiv capturile mai vechi) continuă să deservească toate camerele.

#### Citirea pixelilor de reflectanță (Pix4D / Metashape / scripturile proprii)

Reflectanța este stocată ca număr întreg DN, iar **DN-ul care corespunde valorii ρ = 1,0 depinde de camera sursă**:

| Sursă | ρ = 1,0 este | Cum se identifică |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (marjă până la ρ 2,0) | Fișierul poartă ștampila XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (tăiat la ρ 1,0) | Nu există etichete XMP `Chloros:*` — această absență *este* semnalul. |

**Citiți `Chloros:PixelScale` și împărțiți la aceasta**, în loc să presupuneți o constantă. Eticheta este definită în domeniul uint16, deci rămâne `32768` în toate formatele de ieșire care redimensionează — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` și `TIFF (32-bit, Percent)` sunt toate autodescriptive (normalizați mai întâi tipul de date stocat înapoi la uint16: ×257 din 8 biți, ×65535 din float).

> **Fișierele din versiuni mai vechi nu pot conține nicio scală.** Un fișier TIFF de reflectanță pe 8 biți este rescalat și etichetat `Chloros:PixelScale` la fel ca celelalte. Versiunile anterioare versiunii 1.3.0 îl *tăiau* în schimb la intervalul 0..255, astfel încât orice valoare peste ρ≈0,008 era redusă la 255, iar ele omiteau atât tupletul `Chloros:PixelScale`, cât și tupletul `MicaSense:RadiometricCalibration`. **Dacă eticheta lipsește dintr-un fișier de reflectanță LATTICE, nu presupuneți existența unei scale — reprocesați-l**, în loc să împărțiți pixeli care nu au fost niciodată divizibili.

#### Citirea pixelilor de indice (NDVI și ceilalți indici de vegetație)

**O rasteră de indice întreg este o extindere a contrastului pe imagine, nu o hartă −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` și `JPG (8-bit)` — inclusiv fișierul TIFF de 8 biți în care este înregistrată o captură de 8 biți — stochează

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

unde `IndexMin` / `IndexMax` sunt **valorile finite minime și maxime ale indexului propriu al acelui cadru**. DN 0 nu este −1, iar scala completă nu este +1.

Chloros marchează ambele puncte extreme, astfel încât maparea să rămână inversabilă:

| Etichetă | Semnificație |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — convenția de mai jos |
| `Chloros:IndexMin` | valoarea indicelui stocată ca DN 0 |
| `Chloros:IndexMax` | valoarea indicelui stocată ca scală completă |

Citiți-le și inversați-le, normalizând DN-ul stocat la 0…1 în funcție de valoarea maximă a tipului de date stocat, astfel încât o singură expresie să acopere toate formatele de numere întregi:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Citirea nu necesită `-config pix4d.config` — exiftool citește aceste proprietăți XMP fără ajutor. Configurația este necesară doar pentru a le *scrie*.

> **`IndexMin == IndexMax`** înseamnă că fiecare pixel al acelui cadru avea aceeași valoare de index. Rasterul este umplut la jumătatea scalei și nu există nimic cu care să se împartă — se ia valoarea finală.

> **Absența etichetelor `Chloros:Index*` înseamnă că nu există o mapare specificată.** Un export LUT este o redare colorată ale cărei pixeli sunt culori în gradient corelate cu fereastra de prag a LUT-ului dvs., iar un cadru fără pixeli cu indice finit nu are un interval de specificat. Nu presupuneți existența unei mapări în niciunul dintre cazuri — se aplică aceeași regulă ca în cazul lipsei etichetei `PixelScale`.**Deoarece întinderea se aplică pe fiecare imagine în parte, două cadre dintr-un zbor nu sunt comparabile ca DN**— fiecare are propriile sale puncte finale, astfel încât același NDVI ajunge la valori DN diferite în imagini diferite. Pentru lucrări cantitative pe un zbor (statistici zonale, praguri fixe, serii temporale), exportați**`TIFF (32-bit, Percent)`**: rasterele index sunt scrise acolo ca**valori index** de tip float32, fără a fi necesară rescalarea sau etichetarea. În ciuda denumirii formatului, produsele index nu sunt ×100 — acea scalare se aplică doar reflectanței de tip întreg.

#### Masca de limitare a separării spectrale (pe ce pixeli a fost limitată o bandă)

Separarea spectrală 3×3 poate face ca o bandă să devină negativă — acolo unde lumina conține abia acea bandă sau unde scena se află în afara zonei descrise de matricea de separare a camerei. Aceasta nu este o valoare reală, așa că exportul o fixează la 0 — iar vizualizarea în timp real afișează același 0. O bandă poate fi limitată la câțiva pixeli sau la aproape toți; masca înregistrează ambele situații. `Chloros:Band{R,G,B}UnmixClampFraction` indică cât dintr-un cadru s-a întâmplat acest lucru; masca de limitare indică **care pixeli**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— într-un dosar `clamp_masks/`**lângă** dosarul produsului, niciodată în interiorul acestuia (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; un export din `process` către `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — pentru fiecare produs de radianță, de radianță, de reflectanță și de indice, în care cel puțin un pixel a fost limitat: atât capturile, cât și exporturile `process` (precum și fișierele TIFF pe cadru ale `array-build-video --save-tiffs`). `uint8` monobandă, pe propria grilă de pixeli a produsului (inclusiv orientarea, ROI, binning și alinierea matricei). Fiecare mască specifică camera produsului său (`Make` / `Model` / serial), ora capturii și etichetele obiectivului, și enumeră benzile pe care le reprezintă biții săi (`XMP-Camera:BandName`, bitul 0 primul); marcajele `ImageDescription` și `Software` o identifică ca mască de limitare și nu conține nicio calibrare.
- **Biți**: bitul 0 = banda din canalul R al produsului (`Chloros:BandRName`), bitul 1 = G (`BandGName`), bitul 2 = B (`BandBName`). Un bit setat înseamnă că valoarea pixelului din banda respectivă a fost limitată la 0 — tratați-l, precum și orice indice construit pe baza acestuia, ca având un nivel scăzut de încredere.
- **`Chloros:UnmixClampMask`** este înscris pe fiecare produs care a trecut prin procesul de separare: calea măștii relativă la folderul propriu al produsului (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, întotdeauna separat prin `/` — alăturați-l folderului imaginii pentru a deschide masca), sau `none` atunci când niciun pixel nu a fost limitat. Lipsa etichetei înseamnă că produsul nu a fost supus niciodată procesului de separare (camere RGB sau monocrome, niveluri raw/debayered).
- `lattice capture` / `array-capture` imprimă calea către mască sub fișierul salvat (`clamp mask: …`), iar lista rezultatelor capturii o menționează ca `clamp_mask` a cadrului.
- `lattice index --live --save-multiband` scrie o mască pentru fiecare cameră sursă pentru stiva multibandă — a se vedea [`lattice index` — măști pentru stiva multibandă](#lattice-index--multiband-stack-masks). Un videoclip nu are mască: `array-build-video` raportează în schimb fracțiunile de limitare ale fiecărui cadru (a se vedea [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Deoarece niciun dosar de produs nu conține o mască, la niciun nivel de adâncime, un dosar de produs (`radiance/`, `reflectance/`, un dosar `*_Images/` al unui export `process`) poate fi transmis către Pix4D, Metashape sau alt software de fotogrammetrie așa cum este — inclusiv subfolderele — iar acestea acceptă doar imagini. Transmiteți-i folderul de produs, nu folderul de deasupra acestuia (acela conține și `clamp_masks/` și toate celelalte tipuri de export). Importul propriu al Chloros omite măștile. Citirea unei măști: `(mask & 1) != 0` înseamnă „banda canalului R a fost limitată aici”, `& 2` G, `& 4` B.

#### Datele EXIF transferate în export

`process` copiază **blocul GPS și ExifIFD** ale capturii sursă în fiecare produs, astfel încât un
export conține `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` și
`CameraSerialNumber` alături de georeferențiere.

**`FocalLength` nu este opțional pentru fotogrammetrie.** Pix4D calculează distanța dintre eșantioane la sol (GSD) pe baza
distanței focale plus altitudinea; în absența etichetei, se recurge la o scară extrem de eronată. Într-un
zbor deasupra unei plantații de portocali cu 49 de capturi, lipsa etichetei a transformat un sit de 411 m × 160 m într-unul reconstruit
de 47,8 km × 13 km — o ortofotografie de 455 MP compusă în mare parte din „nodata”, care a fost interpretată inițial ca o problemă de tiling și
o problemă BigTIFF înainte ca cineva să verifice GSD-ul. Dacă ortofotografia dvs. are o scară neverosimilă, rulați mai întâi `exiftool -FocalLength` asupra produsului exportat.

Copia **nu** este în mod deliberat `-all:all`: etichetele structurale ale IFD0&#x27;, deoarece etichetele sale structurale afectează ieșirea LATTICE atunci când
sunt copiate, iar `ExifImageWidth` / `ExifImageHeight` sunt excluse deoarece descriu
captura *sursă* — un export care a fost vreodată redimensionat ar avea altfel dimensiuni
contradictorii cu propriul său raster. Fișierul XMP este scris direct, nu copiat, deoarece ExifTool
elimină etichetele XMP cu aceeași invocare atunci când blocul XMP este copiat (ceea ce ar elimina etichetele de calibrare MAPIR
).

### Unde sunt salvate rezultatele

Produsele sunt salvate **în folderul proiectului, grupate după cameră și apoi după formatul fișierului**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Folderul camerei este `LATT-<sensor>-<lens>-F<filter>` pentru LATTICE (corespunzând codului EXIF al capturii
`Model`) și `<model>_<filter>` pentru Survey3 — două camere care împărtășesc același senzor și filtru, dar diferă
prin obiectiv, păstrează arbori separați, deoarece vignetarea, câmpul vizual și distorsiunea diferă. Formatul
folderului este `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, sau `tiff32` pentru
`TIFF (32-bit, Percent)`.

> **Fiecare produs exportat păstrează numele fișierului SURSĂ.** Un export de radianță al
> `capture_…_raw.tif` se numește în continuare `capture_…_raw.tif` — doar că se află în
> `tiff32/Radiance_Images/`. **Folderul identifică produsul, nu numele fișierului**, așa că o căutare globală
> după `*radiance*.tif` nu găsește nimic; căutați în schimb în director.

### Înregistrări ale senzorului de lumină — calibrate `.daq` + `.csv`

`process` gestionează, de asemenea, înregistrările `.daq` din folderul dvs. de intrare și **nu**
nevoie de nicio imagine pentru a face acest lucru: un DAQ-U / DAQ-M / DAQ-E zburat singur reprezintă o
captură completă, iar un folder care conține doar fișiere `.daq` este o intrare validă.

Un DAQ poate fi înregistrat **fără** calibrarea sa — asta fac în mod implicit înregistratoarele publice
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) fac în mod implicit: ele scriu valorile brute ale senzorilor și marchează fișierul astfel încât
Chloros să preia calibrarea din fabrică a senzorului respectiv **prin portul serial** (mai întâi din cache-ul local,
apoi din cloud-ul MAPIR) și o aplică. `process` scrie rezultatul înapoi:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` conține un rând pentru fiecare citire: marcaj temporal UTC, timp de integrare, putere totală,
lux fotopic/scotopic, PPFD (și defalcarea sa albastru/verde/roșu), lungimea de undă de vârf, apoi
spectrul complet pe grila de lungimi de undă proprie a senzorului. Fișierul `.daq` este reimportat fără a fi
calibrat a doua oară.

`<name>` este numele propriu al fișierului de înregistrare. Două înregistrări din dosare diferite pot împărți
o singură valoare — citirile per captură (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) salvate în
aceeași milisecundă de două camere — și fiecare primește totuși propriile produse: prima păstrează
`<name>`, iar următoarea adaugă folderele care o diferențiază **după** nume
(`<name>__<capture>_daq_calibrated.daq`), astfel încât data și ora înregistrării rămân în frunte.

În caz de succes, execuția raportează `Light-sensor products written: N (calibrated .daq + .csv)`.
Textul dintre paranteze descrie ce a fost scris efectiv, astfel încât se citește
`(RAW COUNTS — this sensor has no calibration bundle)` pentru un senzor fără pachet și
`(N calibrated, M raw counts)` pentru un dosar care le conține pe amândouă. Titlurile proprii ale backend-ului,
`[DAQ-EXPORT]` și `[RUN-SUMMARY]`, își derivă formularea în același mod — niciunul dintre
cele trei nu poate numi un export brut „calibrat”.

O înregistrare DAQ-U / DAQ-M / DAQ-E al cărei pachet de calibrare nu poate fi preluat — sunteți
offline sau senzorul respectiv nu are nicio calibrare înregistrată — este **omise cu un motiv** pe o
linie `[DAQ-EXPORT]`, nefiind niciodată salvate ca fișiere „calibrate” care conțin numere brute.
Conectați-vă la internet și rulați din nou. Motivul este cel pe care cititorul l-a
stabilit efectiv pentru acel fișier (schemă ilizibilă, lipsa pachetului, o eroare de scriere), iar rezumatul
execuției listează motive **distincte** — douăzeci de fișiere omise dintr-un singur motiv sunt afișate ca un singur
motiv, nu ca douăzeci de repetări ale acestuia. CLI-ul numește fiecare înregistrare omisă la sfârșitul
execuției, împreună cu motivul:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Un dosar cu înregistrări care nu exportă **niciuna** dintre ele eșuează: vezi
[O execuție care nu scrie nicio imagine eșuează](#a-run-that-writes-no-images-fails).

O înregistrare al cărei nume se termină în `_calibrated` sau `_raw` (`field_raw.daq`) **nu este
importat**: acestea sunt numele pe care `process` le atribuie propriilor produse, iar un produs citit
ca înregistrare ar fi numărat de două ori. CLI-ul indică acest lucru și menționează numele fișierului —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— în loc să raporteze un folder gol. Redenumiți o astfel de înregistrare înainte de a o procesa.

#### Înregistrările DAQ-A se exportă ca numere brute

Familia **DAQ-A** este anterioară sistemului de pachete pe serie și nu are niciun pachet de calibrare
de preluat — în schimb, este calibrată pe teren în raport cu o țintă de reflectanță, motiv pentru care
nu a avut niciodată nevoie de unul. Refuzarea acestor înregistrări le-a lăsat fără nicio posibilitate de a-și
extrage datele, așa că se exportă sub un **nume diferit**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Un nume de fișier diferit, în loc de un indicator în interiorul fișierului, deoarece informația trebuie să rămână intactă
atunci când este trimisă prin e-mail doar ca nume simplu. Antetul `.csv` indică
`raw spectral sensor counts (NOT irradiance)` și avertizează că valorile sunt comparabile
**în cadrul** fișierului — ceea ce este exact scopul pentru care sunt utilizate în calibrarea bazată pe țintă — și
nu între senzori. Coloanele fotometrice dependente de putere (putere totală, lux fotopic și
scotopic, PPFD) sunt înregistrate ca **NULL** în loc să fie integrate din numărări, iar
rezumatul rulării indică `RAW COUNTS`, astfel încât „exportat” dintr-un jurnal nu poate fi interpretat ca iradianță.

Înregistrările vechi **v1.01 / v1.02** (un DAQ-A-SD le scrie pe acestea) nu conțin o epocă pentru fiecare citire,
ci doar ora de scriere a fișierului. Programul de potrivire imagine↔radiație descendentă încă le respinge — potrivirea unui
cadru cu o oră de scriere ar fi greșită fără a se observa — dar exportatorul le citește, iar
fișierul CSV afișează `clock=daq_created_on`, astfel încât produsul indică ce ceas utilizează.

### Note

- `process` detectează automat dacă folderul dvs. este de tip Survey3, „LATTICE” sau mixt.
- Progresul este transmis prin Server-Sent Events; CLI-ul afișează progresul în timp real pentru fiecare fir de execuție (Detectare, Analizare, Procesare, Exportare).
- Pentru Linux/Jetson, CLI-ul verifică spațiul de swap și poate afișa un avertisment înainte de a procesa foldere mari. De asemenea, debayerul sensibil la textură aplică automat o limită de frecvență a GPU-ului pe dispozitivele Jetson cu consum redus de energie (Nano, Orin Nano).
- În cazul unei execuții reușite, programul raportează câte produse imagistice a generat (`Image products written: N`, iar `(plus M clamp masks)` atunci când procesul de separare a limitat vreun pixel).
- **`process` partajează în siguranță un backend în execuție.** Acesta repornește automat un backend deja în execuție numai atunci când acel backend este inactiv, având încărcat un proiect anterior și neavând în gestionare camere, matrice sau DAQ-uri. Un backend care deține oricare dintre acestea (interfața grafică care transmite în flux o matrice, un script SDK, o înregistrare DAQ) este utilizat așa cum este, fără a fi repornit niciodată, astfel încât `process` poate rula alături de o matrice activă. Dacă backend-ul raportează o execuție în curs, `process` așteaptă până la 15 s ca aceasta să se termine (o execuție care tocmai s-a încheiat necesită aproximativ o secundă pentru a fi ștersă), apoi refuză cu `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. Indicatorul global `--restart` forțează în continuare un backend nou și încheie orice sesiune de cameră, matrice sau DAQ de pe acesta. Un backend rulat de un serviciu systemd dLinux nu este niciodată repornit: `process` îl utilizează, iar `--restart` afișează în schimb `sudo systemctl restart <unit>` (vezi [Rularea backend-ului ca serviciu](#running-the-backend-as-a-service-headless)). Nici backend-ul propriu al aplicației Chloros: cu aplicația deschisă, `process` îl utilizează, încărcându-și proiectul în locul celui al aplicației (un backend pornit în locul său ar bloca accesul aplicației deschise la acesta).
- **O execuție în curs își păstrează proiectul.** În timp ce o execuție de procesare este în curs — pornită de aplicație, `process`, un script SDK sau `chloros-mcp` — backend-ul refuză să creeze, să deschidă, să importe în, să adauge sau să elimine fișiere din, să mute, șterge sau modifica setările de procesare ale unui proiect, cu codurile de eroare HTTP 409 `PROCESSING_IN_PROGRESS` și `A processing run is in progress on project '<name>', so ... now is refused`; un `process` care primește acest mesaj iese din `1`. Setările de afișare (dimensiunea miniaturilor) se salvează în continuare. Modul de procesare și setările de reflectanță țintă ale proiectului sunt refuzate în același mod (o execuție păstrează modul în care a început). O captură salvată în proiect în timpul rulării (din aplicație, din CLI, din SDK sau din `chloros-mcp`) este salvată ca de obicei, dar se adaugă la lista de fișiere a proiectului abia la sfârșitul rulării; răspunsul indică acest lucru (`project_add`), la fel ca și o înregistrare a senzorului de lumină oprită în cadrul proiectului. Anterior, un al doilea client putea comuta la proiectul deschis în timpul execuției, iar execuția în curs era exportată în celălalt proiect.
- **Ctrl+C oprește execuția în backend, nu doar în CLI.** CLI-ul solicită backend-ului să oprească procesarea, afișează `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` și așteaptă până la 120 s pentru `Processing stopped.`, astfel încât `chloros-cli process … && next` să nu pornească `next` pe un backend încă ocupat cu execuția. O a doua apăsare de Ctrl+C pune capăt așteptării. `process` iese apoi din `130`, așa cum face orice comandă întreruptă.

#### O execuție care nu scrie imagini eșuează

Dacă ați solicitat produse, iar execuția nu a scris **niciunul** — doar `project.json` și
`calibration_data.json` — `process` tratează acest lucru ca pe un eșec: afișează
`Processing finished but wrote no image products.` și **iese cu un cod diferit de zero**, astfel încât un script să poată
detecta acest lucru. Mesajul menționează numele folderului proiectului și cauzele obișnuite:

- folderul de intrare nu a fost recunoscut ca fiind o captură (verificați structura și `--input-level`), sau
- fiecare produs solicitat a fost omis ca fiind inaplicabil pentru acele camere (de exemplu, solicitarea
  radianță/reflectanță de la camere care funcționează doar în RGB).

Rulați din nou cu `--verbose` și verificați jurnalul backend-ului pentru liniile `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
care explică omisiunile specifice fiecărei camere, care altfel nu apar în ieșirea CLI.

O execuție deliberată numai cu metadate — toate opțiunile de produs dezactivate și fără `--indices` — este totuși un
**succes**, deoarece o ieșire de imagine goală este rezultatul corect în acest caz.

#### O execuție ale cărei exporturi nu au putut fi verificate eșuează

După o execuție, `process` verifică etichetele fișierelor exportate cu ExifTool. Dacă ExifTool nu a putut fi rulat, acesta
afișează `Verificarea exportului a EȘUAT: ExifTool nu a putut fi rulat, astfel încât etichetele a N din M fișiere exportate
nu au fost verificate (consultați liniile [EXIFTOOL] din jurnalul backend-ului)` and `Produsele sunt scrise și rămân
în loc; rulați din nou pentru a le verifica.`, does not print `Prelucrarea s-a încheiat!`, și **iese cu un cod de ieșire diferit de zero**.
Produsele sunt păstrate.

#### Ce capturi include o execuție

Sunt incluse capturile finalizate înainte de rularea `process`. Capturile salvate în timpul execuției nu sunt incluse; `process`
avertizează câte au fost omise, iar o nouă execuție le include.

#### O execuție care a pierdut o imagine eșuează

Dacă o imagine a eșuat în timpul unei rulări, `process` afișează `N image(s) failed and were not exported` cu
cea mai frecventă cauză, nu afișează `Processing complete!` și **se închide cu un cod de ieșire diferit de zero**. Celelalte produse sunt
scrise și rămân la locul lor, iar jurnalul backend conține fiecare eșec.

La fel se întâmplă și în cazul unei **rulări numai cu senzorul de lumină**: un dosar cu înregistrări `.daq` nu conține imagini de exportat
prin definiție, iar execuția este evaluată pe baza fișierelor calibrate `.daq` / `.csv` pe care le-a scris în schimb.
Dacă nu a scris **nimic** — toate înregistrările au fost omise — afișează
`Processing finished but wrote no light-sensor products.`, denumește fiecare înregistrare cu
motivul pentru care a fost omisă și **iese cu un cod de ieșire diferit de zero**.

---

## `chloros-cli login`

Autentifică acest dispozitiv cu un cont cloud Chloros+. Datele de autentificare sunt stocate în cache în mod securizat în `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Exemple

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (eliminând sau duplicând părți din parolă). La un cod de eroare 401, CLI-ul reîncearcă automat cu `$$` re-apărată, apoi cu o jumătate a parolei fără duplicări; dacă o nouă încercare reușește, vă conectează și afișează sintaxa corectă cu ghilimele simple pe care să o folosiți data viitoare.

> **Utilizare fără interfață grafică/prin script: lipsa unei sesiuni stocate în cache înseamnă o solicitare de autentificare.** O comandă de bază (`process`, `status`, `export-status`, `time-sync`, …) executată fără o sesiune stocată în cache solicită `Email:` / `Password:` pe stdin înainte de a continua. Dacă stdin este închis (cron, o unitate systemd), programul eșuează imediat cu `login cancelled`; dacă stdin este un terminal deschis sau o conductă, programul așteaptă introducerea datelor. Rulați `chloros-cli login EMAIL PASSWORD` o dată pe fiecare mașină înainte de a programa sarcini fără interfață grafică.

---

## `chloros-cli logout`

Șterge sesiunea stocată în cache și impune o nouă autentificare la următoarea apelare.

Deconectarea în timp ce o execuție de procesare este în curs de desfășurare nu o oprește: execuția se termină în modul de procesare în care a început, iar următoarea execuție utilizează procesarea standard. `logout` menționează acest lucru (`The processing run in progress finishes in <mode> mode; the next run uses standard`), la fel și aplicația atunci când vă deconectați de acolo.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Afișează nivelul actual al licenței (Iron/Copper/Bronze/Silver/Gold), utilizatorul autentificat și numărul de asocieri ale dispozitivului.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Verifică progresul în timp real al exportului Thread-4. Poate fi apelat în siguranță **în timpul** unei execuții `process` dintr-un alt shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Setează limba de afișare a CLI-ului (38 de limbi acceptate, inclusiv CJK, RTL și Indic). Revine în mod elegant la limba engleză pe consolele vechi care nu pot reda scriptul.

```
chloros-cli language [LANG_CODE] [--list]
```

### Exemple

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Comenzi pentru folderul de proiect

Acestea gestionează locația implicită a folderului de proiect (comună cu interfața grafică).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` afișează folderul în care sunt salvate proiectele noi. Dacă folderul configurat nu mai există (a fost șters, redenumit sau se află pe o unitate care nu este conectată), CLI-ul revine la folderul implicit, la fel ca și interfața grafică: folderul implicit este afișat pe stdout, un mesaj de avertizare care menționează folderul lipsă este trimis către stderr, iar codul de ieșire este `0`.

---

## `chloros-cli update`

Numai pentru Linux/Jetson. Verifică `version_url` față de `/etc/chloros/update.conf` și, atunci când există o versiune mai nouă, descărcă versiunea corespunzătoare `.deb` și o instalează imediat cu `sudo dpkg -i` (fără solicitare de confirmare; `sudo` poate solicita parola). Pe un Jetson, actualizarea este oferită doar sistemelor cu JetPack 6. Pe Windows, comanda nu face nimic și afișează mesajul `up to date`: actualizați cu un nou program de instalare.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Pe Linux/Jetson, CLI-ul efectuează, de asemenea, o **verificare automată a actualizărilor la fiecare pornire** (fără blocare, nu întârzie niciodată comanda): citește `/etc/chloros/update.conf`, stochează rezultatul în cache timp de 1 oră în `~/.chloros/update_cache.json` și afișează `Update available: vX.Y.Z / Run: chloros-cli update` când există o versiune mai nouă. Se omite în mod silențios în cazul oricărei erori și pe Windows; `CHLOROS_DISABLE_UPDATE_CHECK=1` o dezactivează. Datorită cache-ului de 1 oră, o versiune publicată în decursul acelei ore poate fi încă considerată actualizată.

---

## `chloros-cli selftest`

Efectuează un test de funcționare în 7 pași: versiune, disponibilitatea porturilor, pornirea backend-ului, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), prezența modelului de denoizare, disponibilitatea CUDA+denoiser. Utilizează un backend în execuție sau pornește unul pentru test și îl oprește ulterior. Afișează `N/7 checks passed` și verificările eșuate și iese din `0` numai când toate cele șapte sunt reușite; verificarea CUDA+denoiser eșuează pe un computer fără un GPU CUDA (vezi [Verificarea instalării](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Starea și controlul grandmasterului PTP. Gazda Chloros rulează grandmasterul PTP; camerele LATTICE și unitățile DAQ-E funcționează ca slave pentru acesta, pentru a asigura timestamp-uri comune între dispozitive (aproximativ 1 ms între dispozitive). PTP aliniază **marcajele de timp**: camerele dintr-o matrice expun simultan deoarece declanșatorul hardware M8 le activează (în mai puțin de 100 µs între module), nu datorită PTP. `chloros-cli time-sync` fără subcomandă execută `status`.

| Subcomandă | Descriere |
| --- | --- |
| `status` | Afișează starea grandmasterului, prioritățile BMCA, identitatea ceasului și dacă pachetele sale ajung în rețea: `health` este `OK` (`STOPPED` când grandmasterul nu rulează), sau `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` cu un `health_detail`, iar apoi se afișează o linie `WARNING:` pe stderr (stdout rămâne în format JSON). `sync_sent` / `announce_sent` numără doar mesajele care au fost transmise; `*_send_errors` și `last_send_error` le numără pe cele care nu au fost transmise. După ce o conexiune de cameră se întrerupe și revine, grandmasterul își reconstruiește singur socket-urile (`rebinds`, `last_rebind`). |
| `peers` | Lista slave-urilor detectate prin Delay_Req (camere + senzori DAQ-E). |
| `cameras` | Starea PTP pentru fiecare cameră (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Repornirea procesului grandmaster. |
| `set-priority --priority1 N --priority2 N` | Ignorarea priorităților BMCA. |

Grandmasterul transmite anunțuri pe plăcile de rețea ale camerelor (orice adaptor cu o adresă `169.254.x.x` inclusă). Dacă niciunul nu este activ, acesta revine la adaptorul rutei implicite numai atunci când acesta este conectat prin cablu, niciodată prin Wi-Fi sau un adaptor virtual (Tailscale, WSL), deoarece nicio cameră sau DAQ-E nu se află în spatele unui astfel de adaptor. Dacă nu are nimic de anunțat, rulează fără socket-uri (`health` `NO_SOCKETS`), iar următoarea cameră sau DAQ care se conectează îl reconstruiește pe placa de rețea care este activă la momentul respectiv; `time-sync restart` face acest lucru imediat.

Grandmasterul rulează într-un proces propriu, alături de backend (`own_process` / `process_pid` în `time-sync status`), cu prioritate ridicată în Windows (`process_priority`), astfel încât un backend ocupat (o execuție de procesare, un export, mai multe comenzi simultan) nu poate întârzia timpii pe care îi înregistrează; înainte, o astfel de întârziere putea modifica simultan ceasul fiecărei camere. Acest proces se încheie odată cu backend-ul; dacă nu poate fi pornit, grandmaster-ul rulează în interiorul backend-ului ca înainte (`own_process` false, iar jurnalul backend-ului menționează acest lucru).

### Exemple

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Controlul camerelor LATTICE. Fiecare subcomandă trece prin backend-ul Chloros; backend-ul deține grupul de camere, astfel încât apelurile CLI-ului ulterioare reutilizează același identificator deschis.

### Opțiuni comune (împărtășite de majoritatea subcomenzilor)

| Indicator | Descriere |
| --- | --- |
| `-d, --device N` | Indicele camerei (implicit: 0). |
| `-s, --serial SN` | Număr de serie specific; înlocuiește `--device`. |
| `--serials SN1,SN2,…` | Numere de serie separate prin virgulă pentru funcționarea cu mai multe camere. [Comenzile tranzitorii pentru mai multe camere](#multi-camera-transient-sessions) deschid doar aceste camere, prima dintre ele declanșând acțiunea. |
| `--all` | Funcționează pe fiecare cameră detectată. |
| `--exposure US` | Timpul de expunere în microsecunde. |
| `--gain DB` | Câștig în dB. |
| `--pixel-format FMT` | de ex. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` pe un M3M. Numele stabilește adâncimea de biți în cadrul familiei proprii a camerei: pe un M3M, un nume de tip Bayer stabilește formatul Mono al adâncimii sale (`BayerRG8` stabilește `Mono8`), pe un M3C, un nume de tip Mono îl stabilește pe cel de tip Bayer (`Mono12` stabilește `BayerRG12`). |
| `--width N` / `--height N` | Dimensiunile imaginii. |
| `--preset {default,high_quality,high_speed,triggered}` | Aplică o presetare de setări: formatul de pixeli, expunerea, amplificarea și rata de cadre (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` le listează). Pe un M3M, formatul este cel mono, cu aceeași adâncime: `high_speed` capturează `Mono8`, `high_quality` și `triggered` `Mono12`. Un indicator asociat acestuia (`--exposure`, `--gain`, `--pixel-format`) are prioritate față de presetare, iar o setare pe care presetarea nu o menționează păstrează setarea proprie a camerei. Toate execuțiile libere, cu excepția `triggered`, care activează camera pentru o schimbare de stare hardware pe linia 2 — dacă nimic nu acționează acea linie, camera va aștepta la nesfârșit în loc să captureze. |
| `-o, --output DIR` | Director de ieșire (implicit: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Dimensiunea pachetului GVSP. `auto` execută sonde ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Fluxul de lucru pentru prima conectare a camerei LATTICE

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referință subcomenzi

#### Descoperire și informații

| Subcomandă | Scop |
| --- | --- |
| `lattice info` | Afișează lista camerelor conectate (fabricant, model, număr de serie, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analizează sistemul gazdă pentru configurarea optimă a camerei. `--no-discover` omite detectarea camerei (mai rapid, analiză doar a plăcii de rețea). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Verifică/corectează setările plăcii de rețea (pe Linux, inclusiv bufferele de recepție ale socket-urilor din kernel — vezi [gazdele Linux](#linux-hosts--socket-receive-buffers)); estimează lățimea de bandă/FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Recomandare privind capacitatea rețelei backend cu schemă stabilă+ recomandare de matrice (returnează `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` utilizează implicit senzorul complet, `2048` × `1536`. Se iese din `0` pentru `ok`, `auto_capped_fps` și `auto_shrunk`, iar `1` pentru `needs_force_slip` și `error`, cu sau fără `--json`. Ajunge la backend la `--backend-url` (implicit `http://127.0.0.1:5000`), nu la `CHLOROS_BACKEND_URL`. `auto_capped_fps` păstrează rezoluția solicitată, dar limitează numărul de cadre pe secundă (fps) țintă — citiți `recommended.recommended_target_fps` și transmiteți-l ca țintă de conectare; tratați-l ca pe un succes, nu ca o eroare. `auto_shrunk` apare numai atunci când cadrul solicitat nu poate fi livrat la nicio rată de cadre; connect nu reduce niciodată cadrele din proprie inițiativă. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Analiză ipotetică fără deschiderea camerelor (`--width` / `--height` implicit `2048` × `1536`). Atât aceasta, cât și `network-analysis` proiectează ceea ce va transmite connect: fără `--pixel-format`, se aplică formatele specifice fiecărei camere pe care le folosește „connect” (Mono12 / BayerRG12), iar cu `--packed auto` (implicit) decizia pe care „connect” ar lua-o [transport comprimat de 12 biți](#transport-12-biți-comprimat---comprimat) pe care Connect ar lua-o pe acest computer — astfel încât fps-ul indicat este rata la care rulează matricea. Ambele afișează decizia privind comprimarea. **`--n-active` reprezintă numărul total de camere conectate, nu doar al acestei matrice**— măriți-l atunci când camerele independente transmit simultan sau când bugetul de bandă este calculat pe baza unei cereri care le subestimează numărul (implicit: `len(--models)`). Afișează întotdeauna liniile agregate `Wire budget:` (MB/s solicitați vs. plafonul de siguranță împotriva coliziunilor) și `Max cameras:` și semnalează `** OVER-SUBSCRIBED**` când matricea suprasolicită cablul — a se vedea [Modelul fps și burst al matricei](#array-fps--burst-model). |
| `lattice gpu` | Afișează starea GPU-ului. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Verifică sau actualizează firmware-ul camerei. Selecția locală `.fwa` este fixată: fișierul din `firmware/<MODEL_PREFIX>/` care corespunde cu `MIN_FIRMWARE_VERSION` al versiunii de compilare este flash-uit atunci când este prezent (doar versiunea cea mai recentă ca soluție de rezervă), astfel încât o imagine de la furnizor mai nouă stocată pe disc rămâne inactivă până când acea fixare este modificată — versiunile mai noi ajung în mod deliberat la unități prin intermediul manifestului AWS semnat, care este preferat atunci când este mai recent. |
| `lattice presets [--apply NAME]` | Afișează sau aplică presetările camerei. `--apply` stabilește ce reprezintă numele presetărilor (formatul de pixeli, expunerea, amplificarea, rata de cadre, declanșarea) și afișează aceste setări așa cum sunt citite de cameră după: pe un M3M, formatul Mono al adâncimii presetării; rata de cadre la care rulează, care poate fi mai mică decât cea solicitată de presetare (`high_speed` solicită 60 fps; o cameră full-frame pe o legătură de 1 Gbps rulează la aproximativ 35). Toate celelalte setări scrise de comanda „open” sunt restabilite la valorile inițiale. Comanda `--apply default` resetează camera la setările implicite din fabrică: setările din fabrică cu expunere automată liberă pe Line2, limita de lățime de bandă a conexiunii și corectarea defectelor dezactivate, balansul de alb calibrat al unei camere RGB fiind păstrat (unitate pe orice alt tip de cameră). Nimic nu este salvat în memoria camerei, astfel încât la următoarea pornire aceasta pornește din propria memorie, ca și înainte. Ca `--preset` pentru o captură sau o conectare, `default` nu este o presetare. |
| `lattice status` | Starea camerei în timp real. |

#### Captură

| Subcomandă | Scop |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Cadru unic. **Salvează implicit fiecare tip de export**(`--processing all`); consultați [Niveluri de export pentru captură](#capture-export-levels-the-all-default). `--processing` acceptă `all`, `raw`, `debayered`, `radiance`, `reflectance` sau `display` (`preview` este acceptat pentru `display`); orice alt nume este refuzat înainte ca camera să fie atinsă (ieșire `1`). `--levels` salvează un subset explicit (suprascrie `--processing`). O cameră pe care Chloros-ul nu o deține este deschisă pentru captură, iar expunerea acesteia este setată inițial în funcție de scenă (cel mult câteva secunde), așa cum fac [comenzile tranzitorii](#multi-camera-transient-sessions); când capturarea se încheie, indiferent de modul în care se încheie, camera respectivă este readusă la starea inițială (format, expunere, flux de procesare a culorilor, fiecare setare modificată de capturare — lista de sub [Multi-Camera (Sesiuni tranzitorii)](#multi-camera-transient-sessions)). Date de DAQ-ul care a citit captura utilizată sunt salvate ca fișier sidecar `.daq`, incluzând o captură doar în format brut; `--no-daq` / `--daq-tolerance-ms` o ajustează ([DAQ-ul care citește o captură salvează](#the-daq-reading-a-capture-saves)). `--force-daq` este încă acceptat. `--jpeg-quality` = calitate JPEG 1–100 (implicit 95).**Fișierele salvate în proiectul deschis sunt adăugate la acesta.** La fel ca în aplicație, o captură ale cărei fișiere ajung în folderul proiectului deschis în Chloros le adaugă la lista de fișiere a acelui proiect; fișierele salvate în altă parte sunt lăsate așa cum sunt. Comanda afișează o linie care indică acest lucru (`Project: ...`) sau, în timp ce o sesiune de procesare este în curs, că acestea sunt reținute și se alătură proiectului la finalizarea sesiunii. La fel se întâmplă și cu `multi-capture`, `array-capture` (la fel și `--smart`) și `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Transmite pe disc până la apăsarea Ctrl+C. O execuție `tiff`sunt fișierul `raw`, un tablou pe care captura îl scrie de la acea cameră: mozaicul senzorului, un plan cu propriile valori, necomprimat, cu pachetul de calibrare (un cadru de 2048 x 1536 pe 12 biți are 6,0 MiB plus pachetul, indiferent de scenă), așa că `process` pornește de la acestea. `png` și `jpg` conțin 8 biți, deci aceste serii sunt cadre demosaicate pentru vizualizare, de la care `process` nu pornește. Cu o limită DAQ (`daq pool-bind`) sau în grup, fiecare cadru primește propriile fișiere `.daq` și `.match.json` în folderul `daq/` al seriei, întrucât `lattice capture` salvează unul; `--no-daq` dezactivează această funcție, iar la oprire se afișează numărul de cadre. Fiecare cadru este scris o singură dată, împreună cu metadatele complete; la apăsarea Ctrl+C, cadrele deja capturate sunt scrise înainte ca camera să fie eliberată. Se oprește automat când unitatea de stocare se umple, afișează `Failed:` / `Disk full:` și iese din `1`; orice cadru pe care nu l-a putut scrie iese, de asemenea, din `1`. O cameră lăsată armată pentru un declanșator hardware (`TriggerMode` `On`) este setată pe funcționare liberă pentru această sesiune. La sfârșitul sesiunii, camera este readusă la starea inițială — setările de declanșare și toate setările modificate în timpul sesiunii (expunere, amplificare, format, …) — cu excepția cazului în care se utilizează `--keep-settings`, care salvează setările sesiunii în memoria proprie a camerei (funcționare liberă), sau `--clear-settings`, care resetează memoria respectivă la setările din fabrică. O sesiune care nu înregistrează niciun cadru este considerată eșuată: dacă nu se înregistrează niciun cadru timp de 10 s, se oprește automat, afișează mesajul `No frames:` cu motivul și iese din program cu codul `1`. Cadrele pe care dispozitivele de scriere nu au reușit să le proceseze sunt considerate pierdute, iar oprirea afișează mesajul `Dropped:` împreună cu motivul (rata de cadre a camerei, a dispozitivelor de scriere și a cozii); o rată de cadre mai mică sau un format de pixeli pe 8 biți le păstrează pe toate. Comanda verifică starea backend-ului la fiecare 2 s; dacă se oprește fără a opri înregistrarea (`kill -9`, memorie insuficientă), backend-ul o oprește la 30 s după ultima verificare și readuce camera (`stop_reason` `client_gone`, și o linie în jurnalul backend-ului). `multi-stream` și `align-stream` fac același lucru. O pornire în care camera durează mai mult să se deschidă decât timpul de așteptare al comenzii (60 s; 180 s pentru `multi-stream` / `align-stream`) — o cameră care încă se inițializează — este anulată de backend: camera este repusă în stare inactivă și închisă, iar nicio înregistrare nu rămâne în desfășurare (504 HTTP `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Previzualizare MJPEG live în browser: conectează camera, afișează URL-ul fluxului și îl deschide în browserul dvs. `--ae-damping` setează amortizarea expunerii automate (0,4–100). |

##### Fluxuri live (MJPEG; fără RTSP)

Chloros transmite video live ca **MJPEG prin HTTP**(`multipart/x-mixed-replace`, limită `frame`) din backend.**RTSP nu este acceptat.** Deschideți un URL-ul într-un browser sau citiți-l cu orice client MJPEG (`cv2.VideoCapture(url)` din OpenCV, ffmpeg, VLC). Camera trebuie conectată mai întâi în backend (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` sau prin interfața grafică).

| URL (`http://127.0.0.1:5000` + …) | Flux |
| --- | --- |
| `/api/camera/stream/<serial>` | Previzualizarea pe ecran, până la aproximativ 30 fps. Pentru un element al matricei, `?max_h=` (implicit `720`) și `?jpeg_q=` (implicit `80`) stabilesc înălțimea și calitatea JPEG. Acesta este URL-ul pe care îl deschide `lattice viewer`. |
| `/api/camera/<serial>/stream-annotated` | Aceeași previzualizare cu suprapunerile GUI desenate în: `zebra=1`, `crosshair=1`, `grid=3x3` (sau `9x9`), `histogram=1`, `peaking=1`, `spot=1`; fără nicio interogare, proiectul utilizează setările de suprapunere salvate pentru camera respectivă. |
| `/api/camera/<serial>/stream-reflectance` | Reflectanță pe bandă de 8 biți: ρ = pixel ÷ partea`X-Chloros-Pixel-Scale` (255, cu excepția cazului în care este generată excepția `refl_max`). `fps` (implicit `10`), `jpeg_q` (implicit `95`), `max_h` (implicit `0` = nativ), `band=<name>`, `align=1`. JPEG, deci cu pierderi. Refuzat cu codul HTTP 409 când camera nu are reflectanță (o cameră RGB, fără pachet de calibrare, fără citire DAQ). |

- **Potrivirea cadrelor între camere.**Fiecare parte trimisă de un**element al matricei**, pe toate cele trei fluxuri, conține anteturile `X-Chloros-Group-Id` (contorul grupului sincronizat al matricei), anteturile `X-Chloros-Frame-Id` și `X-Chloros-Timestamp-Ns`. Părțile provenite de la camere diferite, dar care au același `X-Chloros-Group-Id`, provin de la același declanșator. O cameră autonomă nu are grup și nu trimite niciun ID de grup. Browserele ignoră anteturile specifice fiecărei părți; un client care analizează el însuși fluxul multipart le citește.
- **Pauză cât timp vizualizarea matricei este deschisă.** `/api/camera/stream/<serial>` al unui membru al matricei trimite un cadru static „pauză” cât timp fluxul live combinat al matricei este deschis (de exemplu, vizualizarea matricei din GUI) și reia transmisia când acesta se închide.

#### Reglarea senzorului

| Subcomandă | Scop |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Citește/scrie orice nod GenICam. `--set ExposureTime=…` (`Gain=…`) fără comutatoarele `ExposureAuto` (`GainAuto`) care dezactivează mai întâi modul automat, așa cum face `lattice exposure --set`; o valoare pe care camera o refuză afișează `Failed to set <node>: …` pe stderr și iese din `1`. `--get` și vizualizarea grupată implicită afișează un `note:` sub `ExposureTime` / `Gain` în timp ce modul automat rulează — vezi [Citirea expunerii și a amplificării în modul automat](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Expunere și AE. O expunere `--set` sau o țintă `--brightness` este păstrată ca fiind a ta (controlul automat al expunerii al lui Chloros nu o modifică); câștigul rămâne așa cum era, așa că în modul automat se ajustează în continuare către țintă până când setați `lattice gain`. `--auto` restabilește controlul asupra expunerii, iar `--brightness-auto` restabilește ținta la cea automată a lui Chloros. Fără indicatori, citește nodurile de expunere, cu același `note:` ca și `configure --get`. O valoare `--set` pe care camera o refuză afișează `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` pe stderr — modul automat, a cărui scriere a fost dezactivată, este restabilit în loc să rămână dezactivat — nu afișează nicio linie `Exposure: … us (auto OFF)` și iese din `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Amplificare și amplificare automată. La o valoare `--set`, camera refuză să afișeze `Failed to set Gain: …` pe stderr, nu afișează linia `Gain: … dB (auto OFF)` și iese din `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Zona de interes (ROI) a senzorului și binning. Binning-ul este întotdeauna **Medie** — vezi mai jos. |
| `lattice format [--set FMT] [--list]` | Formatul pixelilor. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Declanșare hardware/software. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (fără indicatori = balans de alb one-shot) | Operațiuni de balans de alb. Numai pentru camerele RGB/Bayer; o operație nulă (omise) pe M3M monocrom. Modul „one-shot” păstrează balansul de alb pe care îl calculează (`BalanceWhiteAuto` și amplificările); orice altă setare pe care o scrie în memoria deschisă revine la valoarea inițială, iar toate setările sunt resetate atunci când modul „one-shot” nu poate rula. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Fluxul de culoare pentru afișajul RGB. `natural` (implicit) reprezintă procesarea live simplă; `enhanced` adaugă eliminarea franjurilor + vibranță + contrast local CLAHE pentru un aspect complet de paritate a hub-ului, la un cost de procesare de aproximativ 2× pe cadru, ceea ce duce la o rată de cadre **în timp real** mai mică — capturile salvate beneficiază întotdeauna de procesarea completă, indiferent de situație. Numai pentru camere RGB/Bayer; omise pe M3M mono. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Saturație/contrast în previzualizarea live (camere cu filtru RGB); `100` este neutru, `--reset` le setează pe ambele la 100. Capturile salvate nu sunt afectate. Omitut pe M3M mono. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (fără indicatori = citire) | Afișează **zonele luminoase** (camere cu filtru RGB): reduce intensitatea celor mai luminoși și mai albi pixeli — de exemplu, o suprafață albă strălucitoare care se redă ca o pată albă uniformă. `0` = dezactivat, `-100` = intensitate maximă. Previzualizarea live și capturile la nivel de afișare respectă această setare; radianța/reflectanța nu o respectă niciodată, iar această opțiune nu poate recupera datele senzorului care au fost decupate (pentru asta, reduceți expunerea). Omit pe M3M monocrom. |
| `lattice filter [--list]` | Afișează lista modelelor de filtre cunoscute de Chloros (`RGN-IMX265`, `OCN`, `NGB`, …). Filtrul unei camere este citit din șirul modelului acesteia și nu poate fi setat: `--set NAME` doar caută un model, apoi iese din `1` cu `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Verifică nodurile de alimentare/termice; comută modul de inactivitate cu consum redus de energie. `--sleep` lasă camera în stare de repaus (limita de transfer a legăturii sale activată (`DeviceLinkThroughputLimitMode` `On`) la valoarea minimă, modul de economisire a energiei activat acolo unde camera dispune de unul) cu toate celelalte setări readuse la starea inițială; o trecere în repaus care eșuează este readusă integral. Trezirea readuce legătura așa cum a găsit-o în repaus (limita, modul și întârzierea pachetelor `GevSCPD`, pe care camera o menține la 0 cât timp limita este activată; înregistrarea este păstrată pe acest computer, astfel încât o trezire după repornirea serverului să o găsească la fel), sau, pentru o cameră pe care acest computer nu a pus-o în repaus, dar care pare a fi în repaus (limita sa activată la valoarea minimă), cu limita dezactivată și un `GevSCPD` lăsat la 0 la 80 (setările din fabrică); o limită setată peste pragul minim rămâne așa cum este. Sonda și trezirea restabilesc ceea ce au scris la deschidere. |

> **Binning-ul este întotdeauna de tip „Average”.** `lattice resolution --binning N` setează binning-ul de tip „Average” și
> nu oferă opțiunea de schimbare a modului. Binningul de tip „Sum” înmulțește fiecare DN cu N², costând aproximativ două stopuri de
> marjă pentru zonele luminoase la fiecare pas de binning; Chloros împarte din nou acel factor înainte de
> lanțul radiometric, dar pixelii pe care „Sum” i-a împins până la plafonul senzorului sunt tăiați și nu pot fi
> recuperați. Nu oferă nimic în plus față de „Medie” — un indice este invariant la scară, iar scalarii de radianță
> sunt calibrați la scara „Medie”. Dacă o cameră ajunge deja setată pe „Sum” (lăsată
> așa de un alt instrument sau printr-o resetare la setările din fabrică), Chloros o detectează din citirea unui nod și
> compensează; rularea `lattice resolution --binning N` o readuce la „Average”. În versiunea 1.2.0
> `--binning-mode` este încă acceptat, astfel încât scripturile mai vechi rulează, dar afișează doar un avertisment:
> binning-ul rămâne la „Medie”.

> **Modificarea binning-ului, a formatului pixelilor și a declanșatorului pe o cameră nu are efect.** Pe o cameră,
> backend-ul rămâne deschis și transmite în flux (`cam-connect`, interfața grafică), `lattice resolution --binning`
> nu produce efect (afișează că binning-ul este încă activ, `Binning: 1x1`), iar
> `lattice format --set` și `lattice trigger --mode` sunt respinse de cameră
> (`SC_ERR_ERROR -1001`). Eliberați-l mai întâi (`lattice cam-disconnect -s <SN>`) sau setați-le
> la conectare (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Citirea expunerii și a amplificării în modul automat

În timp ce `ExposureAuto` (`GainAuto`) al unei camere rulează, nodul său `ExposureTime` (`Gain`) păstrează **ultima valoare scrisă**, nu ceea ce face camera în prezent. Pentru o cameră a cărei interfață backend este deschisă (`cam-connect`, `array-connect`, interfața grafică), `lattice configure --get`, vizualizarea implicită `lattice configure` și `lattice exposure` (fără indicatori) afișează, prin urmare, valoarea nodului și, sub aceasta, o notă cu cel mai recent cadru al camerei:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Citiți valoarea cadrului din notă; valoarea nodului este doar ultima scriere. Prin HTTP, același răspuns `POST /api/camera/<serial>/read-nodes` menține `nodes` neschimbat și adaugă `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, cel mai recent cadru) și `notes` (propoziția de mai sus). Ambele sunt prezente doar atâta timp cât modul automat nu este `Off` și se cunoaște un cadru; o cameră deschisă doar pentru citire nu are cadru și nu primește niciuna dintre ele. `lattice gain` (fără indicatori) afișează doar nodul — folosiți `lattice configure --get Gain` pentru notă.

##### Expunerea pe un element al matricei

În timp ce o matrice transmite în flux, fiecare element rulează cu **`AcquisitionFrameRateEnable`**, la o rată de cadre puțin mai mare decât rata de declanșare a matricei: perioada sa este perioada de declanșare minus 15 % (cel puțin 3 ms) sau rata maximă proprie a camerei atunci când rata de declanșare este deja aproape de aceasta. Prin urmare, activarea**`configure --get`**pe un element citește**`AcquisitionFrameRateEnable`**ca**`True`**și**`AcquisitionFrameRate`**la acea setare; activarea se dezactivează când matricea se oprește.**Nu scrieți aceste două noduri pe un element al matricei** — matricea le menține sincronizate cu declanșatorul său și le rescrie.

Această setare reprezintă, de asemenea, limita maximă de expunere a elementului: aproximativ 85 ms la un declanșator de 10 fps (perioadă de 100 ms), la **orice ROI sau binning**. Înainte de versiunea 1.3.0, o zonă de interes (ROI) mică limita expunerea unui element la perioada de funcționare liberă a senzorului (4,9 ms la 768 × 304), indiferent de cât de lent se declanșa matricea.**Un `ExposureTime` manual mai lung decât limita maximă încetinește declanșarea pentru a se adapta la acesta** atunci când este scris cu `lattice exposure --set` sau `configure --set` (sau din interfața grafică). Matricea deschide mai întâi limita maximă a acelui element pentru valoarea respectivă și lasă rata mai lentă să ruleze câteva perioade de declanșare, astfel încât scrierea poate dura cu câteva secunde mai mult; declanșatorul se activează apoi nu mai repede decât permite expunerea. Pe o matrice cu rată automată, rata la care funcționa anterior revine odată ce fiecare element a cărui expunere manuală a încetinit-o revine la expunerea automată. O valoare pe care camera însăși nu o poate înregistra este totuși refuzată, cu intervalul acceptat indicat în mesajul de eroare, iar expunerea automată a elementului este restabilită dacă a fost dezactivată doar pentru acea scriere. O expunere de tip `apply` nu încetinește declanșatorul: acesta este menținut la limita superioară. Chloros nu scurtează niciodată o expunere manuală pentru a se adapta unui declanșator mai rapid; o rată fixată (`array-connect --target-fps F`) mai rapidă decât permite o expunere lasă acelui membru un plafon suficient de lung pentru aceasta, iar membrul respectiv nu acceptă declanșări mai rapide decât permite expunerea sa.

#### Calibrare și senzori

| Subcomandă | Scop |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Calibrează pe baza unei ținte de reflectanță. Se caută până la `--attempts` cadre (implicit `30`) pentru aceasta; un cadru incomplet (pierdere de pachete GVSP) costă o încercare, nu întreaga execuție. Eșuează la `no target detected in N attempts`, sau la `no complete frame in N attempts: <last error>` atunci când niciun cadru nu a ajuns intact. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | Spectrul și magnitudinile de bandă ale unui DAQ-M, citite prin Bluetooth (BLE) prin intermediul backend-ului. `--mac` este adresa MAC BLE proprie a senzorului și este necesară pentru oricare dintre cele trei acțiuni: nu există o valoare implicită, deoarece fiecare unitate are propria adresă (`daq pool-discover --only ble` le listează); fără acesta, comanda iese din `1`. Dacă nu se specifică nicio acțiune, afișează ajutorul. `--camera SERIAL` integrează magnitudinile de bandă afișate prin răspunsul spectral măsurat al camerei respective, în loc de curba de flotă a modelului de filtru — camere RGB, cu pachetul de calibrare al camerei încărcat (conectat) sau stocat în memoria cache a acestui computer. Titlul indică a cui curbă a fost utilizată: `Band magnitudes (camera <SERIAL>):` sau `Band magnitudes (model <filter>):` atunci când camera nu are una (nu este o cameră RGB, sau nu are pachetul încărcat sau stocat în cache), caz în care s-a utilizat în schimb curba standard. Prin HTTP, `POST /api/dls/cli-read` preia `camera_serial` și returnează `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Aplică corecția de vignetare imaginilor existente, fiecare cu câmpul plat specific obiectivului său (din modelul EXIF al acestuia). `--lens-model` se aplică imaginilor al căror model nu menționează nicio cameră; o imagine al cărei model contrazice acest lucru este omisă. |

#### Multi-cameră (sesiuni tranzitorii)

O comandă tranzitorie deschide camerele pentru o singură execuție și le închide la finalul acesteia: `multi-capture` fără o matrice persistentă conectată, `multi-stream` și `multi-test` aici, și `align-calibrate`, `align-apply`, `align-stream` și `index --live` mai jos. Aceste reguli se aplică tuturor:

- **`--serials SN1,SN2,…` alege camerele.** Numai acestea sunt deschise, iar prima declanșează semnalul (pentru `align-calibrate`, `--reference`, dacă sunt specificate). O cameră serială care nu se află în rețea determină eșuarea execuției înainte ca vreo cameră să fie deschisă. Fără `--serials`, toate camerele din rețea sunt deschise. O cameră pe care Chloros o deține deja (interfața grafică, `cam-connect`, `array-connect`) refuză execuția, indiferent dacă `--serials` o numește sau nu: aceasta utilizează rețeaua pentru care este configurată execuția. (`index --live` fără `--serials` deschide în schimb camerele activate din profilul său: vezi [Index / Matematică a vegetației](#index--vegetation-maths).)
- **Camerele împart rețeaua în același mod în care o împarte `array-connect`.** Fiecare cameră este sincronizată la cota sa din ceea ce transportă legătura gazdă (o întârziere între pachete, nu limita de transfer a camerei, a cărei valoare minimă de 31,25 MB/s este deja depășită de patru camere pe 1 GbE), astfel încât patru camere funcționează pe o legătură de 1 Gbps. Când camerele nu pot transmite toate simultan la cel mai lent ritm pe care îl ating, ele sunt declanșate una după alta, fiecare având legătura dedicată exclusiv ei, iar comanda afișează `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Când chiar și o singură cameră nu se încadrează, execuția este refuzată înainte ca vreo cameră să transmită: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, ieșire `1`.
- **Fiecare cameră trebuie să transmită, altfel execuția eșuează.** O cameră care nu transmite niciun cadru reprezintă o eroare: comanda afișează `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, unde `<why>` reprezintă eroarea de preluare a acelei camere, urmată de cauzele care au condus la aceasta, și se închide cu `1`. Un cadru care a sosit cu pierderi de pachete GVSP (`IncompleteBufferError: GVSP packet loss …`) înseamnă că rețeaua nu suportă lățimea de bandă combinată a camerelor (`Every missing frame arrived with GVSP packet loss: …`): verificați viteza de legătură a gazdei și a legăturii ascendente a switch-ului (`lattice network`), utilizați cadre jumbo, sau porniți mai puține camere cu `--serials`. O cameră care nu a transmis nimic nu primește, de obicei, semnalul de declanșare de la camera principală: verificați cablul de sincronizare M8 al acesteia, apoi rulați `lattice multi-detect`. Ce face fiecare comandă în cazul unui grup de cadre din care lipsește o cameră este indicat în rândul corespunzător.
- **Fiecare cameră este repusă în starea în care a fost găsită.** La finalul execuției, fiecare cameră revine la starea în care se afla la deschiderea execuției: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, și `LineMode` / `LineSource` pe `Line2` și `Line3` (iar semnalul de deglitch al intrării de declanșare dintr-un tablou furnizează linia unui slave, `LineFilterSelector` / `LineFilterWidth`); acesta este `PtpEnable` (și setul `PtpSlaveOnly` / `PtpDomainNumber` asociat acestuia); `ExposureAuto` / `GainAuto` (și `ExposureTime` / `Gain`, unde avea o valoare setată manual); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (și fiecare `BalanceRatio` în care balansul de alb era manual); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (și `Gamma` în cazul în care gama era activată), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; dimensiunea pachetului (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); ritmul de transmisie, `DeviceLinkThroughputLimitMode` (și `DeviceLinkThroughputLimit` în cazul în care modul era `On`), întârziereaîntârzierea între pachete `GevSCPD` și decalajul de pornire `GevSCFTD` pe care o matrice le atribuie fiecărei camere; datele de bloc pe care fiecare flux le activează (`ChunkModeActive`, fiecare `ChunkEnable` sub `ChunkSelector`); `AcquisitionFrameRateEnable` (și `AcquisitionFrameRate` unde activarea era pornită — când este oprită, rata este doar pentru citire și citește maximul permis de ritm, deci nu este o setare). Se scrie doar ceea ce diferă, camera s-a oprit mai întâi, ritmul și rata de cadre înainte de o expunere care necesită perioada lor de cadru (o cameră limitează `ExposureTime` la o singură perioadă de cadru). Cu `--keep-settings`, cablajul de declanșare și de linie de sincronizare este deconectat, camera funcționează în mod liber, iar restul este salvat în memoria proprie a camerei conform setărilor de funcționare; `--clear-settings` restabilește setările din fabrică.
- **Expunerea este setată pe baza scenei înainte ca primul cadru să fie păstrat.** Seria de cadre preia grupuri de cadre sincronizate și le elimină în timp ce setează expunerea și amplificarea fiecărei camere pe baza acelor cadre. Scopul este obținerea unei luminozități medii pe ținta camerei, cu cel mult 1 % din cadru decupat; o scenă cu zone foarte luminoase nu atinge ținta. Acest proces durează de obicei 2-6 grupuri de cadre și cel mult 5 s.
  - Fiecare cameră vizează ținta de la care o matrice conectată o pornește, prin filtrul său: TargetBrightness **40**pe o cameră multispectrală (RGN, OCN, NGB) și**60** pe una RGB. Aceasta reprezintă jumătate din valoarea țintă de expunere dată de 80 (trei sferturi în cazul RGB), astfel încât un panou cu reflectanță ridicată are o probabilitate mai mică de a fi decupat. `--brightness N` (0-255) setează în schimb valoarea țintă pentru fiecare cameră (`multi-test` nu are `--brightness`: expunerea automată a acesteia este dezactivată). Jurnalul backend-ului denumește ținta fiecărei camere: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Expunerea are prioritate, până la limita proprie de expunere automată a camerei. Amplificarea acoperă doar ceea ce rămâne.
  - Expunerea și amplificarea stabilite sunt menținute pe toată durata rulării, astfel încât fiecare cadru al unei camere `multi-stream` este capturat cu aceeași expunere. O captură care trebuie să urmărească schimbările de lumină aparține unei matrice conectate (`array-connect`), a cărei expunere automată se ajustează continuu.
  - `--exposure` și `--gain` sunt păstrate exact așa cum au fost specificate. Atunci când ambele sunt specificate, niciun cadru nu este eliminat.
  - La finalizarea execuției, expunerea automată a fiecărei camere este reactivată.
  - Jurnalul backend-ului raportează rezultatul: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` sau `exposure NOT solved after …` cu motivul, iar `(not settled)` pentru fiecare cameră care nu a fost inclusă. În acest caz, cadrele sunt totuși capturate.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` modifică limita de 5 s; `0` dezactivează această opțiune (vezi [Variabile de mediu](#environment-variables)).

**În timp ce Chloros deține orice cameră** (interfața grafică, `cam-connect` sau o matrice conectată), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` și `index --live` refuză să ruleze, deoarece deschid toate camerele din rețea, iar o a doua deschidere a unei camere reținute poate bloca backend-ul. (`index --live` refuză atunci când o cameră pe care ar urma să o deschidă este reținută: fie `--serials`, fie camerele profilului său, precum și orice cameră pentru un profil care nu specifică niciuna.) Backend-ul răspunde cu codul HTTP 409 `CAMERA_HELD`, menționând fiecare cameră reținută și comenzile de eliberare (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), iar comanda se încheie cu `1`. Cu o matrice conectată, `array-capture --continuous` o înregistrează în locul lui `multi-stream`, iar `array-capture` (aliniat în mod implicit) salvează un set aliniat în locul lui `align-apply`; `multi-capture` și `align-calibrate` utilizează un șir conectat care conține camerele lor în loc să le deschidă.

| Subcomandă | Scop |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Afișează toate camerele cu roluri de sincronizare; cu `--serials`, doar acelea, iar prima ca `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Un cadru sincronizat de la fiecare cameră. Salvează **toate tipurile de export în mod implicit**atunci când este conectată o matrice persistentă și citirea DAQ utilizată ([Citirea DAQ a unei capturi salvează](#the-daq-reading-a-capture-saves)); soluția de rezervă tranzitorie fără matrice este**doar debayered** (rulați mai întâi `array-connect` pentru restul). `--brightness` se aplică numai soluției de rezervă: o matrice conectată își păstrează propria expunere automată. Soluția de rezervă nu salvează `.daq` (`--no-daq` / `--daq-tolerance-ms` nu au niciun efect acolo), iar `multi-capture` nu acceptă `--force-daq`. În soluția de rezervă, o cameră care nu furnizează niciun cadru eșuează la captură (`Error: multi-capture failed: no frame from …`, ieșire `1`) și nu se salvează niciun fișier din acel grup de cadre. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Cadre sincronizate în flux (tranzitorii), `--fps` implicit `3`, până la Ctrl+C sau `--count` grupuri. Un grup de cadre pentru care o cameră nu a transmis nimic nu este scris, iar trei la rând încheie fluxul. Dacă a fost omis vreun grup, execuția se încheie prin afișarea mesajului `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, plus `Error: the stream stopped: …` care menționează camera la momentul încheierii fluxului, și iese din `1`X. Se oprește automat când unitatea de stocare se umple, afișează `Failed:` / `Disk full:` și iese din `1`; orice fișier pe care nu l-a putut scrie determină, de asemenea, ieșirea din `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | Test de sincronizare GPIO (tranzitoriu), `--count` declanșatoare implicite `10`. Afișează modul în care este partajată rețeaua (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, iar linia `Serial emit:` atunci când camerele sunt declanșate una după alta, iar marcajele de timp includ apoi decalajul dintre acestea), o linie `Sync group: master SN:<serial> -> SN:<serial>, …` pentru fiecare grup de sincronizare (`(no slaves: not synced)` pentru o cameră fără cablu de sincronizare, care se declanșează independent), apoi un rând pentru fiecare declanșare: `[  0] SN:<serial>=<timestamp> \| …`, cu `SN:<serial>=(no frame)` pentru o cameră care nu a transmis nimic pentru acea declanșare. Apoi `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` și o linie `SN:<serial>: mean <±x> us, jitter <y> us` pentru fiecare cameră, fiecare cameră raportată la master într-un singur ciclu de ceas: un declanșator ale cărui marcaje combină ceasurile dispozitivelor camerelor și PTP (PTP dezactivat sau încă în curs de sincronizare) este omis, cu un `Note:` care le numără. Orice cadru omis duce la eșuarea testului (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, ieșire `1`); trei declanșări incomplete la rând îl încheie mai devreme. De asemenea, testul eșuează (ieșire `1`) când mai puțin de două declanșări au toate camerele sincronizate la același ceas (`… the clocks cannot be compared`) sau când ceasurile nu se sincronizează niciodată: ceasurile PTP au o abatere de peste 1 ms (plus orice decalaj la nivel serial) față de ceasul principal (`… PTP is not locked`) sau ceasurile dispozitivelor ale căror abateri variază cu mai mult decât această valoare între declanșări. Pornește grandmasterul PTP dacă acesta nu rulează, activează PTP-ul fiecărei camere (așa cum face `array-connect`, indiferent de configurația rețelei) și, înainte de primul declanșator, așteaptă ca ceasul PTP al fiecărei camere să se sincronizeze cu grandmasterul și să se stabilizeze: `PtpStatus` Slave pe un singur grandmaster, la o abatere de maximum 100 us față de acesta și stabil timp de 3 s, iar fiecare cameră să fie sincronizată astfel, fără întrerupere, timp de 120 s — aproximativ 2 minute de la pornire. Un ceas care tocmai s-a sincronizat prezintă o abatere de până la ~200 us timp de aproximativ 2 minute după ce afișează „Slave”, iar indicatorul `PtpOffsetFromMaster` al camerei nu o arată; 120 s aduc camerele la o abatere de ~10 us față de un ceas stabilizat. O cameră rămâne sincronizată încă de la un test anterior care a lăsat PTP activat, așa cum a observat acest backend, numără pornind de la acea sincronizare, astfel încât un test efectuat imediat după altul nu mai așteaptă din nou; o cameră pornită de test sau una găsită cu PTP activat, pe care acest backend nu a văzut-o sincronizată (pornită în altă parte sau înainte de o repornire a backend-ului), așteaptă cele 120 s, iar într-un grup care include ambele camere, cele care tocmai s-au sincronizat încă își așteaptă rândul. O cameră care iese din starea de Slave, depășește 100 us sau urmează un alt grandmaster pentru 3 citiri consecutive (1,5 s) în timpul așteptării își reia numărătoarea; una sau două astfel de citiri sunt înregistrate (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), iar numărătoarea continuă; un ID de „grandmaster” care nu poate fi citit este considerat necunoscut, nu ca o schimbare. Faptul că numărătoarea fiecărei camere repornește simultan (în decurs de o secundă) indică faptul că timpul „grandmaster”-ului se modifică, nu al camerelor: jurnalul backend-ului menționează `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` îl listează în `grandmaster_steps`, iar o așteptare care apoi expiră își începe eroarea cu `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Limita nu este extinsă pentru aceasta. `--ptp-settle SECONDS` setează timpul de stabilizare pentru execuție (0-600; `0` pentru fără așteptare de stabilizare). Așteptarea este limitată la timpul de stabilizare plus 90 s (210 s implicit, cel puțin 90 s), iar comanda așteaptă backend-ul 240 s plus timpul de stabilizare (360 s implicit). Afișează `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, iar jurnalul backend-ului conține o linie la fiecare 5 s cu starea și decalajul fiecărei camere, cât timp a fost blocată și cât timp mai are: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Fiecare cameră pornește cu PTP dezactivat; de acolo ceasurile se sincronizează în aproximativ 10 s. Ulterior, fiecare cameră este readusă la starea în care a fost găsită de test — PTP-ul său, expunerea automată și amplificarea pe care testul le menține dezactivate pe durata rulării, iar restul (cu excepția cazului `--keep-settings`) — astfel încât o cameră lăsată cu PTP activat (`lattice configure -s <serial> --set PtpEnable=True` sau o execuție cu `--keep-settings`) rămâne sincronizată, iar odată ce acest backend a constatat că a fost sincronizată pentru timpul de stabilizare, un test ulterior nu o mai așteaptă. Dacă ceasurile nu se stabilizează în timpul așteptării, testul eșuează înainte de declanșare (`PTP: NOT locked …`, ieșire `1`) și indică ce trebuie făcut: o cameră care nu s-a blocat niciodată (`PtpStatus Listening`) înseamnă că grandmasterul nu o accesează (`time-sync status`); una care a continuat să se deconecteze este denumită în funcție de cât timp a fost blocată (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); ceasurile încă converg: verificați `time-sync status`, iar pentru a le sincroniza înainte de test, activați PTP și lăsați-l activat (`lattice configure -s <serial> --set PtpEnable=True` pentru fiecare cameră sau rulați testul cu `--keep-settings`; testul restabilește setările inițiale, așa că rămâne activ), apoi rulați-l. |
| `lattice multi-detect [--line LINE] [--json]` | Identificați camerele de pe linia de sincronizare M8 și o cameră master implicită: camera RGB (modelul se termină cu `-FRGB`) dacă este conectată prin cablu, altfel prima cameră conectată prin cablu în ordinea numerelor de serie. Cablurile M8 se întâlnesc la o singură joncțiune, astfel încât **orice cameră conectată prin cablu poate fi master** — modelul tipărit `MASTER` este o valoare implicită, nu o caracteristică a cablării. Fiecare cameră, pe rând, activează linia, iar celelalte sunt verificate pentru a detecta impulsul acesteia; o cameră deja detectată în grupul cablat nu este verificată din nou, așa că o rețea complet cablată necesită o singură rundă. În absența codului `--line`, se verifică `Line2`, apoi `Line3`. Afișează `MASTER: SN:…`, câte un `SLAVE: SN:…` pentru fiecare altă cameră cablată și `(probed on <line>)`; `--json` adaugă rezultatul complet (`master`, `slaves`, `no_cable` — camerele la care impulsul nu a ajuns — `line` și, pentru fiecare cameră, `results`). La finalizare, fiecare cameră revine la setările inițiale de declanșare și linie de sincronizare (aceleași noduri ca mai sus). |

#### Aliniere

| Subcomandă | Scop |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — plus butoane de detectare/potrivire `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, parametrii RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, combinația multi-cadru `[--averaging mean\|median\|inlier_weighted]`, constrângerile geometrice `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, restricția spațială `[--roi X0,Y0,X1,Y1] [--mask PATH]` șisuprascrieri per-slave `[--per-cam-override SN:KEY=VALUE]` (repetabile) | Calculează profilul de aliniere din camerele live. Valori implicite: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (în folderul din care se execută), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` are propriile setări implicite; consultați [tabelul său](#project-align-calibrate-options).) `--prefilter` are ca setare implicită `gradient` (hartă de margini; se potrivește cu aliniatorul GUI/matrice — marginile se mențin pe toate benzile spectrale). `--matcher flann` dă rezultate bune la peste ~5000 de caracteristici; `--averaging median` este robust față de o captură defectuoasă, `inlier_weighted` ponderează în funcție de numărul de potriviri; `--lock-scale` proiectează către cea mai apropiată rotație (fără scalare), `--lock-axis` pune la zero o componentă de translație; `--mask` se aplică tuturor camerelor (utilizați `--per-cam-override` pentru setări specifice fiecărei camere, de exemplu `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` refuză să salveze o calibrare a cărei valoare RMS a reproiectării depășește pragul. `--method manual` preia punctele sale din `--points PATH`: un obiect JSON `{"<serial>": [[x, y], ...], ...}` care furnizează coordonatele în pixeli ale acelorași caracteristici pentru fiecare cameră, inclusiv cele ale camerei de referință (cel puțin 2 per cameră); fiecare dintre `--frames` este rezolvat pe baza acestor puncte. Atunci când camerele sunt stocate într-un tablou conectat, calibrarea utilizează cadrele live ale acelui tablou, iar profilul devine, de asemenea, alinierea tabloului pentru capturile sale. În caz contrar, se deschide un [tablou tranzitoriu](#multi-camera-transient-sessions): o cameră care nu furnizează niciun cadru în oricare dintre `--frames` eșuează la calibrare (ieșire din `1`, denumind-o), iar profilul nu este salvat. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Capturează un cadru multibandă aliniat. `--bit-depth` se potrivește implicit cu camera; `--no-crop` păstrează cadrul complet (completat cu negru); `--interpolation` (implicit `linear`) și `--border-mode`/`--border-value` (implicit `constant`/0) controlează deformarea CPU — calea GPU este oricum biliniară. Deschide un [tablou tranzitoriu](#multi-camera-transient-sessions): o cameră care nu furnizează niciun cadru eșuează la captură (iese din `1`, denumind-o), și nu se scrie nimic. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Cadre multibandă aliniate la flux (aceleași parametri de warp ca `align-apply`), `--fps` implicit `3`. Deschide un [tablou tranzitoriu](#multi-camera-transient-sessions): un grup de cadre pentru care o cameră nu a furnizat nimic nu este salvat, iar trei consecutive încheie fluxul. Dacă a fost omis vreun grup, execuția se încheie prin afișarea mesajelor `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` și `Error: …`, și se închide cu `1`. Un flux care se încheie din cauza oricărei alte erori afișează, de asemenea, `Error: …` și se închide cu `1`. |
| `lattice align-info --profile PATH [--json]` | Afișează detaliile profilului. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Modifică ordinea straturilor. |

#### Index / Matematică privind vegetația

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` mapează simbolurile unei presetări pe benzi** (NDVI: `red`, `nir`), câte un `--channel SYMBOL=BAND` pentru fiecare.

- **`--live`: numai pentru un simbol la care stiva nu poate răspunde.** La fel ca în cazul `--input`, un simbol al unei presetări pe care îl omiteți preia acea bandă din stivă (benzile camerelor activate din profil) al cărei nume începe cu acesta. Acest lucru este verificat înainte ca orice cameră să se deschidă: un simbol la care nu răspunde nicio bandă sau la care răspund mai multe benzi este refuzat atunci, numindu-le, ieșind din `1`. Un `--formula` personalizat necesită un `--channel` pentru fiecare simbol.
- **Produsele `--live` indică cui aparțin.** `index.tif` și `index_LUT.png` conțin informații despre camera din stivă (marcă, model, număr de serie), ora capturii (expunerea camerei de referință, UTC) și obiectivul acesteia, iar ambele denumesc indexul ca fiind banda lor unică (`XMP-Camera:BandName` `NDVI`; fișierul PNG color denumește indexul, nu canalele sale de culoare), la fel cum face un produs `--input`. Un index format din benzile mai multor camere le denumește pe fiecare dintre ele, iar `Model` este `LATTICE` — vezi [Fișiere create din mai multe camere](#files-made-from-several-cameras).
- **`--input`: doar pentru un simbol la care fișierul nu poate răspunde.** Un simbol pe care îl omiteți preia singura bandă din fișier al cărei nume începe cu acesta: `red` preia `Red_660` sau `Red_660_RGN`, iar `nir` preia `NIR_850`. Numele provin din fișierul însuși: lista de benzi a stivei sau, în cazul unei capturi de radianță/reflectanță Chloros sau al unui produs `process`, etichetele sale `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName`. Așadar, `lattice index --input <reflectance capture> --preset NDVI` nu are nevoie de `--channel`.
- **Două benzi care corespund unui singur simbol nu sunt diferențiate** (un stack cu 9 camere are două benzi NIR): denumiți-o pe aceea. Un simbol căruia nu îi corespunde nimic este respins cu `channel_map missing entries`, ieșiți din `1`.**Denumirile benzilor dintr-un stack** indică filtrul camerei: benzile unei camere M3C sunt `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` pe un RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` pe un OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` pe un RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` pe un NGB — iar o cameră M3Mbanda unică a camerei M3M este `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` listează benzile unui profil.

Setul complet de indicatori: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (repetabil), `--capture-level raw|debayered|radiance|reflectance|unknown` (înlocuiește nivelul de captură înregistrat în fișierul TIFF sursă; implicit: citit din metadatele TIFF, revenind la `Chloros:ProcessingLevel` al unui fișier Chloros, astfel încât o captură de reflectanță raportează `reflectance`), `--output PATH` / `-o` (offline: o cale; cu o extensie de imagine, fișierele sunt denumite pornind de la rădăcina acesteia — `-o ndvi_FRGN.png` scrie `ndvi_FRGN.tif` și `ndvi_FRGN_LUT.png` — iar fără o astfel de extensie, se folosește un prefix, `-o ndvi` -&gt; `ndvi.tif`, `ndvi_LUT.png`; implicit lângă intrare, ca `<input name>_<preset>`; cu `--live` și fără `-o`, un folder `output/` sub folderul din care executați comanda), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. În cazul fișierului `--live`, se aplică și butoanele de aliniere: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (ținta de expunere automată a camerelor, vezi regulile privind tranzitorii), `--serials SN1,SN2,…`. `--live` deschide un [tablou de tranzitorii](#multi-camera-transient-sessions): o cameră care nu transmite niciun cadru eșuează execuția (ieșire din `1`, denumind-o), iar stiva sau indexul nu sunt scrise. Se deschide `--serials` (prima declanșează triggerul), altfel camerele din profil ale căror intrare `layer_order` este activată, cu referința sa prima — referința chiar și cu benzile sale dezactivate, deoarece celelalte sunt aliniate la aceasta și declanșează triggerul. O cameră ale cărei benzi sunt dezactivate nu este niciodată calibrată, astfel încât o cameră RGB dezactivată în profil nu mai eșuează la un indice de reflectanță (`no-absolute-radiance-scale`). Un profil care nu numește nicio cameră deschide toate camerele din rețea, ca și până acum.

Fiecare formular rulează pe backend, deci acesta trebuie să fie în funcțiune: porniți-l din interfața grafică Chloros sau cu comanda afișată de CLI atunci când nu îl poate accesa. În modul offline, backend-ul citește `--input` și scrie fișierul TIFF cu valoare flotantă și fișierul PNG colorat, astfel încât un `--input` relativ sau `-o` se referă la folderul din care executați comanda (a se vedea [Convenții](#conventions)). Ambele produse offline conțin înregistrarea camerei sursei de intrare, la fel ca un export de index `process`: marcă / model / număr de serie, GPS, etichetele obiectivului, ora capturii (cea a sursei de intrare; când EXIF-ul acesteia nu indică nimic, momentul expunerii înregistrat de aceasta sau data din numele fișierului, în UTC) și identitatea de calibrare MAPIR — niciodată calibrarea raw-DN, care nu se aplică unui index — plus `Chloros:ProcessingLevel=index` și expresia indexului; numele TIFF de tip float indică o bandă, presetarea. `--list-presets` și `--list-gradients` afișează tabelele de presetări și de gradient ale backend-ului. O formulă respinsă de backend sau o bandă `--channel` pe care fișierul TIFF nu o conține determină ieșirea din `1` cu motivul indicat de backend.

> **Simbolurile `--channel` disting între majuscule și minuscule.** Partea cu simbolul trebuie să corespundă exact cu numele canalelor presetării (presetările folosesc litere mici, de exemplu NDVI = `red`,`nir` — verificați `--list-presets`), iar partea cu banda trebuie să corespundă cu un nume de bandă din stiva aliniată (sau să fie un indice de bandă începând de la 0 în modul offline). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` funcționează; `--channel RED=660` generează o eroare `channel_map missing entries` .

#### `lattice index` — măști de stivă multibandă

La `--capture-level radiance` sau `reflectance`, procesul de separare 3×3 al fiecărei camere poate limita o bandă la 0 (vezi [Mască de limitare a separării](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). O stivă multibandă conține benzi de la mai multe camere, iar fiecare cameră își limitează propria bandă, astfel încât `--live --save-multiband` scrie **o mască pentru fiecare cameră sursă** lângă stivă:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Fiecare mască este de tip `uint8` cu o singură bandă pe grila de pixeli a **stivei** (prin aceeași deformare de aliniere și decupare), cu biții proprii ai acelei camere: bitul 0 = banda canalului R al acesteia, bitul 1 = G, bitul 2 = B. O bandă a stivei preluată din canalul *c* al unei camere este limitată acolo unde este setat bitul *c* al acelei camere (R = 0, G = 1, B = 2).
- Proveniența încorporată a fiecărei benzi indică numele măștii camerei respective (`unmix_clamp_mask`, relativ la folderul stivei, sau `none`) și bitul de citit (`unmix_clamp_mask_bit`).
- O cameră a cărei operație de separare a culorilor nu a limitat nimic nu primește niciun fișier; o cameră monocromă sau RGB nu execută operația de separare a culorilor și, de asemenea, nu primește niciun fișier. Interfața CLI afișează fiecare mască sub stivă (`clamp mask SN…: …`); răspunsul le listează ca `clamp_masks`.
- Fișierul index TIFF/PNG al aceleiași rulări este calculat din același stivă, pe aceeași grilă, astfel încât măștile îl descriu și pe acesta. `--capture-level raw` nu execută operația de separare a culorilor și nu scrie nicio mască.
- Stiva și indexul se află direct în `--output`, astfel încât măștile se află în propriul `clamp_masks/` al `--output` — niciodată deasupra folderului pe care l-ați numit. Acel folder conține tipuri diferite de produse (stivă, index, randare colorată): selectați fișierele din acesta, nu transmiteți întregul folder către software-ul de fotogrammetrie.

#### Conexiuni persistente (Smart-Prep, flux echivalent GUI)

Aceste comenzi mențin camerele deschise în pool-ul backend între invocările CLI.

| Subcomandă | Scop |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Adăugă o cameră la grup (o singură cameră, fără matrice). [Opțiunile comune](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` și `--packet-size` (`standard` = 1500, `jumbo` = 9000 sau octeți; sondele `auto`) sunt aplicate la conectare (un comutator de tip `--exposure` sau `--gain` care dezactivează modul automat). Comanda se întoarce odată ce fluxul camerei a început (așteptând aproximativ 15 s pentru asta, apoi 10 s pentru primul său buffer complet) și afișează `stream: streaming` sau `stream: no_frames -- connected, no frames yet: …` când fluxul nu a livrat nimic — în cazul unei camere care utilizează pachete jumbo, cu recomandarea de a vă reconecta cu `--packet-size standard`. În timp ce o cameră se conectează (prima conectare descarcă calibrarea acesteia, aproximativ 70 s), o altă comandă care necesită o cameră așteaptă cel mult 5 s, apoi eșuează cu codul HTTP 503 `CAMERA_POOL_BUSY`, indicând camera care se conectează; rulați-o din nou peste un moment. `lattice status` și lista de camere nu așteaptă niciodată: o cameră în curs de conectare apare ca fiind în curs de conectare, cu faza sa. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Eliberare. Camera este readusă la starea în care a fost găsită la conectare (cablarea declanșatorului, PTP, setări); `--keep-settings` își salvează setările în propria memorie, iar `--clear-settings` resetează memoria respectivă la setările din fabrică (vezi [Memoria camerei](#camera-memory----keep-settings----clear-settings)). O cameră care face parte dintr-un grup eliberează întregul grup. |
| `lattice cam-list` | Listează camerele din grup. |
| **`lattice array-connect`**|**Conectează o matrice sincronizată persistentă (punctul de intrare recomandat).** Rulează fluxul complet de pregătire inteligentă prin interfața grafică. Aproximativ un minut pentru 9 camere; imediat după pornirea camerelor poate dura câteva minute. În timp ce așteaptă, afișează fiecare etapă atinsă de backend, ca `    <s> s: <step>` (de exemplu `    41 s: open SN:214000535 (4 of 9)`), iar jurnalul backend-ului conține o linie de cronometrare pentru fiecare etapă (`[array-connect <id>] <step>: <s> s …`) și o ultimă linie cu etapele cele mai lente. Așteaptă până la 600 s. O conexiune care nu s-a realizat până atunci este anulată de către backend: fiecare cameră pe care a deschis-o este readusă la starea în care a fost găsită și închisă, comanda afișează `Error:` cu pasul la care ajunsese și iese la `1`, iar nicio matrice nu este conectată (504 HTTP `CLIENT_DEADLINE_PASSED`). Ctrl+C anulează conexiunea în același mod (`Interrupted: …`); dacă matricea se conectase deja, linia indică acest lucru și menționează `array-disconnect` care o eliberează. O comandă oprită în timp ce așteaptă (`kill -9`) face ca conexiunea să fie anulată atunci când backend-ul nu a mai primit nicio informație de la aceasta timp de 30 de secunde. Un array care este activ, dar nu are încă un grup de cadre sincronizate, este totuși raportat ca fiind conectat, cu fluxul său indicat ca `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Eliberarea unei matrice. Fiecare cameră este lăsată să funcționeze liber (`TriggerMode` `Off`), cu toate celelalte setări așa cum au fost găsite la conectare — setările de declanșare și de linie de sincronizare (cele ale masterului`TriggerSource` revine la starea inițială, iar linia de sincronizare revine la o intrare), PTP-ul și setările camerei modificate de sesiune — astfel încât o comandă pentru o singură cameră aplicată unui membru funcționează imediat. `--keep-settings` salvează setările fiecărei camereașa cum au fost lăsate de sesiune în memoria proprie a camerei; `--clear-settings` resetează memoria respectivă la setările din fabrică. Vezi [Memoria camerei](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Listează matricea conectată. |
| `lattice array-status [--array-id ID]` | fps live, PTP, ultima eroare. `ptp_enabled` este egal cu `True` numai atunci când fiecare membru are opțiunea PTP activată; `ptp_off:` indică un membru care nu o are. Linia `state:` citește `restarting (applying settings)` în timpul unei reporniri a setărilor, `recovering (…)` în timp ce un membru care a rămas fără alimentare este redeschis (vezi nota de mai jos), `stalled (…)` în timp ce bucla de captare rulează, dar nu publică niciun grup de cadre, și `stopped (…)` odată ce bucla de captare a renunțat (reconectați matricea). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | O captură sincronizată din matricea live — Singură / Continuă / La intervale / Cel mai rapid. **Implicit este `all`**(un fișier per tip de export aplicabil per cameră),**aliniat** la modul de captură al aplicației (`--no-aligned` pentru fișiere native). Camerele omise (de ex. RGB, excluse din radianță/reflectanță) sunt raportate cu `Skipped: SN:<serial> (<reason>)`; citirea DAQ a capturii utilizate este salvată împreună cu aceasta, se include o captură doar în format brut și se raportează cu `DAQ: <path>` ([DAQ-ul care citește o captură salvează](#the-daq-reading-a-capture-saves)). A se vedea [Moduri de captură, înregistratoare și reprocesare offline](#capture-modes-recorders--reprocesare-offline). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Înregistrează vizualizarea combinată a indicilor în timp real ca video/GIF (calitate de monitorizare; fără nicio fereastră GUI deschisă, backend-ul redă NDVI-ul matricei, sau `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Rafală raw-Bayer cu fps ridicat (calitate de analiză; reprocesare offline). La fel ca `array-record` și `array-capture --continuous`, se conectează la backend la fiecare interogare: oprit fără a se opri (`kill -9`), backend-ul îl oprește la 30 de secunde după ultima verificare (`stop_reason` `client_gone`; manifestul seriei este finalizat și se începe construirea indexului combinat, la fel ca în cazul unei opriri), iar matricea rămâne conectată. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Reprocesarea unei rafale brute salvate în videoclipuri calibrate. |

> **O cameră care rămâne fără alimentare sau își pierde conexiunea revine automat.** Când un membru nu mai răspunde (un ciclu de repornire a comutatorului reinițializează fiecare cameră pe care o alimentează), matricea așteaptă, o redeschide când aceasta răspunde din nou, și îi restabilește setările stabilite de `array-connect`: PTP, linia de declanșare și sincronizare, ritmul de rețea, formatul pixelilor și dimensiunea cadrului, precum și expunerea automată. Apoi reia transmisia fără un nou `array-connect`. Între timp, `array-status` afișează `state: recovering (...)`, iar `last_error` indică camera pe care o așteaptă; ulterior, afișează `re-opened:`. O cameră care s-a repornit revine cu PTP dezactivat, așa că, până când este redeschisă, `ptp_enabled` este `False`, iar `ptp_off:` îi atribuie un nume.

> **O cameră reținută nu este niciodată deschisă a doua oară.** O a doua deschidere oprește fluxul de date al celui care o reține și îi rescrie setările, putând provoca blocarea backend-ului. Așadar, în cazul unei camere deținute de pool (interfața grafică sau `cam-connect`), a unui element al unui array sau a unei camere deținute de o sarcină de fundal (un `lattice continuous` în execuție, `multi-stream` sau `align-stream`, al său sau al altui client), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, o-shot `white-balance` și `presets --apply` refuză cu codul HTTP 409 `CAMERA_HELD`, menționând deținătorul și comanda de eliberare (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>` sau, în cazul unei sarcini, ID-ul acesteia și cine a inițiat-o) și ieșire `1`. La fel se întâmplă și cu o cameră pe care o altă comandă o folosește chiar acum (o comandă `multi-test` sau `multi-capture` care își deschide camerele, o altă comandă de tip „one-shot”), pentru fiecare comandă de aici: eroarea indică să aștepți ca acea comandă să se termine. `lattice power` fără `--sleep` citește prin propriul identificator al deținătorului, acolo unde există unul (pool-ul, un array, o cameră deschisă a unei sarcini), și este refuzat pentru o cameră pe care o utilizează o altă comandă. Fiecare răspuns `CAMERA_HELD` conține codul atât în `code`, cât și în `error_code`. Pentru comenzile care deschid fiecare cameră, consultați [Multi-Camera (Sesiuni tranzitorii)](#multi-camera-transient-sessions).

##### Opțiuni `array-connect`

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `--serials SN1,SN2,…` | detectare automată a tuturor camerelor LATTICE (necesită ≥2) | Prima cameră din serie este MASTER. Când este omisă, detectarea filtrează modelele LATTICE (M3C / M3M) și le conectează pe toate. |
| `--line {Line0,Line2,Line3}` | `Line2` | Linie de sincronizare GPIO. |
| `--target-fps F` | automat | Rata de declanșare a Masterului. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automat | Ignoră selectorul de nivel. |
| `--wire-ceiling-mbps MB_PER_S` | detectat automat | **Bugetul de lățime de bandă susținută al gazdei, în MB/s — valoarea de care depinde întreaga alocare a matricei.** Reduceți-l când matricea raportează cadre corupte GVSP: valoarea automată este derivată din rata de legătură anunțată de placa de rețea, care supraestimează adaptoarele USB, benzile PCIe subțiri și structurile partajate aglomerate. Se păstrează în blocul de captură a matricei din proiect, astfel încât o reconectare prin redeschidere / CLI / SDK o restabilește. Consultați [Starea matricei](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | niciunul (senzor complet) | Binning hardware (întotdeauna „Medie”). |
| `--width N` / `--height N` / `--pixel-format FMT` | senzor complet; format per cameră | O dimensiune explicită a cadrului sau un format de pixeli pentru fiecare cameră, în loc de senzorul complet și alegerea formatului per cameră (Mono12 / BayerRG12). |
| `--no-recommend` | dezactivat | Omite etapa de analiză a rețelei (fiecare cameră își păstrează atunci dimensiunea curentă, în loc să fie setată la senzorul complet). |
| `--no-ptp` | dezactivat | Dezactivează PTP (marcajele de timp între camere **nu** sunt atunci comparabile). |
| `--packed {auto,on,off}` | `auto` | Transmite cadre de 12 biți comprimate și decomprimă-le pe acest computer — mai multe cadre pe secundă la capacitatea maximă a senzorului, date identice. `auto` comprimă doar atunci când este util și acest computer are resurse suficiente; `off` nu comprimă niciodată. Vezi [Transport comprimat pe 12 biți](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | Dimensiunea pachetului GVSP pentru fiecare cameră din matrice. `auto` permite fiecărei camere să sondeze rețeaua; `jumbo` = 9000, `standard` = 1500 sau o dimensiune în octeți o blochează. Înainte de versiunea 1.3.0, `array-connect` accepta acest indicator și îl ignora. |

> **Senzor complet în mod implicit.** Fără `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect` setează fiecare cameră pe senzor complet — fără
> binning, fără decupare ROI — și permite ca rata de cadre să scadă la nivelul pe care îl pot oferi acest computer și
> rețeaua, în loc să micșoreze cadrele pentru a menține rata ridicată. Linia
> de analiză a rețelei afișează rata proiectată, iar linia următoare listează
> opțiuni mai rapide (binning sau un ROI mai mic) împreună cu ratele lor proiectate; treceți
> `--binning` sau `--width`/`--height` pentru a alege una dintre acestea. Comanda „Connect” nu efectuează niciodată binning sau decupare
> în mod automat.

##### Transport comprimat pe 12 biți (`--packed`)

La capacitate maximă a senzorului, rata de cadre a unei matrice este determinată de viteza cu care fiecare cameră poate transmite
cadrele sale. Formatele pe 12 biți (`BayerRG12`, `Mono12`) sunt transmise în mod normal ca cuvinte pe 16 biți;
transportul **comprimat** trimite aceiași 12 biți în 1,5 octeți pe pixel și
le despachetează pe acest computer, astfel încât fiecare cadru necesită cu 25% mai puțin timp pentru a fi transmis —
aproximativ **cu 33% mai multe cadre pe secundă la capacitate maximă a senzorului** (măsurat: 10,7 → 14,0 fps
pe o matrice de 9 camere). **Datele salvate sunt identice**: aceleași valori, aceleași
fișiere pe 16 biți, aceleași metadate.

- `auto` (implicit) comprimă doar atunci când este util **și** acest computer își poate permite
  decomprimarea. La conectare, backend-ul își măsoară propria viteză de dezarhivare, numărul de nuclee CPU,
  încărcarea curentă a procesorului și lățimea de bandă disponibilă, astfel încât un computer desktop cu nuclee disponibile comprimă
  la capacitate maximă a senzorului, în timp ce un computer ocupat sau cu putere redusă transmite fluxul fără comprimare. ROI-urile mici și cadrele grupate câștigă
  puțin.
- `off` nu comprimă niciodată — folosiți-l pentru a exclude comprimarea în timpul diagnosticării sau pe un
  computer care trebuie să-și păstreze toată puterea procesorului pentru alte sarcini.
- `on` comprimă întotdeauna formatele de 12 biți.

Ieșirea „connect” afișează decizia și motivul (`comprimat 12 biți: pe (auto:
proiectat 11,0 -&gt; 14,4 fps, …)`); `/api/camera/array/status` raportează
`packed_transport` și `packed_decision`. Pentru a seta o valoare implicită la nivel de mașină
în loc să treceți indicatorul de fiecare dată, setați `CHLOROS_ARRAY_PACKED12=off` (sau
`on`) în mediul backend-ului — un `--packed on|off` explicit are totuși prioritate.
Formatele pe 8 biți și 16 biți nu sunt niciodată comprimate.

### Un singur fișier de proiect, pentru toate modurile

Un dosar de proiect conține setările fiecărui dispozitiv: `cameras.json` — ale fiecărei camere (`cameras[].settings`, pentru o singură cameră și pentru fiecare membru al matricei) și setările la nivel de matrice ale fiecărei matrice (`arrays[].capture`: format, ROI, binning, rata de declanșare, buget de cablare) — precum și `sensors.json`, ale fiecărui senzor de lumină (timp de integrare, medie pe cadru, limită maximă). Aplicația, CLI-ul, SDK-ul și serverul MCP citesc și scriu toate aceleași fișiere, în același mod:

- **Un dispozitiv conectat în cadrul unui proiect este salvat în acesta și înregistrează fiecare modificare în el.** „În cadrul unui proiect” înseamnă cu un proiect deschis în backend (al aplicației) sau — din partea SDK-ului — prin intermediul unui `ChlorosProject`, care își denumește propriul folder. O cameră, o matrice sau un senzor de lumină pe care proiectul nu le conține încă sunt adăugate la fișierele acestuia atunci când se conectează (`lattice cam-connect`, `array-connect`, `daq pool-connect`, aplicația, SDK-ul, `chloros-mcp`), împreună cu setările sale. O modificare din orice mod (`lattice exposure`, `gain`, `configure --set`, `resolution`, setările unei matricesetările și frecvența de declanșare a unui array, capacul unui senzor de lumină, comenzile aplicației) este salvată în fișierele acelui proiect o clipă mai târziu. O expunere, un câștig sau o țintă AE setată pe o cameră este salvată ca fiind menținută, astfel încât redeschiderea proiectului o menține din nou.
- **Deschiderea unui proiect configurează dispozitivele sale în conformitate cu acesta**, în orice mod: o cameră, o matrice sau un senzor de lumină conectat fără setări proprii preia setările salvate ale proiectului (`lattice cam-connect` / `array-connect` cu un proiect deschis, `chloros-cli project connect`, `open_project().connect_all()`, redeschiderea aplicației). O matrice conectată sub un ID nou actualizează matricea salvată de proiect cu aceleași camere; o matrice cu alte camere este adăugată alături de aceasta.
- **Salvarea propriu-zisă a aplicației nu suprascrie niciodată setările active ale unui dispozitiv** cu o copie mai veche a acestora și păstrează modificările specifice fiecărui membru salvate de un apelant CLI sau SDK.
- **Formatul de pixeli pe care îl setați este cel păstrat de proiect.** Un format setat în Setările matricei, prin `--pixel-format` sau prin `connect_array(pixel_format=)`, inclusiv cel de 8 biți, este salvat și reconectat exact. Dacă nu este setat nimic, fiecare cameră se conectează la setarea implicită de 12 biți (`BayerRG12`, `Mono12` pe un senzor monocrom). Numele unui format indică adâncimea de biți a acestuia; familia aparține senzorului, astfel încât o cameră color nu este niciodată setată pe un format monocrom, iar o cameră monocromă nu este setată pe un format Bayer, iar linia `[PixelFormat <serial>] ... written` indică acest lucru atunci când un format a fost mapat. Exporturile capturilor de 8-biți sunt de 8 biți, inclusiv reflectanța: redimensionate și marcate cu `PixelScale`.
- Un dispozitiv conectat fără a fi asociat unui proiect nu înregistrează nimic.
- **O înregistrare urmează unui proiect creat după ea; o deschidere o lasă cu proiectul respectiv.**Când se creează un proiect — un SDK `create_project()`, `chloros-mcp` al lui `create_project` sau opțiunea**Proiect nou**— sau duplicat (opțiunea**Duplicare**din aplicație, care deschide copia) în timp ce o înregistrare încă scrie în proiectul care era deschis anterior, înregistrarea continuă, iar ceea ce scrie ajunge în noul proiect, în același loc din interiorul acestuia (mai jos). Deschiderea unui proiect existent — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, opțiunea**Deschide proiect** a aplicației — lasă o înregistrare încă în curs de desfășurare în proiectul în care se înregistrează: aceasta se finalizează acolo, iar fișierele sale se alătură listei de fișiere a acelui proiect (aplicația oprește fiecare înregistrare înainte de a deschide un proiect).
  - `array-record` și `array-burst`: videoclipul sau folderul cu toate cadrele se mută integral acolo când înregistrarea se oprește (este `--duration`, Ctrl+C). Fișierul `Burst dir` generat este cel nou, iar `--build` și `array-build-video` funcționează pe baza acestuia. Dacă unitatea de stocare refuză mutarea, înregistrarea rămâne acolo unde a fost scrisă; jurnalul de la sfârșitde la sfârșit indică motivul, la fel și răspunsul HTTP-ului la oprire (`relocation_error`). Cadrele brute ale unei secvențe oprite și `.daq` se adaugă la lista de fișiere a proiectului în care ajung; un videoclip (un `array-record`, un videoclip generat dintr-o serie) nu este adăugat la lista de fișiere a unui proiect.
  - `array-capture --continuous`: fiecare captură realizată după modificare este salvată în noul proiect; cele salvate anterior rămân unde sunt.
  - `daq pool-record` și `project sensor log` (precum și înregistrarea propriu-zisă a aplicației cu senzorul de lumină): fișierul `.daq` se mută în noul proiect`light_sensor` al noului proiect atunci când înregistrarea se oprește și se alătură listei de fișiere a acelui proiect.

  O înregistrare într-un folder din afara proiectului deschis (un `-o` situat în altă parte) rămâne acolo unde se află, iar `lattice continuous` și `multi-stream` păstrează folderul care le-a fost atribuit.
- **În aplicație, capturile efectuate după crearea unui proiect nou sunt incluse în acesta.** Crearea sau duplicarea unui proiect menține active toate sesiunile de captură și înregistrările în curs, iar fiecare captură efectuată după aceea (Captură totală, Începe capturarea) este salvată în noul proiect și se adaugă la lista de fișiere a acestuia. Deschiderea unui proiect existent în aplicație le oprește mai întâi pe toate.
- **Un proiect nu este mutat în timp ce se înregistrează.**Mutarea proiectului deschis într-un alt director de lucru (din**Setări → Director de lucru** al aplicației; nicio comandă CLI nu mută un proiect, ci doar un client HTTP care trimite date către `/api/move-project`) este refuzată cu codul HTTP 409 `PROJECT_RECORDING` în timp ce o înregistrare matricială sau o serie de capturi, o captură matricială continuă, o sarcină `lattice continuous` / `multi-stream` sau o înregistrare a senzorului de lumină scrie în acesta: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Aplicația refuză în același mod, înainte de a solicita un dosar, în timp ce propriile funcții „Capture All”, „Start Capturing” sau înregistrarea cu senzorul de lumină sunt în desfășurare.

Acest lucru este separat de starea în care sunt lăsate camerele la deconectare (vezi [Memoria camerei](#camera-memory----keep-settings----clear-settings)): un proiect își configurează singur camerele la deschidere.

### Memoria camerei — `--keep-settings` / `--clear-settings`

Când este conectat un ansamblu, camerele sale preiau setările ansamblului, iar tu le poți modifica ulterior. Când Chloros dezactivează camerele — `array-disconnect`, sfârșitul unei comenzi tranzitorii (`multi-capture` fără matrice, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), oprirea lui `lattice continuous`, sfârșitul unui `lattice capture` al unei camere pe care nu o deține — le readuce pe fiecare în starea în care se aflau înainte de conectare: cablajul declanșatorului și al liniei de sincronizare, setările PTP și ale camerei (lista completă se găsește la [Multi-Camera (Sesiuni tranzitorii)](#multi-camera-transient-sessions)).

Pentru utilizarea camerelor în afara unui proiect, două indicatori din oricare dintre aceste comenzi scriu în **memoria proprie** a camerei — setările cu care pornește, indiferent de software:

- **`--keep-settings`** salvează setările fiecărei camere așa cum au fost lăsate în memoria acesteia la încheierea sesiunii (UserSet2, setul de pornire). Camerele rămân în continuare să funcționeze liber (`TriggerMode` `Off`), cablajul liniei de sincronizare rămânând așa cum a fost găsit. O cameră fără UserSet2 salvează în UserSet1; acest computer reține acest lucru, astfel încât sistemul de auto-reparare al matricei Chloros — care resetează un UserSet1 care s-a abătut de la valoarea de referință a Chloros — nu intervine asupra acestuia.
- **`--clear-settings`** resetează memoria fiecărei camere la setările din fabrică: valoarea de referință a Chloros, cea pe care funcția de autoreparare a matricei o încorporează — setările implicite din fabrică cu `TriggerMode` `Off`, `TriggerSource` `Line2`, expunere automată, balansul de alb calibrat al unei camere RGB păstrat — în UserSet1, care a devenit setul de pornire.
- Cele două nu pot fi combinate. Fiecare cameră primește o linie: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` sau `SN:…: camera memory reset to the factory settings.`; răspunsul `camera_memory` este identic pentru fiecare cameră.

Aplicația nu oferă aceste indicatori.

**Orice ieșire restabilește camerele la starea inițială.**Aceeași restabilire la starea inițială se execută pe fiecare cale de ieșire, nu doar la comenzile de mai sus: `cam-disconnect` (și camera**X**a aplicației);**X**-ul aplicației pe un tablou sau pe rândul oricărui element al tabloului (un tablou nu poate rula fără un element, așa că întregul tablou este eliberat); închiderea aplicației (aceasta acordă backend-ului până la 20 de secunde pentru a repune fiecare cameră, matrice și comanda `lattice continuous` în funcțiune înainte de a o opri; o comandă care încă rulează în acel moment — o comandă tranzitorie care își deschide camerele sau așteaptă ceasurile PTP ale acestora, o comandă `lattice capture` de o singură execuție, `power`, `white-balance` sau `presets --apply` — se oprește la următorul punct de siguranță, repune mai întâi camerele, timp de până la 8 s, și răspunde cu HTTP 503 `SHUTTING_DOWN`; o conexiune de tip „array” încă în desfășurare — `lattice array-connect`, conexiunea aplicației — se oprește înainte de următoarea cameră și închide camerele pe care le-a deschis, pentru o perioadă de până la 10 s, și răspunde cu HTTP 503 `SHUTTING_DOWN`; iar una care ajunge în timpul închiderii nu deschide nimic: HTTP 503 `SHUTTING_DOWN`); și o comandă CLI care este oprită sau al cărei terminal se închide (SIGTERM / SIGHUP îi opresc activitatea — și își închide camerele — înainte de a ieși din `143` / `129`, ca și Ctrl+C). Un `lattice continuous`, `multi-stream` sau `align-stream` oprit definitiv (SIGKILL), care nu poate opri nimic, are activitatea oprită și camerele readuse la starea inițială de către backend la 30 de secunde după ultima sa verificare. Un backend care se blochează sau este oprit forțat nu poate restabili nimic; o repornire readuce fiecare cameră la setările sale din memorie, iar următoarea conexiune Chloros reconfigurează o cameră pe care o găsește activată.**Un proiect își configurează singur camerele.** Când o matrice se conectează cu un proiect deschis, fiecare membru primește setările pe care proiectul le-a salvat pentru el — o expunere, un câștig sau o țintă AE menținute pe el (considerate ale tale, astfel încât expunerea automată nu le modifică) și modul său de balans de alb (corecția Chloros; balansul de alb propriu al camerei rămâne dezactivat) — indiferent de ce conține memoria camerei.

### Smart-AE / Smart-Capture

Rețelele LATTICE rulează AE continuu în fundal imediat ce sunt conectate, dar o scenă nou orientată are nevoie de un moment pentru a converge. `array-capture --smart` reprezintă **soluția convenabilă**: așteaptă ca AE să se stabilizeze pe toate camerele din matrice, apoi declanșează capturarea. Folosiți-l atunci când schimbați scenele în timpul sesiunii.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Politica de stabilizare este conservatoare în mod implicit: timp de așteptare de 5 s, fereastră de stabilitate de 1,5 s, toleranță de variație a expunerii de ±5 %. Reglați prin intermediul SDK-ului (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) dacă aveți nevoie de un comportament diferit din partea automatizării.

#### Cum funcționează expunerea automată

- **Mai întâi expunerea, apoi amplificarea.** Cu amplificarea setată pe „Auto”, aceasta rămâne la 0 dB până când expunerea atinge cea mai mare valoare permisă.
- **O expunere manuală menține câștigul pe „Auto”** până când modificați câștigul sau îl setați pe „Manual”. Revenind la „Auto”, limita câștigului este aceeași ca înainte.
- **Zonele foarte luminoase nu întunecă scena.** O reflexie de până la 1 % din cadru este ignorată, iar în cazul panourilor unei ținte de calibrare cunoscute, până la 0,5 % dintr-un panou.
- **Limita maximă de expunere DAQ se aplică numai cât timp o țintă de calibrare se află în câmpul vizual.** Aceasta împiedică decuparea unui panou alb; dacă nu se vede niciun panou, camera expune pentru scenă.
- **Lumina constantă rămâne stabilă, iar lumina modificată se stabilizează în câteva secunde.** La lumină constantă, expunerea și limita acesteia rămân neschimbate, iar după o schimbare majoră a luminii, imaginea se stabilizează în câteva secunde. O suprasaturare nu mai limitează expunerea definitiv, iar o modificare a zonei de interes (ROI) restabilește expunerea.
- **Ținta de luminozitate rămâne la îndemână.** Aceasta nu crește cât timp camera se află la limita de expunere și amplificarea sa nu poate crește, astfel încât o lumină care revine nu produce o strălucire excesivă în cadru.
- **O cameră a cărei expunere automată nu mai răspunde este menținută la nivelul corespunzător luminii în care se află.** În lumina puternică a soarelui, o cameră ar putea coborî la expunerea sa minimă și să rămână acolo cu un cadru întunecat, toate setările sale indicând valori normale. Chloros monitorizează cadrul, menține camera la nivelul stabilit pentru acea lumină și o eliberează când lumina se schimbă. Un senzor de lumină întunecat sau acoperit nu afectează acest proces.

### Niveluri de captură și export (setarea implicită `all`)

Începând cu această versiune, `lattice capture`, `lattice multi-capture` și `lattice array-capture` **sunt setate implicit la `--processing all`** — un fișier salvat pentru fiecare tip de export, care se aplică fiecărei camere, corespunzând comportamentului opțiunii „Capture All” (Captură totală) din interfața grafică. Nivelurile sunt:

| Nivel | Ieșire | Se aplică la |
| --- | --- | --- |
| `raw` | Bayer monocanal (camere monocrome: banda unică) direct de la senzor. | Toate camerele. |
| `debayered` | Culoare pe 3 canale, salvată în ordinea canalelor **RGB**(un fișier TIFF cu interpretare fotometrică RGB; fișierele PNG/JPEG se deschid, de asemenea, ca RGB): offsetul de întuneric (DSNU) din pachetul de calibrare este eliminat din datele brute, apoi se aplică demosaicarea bazată pe margini a OpenCV— fără LUT de liniaritate, câmp plat sau demixare (camere monocrome: 1 canal în tonuri de gri). Cadrele demosaicate în memorie pe care le returnează Python SDK (`grab()`, `frame_stream()`) sunt**BGR**, în ordinea OpenCV; doar fișierul salvat este în format RGB. | Toate camerele. |
| `radiance` | float32 W/m²/sr/nm prin lanțul radiometric complet. | Numai multispectral (M3C/M3M) — **omise pentru camerele cu filtru RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), compatibil cu Pix4D. | Numai multispectral și **numai când un DAQ este asociat + camera este calibrată**; în caz contrar, se omite. |
| `display` | Lanț complet de previzualizare GUI (CCM + WB + gamma conform profilului camerei). Salvate într-un folder `preview/`. `lattice capture` acceptă, de asemenea, numele `preview` pentru acesta; `array-capture` / `multi-capture` acceptă doar `display`. | Toate camerele. |

Treceți un singur nivel pentru a salva doar acela (`--processing debayered`). Când solicitați `all`, nivelurile care nu se aplică unei anumite came sunt omise (și raportate), nu sunt tratate ca erori — o cameră neconectată sau necalibrată primește în continuare `raw` / `debayered` / `preview`.

#### Citirea DAQ-ului care realizează o captură este salvată

Ori de câte ori un DAQ este asociat cu camera sau matricea, sau este inclus într-un grup, citirea în direcție descendentă utilizată pentru o captură este scrisă într-un fișier **`.daq`** din subfolderul `daq/` al folderului de capturi, împreună cu `.match.json` (cât de departe a fost citirea față de expunere) și `.cameras.json` (a cui citire este), și este raportată pe o linie `DAQ:`. Acest lucru este valabil pentru fiecare nivel, inclusiv pentru o captură brută sau debayered, astfel încât orice captură poate fi reprocesată ulterior în radianță, reflectanță sau un indice. Este același fișier, cu același nume și fișiere auxiliare, indiferent dacă captura a provenit din GUI, CLI sau SDK. În absența oricărei citiri DAQ, nu există `.daq`, iar un `multi-capture` fără matrice conectată (soluția de rezervă tranzitorie) nu salvează nimic.

| Indicator | Efect |
| --- | --- |
| `--no-daq` | Nu salvați citirea unei capturi ale cărei niveluri nu o utilizează (`raw`, `debayered`). Un nivel `radiance`, `reflectance` sau `display` o scrie totuși, deoarece reimportarea are nevoie de citirea cu care a fost realizat produsul respectiv. |
| `--daq-tolerance-ms MS` | Cât de departe de expunere poate fi valoarea măsurată pentru a fi încă considerată în limitele de toleranță (implicit 500). Se schimbă doar verdictul (`.match.json`, `in_tolerance` și `tolerance_ms` pentru , respectiv `DlsInTolerance` pentru reflectanță), niciodată valoarea măsurată utilizată. Vezi [O valoare DAQ departe de expunere](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Metoda mai veche de a solicita `.daq` într-o captură exclusiv în format raw. Acum este setarea implicită, iar `lattice capture` și `array-capture` acceptă în continuare acest indicator (`multi-capture` nu îl acceptă). `--no-daq` are prioritate asupra acestuia. |

Care DAQ: cel asociat camerei sau matricei (selecția senzorului de lumină din interfața grafică, o asociere salvată a proiectului sau `chloros-cli daq pool-bind`), altfel cel mai recent pe care îl are backend-ul. O serie de imagini brute (`array-burst`) își păstrează propriul `.daq` pentru fiecare citire distinctă în folderul „burst”, ca și până acum.

**Fișierele PNG și JPEG au 8 biți pe canal, așa că acceptă doar fișierele `display` / `preview` și produsele indexate.** `--format png` sau `jpg` cu un nivel pe care nu îl pot stoca sunt respuse înainte de capturarea oricărui lucru, specificându-se nivelul și TIFF-ul ca format care îl stochează:

- `radiance` (float32) și `reflectance` (uint16, `32768` = 1,0) — întotdeauna, astfel încât `--processing all` (implicit) împreună cu `--format png`/`jpg` sunt, de asemenea, respinse: utilizați `--format tiff` sau `--levels display`.
- `raw` și `debayered` provenite de la o cameră care transmite mai mult de 8 biți (cadrele M3C de 12 biți / Mono12 M3M) — CLI-ul nu poate detecta adâncimea de biți a camerei, așa că backend-ul le respinge (400 HTTP). Cadrul`lattice capture --processing debayered` cu un singur cadru preia un cadru de 8 biți pentru PNG/JPEG și nu este respins, la fel ca și un `multi-capture` fără o matrice persistentă (debayering de 8 biți).

Fiecare rută de captură răspunde în același mod (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst`, precum și rutele HTTP din spatele acestora.

### Cum arată un dosar de capturi

Fiecare tip de export este stocat într-un **subdosar propriu**din cadrul `-o`, astfel încât o captură pe mai multe niveluri nu amestecă niciodată tipurile. Schița ilustrează**captura implicită `all` a unei camere de către `lattice capture`** — mai multe niveluri per cameră, astfel încât fiecare fișier poartă sufixul `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Cum variază numele fișierelor:

- **Câmpul camerei — număr de serie, apoi filtru:** fiecare fișier de captură poartă numele camerei care l-a realizat sub forma `SN<serial>-<filter>`, unde `<filter>` reprezintă partea de filtru a modelului camerei: `SN261902142-F450` pentru un `M3M-L87-F450`, `SN213800234-FRGN` pentru un model `M3C-L87-FRGN`, `SN214701292-FRGB` pentru o cameră RGB. Filtrul se alătură numărului de serie, formând `-`, nu cu `_`, astfel încât un nume are aceleași câmpuri separate de `_` ca și înainte (prefix, dată, oră, milisecunde, cameră, nivel). O cameră al cărei model nu poate fi citit (un DeviceUserID șters) este denumită doar `SN<serial>`. Capturile din versiunile anterioare nu au niciun filtru în nume; Chloros le citește pe ambele.
- **Prefix:** `lattice capture` denumește fișierele `capture_…`; `array-capture`, `multi-capture` cu o matrice conectată, iar `project capture` / `burst` / `stream` dintr-o matrice le denumește `sync_<ts>_SN<serial>-<filter>…`.
- **Sufix de nivel doar în cazul mai multor niveluri.**O cameră care salvează**mai multe**niveluri într-o singură captură (`all` sau `--levels a,b`) adaugă `_<level>`. O cameră care salvează**un** nivel păstrează numele simplu, fără sufix — `--processing reflectance` scrie `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — astfel încât scripturile scrise pentru capturi cu un singur nivel să funcționeze în continuare. Nivelul este în continuare indicat de folder.
- **Măștile de limitare urmează numele produsului:** `<product name>_clampmask.tif`. O captură de reflectanță cu un singur nivel scrie `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; o captură `all` scrie `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Extensia** urmează după `--format`: valoarea implicită `tiff` generează `.tiff`, iar `--format tif` generează `.tif` (măștile de limitare sunt întotdeauna `.tif`).
- **O singură excepție:** `lattice capture --processing debayered` (sau `--levels debayered`) preia o singură-cadru rapid și scrie `capture_<ts>_SN<serial>-<filter>.tiff` direct în `-o` — fără subfolder.
- **Fluxurile și rafalele** conțin același câmp: `lattice continuous` scrie `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` și o serie neprelucrată `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` reprezintă marcajul temporal al capturii, iar `<serial>` numărul de serie al camerei, astfel încât un grup sincronizat împărtășește un
marcaj temporal comun tuturor camerelor. **Rețineți singura asimetrie:** nivelul `display` este stocat într-un folder
numit `preview/`, în timp ce fișierele cu mai multe niveluri păstrează `_display` în numele — folderul și sufixul diferă
numai pentru acel nivel. Nivelurile necunoscute revin la un folder cu numele propriu, iar dacă subfolderul
nu poate fi creat, fișierul este scris în rădăcina de ieșire, în loc să fie pierdut.

**Reprocesarea unui folder de capturi:**indicați `chloros-cli process` către**directorul rădăcină al capturilor**
(`output/`). `process` importă în mod normal doar folderul pe care îl specificați, dar când acel folder nu conține
imagini și are subfoldere, acesta coboară automat în subfoldere — astfel încât subfolderele de nivel și
subfolderul `daq/` sunt preluate toate dintr-o singură mișcare. Fiecare nivel al unei capturi se importă ca o singură imagine,
celelalte niveluri fiind disponibile ca moduri, în loc de o imagine per nivel.

Denumirea directă a unui **subfolder de nivel** (de exemplu, `output/raw/`) funcționează, de asemenea. Procedând astfel, `daq/`
rămâne în urmă, așa că copiați sau indicați citirea DAQ alături atunci cândderivați un produs radiometric
din `raw/` — altfel, potrivirea marcajului temporal nu are cu ce să se coreleze.

**Prelucrarea începe întotdeauna de la `raw`.** În cadrul fiecărei capturi, cadrul brut este sursa procesului de prelucrare;
`debayered`, `radiance`, `reflectance` și `preview` apar ca moduri de vizualizare, dar nu sunt niciodată redirecționate
înapoi prin pipeline. Reprocesarea unui produs derivat ar aplica din nou vignetarea, CCM-ul și
calculele de radianță care sunt deja integrate în pixelii săi, așa că Chlorosează această opțiune în loc să
efectueze o procesare dublă. Două consecințe demne de menționat:

- Renderizările `index/` și `composite/` nu sunt **niciodată** procesate. Sunt ieșiri, nu capturi —
  un randare LUT NDVI nu are o interpretare semnificativă a radianței.
- Un folder de capturi exportat **fără** `raw` (de ex. `array-capture --processing reflectance`) nu are
  nicio sursă legitimă în fluxul de lucru. Aceste capturi se importă și se afișează normal, dar `process` le omite
  și indică acest lucru:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Dacă aveți cu adevărat nevoie să treceți un produs derivat prin proces — o sesiune hub capturată cu
  `demosaic` activat sau dintr-un folder vechi — `--input-level {raw,debayered,processed}` forțează punctul
  de intrare și anulează omiterea. Acest indicator este o soluție de rezervă deliberată; `auto` (implicit)
  nu procesează niciodată o captură care nu conține date brute.

**Etichete de calibrare: numai pe `raw`.** Fișierul `raw` al unei capturi conține pachetul de calibrare al camerei: încărcătura utilă `CalibrationJSON` și etichetele compatibile cu MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), care descriu valorile brute ale senzorului. Fiecare fișier de la celelalte niveluri (`debayered`, `display`, `radiance`, `reflectance`, randarea `index/`) și fiecare cadru `lattice continuous` / `multi-stream` conține identitatea camerei (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) și înregistrarea propriu-zisă a cadrului, `XMP-mapir:FrameJSON` (expunere, amplificare, temperatură, format pixel, timp de captură, potrivire DAQ), deoarece fiecare produs `process` poartă această identitate. `process` găsește calibrarea unui astfel de fișier după numărul de serie al acestuia, iar `CalBundleSha256` o găsește în cache-ul de calibrare al Chloros, descărcând pachetul o singură dată dacă computerul nu îl are.

#### Fișiere create din mai multe camere

Un fișier creat din benzile mai multor camere — o stivă multibandă (`align-apply`, `align-stream`, `index --live --save-multiband`), un index live creat din mai multe camere (`index.tif`, `index_LUT.png`), o imagine `composite/` dintr-o captură matricială — menționează fiecare cameră care a contribuit la crearea sa:

- **Liste pe cameră, în ordinea benzilor** (imaginea compusă: stratul de fundal, apoi prim-planul; un strat index, camerele din care provin benzile sale): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (pachetul de calibrare prin care au trecut benzile fiecărei camere, altfel — o stivă `align-apply` / `align-stream`, ale cărei benzi nu trec prin niciun pachet — pachetul camerei pe care Chloros l-a stocat în cache, cel după care sunt denumite fișierele sale de captură), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — liste paralele, elementul *n* din fiecare descriind aceeași cameră; `-`, unde o cameră nu are nicio valoare (fără pachet de calibrare, un obiectiv necunoscut). Fiecare aparat foto reprezintă **un** element: în cazul în care pachetul său conține mai multe valori pentru un câmp — certificatele lămpilor cu halogen ale unui pachet — acestea sunt unite cu `;` (`F-1479-1;F-1480-1;F-1481-1`), niciodată cu o virgulă, astfel încât o listă conține un singur element per cameră, indiferent de modul în care este afișată.
- **EXIF `SerialNumber`** conține aceleași numere de serie, separate prin virgulă (`214000535,214000551`).
- Nu conține **nicio** identitate specifică unei singure camere (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): acestea denumesc o cameră și pachetul acesteia, iar `process` ar determina calibrarea fișierului pe baza acestora. Un index calculat din benzile mai multor camere (`index.tif`, `index_LUT.png`) conține date EXIF `Model` `LATTICE` (riga), nu modelul unei singure camere `LATT-…`.

Un fișier cu benzile unei **singure** camere — un fișier TIFF pentru fiecare bandă (`--per-band`), un cadru `align-apply --per-camera`, o stivă sau un index live al unei singure camere — conține etichetele de identitate proprii ale acelei camere, la fel ca orice alt fișier de captură. Stiva `align-apply`, fișierele pe bandă și pe cameră, precum și fiecare stivă `align-stream` și fișierele sale pe bandă conțin ora de captură a grupului lor (expunerea camerei de referință, UTC), iar un cadru `--per-camera` al unei camere mono este un singur plan, la `--bit-depth`, la fel ca stiva.**Orientarea camerei (răsucire / rotire).** Setarea de orientare a unei camere (opțiunile Mirror H /
Mirror V / rotație) din interfața grafică determină modul în care este salvată o captură:

- **`raw` și `debayered` sunt salvate exact așa cum au fost citite de senzor**, indiferent de setare. Un
  mozaic Bayer rotit sau răsturnat ar reprezenta un model diferit de filtre de culoare, iar hărțile de calibrare
  (offset întunecat, câmp plat) se află pe grila propriu-zisă a senzorului.
- **`preview`/`display`, `radiance`, `reflectance` și randarea `index/` sunt orientate** — rotite
  o singură dată, după toate calculele de calibrare, astfel încât se deschid în modul în care a fost încadrată camera. Fișierele lor
  `clamp_masks/…_clampmask.tif` se află pe aceeași grilă.
- **Fiecare fișier de captură înregistrează setarea** în XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, în sensul acelor de ceasornic, după
  răsturnări) și `ChlorosOrientationApplied` — `false` pentru imaginile brute și cele debayered (pixeli nativi),
  `true` pe produsele orientate. `ChlorosProcessing` enumeră operațiile aplicate pixelilor acelui fișier
  (`none` când sunt nativi). Aceasta include fiecare cadru transmis de `lattice continuous` (
  setarea proprie a camerei, neaplicată) și fiecare cadru din `lattice multi-stream` (identitatea:
  cadrele sale componente sunt native, ca în cazul unei capturi de tip matrice). Stivele `lattice align-stream` / `align-apply`
  înregistrează setările camerei principale, neaplicate: benzile lor sunt deformate în cadrul camerei principale,
  dar nu sunt rotite.
- **`process` aplică orientarea înregistrată** atunci când procesează un fișier brut: mai întâi compensarea întunecimii, câmpul plat
, demosaicarea și separarea în coordonatele senzorului, apoi deformarea de aliniere a matricei, iar
  orientarea la final — astfel încât produsele de previzualizare, radianță, reflectanță și indice ies exact ca
  cele ale capturii live și se înregistrează cu `ChlorosOrientationApplied=true`. Produsele de rezervă necalibrate
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) sunt orientate în același mod, după
  câmpul plat. Produsul debayerat rămâne în formatul nativ.
- **Vizualizatorul de imagini Chloros afișează un fișier brut sau debayerat rotit conform înregistrării sale** (doar afișare —
  fișierul nu este modificat), astfel încât să arate ca produsele sale; miniatura, vizualizarea la dimensiune completă,
  afișajul cursorului și zona de testare Index/LUT utilizează toate aceeași rotire. Un fișier fără înregistrare și
  un produs (deja orientat) sunt afișate așa cum au fost stocate. Un export în zona de testare înregistrează orientarea pe care o
  au pixelii săi (`ChlorosOrientationApplied=true`; identitatea pentru un strat fără înregistrare).
- **Capturile fără înregistrare sunt procesate ca native** (comportament neschimbat). Înainte de versiunea 1.3.0, o
  captură cu o singură cameră, pentru care era setată o răsucire sau o rotire, își rotea singură fișierele raw și cele debayered;
  Chloros nu poate distinge un astfel de fișier raw deja-rotit dintr-un fișier nativ și nu face presupuneri, așa că
  o procesează așa cum a fost stocată. Hărțile de calibrare se află pe grila propriu-zisă a senzorului, așa că pentru o astfel de
  captură folosiți una realizată cu versiunea 1.3.0 atunci când calibrarea acesteia contează.
- **Matricea nu rotește niciun fișier component în mod implicit** (deformarea de aliniere și fișierul XMP `Alignment*` se află în
  cadrele proprii ale camerelor, nerotite) și înregistrează identitatea. `array-capture --orient` rotește produsele fiecărui
  membru, după transformarea de aliniere: în funcție de orientarea **cadru principal** atunci când au fost
  transformate în cadrul acestuia (astfel încât membrii să rămână co-înregistrați), altfel în funcție de orientarea propriilor lor camere. Fiecare
  fișier înregistrează orientarea cadrului în care se află proprii săi pixeli — un fișier brut, al camerei sale —
  iar un fișier brut care conține transformarea de aliniere înregistrează, de asemenea, `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: orientarea imaginii master, pentru un produs aliniat
  în cadrul masterului. `process` rotește fiecare produs cu valoarea care aparține pixelilor săi:
  cea aliniată atunci când deformează fișierul raw (`--array-alignment`, valoarea implicită — produsele capturii aliniate în timp real
  ), cel al camerei atunci când nu o face (`--no-array-alignment` — cele ale capturii live
  nealiniate). Aceeași opțiune există pentru `array-burst`, `project capture` / `burst` /
  `stream`, rețete și SDK.

### Capturi omise în matrice cu filtre mixte

Când combinați camere RGB și multispectrale într-o singură matrice, `array-capture --processing radiance` (sau `reflectance`) salvează cadrele multispectrale și **omite** camerele RGB — radianța pe pixel Bayer nu are relevanță pentru un senzor de bandă largă. CLI-ul afișează explicit fiecare fișier salvat (împreună cu nivelul său de export), fiecare `.daq` scris și fiecare omitere, astfel încât numărul de fișiere nu este surprinzător:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokenurile care indică motivul omiterii urmează modelul `<level>-not-applicable-to-rgb-cam`. O cameră al cărei pachet de calibrare nu este încărcat omite `radiance` / `reflectance` cu `<level>-needs-calibration (connect the camera so its cal pack loads)`, iar în cazul în care nu există o citire DAQ utilizabilă a reflectanței, se omite cu `reflectance-skipped-no-dls`. Reflectanța poate fi, de asemenea, omisă cu `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, și cu `dls-uncalibrated-band-<nm>` atunci când banda se află în mare parte în afara intervalului calibrat radiometric al senzorului de lumină DAQ (~374–974 nm) — dintre codurile de produs disponibile, doar F988, a cărui cale acceptată este fluxul de lucru cu panoul de reflectanță.

Utilizați `--processing debayered` (sau `display`) pentru a include toate camerele, indiferent de tipul de filtru, sau valoarea implicită `all` pentru a obține toate nivelurile aplicabile pentru fiecare cameră dintr-o singură operațiune.

### O citire DAQ la distanță de expunere

Fiecare valoare `.daq` salvată la o captură reprezintă citirea DAQ corespunzătoare acelei expuneri, iar valoarea `.match.json` de lângă aceasta înregistrează cât de departe erau cele două (`gap_ms`) și dacă aceasta se încadrează în toleranță (`in_tolerance`, 500 ms, cu excepția cazului în care setul de capturi `--daq-tolerance-ms`). O valoare care depășește limita este totuși utilizată, astfel încât niciun cadru nu se pierde, iar aceasta este etichetată: fișierul XMP al fișierului de reflectanță conține `DlsInTolerance=False` și `DlsMatchGapS`. `lattice capture`, `multi-capture` și `array-capture` o afișează, de asemenea, sub `.daq` la care se referă, cu eroarea de flux a DAQ-uluiatunci când se explică:

**Modul în care o citire este corelată cu o expunere.** O captură așteaptă prima citire a DAQ-ului după expunere, astfel încât să poată combina citirile de pe ambele părți ale acesteia (`method` `interpolated`). Un DAQ-E trimite o citire la fiecare 0,4 s în lumina zilei și la fiecare 1-2 s în interior, astfel încât timpul de așteptare se adaugă la aproximativ un interval și jumătate la comandă, nu la expunere. Un DAQ care a încetat să mai trimită nu este așteptat. O citire-E reprezintă media luminii pe intervalul de timp scurs de la citirea anterioară, așa că este plasată la mijlocul acelui interval: `gap_ms` este măsurată de la expunere până la acel punct median, iar `bracket_stamps_ns` indică propriile timestamp-uri ale citirilor. `array-capture --continuous` așteaptă fără a încetini captarea: fișierele sale brute sunt scrise imediat, iar `.daq` odată ce citirea ajunge. Cu `--interval`, fiecare trecere așteaptă ca o singură captură.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Când nu a fost scris niciun fișier de reflectanță din acea citire, prima linie se termină în schimb cu `saved anyway; its .match.json says in_tolerance: false`.

Captura se încheie totuși cu `0`: fiecare fișier a fost scris și indică conținutul său. Un DAQ al cărui flux s-a blocat continuă să furnizeze ultima sa citire, așa că verificați-l cu `chloros-cli daq pool-list` înainte de a vă baza pe reflectanța acelei capturi. Backend-ul reconectează un DAQ-E din grup care a încetat să mai transmită (vezi [Un DAQ-E care încetează să mai furnizeze date](#a-daq-e-that-stops-delivering)). O citire în afara toleranței afișează doar linia sa `DAQ:`, cu excepția cazului în care este o citire păstrată (mai jos).

**O citire păstrată după ce DAQ-ul său a dispărut.** Când niciun DAQ nu poate furniza o citire pentru o expunere (DAQ-ul asociat nu este disponibil sau niciun DAQ din grup nu are una, de exemplu după `daq pool-disconnect`), capturarea reutilizează ultima citire pe care o avea Chloros, iar codul său `.match.json` descrie acea citire în raport cu această expunere: `method`, `retained`, `gap_ms` de la această expunere la probele din care a fost obținută citirea, `in_tolerance` din acel interval și `retained_from_epoch_ns`, expunerea cu care citirea a fost inițial asociată. Prin urmare, o captură realizată la mult timp după ce DAQ-ul a fost oprit afișează liniile de mai sus și, sub acestea încă una, care este afișată chiar și atunci când valoarea măsurată se încadrează încă în limitele de toleranță:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

`.daq` indică în continuare DAQ-ul care a efectuat citirea, împreună cu calibrarea și capacul sub care a fost efectuată citirea, astfel încât o reimportare știe care DAQ, calibrare și capac au generat-o.

### Fișiere care nu pot fi salvate

O cameră al cărei fișier nu poate fi salvat — un disc plin, o eroare de permisiuni — nu împiedică salvarea fișierelor celorlalte camere. Fiecare comandă de captură (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; precum și `capture`, `burst` și `stream` ale unei rețete) afișează fiecare fișier care nu a putut fi salvat în stderr, apoi **iese din `1`** chiar dacă restul au fost salvate:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Fiecare fișier aferent unei capturi este acoperit, inclusiv singurul`lattice capture --levels debayered` cu un singur cadru debayerat inclus. O citire `.daq` al cărei fișier sidecar de proveniență `.match.json` nu a putut fi scris este listată ca `[match]` (fișierul `.daq` în sine a fost salvat).

Pe o unitate plină, o captură repetată (`array-capture --continuous` / `--interval`, `project burst` / `stream`, o rețetă `burst` / `stream`) **se oprește la acea trecere** în loc să nu scrie nimic pe aceeași unitate pentru restul execuției. Orice altă eroare este raportată la fiecare trecere în care apare, iar execuția continuă până la sfârșit, apoi iese din `1`. `array-capture --continuous` (captura continuă a backend-ului) nu listează niciun fișier, inclusiv cele eșuate: le numără în liniile de progres și de rezumat (`N failed`, `N file(s) failed to write.`) și iese din `1` — vezi [Modul continuu](#continuous-mode---continuous).

Înregistratoarele — `array-burst` și `array-record` — raportează propriile eșecuri. `array-burst` se oprește automat când unitatea se umple sau chiar înainte de aceasta. Ambele continuă să funcționeze chiar și după orice altă scriere eșuată. Când una dintre ele raportează o eroare, aceasta este afișată după contoare, iar comanda **iese din `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` în locul lui `Disk full:` înseamnă că seria de înregistrări s-a oprit înainte de a umple unitatea. Cadrele înregistrate înainte de oprire sunt păstrate. `array-burst --build` omite compilarea când s-a epuizat spațiul, deoarece videoclipurile ar fi fost salvate pe aceeași unitate. Eliberați mai întâi spațiu, apoi rulați `array-build-video --burst-dir …`. După orice altă eroare, compilarea continuă să ruleze.

Un `array-record` care nu a scris niciun cadru reprezintă, de asemenea, o înregistrare eșuată (`Recorder error: no-frames: the combined view published no frame to record …`, ieșire din `1`) — consultați [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Moduri de captură, înregistratoare și reprocesare offline

Toate acestea funcționează pe un **array persistent** (rulați mai întâi `array-connect`). Ele reflectă panoul de captură din interfața grafică.

### Modurile `array-capture`

`array-capture` este o singură comandă cu patru moduri de declanșare, plus un set de opțiuni de export:

| Mod | Indicator | Comportament |
| --- | --- | --- |
| **Singur** *(implicit)* | (niciunul) | Un singur grup de capturi sincronizate, apoi ieșire. |
| **Continuu** | `--continuous` | Capturi consecutive până când `Ctrl+C`, `--count N` capturează, `--duration S` sau până la umplerea completă a unității. Rulează propria captură continuă a backend-ului (modul Continuu al GUI) — consultați [Modul Continuu](#continuous-mode---continuous) mai jos. |
| **Interval** | `--interval S` | O trecere la fiecare `S` secunde (măsurate de la începutul fiecărei treceri), aceleași limite. |
| **Cel mai rapid** | `--fastest` | Doar date brute+ citirea DAQ atribuită + compozitul cu indice combinat; omite calculele de radianță/reflectanță/afișare, astfel încât cadrul să fie afișat rapid. Implică `--processing raw --force-daq`. Reprocesați ulterior `.daq` salvat în produse calibrate. |

Opțiuni de export (se pot combina cu orice mod; toate utilizează același punct final GUI/SDK):

| Indicator | Efect |
| --- | --- |
| `--array-id ID` | Matricea din care se va captura (se omite când este conectată o singură matrice). |
| `-o, --output DIR` | Dosarul de ieșire (implicit `output`, în dosarul din care executați comanda). |
| `--format {tiff,tif,png,jpg,jpeg}` | Formatul fișierului (implicit `tiff`). PNG / JPEG acceptă doar `display` și randarea indexului; consultați [Niveluri de export pentru captură](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Nivel unic de export (`raw`, `debayered`, `radiance`, `reflectance`, `display`) sau `all` (implicit). |
| `--levels L1,L2,…` | Subset explicit de tipuri de export (de ex. `raw,radiance,reflectance`); **înlocuiește `--processing`**. |
| `--aligned` / `--no-aligned` | **Implicit: aliniat, așa cum este capturarea aceluiași tablou de către aplicația Chloros**(setarea de captură**Aliniat** pentru fiecare tablou, activată în mod implicit, cu excepția cazului în care o dezactivați). Exportul non-raw al fiecărui membru (`debayered`, `display`, `radiance`, `reflectance`, randarea indexului) este transformat în cadrul camerei principale și decupat la suprapunerea comună a membrilor, astfel încât fișierele sunt aliniate la nivel de pixeli și au dimensiunea, decupajul și etichetele `Alignment*` specifice aplicației — mai mici decât senzorul (sau a ROI) cu valorile de decalaj ale membrilor; în cazul unui ROI, transformarea este recalculată pe fereastră. `XMP-mapir:AlignmentJSON` indică `"applied": true` și generează `crop_roi`. Fișierul brut rămâne netransformat la dimensiunea nativă, dar conține transformarea în metadate (`"applied": false`). `--no-aligned` salvează fiecare fișier la dimensiunea nativă, netransformat. `--aligned` este setarea implicită. Indiferent dacă deformația este aplicată sau nu, fiecare fișier membru înregistrează transformarea `Alignment*` a camerei sale odată ce matricea are un profil de aliniere — cu excepția unei camere pe care profilul nu o aliniază (omise sau forțate la identitate din cauza unei rezolvări eșuate) și a camerei de referință atunci când nicio altă cameră nu este aliniată: acestea nu conțin etichete `Alignment*`. În absența unui profil pentru matrice, captura este salvată nealiniată, iar CLI-ul afișează motivul pe o linie `Alignment:`, așa cum procedează în orice alt caz în care deformarea nu a putut fi aplicată (elemente cu rezoluții diferite, o matrice nesincronizată prin cablu, un cadru pe care profilul nu poate fi mapat). Alinierea automată a matricei creează profilul odată ce expunerea se stabilizează; `align-calibrate` creează unul la cerere. |
| `--orient` | Comutați **produsele**fiecărui element — `display`, `radiance`, `reflectance`, randarea `index/` și `composite/` — în funcție de orientarea camerei, după deformarea de aliniere: un produs deformat în cadrul masterului (implicit) în funcție de**orientarea**cadrului principal, astfel încât elementele să rămână co-înregistrate; unul nealiniat (`--no-aligned` sau o captură la care nu s-a putut aplica deformarea) în funcție de orientarea propriei sale camere.**Implicit: dezactivat — fiecare fișier element rămâne în formatul nativ**, așa cum se întâmplă întotdeauna în cazul capturilor de tip matrice. `raw` și `debayered` rămân native în ambele cazuri. Fiecare fișier înregistrează orientarea pe care o au produsele sale (vezi [Orientarea camerei](#cum-arată-un-dosar-de-capturi)); fișierul XMP al `Alignment*` rămâne transformarea cadrului nativ, aplicată înaintea orientării, astfel încât `process` (deformare, apoi orientare) reproduce produsul în timp real. |
| `--index` / `--no-index` | Salvează / omite suprapunerea indicelui de vegetație specific fiecărei camere, acolo unde este configurată una. Implicit: o redă. |
| `--no-daq` | Omite fișierul sidecar `.daq` atunci când niciun nivel selectat nu utilizează DAQ (o captură doar în format brut). În mod implicit, citirea este salvată, astfel încât cadrele pot fi reprocesate offline în reflectanță/indice. `--force-daq` este încă acceptat (implicit). |
| `--daq-tolerance-ms MS` | Toleranța de potrivire DAQ pentru aceste capturi (implicit 500 ms). |
| `--smart` | Așteaptă stabilizarea AE pe toate camerele înainte de declanșare (vezi [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Compresie de pixeli TIFF. `none` (implicit începând cu versiunea 1.3.0; versiunea 1.2.0 scria `deflate`) = necomprimat — pixelii senzorului exact așa cum sunt citiți, ~6,3 MB per cadru la rezoluție maximă și cea mai rapidă viteză susținută; `deflate` = zlib L1 fără pierderi + predictor orizontal, ~4,1 MB per cadru, mai lent. Ambele sunt fără pierderi și se citesc identic la import. |

> **TIFF cu scriere unică + modelul cu rată susținută.**Fiecare fișier este scris**o singură dată**: pixelii, pachetul XMP, IFD0 Make/Model/CameraSerialNumber și etichetele EXIF `ExposureTime` / `SerialNumber` într-o singură trecere, fără ca nimic să le rescrie ulterior. `--continuous` captează grupuri consecutive sincronizate — fiecare captură preia grupul de după cel anterior, inclusiv unul publicat în timp ce acea captură era încă în curs de salvare — astfel încât funcționează la rata matricei. DEFLATE deține GIL-ul Python, astfel încât scrierile comprimate**nu**se paralelizează între firele de scriere per cameră; captura susținută la rezoluție maximă, la rata senzorului, necesită scrieri necomprimate (implicit)**și** un disc de clasă NVMe (~600 MB/s susținut pentru o matrice de 9 camere la ~10 fps). Același parametru este expus ca `compression` pe `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Mod continuu (`--continuous`)

`array-capture --continuous` (fără `--interval` sau `--smart`) predă controlul către **captura continuă proprie a backend-ului** — sesiunea în care rulează modul Continuu al GUI-ului. Backend-ul salvează fiecare grup sincronizat în mod independent, cu două capturi în curs de desfășurare și fără nicio solicitare din partea CLI-ului între grupuri, astfel încât ține pasul cu o matrice rapidă în care o solicitare pe captură rămâne în urmă (măsurată la 73–77 capturi/s pe o matrice de 9 camere la o zonă de interes (ROI) de 512 pixeli). Fiecare comutator de export de mai sus se aplică fiecărei capturi.

- **Limite:** `--count N` se oprește după N capturi (o captură = un grup sincronizat: fișierele fiecărui membru); `--duration S` după S secunde. În absența oricăreia dintre acestea, programul rulează până la `Ctrl+C`, până la umplerea unității de stocare, până la apariția unei erori sau până la oprirea matricei. `Ctrl+C` oprește sesiunea backend-ului, apoi afișează rezumatul.
- **Progres** la fiecare 2 s, cu `, N failed` și `, N groups skipped` adăugate când valoarea este diferită de zero (un grup omis este unul pe care matricea l-a publicat, dar pentru care nu s-a efectuat nicio captură):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Nu se afișează lista pe fișiere.** Rularea se încheie cu o singură linie de rezumat — capturi, rată, fișiere și folderul de ieșire — urmată de `N file(s) failed to write.` / `N group(s) skipped.` atunci când valoarea este diferită de zero:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Se oprește** atunci când unitatea de stocare este plină, afișând `Continuous capture stopped: the drive is full.`, și în cazul unei erori sau al dispariției matricei, afișând `Continuous capture stopped: <reason>.` (ambele pe stderr).
- **Stare de ieșire:** `0` atunci când a salvat capturile și toate fișierele au fost scrise, inclusiv `Ctrl+C`; `1` în cazul oricărui fișier care nu a putut fi scris, un disc plin, o eroare, dispariția matricei sau lipsa completă a capturii.
- **Microsoft Defender:** atunci când Defender utilizează un nucleu de procesor sau mai multe pentru scanarea fișierelor scrise de captură, execuția afișează o singură dată, pe stderr: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Consultați [Setările de sistem](#windows-exe).
- O singură captură continuă pe matrice la un moment dat; o a doua este refuzată cu `a continuous capture is already running for this array`. Prin HTTP, sesiunea este `POST /api/camera/array/continuous/start` (corpul `/api/camera/array/capture` plus `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` și `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, cu sau fără `--continuous`) și**`--continuous --smart`** păstrează bucla pe cerere: o cerere de captură pe trecere — la fiecare moment programat pentru Interval, după stabilizarea AE pentru `--smart` — cu fișierele fiecărei treceri listate ca în modul Single, o linie `Done — N capture pass(es).` la sfârșit și `--count N` care numără trecerile.

### `array-record` — index combinat video/GIF (calitate de monitorizare)

Înregistrează **vizualizarea în timp real a indexului combinat** a matricei într-un fișier `.avi` (și, opțional, într-un fișier `.gif`). Cu vizualizarea combinată a interfeței grafice deschisă, înregistrează acea vizualizare așa cum este afișată. Dacă nu este deschisă nicio fereastră, backend-ul redă vizualizarea în sine: indexul pe care îl transmiteți cu `--expr` sau, în mod implicit, NDVI-ul matricei, odată ce matricea este aliniată. NDVI-ul implicit utilizează benzile roșie și NIR cele mai apropiate de 660 nm și 850 nm, la fel ca vizualizarea în timp real. O vizualizare GUI deschisă în timpul înregistrării este înregistrată atâta timp cât rămâne deschisă. Ea verifică progresul la fiecare 2 s și se oprește la `--duration`, `Ctrl+C`, sau când înregistratorul se oprește automat.

O înregistrare care nu scrie niciun cadru eșuează: înregistratorul raportează `no-frames: …` (matricea poate fi încă în curs de aliniere sau de stabilizare a expunerii) și comanda **iese din `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `--array-id ID` | numai matrice | Matrice țintă (se omite dacă este conectată doar una). |
| `-o, --output DIR` | `output` | Director de ieșire. O cale relativă se află în folderul din care executați comanda. |
| `--fps F` | `10` | Rata de cadre de înregistrare. |
| `--duration S` | până la Ctrl+C | Oprire automată după `S` secunde. |
| `--gif` | dezactivat | Scrie și un GIF animat. |
| `--gif-only` | dezactivat | Salvează doar un GIF (fără `.avi`). |
| `--expr EXPR` | NDVI-ul matricei | Expresie de indexare de înregistrat atunci când nu este deschisă nicio fereastră GUI, în denumirile benzilor matricei (de ex. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; vezi [denumirile benzilor](#index--vegetation-maths)). O denumire scurtă a culorii (`NIR`, `Red`) funcționează numai atunci când o bandă a matricei o conține. În schimb, o fereastră GUI deschisă este înregistrată așa cum este afișată. |

### `array-burst` — serie de cadre raw-Bayer cu frecvență ridicată (calitate de analiză)

Citește direct bufferul grupului sincronizat al buclei de captură — **fără lanț de calibrare, fără exiftool, fără vizualizare live** — astfel încât rulează la rata maximă de captură a camerei. Scrie cadre raw + un manifest per cadru + un `.daq` pentru fiecare citire DLS distinctă sub `<output>/bursts/<base>/`. Reprocesați offline (comanda următoare) sau transmiteți `--build` pentru a efectua procesarea imediat la oprire.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `--array-id ID` | numai matrice | Matrice țintă. |
| `-o, --output DIR` | `output` | Director de ieșire (datele sunt stocate în `<DIR>/bursts/<base>/`). |
| `--duration S` | până la Ctrl+C | Oprire automată după `S` secunde. |
| `--max-frames N` | nelimitat | Oprire automatădupă `N` cadre brute. |
| `--orient` | dezactivat | Produsele generate de o compilare din această serie sunt rotite în funcție de orientarea camerei — produsele specifice fiecărei camere sunt rotite în funcție de orientarea camerei respective, iar indexul combinat în funcție de orientarea camerei principale — așa cum face `array-capture --orient`. Cadrele brute rămân în formatul lor original; `manifest.json` al seriei înregistrează orientarea și opțiunea fiecărui element. |
| `--build` | dezactivat | După oprire, reprocesează imediat seria (la fel ca `array-build-video`). |
| `--products …` | `combined:index` | Cu `--build`: ce videoclip(uri) să se genereze (vezi mai jos). |
| `--fps F` | `10` | Cu `--build`: numărul de cadre pe secundă (fps) al videoclipului de ieșire. |
| `--save-tiffs` | dezactivat | Cu `--build`: salvează și fișiere TIFF calibrate pentru fiecare cadru (fiecare cu propria [mască de limitare unmix](#unmix-clamp-mask-which-pixels-banda-pe-care-s-a-aplicat-limitarea) atunci când un pixel este limitat; rezultatul compilării le listează ca `clamp_masks`). |
| `--gif` | dezactivat | Cu `--build`: scrie și fișiere GIF animate. |

### `array-build-video` — reprocesează offline o serie salvată

Sincronizează temporal fiecare cadru brut cu cea mai apropiată citire `.daq` salvată și îl trece prin **același lanț de radianță / reflectanță / indice ca și pipeline-ul de import**, generând unul sau mai multe videoclipuri.

`--products` este o listă separată prin virgulă de elemente `kind:level`, unde `kind` ∈ `per_cam` | `combined` și `level` ∈ `radiance` | `reflectance` | `index`. Un `level` simplu (fără `kind:`) are ca valoare implicită `per_cam`. Valoarea implicită este `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `--burst-dir DIR` | (obligatoriu) | Calea către folderul de imagini în serie (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level` listă, ca mai sus. |
| `--fps F` | `10` | Numărul de cadre pe secundă (fps) al videoclipului de ieșire. |
| `--save-tiffs` | dezactivat | Salvează, de asemenea, fișiere TIFF calibrate pentru fiecare cadru, alături de videoclip(uri). |
| `--gif` | dezactivat | Salvează, de asemenea, fișiere GIF animate. |

**Un videoclip nu conține nicio mască de limitare** (un singur flux de imagini, 8 biți). În schimb, rezultatul compilării conține `clamp_fractions`: pentru fiecare produs video, proporția de pixeli a fiecărui cadru pe care decombinarea a limitat-o în banda pe care o deține canalul său R / G / B — produsul `Chloros:Band{R,G,B}UnmixClampFraction` dintre radianță și reflectanță al acelui cadru ar fi imprimat (pentru indicele combinat, al fiecărei camere membre). Execuția afișează cel mai slab cadru pentru fiecare produs pe o linie `Clamp:`. `--save-tiffs` adaugă măștile pe pixel pentru fișierele TIFF, în `clamp_masks/<level>/` lângă folderul `<level>/` al fișierelor TIFF.**O compilare întreruptă nu lasă în urmă niciun videoclip incomplet.** Fiecare fișier video este înregistrat ca `<name>.partial.avi` și își primește numele final abia după finalizare, astfel încât un fișier video mai vechi cu acel nume rămâne până când înlocuitorul său este complet. Rulați din nou compilarea pentru a înlocui un fișier `.partial.avi` rămas în urmă.

Procesul afișează fiecare fișier video pe care l-a generat, apoi orice produs de tip `Skipped:` (o cameră/un nivel solicitat — sau indicele combinat — care nu a generat niciun fișier video, împreună cu motivul — de exemplu, lipsa pachetului de calibrare, lipsa citirii DLS pentru reflectanță sau lipsa perechii de benzi roșu/NIR pentru indicele combinat) și orice `Note:`. O cameră RGB pentru care s-a solicitat radianță sau reflectanță primește în schimb videoclipul de afișare, cu balansul de alb ajustat atunci când pachetul de calibrare al camerei este disponibil; o cameră redată fără acesta este menționată într-un `Note:`. Acesta **returnează o valoare diferită de zero** atunci când un produs solicitat a fost omis sau nu a fost generat conform cererii (o expresie de indice configurată nu este evaluată offline); videoclipurile care au fost generate sunt totuși scrise și listate.

> **Alegeți înregistratorul potrivit.** `array-record` este de *nivel de monitorizare* — înregistrează compoziția live redată: vizualizarea combinată a interfeței grafice (GUI) sau, dacă nu este deschisă nicio interfață grafică, vizualizarea NDVI (sau `--expr`). `array-burst` → `array-build-video` este de *nivel de analiză* — salvează datele brute ale senzorului la viteză maximă și reconstruiește ulterior videoclipuri calibrate de radianță/reflectanță/indice, fără a fi necesară o vizualizare în timp real.

### Camere mono (M3M) cu o singură bandă

Gama **M3M**este varianta monocromă a modelului Bayer**M3C**: un filtru de interferență de bandă îngustă per cameră (`M3M-<lens>-F<wavelength>`, de exemplu `M3M-L87-F685`), astfel încât senzorul oferă o**singură bandă în tonuri de gri**, fără mozaic Bayer. Nu este nevoie de demosaic, nu există interferențe între canale care să necesite separare și nu trebuie setat balansul de alb — întregul proces de afișare a culorilor RGB pur și simplu nu se aplică.

Ce înseamnă asta în CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detectează o cameră monocromă și**trec mai departe cu un mesaj de o singură linie** în loc să impună setări fără sens. Acestea funcționează în continuare normal cu o cameră RGB/Bayer M3C în aceeași sesiune.
- **Un format de pixeli Bayer indică adâncimea acestuia.** Un M3M oferă doar formate mono, așa că un format Bayer din `--pixel-format` sau `--preset` setează formatul mono cu aceeași adâncime: `BayerRG8` și `--preset high_speed` dau `Mono8`; `BayerRG12`, `high_quality` și `triggered` dau `Mono12`; un nume compactat rămâne compactat (`BayerRG12p` generează `Mono12p`). Fișierele și `lattice format` denumesc formatul Mono. Un tablou care combină camerele M3C și M3M adoptă un singur format în același mod: Bayer pe fiecare M3C, adâncimea sa Mono pe fiecare M3M. Un nume corespunde adâncimii sale de biți: același lucru este valabil și invers, astfel încât o cameră M3C nu este niciodată plasată într-un format Mono (`--pixel-format Mono12` este `BayerRG12` pe o cameră M3C, indiferent care dintre camere este masterul matricei), iar `lattice format --set` se mapează în același mod și imprimă formatul pe care l-a setat.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** funcționează în continuare — radianța și reflectanța sunt hărți radiometrice *pe bandă* și sunt perfect bine definite pentru o singură bandă. Cadrele monocrome conțin o matrice **de identitate** a răspunsului senzorului (fără demixare 3×3), astfel încât planul trece nemodificat prin calculele de calibrare.
- **O singură cameră monocromă nu poate produce un indice de vegetație.**NDVI/NDRE/etc. necesită cel puțin două benzi (de exemplu, Red + NIR). Pentru a obține un indice de la un dispozitiv monocrom, îndreptați**mai multe** camere M3M către lungimi de undă diferite, aliniați-le într-un singur stack multibandă și calculați indicele *acestuia*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` deschide singur camerele (o [sesiune tranzitorie](#multi-camera-transient-sessions)), așa că nu conectați nimic mai întâi: `--serials`, altfel camerele activate din profil (vezi [Index / Matematică a vegetației](#index--vegetation-maths)). În timp ce Chloros deține orice cameră (interfața grafică, `cam-connect`, `array-connect`) acesta este respins cu codul HTTP 409 `CAMERA_HELD` și se închide `1`; eliberați-le cu `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

Simbolurile `--channel` trebuie să corespundă **exact** cu numele canalelor din presetare (se ține cont de majuscule și minuscule; cele ale NDVI sunt cu litere mici `red`, `nir` — a se vedea `--list-presets`), iar denumirea laterală a benzii corespunde unei benzi din stiva aliniată (modul offline acceptă și indici de bandă începând de la 0, de exemplu `--channel red=0 --channel nir=1`).

Elementul de discriminare în întreaga stivă este tokenul `M3M` din șirul modelului (acesta nu apare niciodată într-un șir `M3C`), afișat în GUI/SDK ca `is_mono`.

---

## Configurarea și reglarea plăcii de rețea a gazdei (matrice LATTICE)

Camerele LATTICE transmit GVSP prin adaptorul Ethernet al gazdei, astfel încât, pentru matricile cu mai multe camere, **driverul**adaptorului și**dimensiunea inelului de recepție** sunt la fel de importante ca și rata de transfer. Setările incorecte se manifestă ca o poartă `FRAMES WILL DROP` / `Reduce ROI to enable` din panoul „Setări matrice” (și în `lattice network-analysis` / `analyze_array_network()` din SDK), chiar și atunci când camerele în sine funcționează corect.

### Adaptoare USB 10GbE — Realtek RTL8157 („Realtek USB 10GbE Family Controller”)

Setările de mai jos sunt proprietățile driverului pentru Windows. Pe Linux, un adaptor USB are și el o limită (400 MB/s pe x86_64, 200 MB/s pe NVIDIA Jetson) (vezi [Modelul fps și burst al matricei](#array-fps--burst-model)), iar bufferele de recepție ale socket-urilor din kernel sunt importante pentru fiecare adaptor — vezi [Gazdele Linux](#linux-hosts--socket-receive-buffers).

| Element | Valoare necesară | De ce este importantă |
| --- | --- | --- |
| **Versiunea driverului**|**≥ v10.67 (ianuarie 2026)**, INF `rtump64x64sta.inf` | Driverul vechi din**2016**(v10.65, `rtump64x64.inf`) gestionează incorect oprirea și verificările de erori cu**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**la oprire/repornire/modul de repaus. Tranziția se blochează (timp de așteptare de ~5 minute), utilizatorul oprește forțat sistemul, iar opririle necorespunzătoare repetate**corup depozitul WMI**(PowerShell/instrumentele încep să dea erori cu `Invalid class`) și**blochează stiva USB** la următoarea pornire (placa de rețea nu se activează; unitățile USB nu mai sunt enumerate). Efectuați o actualizare de pe realtek.com (sau de la furnizorul dongle-ului) înainte de a vă baza pe reporniri corecte. |
| **Bufferele de recepție**— cuvânt cheie `ReceiveBufferLen` |**256**(maximul driverului) | Inelul RX al plăcii de rețea. Valoarea implicită a driverului,**32**, lasă doar ~0,26 MB de inel utilizabil — mult prea puțin pentru o rafală multi-cameră — așa că panoul matricei raportează `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` și blochează conectarea. La**256**inelul este mare (**~13,5 MB măsurat pe gazda de laborator 10GbE**), oferind pipeline-ului RX un spațiu real de manevră pentru rafale GVSP cu mai multe camere. (Dacă o anumită configurație se *conectează* efectiv este decis de două verificări — verificarea **de admisie conștientă de drenaj**și verificarea**supra-subscrierii agregate** — nu o comparație brută între rafală și inel; vezi [Modelul fps și rafale al matricei](#array-fps--burst-model).) |
| **URB-uri de recepție**— cuvânt-cheie `PendingReceives` |**64** (max) | Blocuri de cereri USB în tranzit; măriți-le odată cu bufferele de recepție pentru absorbția rafalelor. |
| **Cadru Jumbo** — cuvânt cheie `*JumboPacket` | **9014** | Necesar pentru pachetele GVSP de 9000 de octeți (de 6 ori mai puține pachete/cadru decât la 1500). |

> ⚠️ **O actualizare a driverului plăcii de rețea RESETEAZĂ aceste proprietăți avansate la valorile implicite.**După actualizarea sau înlocuirea driverului adaptorului,**reaplicați** `ReceiveBufferLen=256` și `PendingReceives=64`, altfel panoul matricei se va bloca din nou, chiar dacă „nu s-a schimbat nimic la nivel hardware”. Aceasta este cauza principală pentru care un sistem care funcționa anterior refuză brusc să se conecteze.

Aplicați dintr-un PowerShell **cu drepturi ridicate** (înlocuiți numele adaptorului dvs., de exemplu `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` acoperă adaptoarele USB 10GbE.** Acum detectează tipul adaptorului și setează corect cuvântul-cheie pentru inelul de recepție: `*ReceiveBuffers`→2048 pentru plăcile de rețea PCIe (Intel I219, etc.) sau `ReceiveBufferLen`→256 + `PendingReceives`→64 pentru controlerul Realtek **USB** 10GbE (care nu expune `*ReceiveBuffers`). Valorile țintă sunt limitate la valoarea maximă raportată de fiecare driver (`NumericParameterMaxValue`), astfel încât nu se scrie niciodată o valoare în afara intervalului. Rulați-l dintr-un terminal **cu drepturi ridicate**; ca orice optimizare bazată pe registru, modificarea intră în vigoare după o repornire a adaptorului sau a sistemului. Comenzile manuale `Set-NetAdapterAdvancedProperty` de mai sus rămân o alternativă bună — ele se aplică în timp real (reconfigurează adaptorul) fără a fi necesară o repornire.

### Hosturi Linux
— tampoane de recepție pentru socketuri

Pe Linux
, inclusiv pe NVIDIA Jetson, tampoanele de recepție pentru socketuri ale kernelului sunt la fel de importante ca inelul de recepție al plăcii de rețea (NIC). La setările implicite ale kernelului (`net.core.rmem_max` / `net.core.rmem_default` = 212.992 octeți), un flux intens de pachete GVSP de la o cameră provoacă depășirea capacității socketului de flux înainte ca Chloros
să le citească. Pachetele sunt pierdute în kernel, în timp ce placa de rețea (NIC) nu raportează nicio pierdere, iar o matrice pierde grupuri de cadre și își reduce rata de cadre. Măsurat pe o matrice de 4 camere conectate la un adaptor USB 10 GbE, mărirea ambelor valori la 32 MB a dus la creșterea ratei de cadre a matricei de la 4,96 la 7,52 fps după conectare, iar a capturii brute continue de la 4,7 la 7,6 capturi/s, fără ca niciun pachet să fie pierdut în kernel.

- **Fișierul .deb le configurează.** Instalarea pachetului scrie valorile `/etc/sysctl.d/60-chloros-gige.conf` (ambele la `33554432`) și le aplică imediat. O valoare deja setată la un nivel mai ridicat rămâne neschimbată.
- **`lattice network` le verifică.** Când oricare dintre ele este mai mică, afișează `socket receive buffers too small for GigE Vision (…)` la secțiunea *Probleme găsite*, iar jurnalul backend-ului conține același avertisment la prima conectare a camerei sau a matricei.
- **Pentru a le mări manual** (până la următoarea repornire; fișierul pachetului le aplică la fiecare pornire):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` le mărește și le menține, dar numai atunci când backend-ul rulează ca root. Un backend care rulează ca utilizatorul dvs. (unul pornit cu `chloros-backend` sau [`chloros-backend@<user>`](#running-the-backend-as-a-service-headless)) nu poate modifica setările kernelului și afișează în schimb linia `sudo sysctl`.

**Reglarea automată a adaptoarelor vizează doar adaptoarele conectate.** La prima conectare a camerei, un backend care rulează ca root reglează adaptoarele cu fir al căror inel de recepție este mic. Acum reglează doar adaptoarele cu o legătură activă, astfel încât un port integrat deconectat își păstrează MTU-ul, iar bufferele de recepție rămân neschimbate. `lattice network` nu mai menționează MTU-ul unui adaptor deconectatca o problemă. `lattice network`, `--fix`, `lattice probe` și reglarea adaptorului iau în considerare doar porturile Ethernet cu fir, nu și magistrala CAN a unui Jetson (`can0`) sau legăturile în modul dispozitiv USB (`usb0`, `usb1`) pe care le oferă unui PC gazdă; un adaptor Ethernet USB pentru camere video este totuși luat în considerare.

### Noțiuni de bază despre rețea (toate legăturile LATTICE)

- **Adresare:** adresă locală de legătură `169.254.0.0/16` (GigE Vision LLA). Gazda utilizează o adresă statică `169.254.x.x/16`; camerele și DAQ-E își atribuie automat adrese în același interval. Nu este necesar DHCP/gateway.
- **Dimensiunea pachetului:**se preferă jumbo (9000), dar lăsați funcția de detectare automată să o găsească — aceasta trece de limita ICMP de 1500 de octeți a camerei prin intermediul unei probe GVSP, astfel încât ajunge la jumbo oriunde cablul o suportă cu adevărat.**Fiecare cameră este măsurată o dată pe sesiune de backend și din nou când legătura sa se modifică (un alt adaptor, o altă viteză de legătură sau o legătură întreruptă, cum se întâmplă la schimbarea unui switch); reconectările reutilizează rezultatul. După o modificare în spatele unui switch care a rămas activ, reporniți backend-ul pentru a măsura din nou.**O primă conectare poate dura ocazional câteva minute la această verificare, iar jurnalul menționează acest lucru atunci când se întâmplă. Utilizați pinul cu `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` numai când știți mai bine decât sonda și preferați setarea pe comandă în locul uneia permanente: un pin ocolește sonda, așa că, dacă traseul nu poate transporta efectiv 9000,**fiecare** captură va expira cu `SC_ERR_TIMEOUT -1011` (vezi [Variabile de mediu](#environment-variables)).
- **Inelul RX se scalează cu `ReceiveBufferLen`:**la valoarea implicită `32`, inelul utilizabil este de ~0,26 MB (prea mic pentru orice rafală multi-cameră); la valoarea maximă `256`, acesta este mare (~13,5 MB măsurat pe gazda de laborator 10GbE), oferind o marjă reală de manevră. Dacă o configurație se conectează sau nu este decis apoi de verificarea de admisibilitate care ține cont de consum**și** de verificarea agregată a suprasubscrierei de mai jos — nu de o comparație brută între rafală și inel.

### Modelul de fps și rafale al matricei

Cum se interpretează panoul de setări al matricei (și `lattice analyze-array` / `analyze_array_network` din SDK
):

- **Burstul este însumat pe cameră, la formatul real de pixeli al fiecărei camere.**Camerele mono**M3M**transmit**Mono12 (2 biți/px)**, sau, cu `--pixel-format`, formatul mono al adâncimii sale (`BayerRG8` este `Mono8`, 1 B/px); camerele**M3C**de tip Bayer transmit date pe 8 sau 12 biți (firmware-ul M3C emite în mod implicit BayerRG12 chiar și atunci când se solicită BayerRG8). Așadar, un cadru la rezoluție maximă cu 4 camere are**~12,6 MB dacă toate sunt de 8 biți, dar ~25 MB cu trei camere mono de 12 biți**. Proiecția determină formatul fiecărei camere pe baza modelului acesteia (cache de identitate), astfel încât fluxul de date corespunde cu ceea ce transportă efectiv cablul — nu o presupunere universală de tip BayerRG8.
- **Un adaptor USB Ethernet are o limită maximă de 200 MB/s, indiferent de specificațiile de pe plăcuța de identificare.** Tabelul de eficiență care transformă rata de legătură într-o valoare susținută este derivat din PCIe; o placă de rețea USB anunță rata sa de legătură *Ethernet*, dar este limitată de magistrala USB și de driverul acesteia. Un dongle USB 10GbE a înregistrat ~1063 MB/s „susținută” — o valoare care nu a fost niciodată verificată — iar ritmul de transfer rezultat a corupt 6–18 % din cadre, în timp ce raporta în continuare o valoare țintă sănătoasă a fps-urilor. Plăcile de rețea conectate prin USB sunt acum limitate la o valoare absolută: **200 MB/s** pe Windows
și pe NVIDIA Jetson, **400 MB/s** pe Linux
x86_64 (pe Linux
, unui adaptor conectat la un port USB 2.0 i se alocă **30 MB/s**, iar specificațiile tehnice recomandă conectarea acestuia la un port USB 3) (limita este dată de magistrala USB și de driverul gazdei, deci nu se adaptează la specificațiile tehnice; un adaptor USB 1 GbE atinge ~80 MB/s și nu este afectat). Pe Linux
, adaptorul este identificat prin sysfs (recurgând la `ethtool -i` și `bus-info`). Fiecare valoare este măsurată: pe Windows
, aceeași configurație cu 4 camere a corupt 6–10 % din cadre la viteze de peste 200 MB/s; pe Linux
x86_64, același adaptor a funcționat fără erori la orice viteză maximă de până la 480 MB/s (273 MB/s livrați, limita propriu-zisă a camerelor). Pe NVIDIA Jetson, același adaptor a funcționat fără erori doar la 200 MB/s și a pierdut cadre intermitent la viteze mai mari. `wire_ceiling_source` din înregistrarea de capacitate menționează acest lucru în mod explicit, iar `nic_is_usb` îl semnalează. Se poate suprascrie în orice mod cu `--wire-ceiling-mbps`.
- **Admitanța ține cont de consumul de curent, nu de raportul între întreaga rafală și inel.** Un burst simultan trebuie să se încadreze doar în *backlog-ul tranzitoriu* = `max(0, Σ per-cam arrival − host drain) × emit_window`, nu în întregul burst. Pe o structură cu gazdă rapidă / cameră lentă (un **PCIe**gazdă de 10G + 4× camere de 1 GbE: sosire ≈ 320 MB/s, descărcare ≈ 1063 MB/s), gazda evacuează mai repede decât se umplu camerele, backlog-ul fiind ≈ 0, astfel încât emisia simulată la rezoluție maximă**este permisă**chiar dacă rafala de 25 MB depășește inelul de 13,5 MB. Dacă se conectează aceleași patru camere la un adaptor**USB** de 10 GbE pe Windows
, viteza de evacuare este de 200 MB/s, nu de 1063 — sosirea o depășește, iar pierderea se manifestă sub formă de cadre corupte, nu ca o rată de cadre mai mică. Pe o gazdă de 1 GbE, pragul DLThr de 31,25 MB/s al camerelor face ca sosirea să depășească debitul → acesta **blochează** corect (pentru *această* clasă de blocare, reduceți ROI sau utilizați binning ≥ 2). Admitanța este una dintre **cele două** porți de conectare — cealaltă este verificarea agregată a suprasubscrierii de mai jos.
- **FPS-ul proiectat este ciclul celei mai lente camere.**În cazul emiterii simultane, fiecare cameră își trimite cadrul în același timp, astfel încât matricea funcționează la ciclul mai lung dintre timpul de citire a senzorului celui mai lent membru al său și timpul de emisie (octeți de cadru ÷ rata sa de emisie:**legătura de acces** a camerei, 1 GbE ≈ 80 MB/s, sau cota sa din limita maximă a conexiunii gazdei, atunci când aceasta este mai mică). Fiecare expunere se suprapune peste citirea cadrului anterior, astfel încât expunerea nu intră în ciclu: expunerea automată umple perioada determinată de rată, apoi crește amplificarea, iar o scenă slab iluminată nu mai reduce rata (în versiunea 1.2.0 se întâmpla acest lucru). Doar o expunere manuală mai lungă decât perioada încetinește declanșarea (vezi [Expunerea pe un element al matricei](#exposure-on-an-array-member)). Adăugarea de camere nu încetinește matricea până când lățimea de bandă a acestora nu se reduce. În emisia serială (`slip-emit-and-capture`), timpii de emisie ai camerelor se cumulează, iar expunerea fiecărui element este luată în calcul la stabilirea intervalului de eșalonare. Măsurat la senzor complet, 12 biți, pe o matrice de 9 camere cu un gazdă PCIe 10 GbE: 10,7 fps sau 14,0 fps cu [transport comprimat](#packed-12-bit-transport---packed).
- **După o captură continuă, proiecția indică rata la care s-a stabilit.** O captură de 30 de secunde sau mai mult la o rată automată înregistrează rata la care s-a stabilit pentru acea dimensiune a matricei pe acest computer; încercările scurte ale controlerului de rată la o rată mai rapidă sunt omise. De atunci încolo, linia `array-connect` și panoul „Setări matrice” indică acea rată atunci când aceasta este inferioară celei a modelului, precizând ce a limitat-o: `clock-bound` (matricea a funcționat la rata la care s-a stabilit, bucla de captură așteptând ceasul de declanșare), `loop-bound` (bucla de captură nu mai avea timp disponibil), `capture-bound` (captura a rămas în urmă și a omis grupuri) sau `measured` (nu se specifică care dintre acestea). Valoarea modelului se află în `wire_fps_max`. Matricea atinge totuși limita maximă pe care o suportă rețeaua.
- **O matrice funcționează pe orice rețea — rata de cadre este cea care dictează.**Fiecare cameră primește o parte egală din**limita maximă a cablului fără coliziuni**(`sustained × sim_emit_factor`), sincronizată cu GevSCPD, iar toate expun în continuare la același declanșator. Pe o conexiune lentă, cota este mică, iar rata de cadre scade odată cu aceasta: 8 camere de 12 biți la rezoluție maximă pe o rețea de 1 GbE fără cadre jumbo primesc 52 / 8 = 6,5 MB/s fiecare, aproximativ**1 fps**(aproximativ 1,3 fps comprimate), în timp ce un host de 10G le rulează la 11-14. Dacă camerele nu pot fi sincronizate atât de lent — conectați-vă și citiți intervalul GevSCPD al fiecărei camere pentru a afla — matricea comută la emisie**serială**(`slip-emit-and-capture`): camerele se succed pe cablu la viteza maximă a legăturii, declanșate una după alta, astfel încât fiecare cadru ajunge în continuare, dar elementele**nu sunt expuse simultan**. `analyze-array` și numele de ieșire al comenzii „connect” indică modul; setați `--force-tier sim-capture-sim-emit` pentru a refuza trecerea la modul de rezervă. Dimensiunea cadrului nu influențează această partajare, astfel încât**binning-ul și o zonă de interes (ROI) mai mică cresc rata cadrelor, dar nu modifică capacitatea matricei de a se încadra**. Doar o linie care nu poate transporta matricea nici măcar în mod serial este**supraîncărcată**— de exemplu, mult mai multe camere decât poate gestiona o linie sau multe alte camere care transmit pe aceasta — iar în acest caz simptomul este pierderea de pachete GVSP, nu o rată mai mică: `analyze-array` resetează la zero valorile fps realizabile și afișează `** OVER-SUBSCRIBED**`, conexiunea se realizează cu un avertisment sonor, iar o conexiune HTTP
cu `pin_resolution: true` (mod strict) **refuză**. Soluțiile sunt: un număr mai mic de camere, cadre jumbo de la un capăt la altul sau o placă de rețea mai rapidă. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` reduce refuzul la un avertisment sonor pentru testele de banc — vezi [Variabile de mediu](#environment-variables).

### Starea matricei — care subsistem pierde cadre

`GET /api/camera/array/<array_id>/capability` al unei matrice conectate conține un bloc activ
`health`, reevaluat într-o fereastră rulantă de **10 secunde**. Acesta împarte pierderea de cadre
în cele două cauze care necesită remedieri opuse, în loc să raporteze o singură rată „incompletă”
care nu specifică niciuna dintre ele:

| Câmp | Ce înseamnă | Ce subsistem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pe serial) | Cadrul **a sosit și era defect din punct de vedere structural**— pierdere de pachete GVSP. |**Rețea**: lățime de bandă, ritm, inelul de recepție al plăcii de rețea, MTU |
| `never_arrived_rate_pct` (pe serial) | Cadrul **nu a sosit deloc**— camera nu s-a declanșat sau nu a transmis nimic. |**Declanșare / sincronizare**: cablu M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Cea mai slabă rată a camerei pentru fiecare. | — |
| `per_cam_rate_pct` | Rata combinată de cadre incomplete pe cameră (ambele cauze împreună). | — |
| `stable_for_seconds` | Cât timp a rămas fiecare cameră sub 0,01 %. | — |

Peste 5 %, backend-ul înregistrează o linie `[array-health <id>] WARN` care specifică diviziunea — la
prima încălcare, la o schimbare a benzii de gravitate, o dată pe minut cât timp persistă și o dată când
se remediază. Jumătatea coruptă scrie `[gvsp-corrupt <SN>]` în fișierul jurnal al backend-ului la
prima depășire per cameră și motiv, apoi o sinteză la fiecare 60 de secunde; panoul de jurnal al aplicației afișează în schimb linia
linia `[array-health <id>]`. Fiecare evaluare ajunge totuși în fișierul de jurnal din backend;
contoarele se actualizează la fiecare buffer, indiferent de ce se afișează.

Aceeași înregistrare raportează numărul de care depinde întreaga alocare:

| Câmp | Ce înseamnă |
| --- | --- |
| `wire_ceiling_mbps` | Bugetul de transfer susținut al gazdei, în vigoare, MB/s. |
| `wire_ceiling_source` | De unde provine acea valoare, în cuvinte — de exemplu, `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (pe Linux
x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) sau `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` atunci când `--wire-ceiling-mbps` (sau câmpul **Buget de transfer** din interfața grafică) îl setează. |
| `nic_is_usb` | `true` pentru un adaptor Ethernet USB — consultați limita USB de mai sus. |

**Interpretare:** o valoare diferită de zero pentru `gvsp_corrupt_rate_pct`, cu `never_arrived_rate_pct` la 0
înseamnă că declanșarea și sincronizarea cablului sunt perfecte, iar 100 % din pierderi se află pe calea de rețea
— reduceți valoarea `--wire-ceiling-mbps` și reconectați-vă. Modelul invers indică, în schimb,
cablul de sincronizare sau linia de declanșare.

> **`--target-fps` nu este factorul determinant pentru cadrele corupte.** Ritmul GevSCPD este stabilit
> o singură dată la conectare, astfel încât reducerea ratei de declanșare modifică ciclul de lucru și nu
> rata de transmisie simultană a rafalelor. O reducere măsurată de 5× a cererii nu a produs nicio îmbunătățire;
> reducerea limitei maxime a cablului de la 240 la 200 MB/s a dus aceeași platformă de la 10,4 %
> cadre corupte la 0,00 %.

> **O matrice în funcțiune nu se poate replanifica singură pe firmware-ul LATTICE (M3C / M3M).** Acesta
> nu poate remedia singur această problemă; deconectați și reconectați, astfel încât selectorul de timp de conectare să poată
> replanifica cu noua limită maximă.

### Simptom → remediere

| Simptom (Setări matrice / conectare / `analyze_array_network`) | Cauză | Remediere |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` se resetează la 32 (de obicei după o actualizare a driverului) | Setați `ReceiveBufferLen`→256, `PendingReceives`→64; redeschideți panoul (reporniti backend-ul dacă a stocat în cache vechea dimensiune a inelului) |
| Blocări la repornire/oprire; ulterior erori WMI `Invalid class`, placa de rețea nu se activează, unitățile USB lipsesc | Driver Realtek USB 10GbE vechi din 2016 → BSOD `0x9F` → opriri forțate | Actualizați driverul adaptorului la versiunea ≥ v10.67 (2026), apoi reaplicați setările inelului de recepție de mai sus |
| Matricea se conectează la capacitatea maximă a senzorului, dar cu o rată de cadre scăzută | Modul „Auto” menține capacitatea maximă a senzorului și permite ca rata să scadă până la capacitatea cablului | Alegeți una dintre opțiunile mai rapide din lista de ieșiri de conectare (`--binning 2` / `4`, sau un `--width`/`--height`), activați cadrele jumbo sau actualizați legătura |
| Array raportează o rată țintă de cadre pe secundă (fps) corectă, dar livrează doar o fracțiune din aceasta; `health.gvsp_corrupt_rate_pct` diferit de zero, `never_arrived_rate_pct` 0 | Bugetul de bandă dedus al gazdei supraestimează ceea ce susține de fapt (tipic pentru un adaptor USB Ethernet, o bandă PCIe subțire sau o structură partajată) | Reconectați-vă cu o valoare mai mică pentru `--wire-ceiling-mbps` și verificați din nou blocul de stare. **Nu** `--target-fps` — Ritmul GevSCPD este fixat la conectare |
| Linux
: o matrice pierde grupuri de cadre și își reduce rata de cadre, în timp ce placa de rețea nu raportează pierderi; `lattice network` listează `socket receive buffers too small` | Bufferele de recepție ale socket-urilor din kernel sunt la valorile implicite, astfel încât pachetele de la cameră sunt pierdute în kernel | Măriți `net.core.rmem_max` / `rmem_default` la 32 MB — consultați [hosturile Linux
](#linux-hosts--socket-receive-buffers). Fișierul .deb face acest lucru la instalare |
| Camerele lipsesc din grupurile publicate; `health.never_arrived_rate_pct` diferit de zero, `gvsp_corrupt_rate_pct` 0 | Declanșare / cale de sincronizare — camerele nu se activează, nu este o problemă de rețea | Verificați cablul de sincronizare M8 și `--line`; confirmați că fiecare membru este activat (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` depășit în `analyze-array` sau refuz de conectare în modul strict (`pin_resolution`; `array over-subscribes the wire`) | Cablul nu poate suporta matricea la niciun ritm atins de camere, simultan sau pe rând — sunt mult mai multe camere decât poate suporta cablul, sau alte camere care transmit pe acesta | Mai puține camere, cadre jumbo de la un capăt la altul sau o placă de rețea mai rapidă. **ROI/binning NU vor ajuta** (partajarea este independentă de dimensiunea cadrului). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` se aplică prioritar în testele de laborator (acceptă pierderea de pachete) |
| Matricea se conectează în `slip-emit-and-capture` (serial) pe o rețea lentă | Camerele nu pot fi sincronizate suficient de lent pentru a partaja cablul simultan, așa că funcționează pe rând; cadrele nu sunt expuse în același moment | Cadrele jumbo sau o placă de rețea mai rapidă restabilesc captarea simultană; `--force-tier sim-capture-sim-emit` insistă asupra acestui lucru (și pierde pachete în schimb) |

---

## `chloros-cli daq`

Comenzi pentru senzorul spectral. Comenzile `pool-*` sunt clienți HTTP
„subțiri” care controlează senzorul prin intermediul pool-ului persistent al backend-ului, așa că au nevoie de un backend în funcțiune (vezi [Pornirea backend-ului](#start-the-backend)). Backend-ul deține controlul asupra transportului, astfel încât scripturile CLI
și SDK
împart un singur identificator activ, în loc să se lupte pentru portul serial. `pool-discover` este singura modalitate de a afla adresa MAC BLE a unui DAQ-M.

> **`chloros-cli daq --help`** (sau `-h`, sau `daq help`) listează subcomenzile `pool-*` (pe Linux
, împreună cu comenzile directe de hardware incluse în compilare), iar `chloros-cli daq <subcommand> --help` (sau `daq help <subcommand>`) prezintă opțiunile fiecăreia. Acestea acoperă conectarea, stream, înregistrarea fișierelor `.daq` calibrate, schimbarea profilurilor de capacitate și conectarea unui DAQ la o cameră, iar pentru un DAQ-E — setările de rețea, PTP, repornirea și actualizările de firmware. DAQ-ul poate fi, de asemenea, controlat din Python
cu `chloros_sdk.connect_daq_sensor()`, care utilizează aceeași cale comună.
>
> **Utilizați comenzile `pool-*` pe toate platformele.** Acestea se comportă la fel pe Windows
, Linux
și Jetson. Pe Windows
, `daq discover` și `daq list` sunt aliasuri ale `pool-discover`, iar orice altă comandă `daq` care nu este `pool-*`, se termină cu `1` și `DAQ direct-hardware commands require the full daq package`. Pe Linux
și Jetson, `daq discover`, `daq list` și celelalte comenzi `pool-*`, care nu sunt `daq`, sunt comenzi hardware directe-hardware: ele deschid senzorul în procesul propriu al CLI
, în afara grupului de resurse al backend-ului (deci nu pe un senzor deținut de backend, de GUI sau de un script), iar acolo se acceptă `daq discover`, `--only` și `--timeout`, dar nu și `--json`. Această referință documentează numai comenzile `pool-*`.

### Fluxul de lucru pentru prima conectare a senzorului DAQ

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referință `pool-*`

| Subcomandă | Scop |
| --- | --- |
| `daq pool-connect` (smart-detect) | Deschide un senzor din grupul de fundal. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U pe un port serial specific. |
| `daq pool-connect --ble` | DAQ-M prin BLE, scanare automată a adresei MAC. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M prin BLE la o adresă MAC cunoscută (implică `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E prin Ethernet la un nume de gazdă sau o adresă IP cunoscută (implică `--eth`). |
| `daq pool-connect --eth` | DAQ-E prin Ethernet, gazdă detectată automat (mDNS + ARP de rezervă; funcționează cu un cache ARP gol pe Windows
și Linux
). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Reglare fereastră de integrare / stare AE. |
| `daq pool-connect --no-stream` | Conectare, dar fără a începe încă transmisia (reia cu `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil de corecție a plafonului. Valoarea implicită la nivelul backend-ului este `sunshine_cosine`; `none` este un senzor fără capac. Un [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) îl ignoră (difuzor fix). **Senzorii sunt livrați fără capac, așa că denumiți capacul pe care l-ați montat:** valoarea implicită este înregistrată ca fiind (`cap_id_source=auto_default`), iar procesarea afișează o avertizare în acest sens, deoarece un DAQ-U fără capac, în condiții de corecție pentru lumina soarelui, înregistrează valori de aproximativ 30 de ori mai mari. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Scanează fiecare transport pentru a identifica senzorii la care te-ai putea conecta, fără a te conecta efectiv. **Așa găsiți adresa MAC BLE a unui DAQ-M.** `--timeout` se aplică pentru fiecare transport (implicit 5 s, 1-20). Senzorii deja deschisi în pool nu sunt afișați — un DAQ-M conectat încetează să mai transmită — așa că folosiți `pool-list` pentru aceștia. `usb` listează numai dispozitivele seriale USB: porturile seriale proprii ale unui computer (`COM1`, `/dev/ttyS*` pentru Linux
) nu pot fi senzori și sunt omise. |
| `daq pool-list` | Afișează fiecare senzor din grupul din fundal. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Leagă o cameră sau o matrice de un DAQ, așa cum face selecția senzorului de lumină din interfața grafică: capturile sale utilizează, corelează și salvează valorile citite de acel DAQ. `SENSOR` este un ID de senzor din `pool-list` sau adresa la care se accesează DAQ-ul. `--clear` (nu există `SENSOR`) anulează asocierea, iar capturile revin la cel mai recent DAQ. Legarea durează până la repornirea backend-ului sau până când camera, matricea sau DAQ-ul respectiv este deconectat: o deconectare (aici, din SDK
-ul sau din aplicație) anulează toate legăturile hardware-ului pe care îl deconectează, indiferent cine le-a stabilit, iar legăturile membrilor unei matrice se anulează odată cu cea a matricei. Camera trebuie să fie conectată, în proiectul deschis, sau să fi fost deschisă anterior pe acest computer, iar matricea să fie conectată sau să se afle în proiectul deschis; orice alt număr de serie sau ID este respins pe baza numelui și nu se realizează nicio legătură. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Eliberare — indiferent cine a deschis senzorul, astfel încât eliberează și un senzor lăsat în pool de un script care s-a blocat (orice înregistrare pe acesta este finalizată mai întâi). Un senzor care nu se află în pool nu are nimic de eliberat: afișează `ID: already released (not in the pool)` și iese din `0`; o eliberare refuzată sau neconfirmată de backend iese din `1`. SDK
: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Cele mai recente N cadre din spectrul N (implicit 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Reluare / pauză streaming. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Pornire / oprire a unei înregistrări .daq. `-o` are ca valoare implicită `~/Documents/DAQ Live View` pe computerul backend-ului (o cale relativă se află în folderul din care executați comanda); `--duration` (`-d`, implicit `0` = până la `--stop`). `--confirm-after S` avertizează când nu a sosit nicio citire la S secunde după începere (implicit 20; 0 dezactivează această funcție); cu `--duration`, `--heartbeat S` afișează numărul de citiri la fiecare S secunde și avertizează când acesta încetează să crească (implicit 30; 0 dezactivează această funcție). O înregistrare pe un DAQ-E care se întrerupe continuă în același fișier odată ce backend-ul s-a reconectat, iar `--stop` listează intervalele de întrerupere ([Un DAQ-E care încetează să mai transmită date](#a-daq-e-that-stops-delivering)). Ctrl+C oprește prematur o înregistrare `--duration` și raportează acest lucru; fără `--duration`, înregistrarea continuă să ruleze după Ctrl+C (ieșire din `130`) până la `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Schimbă profilul de corecție a capacității în timpul rulării. `--cap-id` este necesar; `--cap-id none` elimină corecția capacității (un senzor fără capac). Un DAQ-E-S păstrează `as_recorded` indiferent de setare. |
| `daq pool-enable-ptp --sensor-id ID` | Reactivează PTP pentru un DAQ-E al cărui PTP este dezactivat. Setarea este salvată pe unitate, astfel încât un dispozitiv lăsat oprit rămâne oprit — marcajele sale de timp nu sunt sincronizate prin PTP — până când se execută această comandă. `pool-list` semnalează o astfel de unitate (`!! PTP is switched OFF`) și afișează această comandă pentru ea. Chloros
nu reactivează niciodată o unitate de la sine, deoarece dezactivarea poate fi intenționată. |
| `daq pool-info --sensor-id ID [--json]` | Setările proprii ale unui DAQ-E: firmware-ul său (și dacă această versiune Chloros
conține una mai nouă), adresa sa și modul în care a obținut-o (DHCP / link-local / static), setarea de rețea salvată până când senzorul o execută (una salvată cu `--no-reboot` inclusă), fluxul la pornire și PTP. În cazul unui senzor al cărui flux nu ajunge la acest computer, acesta citește senzorul pe o conexiune nouă și semnalează acest lucru. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Configurarea unui DAQ-E pentru DHCP (`--auto`) sau o adresă statică (`--mask` implicit `255.255.255.0`). Senzorul se repornește pentru a aplica setarea și se reconectează cu setările pe care le avea, așteptând până la `--timeout` secunde (implicit 45) pentru aceasta; `--no-reboot` doar o salvează. Consultați [Setările dispozitivului DAQ-E](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Dacă un DAQ-E începe transmisia la pornire fără software conectat (salvat pe senzor). Chloros
pornește transmisia în orice caz. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Repornirea unui DAQ-E și reconectarea acestuia cu setările anterioare (așteaptă până la `--timeout`, implicit 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Instalează firmware-ul DAQ-E inclus în această versiune Chloros
pe un senzor care rulează o versiune mai veche, apoi reconectează-l (aproximativ un minut; nu-l opriți; `--timeout` implicit 45 s până se repornește). Un senzor care are deja versiunea actuală nu este modificat. |

### Un DAQ-E care nu mai transmite

Un DAQ-E din grup care nu transmite niciun spectru timp de 20 de secunde este reconectat de către backend. Aceasta se realizează printr-o repornire a comutatorului care alimentează senzorul: senzorul repornește odată cu acesta și nu transmite din nou până când nu i se cere acest lucru (cu excepția cazului în care se aplică `pool-set-auto-stream --on`).

- **Mai întâi verifică dacă este vorba de același senzor.** Solicită, la adresa pe care o avea senzorul, fără a modifica nimic, numele senzorului, adresa MAC și numărul de serie. De asemenea, interoghează sub numele `daq-e-<id>.local` al senzorului, dar acceptă acel răspuns doar dacă provine de la propriul adaptor de rețea (MAC) al senzorului. Niciun alt dispozitiv care răspunde nu este reconectat și nu i se scriu date.
- **Se reconectează cu setările sesiunii:** timpul de integrare, medie de cadre, expunere automată și profilul cap, așa cum erau, nu cu valorile implicite la pornire ale senzorului. Un senzor care avea PTP activat și s-a întors cu acesta dezactivat este reactivat. Apoi repornește fluxul și receptorul computerului pentru acesta.
- **Reîncearcă la intervale din ce în ce mai mari:** imediat, apoi după 5, 10, 20 și 40 de secunde, apoi la fiecare 60 de secunde cât timp senzorul este indisponibil. Fiecare încercare se limitează la a întreba senzorul care este acesta. Un senzor care răspunde, dar nu trimite nimic nici după trei încercări de reconectare, este lăsat în pace și raportat (`auto-reconnect GAVE UP`). `pool-stream --start` încearcă din nou.
- **`pool-list` indică starea actuală.** În timp ce senzorul nu transmite nimic, linia sa `STREAM FAULT` arată ce face procesul de reconectare. Linia se șterge odată cu primul spectru, iar senzorul afișează apoi `reconnected automatically N time(s)`.
- **Înregistrarea continuă în același `.daq`.** Fișierul marchează fiecare întrerupere cu două rânduri: `event_type` 5 la ultimul spectru înainte de întrerupere și 6 la primul după aceasta, fiecare conținând doar o marcă temporală. `pool-record --duration` afișează `!! STREAM LOST …` cât timp senzorul este absent și `stream resumed after …` când acesta revine. `pool-record --stop` listează fiecare interval și avertizează, așa cum face și în cazul un fișier gol (o înregistrare care nu conține nicio citire iese din `1`).
- **O înregistrare păstrează o singură calibrare.** Dacă senzorul revine cu o calibrare diferită în timp ce o înregistrare este în curs de desfășurare, restul înregistrării păstrează calibrarea cu care a început, iar avertismentul de oprire menționează acest lucru. `pool-disconnect` + `pool-connect` preia noua calibrare.
- **Comenzile tale au prioritate.** Un senzor pe care îl oprești cu `pool-disconnect` sau pe care îl pui în pauză cu `pool-stream --stop` nu este niciodată reconectat, chiar și atunci când o reconectare este deja în curs.
- **`pool-info` pe un senzor inactiv** citește senzorul la o conexiune nouă și adaugă `!! stream NOT reaching this computer`. Un senzor care nu răspunde este raportat imediat. `pool-list` nu mai afișează starea PTP pentru un senzor care a încetat să răspundă.
- DAQ-U și DAQ-M nu sunt acoperite. Un DAQ-M se reconectează singur prin Bluetooth, iar un DAQ-U deconectat de la USB are nevoie din nou de `pool-connect`.

### DAQ-E-S — difuzor fix și senzor de înclinare

**DAQ-E-S** este un DAQ-E cu două diferențe relevante pentru software. Tot restul — detectarea, conectarea, PTP, înregistrarea, setările dispozitivului de mai jos — funcționează exact ca în cazul unui DAQ-E.**Difuzorul său este permanent, deci nu necesită nicio corecție a capacului.** Difuzorul de tip Sunshine este încorporat și se afla pe unitate atunci când a fost măsurată calibrarea din fabrică, astfel încât calibrarea indică deja iradianța descendentă reală. Chloros
-ul își setează capacul la `as_recorded` (fără profil) indiferent de cerință: un cap numit astfel (`sunshine_cosine`, `fov_*`, `none`) este ignorat cu un mesaj, iar înregistrările sale stochează `cap_id_source=model`, astfel încât avertismentul privind limita presupusă nu se declanșează niciodată pentru acesta. Nu aplicați manual o limită datelor sale: o limită de tip `sunshine_cosine` aplicată ar indica o valoare de aproximativ 11 ori mai mare.

**Este echipat cu un accelerometru pe 3 axe.** Răspunsul cosinus al difuzorului de lumină solară este valabil doar cât timp acesta este orizontal, astfel încât un senzor înclinat raportează o iradianță eronată — iar reflectanța moștenește eroarea — fără alte simptome. Fiecare citire DAQ-E-S conține, prin urmare, orientarea sa:

| Câmp | Semnificație |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Unghiuri în grade. `None` atunci când firmware-ul marchează unghiul ca fiind nedefinit (cădere liberă, o axă inactivă). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Accelerație în mili-g. O valoare `mag_mg` departe de ~1000 înseamnă că unitatea era în mișcare și că unghiurile nu sunt relevante pentru acea probă. |
| `cal_applied` | Setat: axele și unghiurile sunt în sistemul de referință al difuzorului (se aplică calibrarea de montare a unității). Șters: sistemul de referință brut al plăcii, care poate indica o înclinare de până la ~2,3°, care nu este reală. |
| `sample_age_ms` | Vechimea eșantionului accelerometrului la momentul trimiterii spectrului. |
| `temp_raw` | **Tendința** necalibrată a temperaturii cipului — doar diferențe, niciodată o citire a termometrului. |

`daq pool-latest --json` le returnează sub cheia `imu` a fiecărui cadru (`null` pe un DAQ-U, DAQ-M sau DAQ-E simplu). Un `.daq` înregistrat de un DAQ-E-S stochează aceleași valori în coloanele sale `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) lângă fiecare spectru, astfel încât cititorul să poată distinge un nor de un trepied răsturnat. La toate celelalte modele, aceste coloane sunt goale.

Fluxul de atitudine este raportat separat: `daq pool-list` afișează `!! ATTITUDE STREAM FAULT (spectra unaffected)` în timp ce este oprit și câte erori a recuperat ulterior. O eroare acolo nu marchează niciodată o lacună în înregistrarea spectrală. `daq pool-discover` listează unitatea ca `DAQ-E-S` (ca `DAQ-E` dacă nu răspunde la verificarea identității în aproximativ 2 s).

### Setări dispozitiv DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` fără `--gateway` nu salvează nicio poartă de acces** (orice poartă de acces setată anterior este eliminată), ceea ce este adecvat pentru o rețea de camere fără router.
- **O adresă statică trebuie să se afle în rețeaua acestui computer.** Chloros
respinge o adresă a cărei subrețea nu include adresa acestui computer în rețeaua senzorului: după repornire, nimic nu ar putea ajunge la senzor pentru a restabili setarea inițială. Treceți `--force` numai atunci când acest computer va fi redirecționat sau rutate către noua subrețea.
- **Repornirea este refuzată în timp ce senzorul înregistrează.** Opriți mai întâi înregistrarea (`pool-record --stop`). O înregistrare începută chiar înainte de repornire este închisă de aceasta, iar `pool-set-network`, `pool-reboot` și `pool-update-firmware` se încheie cu o linie `Warning:` care menționează numele fișierului de înregistrare (chiar și atunci când senzorul nu revine).
- **După o repornire, Chloros
reconectează doar senzorul în sine.** Așteaptă acolo unde se afla senzorul — și, după `--auto`, și sub numele său `daq-e-<id>.local` — un răspuns cu numele senzorului, numele de gazdă și modelul, de la adaptorul de rețea (MAC) pe care îl avea, și îl reconectează la adresa care a răspuns. Un răspuns sub acest nume este acceptat numai cu acel MAC, astfel încât de la un computer care accesează senzorul printr-un router sau din WSL2 în rețeaua NAT implicită, un senzor pe care DHCP l-a mutat la o nouă adresă nu este reconectat: comanda indică acest lucru și afișează comanda `pool-connect --eth-host <its address> …` care îl readuce cu setările de expunere și limită pe care le avea.
- **`pool-update-firmware` refuză un senzor a cărui versiune de firmware nu o poate citi.** Un senzor care durează mai mult decât timpul de așteptare pentru a se reconecta după declanșarea blițului nu a suferit o defecțiune: odată ce răspunde, este reconectat cu setările noului firmware completate, iar comanda avertizează că a fost lent.
- **O actualizare de firmware necesită o conexiune TCP de intrare.** Senzorul se reconectează la acest computer pentru a prelua imaginea. Chloros
ascultă pe adaptorul care redirecționează către senzor (pe fiecare adaptor, doar atunci când nu poate identifica care anume) și deservește doar adresa propriu-zisă a senzorului; orice altă conexiune este închisă fără a fi deservită. Pe Windows
, regula de firewall `Chloros Backend (TCP-In)` a programului de instalare permite accesul (vezi [Windows
(.exe)](#windows-exe)); un firewall care o blochează eșuează actualizarea cu codul de eroare `… device never connected back on TCP <port> …`.
- **În WSL2, utilizați rețeaua oglindită** (`networkingMode=mirrored` în `%UserProfile%\.wslconfig`). În modul NAT implicit, adresa acestui computer în rețeaua senzorului este cea a WSL, astfel încât orice adresă statică este refuzată (cu excepția `--force`), fluxul live nu ajunge la WSL, iar actualizarea firmware-ului nu se poate finaliza, deoarece senzorul se reconectează la computer pentru a primi imaginea.
- **ID-ul senzorului și numele de gazdă nu se modifică.** Indiferent de adresa sa, un DAQ-E rămâne `daq-e-<id>` și `daq-e-<id>.local`, astfel încât `--eth-host daq-e-<id>.local` îl găsește în continuare.
- **Interfața grafică (GUI) are aceleași setări**: fila „Senzori de lumină”, setările senzorului (rotița de setări), „Rețea”. Interfața grafică menține propria conexiune cu un senzor, așa că, dacă modificați un senzor pe care interfața grafică îl are deschis din interfața grafică: o repornire din CLI
încheie sesiunea GUI cu acesta.
- Echivalentele din SDK
sunt `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` și `update_firmware()`.

---

## `chloros-cli project`

Deschideți, conectați-vă la și controlați un proiect Chloros
salvat (un dosar conținând `cameras.json` + `sensors.json` + `project.json`). Totul se realizează prin intermediul backend-ului, astfel încât interfața grafică (GUI) și interfața de linie de comandă (CLI
) generează o stare identică a hardware-ului.

### Referință subcomenzi

| Subcomandă | Scop |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Afișează lista dispozitivelor din proiect (camere, matrice, senzori), citită din fișierele acestuia, și verifică fișierele sursă ale acestuia (vezi [Un proiect mutat](#a-moved-project---relink)). Nu deschide proiectul în backend: proiectul deschis acolo, precum și dispozitivele și înregistrările sale, rămân așa cum sunt. Iese din `1` atunci când nu se găsește niciunul dintre fișierele sursă ale proiectului. |
| `project devices PATH [--reconnect]` | Afișează lista sau reexecută procesul de descoperire. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Conectează fiecare cameră / matrice / senzor salvat. Fiecare matrice se conectează la fel ca în cazul comenzii `lattice array-connect`: până la 600 s, pașii săi fiind afișați pe stderr (`[array <id>]    <s> s: <step>`; stdout este raportul JSON
), iar o matrice neconectată în acest interval este anulată — camerele sale fiind readuse la starea inițială — și raportată sub `errors`, fără a rămâne niciodată conectată. `--packed` setează transportul comprimat pe 12 biți pentru fiecare matrice (implicit `auto`; un proiect nu salvează setări individuale pentru fiecare matrice; vezi [Transport comprimat pe 12 biți](#packed-12-bit-transport---packed)). Fiecare senzor de lumină este deschis în grupul DAQ al backend-ului la adresa sa salvată, cu timpul de integrare, medierea cadrelor, expunerea automată și limita salvate: un senzor pentru care nu a fost selectată nicio limită primește `sunshine_cosine`, unul salvat fără setări (`none`) rămâne așa, iar un DAQ-E-S nu acceptă niciodată o limită. Panoul DAQ al GUI-ului nu face parte din grup: închideți mai întâi un senzor de acolo. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Captură unică de la o cameră sau o matrice denumită (`--prefix` implicit, `capture`). `--orient` (matrice): matricea salvează datele brute, care rămân în format nativ — aceasta înregistrează faptul că produsele create de `process` pe baza acestora trebuie orientate, conform `array-capture --orient`. Implicit: nativ. `project burst` / `stream` și `capture` / `burst` / `stream` (`orient: true`) au aceeași opțiune. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Serie de N cadre de la o cameră sau o matrice specificată (`-n/--count` implicit 5; `-i/--interval` secunde între cadre, implicit 0). Seriile de cadre din matrice elimină duplicatele grupurilor sincronizate repetate (mecanism de detectare a datelor învechite), astfel încât o matrice cu ciclu parțial nu poate returna N copii ale unui singur cadru; afișează rezultatele pentru fiecare iterație. O captură care eșuează pe parcurs oprește seria de cadre și iese din `1`, indicând câte cadre au fost salvate până în acel moment. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Captură pe disc într-o buclă: `--fps F` capturează o secundă (implicit 3), un cadru pe captură de la o cameră, un grup de cadre sincronizate dintr-o matrice (cu același mecanism de monitorizare a vechimii ca `project burst`), până când `-n N` capturează (implicit `0` = până la apăsarea Ctrl+C, care afișează câte au fost salvate și închide `0`, sau `1` când nu a fost salvat niciunul sau un fișier a eșuat; o apăsare de Ctrl+C înainte ca `-n N` să captureze determină ieșirea din `130`). O captură care eșuează pe parcurs oprește fluxul și închide `1`, indicând câte cadre au fost salvate până în acel moment. `--poll-interval` este învechit și ignorat: este încă acceptat, astfel încât scripturile mai vechi să poată rula, cu o notă pe stderr. |
| `project sensor read PATH NAME [--json]` | Ultimul cadru de spectru. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Înregistrează un fișier .daq pentru `--seconds` (implicit 10). O înregistrare care nu conține nicio citire este raportată pe stderr și se închide `1`. |
| `project run PATH RECIPE.yaml` | Execută o rețetă de captură YAML/JSON
. `--dry-run` validează fără a rula. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Calculează alinierea pentru o matrice — consultați [tabelul de opțiuni de mai jos](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Afișează profilul de aliniere curent. |
| `project align clear PATH NAME` | Șterge profilul stocat în cache. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Modifică ușor transformarea unui slave (implicit `0`, `0`, `0`, `1.0`: fără modificări). |
| `project align export PATH NAME --to FILE` | Salvează profilul în format JSON
. |
| `project align import PATH NAME --from FILE [--no-validate]` | Încarcă un profil salvat. |

#### Opțiuni `project align calibrate`

| Indicator | Implicit | Descriere |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metoda de aliniere. **Aceste denumiri diferă de `lattice align-calibrate`**, care acceptă formele scurte `orb` / `akaze` / `phase`; cele două comenzi nu sunt interschimbabile pentru acest indicator. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformă modelul pentru a se potrivi. |
| `--frames N` | `1` | Instantanee ale cadrelor sincronizate la media. |
| `--reference SN` | camera principală | Numărul de serie al camerei de referință; toate celelalte camere sunt aliniate la aceasta. |
| `--max-features N` | `5000` | Limita numărului de caracteristici ORB. |
| `--ratio-threshold F` | `0.75` | Testul. |
| `--ransac-threshold-px F` | `3.0` | Pragul de puncte interioare RANSAC. |
| `--min-matches N` | `15` | **Prag de calitate** — respinge soluția dacă numărul de potriviri între puncte încadrate este sub această valoare. |
| `--max-reproj-err-px F` | `4.0` | **Prag de calitate** — respinge soluția dacă depășește această eroare RMS de reproiectare. |
| `--checkerboard RxC` | — | Geometria plăcii pentru `--method checkerboard`, de exemplu `9x6`. |
| `--name PROFILE` | gol | Numele profilului încorporat în fișierul JSON
salvat. **Nu este numele matricei** — acesta este `NAME` pozițional. |

Cele două praguri de calitate sunt motivul pentru care o calibrare poate reuși să rezolve problema și totuși
să refuze salvarea: un profil care nu îndeplinește unul dintre ele ar înregistra în mod eronat, fără avertisment, fiecare
captură ulterioară, așa că este respins în loc să fie păstrat.

### Un proiect mutat (`--relink`)

Un proiect `project.json` denumește fiecare fișier sursă — fiecare imagine și înregistrare a senzorului de lumină
— după calea sa completă. Dacă mutați folderul proiectului sau întregul folder „proiecte”
pe o altă unitate, acele căi vor indica în continuare vechea locație.

Deschiderea unui proiect remediază ceea ce poate remedia singur: un fișier care se afla în folderul
proiectului este căutat în folderul în care se află proiectul acum, iar fiecare cale găsită acolo
este redirecționată și salvată (`project.json` este mai întâi copiat ca
`project.json.before-relink-<date>-<time>`). Acest lucru se întâmplă la fiecare deschidere — a aplicației,
`chloros-cli project open`, `chloros-cli process` într-un proiect existent, al SDK
-ului
și al `chloros-mcp`. O cale este redirecționată întotdeauna doar către un fișier care se află acolo.

Fișierele care se aflau în afara folderului proiectului — o sesiune hub, o importare prin „Add Folder” —
nu pot fi găsite în acest mod. Denumiți folderul în care se află acum:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` ia în considerare folderul în care au ajuns fișierele sau orice folder de deasupra acestuia: fiecare folder al unui
fișier lipsă este căutat sub acesta, pe baza numelor de folder care îi urmează
(`Hub Projects/<session>/raw/<serial>`, apoi `<session>/raw/<serial>`, ...), iar fiecare
fișierele din fiecare dosar sunt preluate din locul în care se găsesc cele mai multe dintre ele. `project open` afișează
ce a fost redirecționat, câte fișiere sursă au fost găsite și dosarele în care se aflau restul
acestora. Unele încă lipsesc: un avertisment, ieșire `0` — proiectul se deschide cu restul. Nu s-a
găsit nimic: o eroare care menționează acele foldere și `--relink`, ieșire `1`; backend-ul, la rândul său,
refuză să deschidă un astfel de proiect pentru `process` sau `project devices --reconnect` (409 HTTP
`PROJECT_SOURCES_MISSING`).

### Exemple

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Limbajul DSL pentru rețete

`project run RECIPE.yaml` acceptă un fișier YAML sau JSON care descrie o secvență de acțiuni:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Acțiuni acceptate: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` preia cheile `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (denumirile mai vechi `exposure_time`, `exposure_auto`, `gain`, `gain_auto` sunt acceptate ca aliasuri). O expunere, un câștig sau o țintă setată în acest mod este păstrată ca fiind a ta: controlul automat al expunerii din Chloros (inclusiv expunerea măsurată de DAQ) nu o modifică. O expunere manuală lasă amplificarea așa cum era: în modul automat, camera continuă să ajusteze amplificarea (pe întreaga sa gamă) către luminozitatea țintă până când setați `gain_db` sau `auto_gain: "Off"`. O comandă ulterioară `auto_exposure: "Continuous"` restabilește controlul automat al expunerii, iar `target_brightness_auto: true` restabilește ținta menținută la ținta automată a lui Chloros. Acțiunea `burst` acceptă `name` (obligatoriu), `count` (implicit 5), `interval` (secunde, valoare implicită 0), `output`, `format`, și `settings` (aceeași formă a setărilor per cameră ca `apply`); rafalele de tip matrice utilizează același mecanism de supraveghere a grupului proaspăt sincronizat ca `project burst`.

În etapa `capture`, `burst` sau `stream`, valoarea implicită a `output` este `output`. Un `output` relativ — inclusă implicit — este rezolvată în raport cu folderul din care rulați `chloros-cli project run`, nu cu folderul fișierului de rețetă sau cu folderul proiectului (în versiunea 1.2.0 a ajuns în folderul de instalare al backend-ului).

Rulați-l:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Fiecare execuție este verificată mai întâi în raport cu proiectul.**Fiecare acțiune trebuie să aibă ca cheie exact unul dintre verbele de mai sus — `- capture: {name: FrontLeft}`, nu `- action: capture` sau `- type: capture` — în cazul unui `name`, proiectul conține `cameras.json` / `sensors.json`, o cameră sau o matrice pentru `apply` / `capture` / `burst` / `stream` și un senzor de lumină pentru `sensor`; un `stream` necesită ca `count` să fie &gt; 0, iar un `wait` necesită o durată. O rețetă care conține oricare dintre aceste erori**nu** execută nicio acțiune (`[recipe] error: …`, `[recipe] not run: …`, ieșire `1`), chiar și cu `stop_on_error: false`. Tastele pe care un pas nu le utilizează sunt un `[recipe] warning:`, la fel ca orice matrice sau senzor de lumină pe care rețeta îl controlează și care nu este conectat în prezent. O rețetă nu conectează niciodată dispozitive: rulați mai întâi `chloros-cli project connect PATH`. `--dry-run` afișează rețeta, apoi aceleași erori și avertismente, iar `[recipe] valid: …` / `NOT valid: …`, nu execută nimic și iese din `1` atunci când rețeta prezintă o eroare.

---

## Variabile de mediu

| Variabilă | Efect |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Înlocuiește URL-ul backend-ului (implicit `http://127.0.0.1:5000`) — **respectată numai de familiile de comenzi `lattice`, `project` și `daq pool-*`** (`lattice network-analysis` acceptă în schimb `--backend-url`). Comenzile de bază (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) utilizează `http://127.0.0.1:<port>` și ignoră această variabilă (literalul IPv4 ocolește penalizarea de ~2 s pe cerere din Windows `localhost`→`::1`), astfel încât acestea vizează întotdeauna mașina locală. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Se setează în mediul backend-ului. `1` reduce severitatea refuzului de suprasubscriere — o conexiune de tip array cu `pin_resolution` sau o [comandă tranzitorie multi-cameră](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — la un avertisment puternic, acceptând pierderea de pachete GVSP. Doar pentru testare — consultați [Modelul de fps și rafale pentru matrice](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Se setează în mediul backend-ului: `on` sau `off` setează implicit [transportul comprimat pe 12 biți](#packed-12-bit-transport---packed) pentru fiecare matrice (implicit `auto`). O setare explicită `--packed` are întotdeauna prioritate. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (sau `true` / `yes`) dezactivează [verificarea automată a actualizărilor](#chloros-cli-update) a Linux CLI la pornire. |
| `CHLOROS_CLI_MODE` | Setat chiar de CLI; indică backend-ului să activeze procesarea paralelă. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` omite sonda de rezervă GVSP (doar rezultate ICMP). **Aceasta dezactivează pachetele jumbo, nu doar reduce volumul jurnalului** — camera răspunde la ping-urile DF doar până la 1500 pe fiecare cale, așa că această probă este singura care poate detecta pachetele jumbo. Economisește ~1 s pe cameră per conexiune; costă ~1,45× capacitatea maximă a cablului dacă rețeaua *ar fi putut* transporta pachete jumbo. SDK-ul afișează un avertisment când setați această opțiune. |
| `CHLOROS_GVSP_SESSION_REUSE` | Setat în mediul backend-ului: `0` măsoară dimensiunea pachetului la fiecare conectare, în loc de o singură dată pe cameră pe sesiune de backend. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixează dimensiunea pachetului GVSP la N octeți; omite complet sondarea. Este de preferat setarea pe comandă (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) în locul setării permanente: o dimensiune fixată nu se mai adaptează la rețeaua din fața sa, iar fixarea valorii 9000 pe o cale care nu poate transporta pachete jumbo face ca **fiecare** captură să depășească timpul de așteptare cu `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Se setează în mediul backend-ului: numărul maxim de secunde pentru o [comandă tranzitorie multi-cameră](#multi-camera-transient-sessions) sau o `lattice capture` a unei camere pe care nimic nu o reține, o petrece setând expunerea fiecărei camere din scena anterioară până când păstrează un cadru (implicit `5`). `0` dezactivează această opțiune: expunerea automată a fiecărei camere rulează apoi de la primul cadru, începând de la limita sa de expunere. |
| `CHLOROS_LOG_DIR` | Se setează în mediul backend-ului: folderul în care se salvează jurnalul detaliat. Implicit `%LOCALAPPDATA%\Chloros\logs` pe Windows, `~/.cache/chloros/logs` pe Linux (`$XDG_CACHE_HOME/chloros/logs` atunci când acesta este setat). Serviciul rădăcină al fișierului .deb setează `/var/log/chloros`, accesibil grupului `adm` — consultați [Rularea backend-ului ca serviciu](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Locul unde sunt stocate fișierele temporare. Interfața CLI utilizează automat `/mnt/ssd/tmp` dacă acesta există. |

---

## Coduri de ieșire

| Cod | Semnificație |
| --- | --- |
| `0` | Succes. |
| `1` | Eșec generic (majoritatea erorilor de subcomandă). O comandă de captură iese, de asemenea, cu codul `1` atunci când nu s-a reușit scrierea vreunui fișier solicitat, chiar dacă celelalte au fost salvate, și cu codul `array-burst` / `array-record` se închide la `1` atunci când înregistratorul raportează o eroare — consultați [Fișiere care nu pot fi scrise](#files-that-fail-to-write). `process` se închide, de asemenea, `1` atunci când exporturile unei rulări nu au putut fi verificate sau când s-a pierdut o imagine, deși celelalte produse sunt scrise. |
| `2` | Eroare de argument (un indicator necunoscut sau lipsă; de asemenea, `time-sync set-priority` fără niciuna dintre priorități). |
| `130` | Întrerupt de Ctrl+C: orice comandă pe care o întrerupe — `lattice`, `project`, `daq` și `process` deopotrivă (`process` după ce backend-ul a oprit execuția, vezi [Notele `process`](#notes)). O comandă a cărei oprire normală se face cu Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` și `project stream` cu `-n 0` — se închide în funcție de ce a făcut: `0` când totul a fost scris, `1` când ceva a eșuat. |
| `143` / `129` | Oprit prin SIGTERM / SIGHUP (128 + semnalul), după ce s-a executat curățarea comenzii: o sarcină în execuție a fost oprită și camerele sale au fost repuse la loc, o execuție de procesare a fost oprită în backend. O înregistrare de tip `lattice continuous` sau altă înregistrare a cărei oprire normală se face cu Ctrl+C iese și din `143`, nu din `0`. |

`lattice network-analysis` închide `1` pentru verdictele `needs_force_slip` și `error`, iar `selftest` închide `1`, cu excepția cazului în care toate cele șapte verificări sunt reușite.

---

## Indicații pentru depanare

- **„Este necesară autentificarea”** → Rulați `chloros-cli login EMAIL PASSWORD` o singură dată pe acest computer.
- **„Folderul proiectului este prea adânc pentru acest computer”** → Căile lungi din Windows sunt dezactivate pe acest computer (setarea implicită a Windows), astfel încât nicio cale nu poate depăși 260 de caractere, iar produsele se află la o distanță de până la aproximativ 130 de caractere sub folderul proiectului. Nu s-a procesat nimic. Mesajul indică cel mai lung folder de proiect care se încadrează: utilizați unul mai scurt (`-o` sau `chloros-cli set-project-folder`) sau activați căile lungi dWindows și reporniți Chloros. Programul de instalare Windows oferă această opțiune sub denumirea „Activați căile lungi Windows”, bifată implicit; în caz contrar, utilizați Politica de grup: *Configurare computer &gt; Șabloane administrative &gt; Sistem &gt; Sistem de fișiere &gt; Activați căile lungi Win32*.
- **„backend inaccesibil”** → Porniți aplicația desktop Chloros (Windows) sau porniți manual backend-ul: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` pe Windows (folderul `backend` din folderul de instalare; nu se află în `PATH`), `chloros-backend` pe Linux. Sau verificați `CHLOROS_BACKEND_URL` dacă este la distanță.
- **Comenzile `lattice` eșuează cu mesajul „Drivere cameră LATTICE neidentificate”** → Backend-ul nu a putut încărca mediul de execuție al camerei LATTICE cu care este instalat. Fiecare comandă `lattice` se execută pe backend; interfața CLI-ului în sine nu conține drivere de cameră. Reinstalați Chloros: programul de instalare pentru Windows și fișierul .deb instalează mediul de execuție alături de backend.
- **„Array connect” / „Array Settings” afișează „FRAMES WILL DROP” sau „Reduce ROI to enable”** → Inelul de recepție al plăcii de rețea a gazdei este prea mic (de obicei se resetează la 32 după o actualizare a driverului plăcii de rețea). Consultați [Configurarea și reglarea plăcii de rețea a gazdei](#host-nic-setup--tuning-lattice-arrays) — setați `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Mașina se blochează la repornire/oprire, apoi WMI `Invalid class` / placa de rețea nu se activează / unitățile USB lipsesc** → Driverul învechit al adaptorului USB 10GbE provoacă `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Actualizați driverul adaptorului — consultați [Configurarea și optimizarea plăcii de rețea a gazdei](#host-nic-setup--tuning-lattice-arrays).
- **Sistemul Linux: capturile aparțin utilizatorului root, deci nu le puteți șterge, iar jurnalul backend-ului nu se află în `~/.cache`** → Backend-ul rulează ca root prin `chloros-backend.service`. Treceți la `chloros-backend@<user>.service`, care îl rulează ca utilizator — consultați [Rularea backend-ului ca serviciu](#running-the-backend-as-a-service-headless). Până atunci, jurnalul serviciului root se află în `/var/log/chloros` (accesibil grupului `adm`).
- **Sistem Linux: o unitate backend este `failed` cu mesajul „O altă instanță rulează deja” (stare de ieșire 98)** → Un alt backend deține deja portul: rulează doar unul la un moment dat. Dacă sunt activate două unități backend, păstrați una: `sudo systemctl disable --now <unit>` pentru cea pe care nu o utilizați — consultați [Rularea backend-ului ca serviciu](#running-the-backend-as-a-service-headless). Dacă este activată doar această unitate, un backend pornit în afara systemd (prin `chloros-backend`, `chloros-cli` sau un script SDK) îl deține: opriți acel backend, apoi `sudo systemctl start <unit>`.
- **Avertisment privind swap-ul Jetson** → Adăugați swap bazat pe fișiere; interfața CLI afișează comenzile exacte `fallocate` / `swapon`.
- **„backend-ul nu este accesibil” dintr-o comandă `lattice`, `project` sau `daq`** → nu rulează niciun backend: aceste comenzi nu pornesc niciodată unul. Rulați `chloros-cli backend status` și consultați [Pornirea backend-ului](#start-the-backend).
- **„`daq net ...` nu este disponibil în Chloros”** (Linux și Jetson) → Unele dintre setările proprii ale unui DAQ-E sunt rezervate pentru serviciul MAPIR. Setările de rețea, repornirile și actualizările de firmware sunt `pool-set-network`, `pool-reboot` și `pool-update-firmware`; contactați asistența MAPIR pentru restul.
- **„Comenzile hardware directe DAQ necesită pachetul daq complet”** (Windows) → Utilizați `pool-*`, care controlează același senzor prin intermediul backend-ului, sau `chloros_sdk.connect_daq_sensor()` din Python.

---

## Vezi și

- [Referință SDK-Python pentru Python](sdk-reference.md) — echivalentul programatic al fiecărei comenzi CLI.
- [Senzori de lumină DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) în manualul Chloros — senzorii, limitele și intervalul calibrat al acestora, înregistrarea, rețeaua DAQ-E.
- [Camere LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) în manualul Chloros — conectare, setări ale camerei, moduri de captură, matrice.
- Documentație online: `https://mapir.gitbook.io/chloros/cli`
