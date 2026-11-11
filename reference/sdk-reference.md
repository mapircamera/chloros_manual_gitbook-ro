# Ghid de referință pentru Chloros Python SDK

**Versiune:**

1.3.0**Generat:**29 iulie 2026, ora 19:19 ·**Revizuit:** 6 noiembrie 2026**Pachet:** `chloros-sdk` (PyPI)**Public țintă:** Optimizat pentru utilizarea LLM; ușor de citit de către oameni.**Domeniu de aplicare:** Toate clasele, funcțiile și ajutoarele publice expuse de `import chloros_sdk`, cu exemple care pot fi copiate și lipite, acoperind procesarea imaginilor, controlul unei singure camere, matrice sincronizate, senzori DAQ și automatizarea proiectelor, plus serverul `chloros-mcp` pentru asistenți AI.

Dacă doriți doar informațiile esențiale, accesați:
- [Instalare și ghid de pornire rapidă](#instalare)
- [Smart-Connect pentru matrice LATTICE](#smart-connect-pentru-camere-lattice)
- [Sesiuni cu senzori DAQ](#sesiuni-senzori-daq)
- [Utilitare pentru camere](#camera-utilities)
- [Matrice tranzitorii, profiluri de aliniere și indici](#transient-arrays-alignment-profiles--indices)
- [Dosar de proiect și rețete](#project-folder--recipes)
- [Automatizarea proiectelor](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Cadre live în propriul flux de lucru](#live-frames-in-your-own-pipeline)
- [Server MCP (asistenți AI)](#mcp-server-ai-assistants)

> **Modificări în versiunea 1.3.0** (ce detectează un script existent din versiunea 1.2.0).
>
> **Modificări pe care un script din versiunea 1.2.0 ar putea fi nevoit să le gestioneze:**
>
> - **O versiune pe care backend-ul nu o confirmă generează acum o eroare.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` pe un array deschis de sesiune, `DAQSensorSession.disconnect()` / `force_disconnect()` pe un senzor deschis de acesta, iar `ChlorosProject.disconnect_all()` generează `ChlorosConnectError` atunci când backend-ul refuză eliberarea sau nu o confirmă la timp, deoarece dispozitivul poate fi încă reținut. În versiunea 1.2.0, acestea se întorceau fără nicio notificare. Ieșirea dintr-un bloc `with` nu generează niciodată o eroare.
> - **Un dispozitiv împrumutat nu este niciodată dezactivat.** `with attach_array(...)` și un `disconnect()` explicit asupra acestuia lasă matricea în funcțiune (în versiunea 1.2.0, părăsirea blocului îl dezactiva, chiar și atunci când interfața grafică îl deținea); apelați `force_disconnect()` atunci când doriți să-l închideți. La fel, un `connect_daq_sensor()` care a găsit senzorul deja în pool, inclusiv detectarea inteligentă cu flag zero, îl lasă deschis pe `disconnect()` și la ieșirea din blocul său `with`.
> - **Formatele PNG și JPEG acceptă doar produsele `display` și index.** `capture()` cu `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) și un nivel `radiance` sau `reflectance`, sau `processing="all"`, generează `ValueError` înainte de orice cerere; un `raw` / `debayered` de 12 biți este refuzat de backend. A se vedea [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` deschide senzorul respectiv sau generează** `ChlorosConnectError`, enumerând ceea ce a găsit; nu recurge niciodată la un alt senzor. A se vedea [Transport fixat](#pinned-transport).
> - **`frame_stream()` generează o eroare în loc să rămână tăcut:** `FrameFetchError`, în cazul unui refuz, la următorul ciclu ar repeta (o citire a reflectanței fără DAQ inclus), după 60 de secunde fără cadru și după 60 de secunde fără un nou declanșator. A se vedea [Un singur declanșator pe citire](#one-trigger-per-read--group_id).
> - **O expirare a timpului de așteptare pentru citire sau o conexiune întreruptă generează `ChlorosConnectError`** la fiecare apel smart-connect și `analyze_array_network()`, unde versiunea 1.2.0 genera eroarea de expirare a timpului de așteptare sau eroarea de conexiune subiacentă.
> - **`connect_array()` așteaptă 600 s în mod implicit (`timeout`, 180 s în versiunea 1.2.0), iar o conexiune la care renunță este anulată.** Apelul trimite `timeout` către backend, care readuce fiecare cameră la starea inițială și nu conectează nimic atunci când conectarea nu se realizează la timp; apelul generează `ChlorosConnectError` (504 HTTP `CLIENT_DEADLINE_PASSED`). În versiunea 1.2.0, o conexiune inițiată imediat după pornirea camerelor putea depăși durata apelului și putea conecta matricea după ce acesta generase o eroare. `connect_all()` așteaptă fiecare matrice salvată în același mod (120 s în versiunea 1.2.0). A se vedea [`connect_array()` Semnătură](#connect_array-signature).
> - **`connect_array()` păstrează senzorul complet.** Fără `width` / `height` / `binning`, fiecare cameră este setată pe senzorul complet, iar rata de cadre scade la nivelul oferit de gazdă și de rețea; 1.2.0 a redus sau a grupat cadrele pentru a menține rata ridicată. A se vedea [Matrice sincronizată](#synchronized-array--arraysession-smart-prep).
> - **Un format de pixeli Bayer pe o cameră M3M (mono) reprezintă adâncimea acesteia.** O cameră M3M oferă doar formate mono, iar presetările denumesc cele de tip Bayer: în versiunea 1.2.0, camera a refuzat formatul și și-a păstrat propriul format, în timp ce `settings.pixel_format` a denumit formatul Bayer. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, un `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` și o matriceformat comun stabilesc acum formatul Mono cu aceeași adâncime pe un M3M: `"BayerRG8"` și `high_speed` dau `Mono8`; `"BayerRG12"`, `high_quality` și `triggered` dau `Mono12`; `cam.settings.pixel_format` îi atribuie un nume. Același lucru și invers: un M3C nu este niciodată plasat într-un format Mono, așa că `"Mono12"` — provenind din oricare dintre acestea, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` sau formatul salvat al unui array — generează `BayerRG12` pe un M3C, indiferent care cameră este masterul array-ului. Un `CameraSettings` pe care îl transmiteți își păstrează numele, astfel încât același obiect solicită în continuare fiecărei camere acea adâncime în propria sa familie. `connect_camera(preset=...)` păstrează formatul presetării (versiunea 1.2.0 a setat camera la 12 biți după aplicarea presetării), iar o cameră deschisă pe o presetare nu mai scrie datele capturate în `PRESETS`. Orice alt format pe care o cameră îl refuză (unul care lipsește din lista sa `PixelFormat`, cum ar fi `RGB8` pe un M3M sau `BayerRG16` pe un firmware care nu îl conține) nu este denumit nici ca fiind al său: după ce un `LatticeCamera`, `apply_settings()` sau `set_format()`, `cam.settings.pixel_format` și `packed_transport` indică formatul în care camera transmite fluxul, iar o linie tipărită indică formatul pe care l-a refuzat.
> - **`analyze_array_network()` indică ce conexiune se aplică:** `pixel_format=None` (anterior `"BayerRG8"`) înseamnă formatele setate de conexiune (Mono12 / BayerRG12), iar `packed="auto"` indică opțiunea de transport comprimat, astfel încât valoarea fps afișată poate diferi de 1.2.0'. Un `pixel_format` pe care îl transmiteți proiectează un fișier M3M la acea adâncime. Consultați [Asistentul de analiză a rețelei](#network-analysis-helper).
> - **`CameraSession.capture()` returnează un `CaptureResult`** (în continuare o listă de dicționare de cadre) cu `.skipped`, `.failed`, `.daq_files` și `.daq_matches`. Un rezultat `ArraySession.capture()` conține, de asemenea, `.alignment` (dacă s-a aplicat alinierea și, dacă nu, de ce) și `.oriented`.
> - **Capturile de matrice sunt aliniate în mod implicit, la fel ca cele ale aplicației.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` și `ArrayHandle.capture()` / `capture_smart()` aliniază fișierele `debayered`, `display`, `radiance`, `reflectance` și fișierele index în cadrul camerei principale și le decupează la suprapunerea membrilor (mai mică decât senzorul cu offset-urile camerelor, `XMP-mapir:AlignmentJSON` `"applied": true`), astfel încât acestea să fie aceleași fișiere ca și capturile aceleiași scene realizate de aplicația Chloros; `raw` rămâne în formatul nativ. Anterior, o captură fără `aligned=True` salva fiecare fișier la dimensiunea nativă, fără deformare. `aligned=False` păstrează această setare. `chloros-mcp``capture_array` și `start_capture_run` adoptă aceeași setare implicită. Consultați [Metodele `ArraySession`](#arraysession-methods).
> - **`connect_all()` indică un backend care refuză SDK-ul.** Un backend în funcțiune care este deconectat sau care are un plan inferior planului Copper este raportat sub `errors` cu propriul motiv (HTTP 401 / 403), nu ca „niciun backend Chloros nu rulează”, iar fiecare apel `connect_all()` verifică din nou backend-ul, astfel încât apelarea acestuia după autentificare funcționează pe același obiect de proiect.
> - **Un `output_dir` relativ funcționează cu un backend local.** Căile relative sunt transformate în căi absolute față de directorul de lucru al scriptului înainte de a fi trimise. În versiunea 1.2.0, backend-ul le rezolva în propriul folder de instalare, ceea ce ducea la eșecul operațiunii cu mesajul „Accesul este refuzat”.
> - **`numpy` este acum o cerință** a SDK-ului (apelurile live-frame returnează matrice numpy).
> - **Un nume de nivel de export care nu este „one” este respins.** `processing=` și `levels=` acceptă `raw`, `debayered`, `radiance`, `reflectance` sau `display` (sau un alias precum `preview`; `processing=`, precum și `"all"`). Orice alt nume este respins înainte de atingerea camerei (400 HTTP `UNKNOWN_LEVEL`: `capture()` generează `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); versiunea 1.2.0 a salvat sau a returnat `raw` pentru aceasta.
> - **Funcția „Texture Aware” este refuzată pentru capturile LATTICE.** Într-un proiect care conține capturi LATTICE, `configure(debayer="Texture Aware (Slow, Highest Quality)")` generează `ChlorosConfigurationError`, la fel și `process()` după un `configure()` care a solicitat acest lucru înainte de import. O valoare „Texture Aware” salvată anterior de proiect este setată la „Standard”, cu o eroare `UserWarning`. Capturile LATTICE beneficiază întotdeauna de demosaic standard, așa că versiunea 1.2.0 a acceptat setarea și a ignorat-o; nici aplicația Chloros nu oferă această opțiune pentru ele. Vezi [Valori acceptate](#supported-values).
> - **Camerele revin la starea în care au fost găsite — cu excepția cazului în care se utilizează `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, la sfârșitul fiecărei funcții tranzitorii (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) și oprirea funcției `start_camera_continuous()` readuc fiecare cameră la starea inițială: cablajul declanșatorului și al liniei de sincronizare, conexiunea PTP și toate setările camerei modificate în timpul sesiunii (expunere și amplificare, ținta AE, limitele și amortizarea, balansul de alb, nivelul de negru, formatul pixelilor, binningul și ROI, răsturnările, rata de cadre). În schimb, `keep_settings=True` salvează setările fiecărei camere în memoria proprie a acesteia, astfel încât să pornească cu ele; `clear_settings=True` resetează memoria respectivă la setările din fabrică. În ambele cazuri, camerele sunt lăsate să funcționeze în mod liber. Un proiect își configurează camerele pe baza propriilor setări salvate atunci când le conectează. Consultați [Memoria camerei](#camera-memory--keep_settings--clear_settings).
> - **Fiecare mod salvează un proiect în același mod.** Orice modificare adusă unei camere, unui array sau unui senzor de lumină conectat la un proiect — din SDK, aplicație, CLI sau serverul MCP — este salvată în fișierul `cameras.json` / `sensors.json`, iar deschiderea proiectului în orice mod configurează dispozitivele pe baza acestuia; un `ChlorosProject` își denumește propriul folder în backend. Anterior, doar aplicația salva proiectul. Consultați [O singură înregistrare de proiect, pentru fiecare mod](#one-project-record-every-mode).
> - **Fișierele create din mai multe camere menționează fiecare dintre ele.** O stivă multibandă, un index live creat din benzile mai multor camere și o imagine `composite/` dintr-o captură matricială conțin seria, modelul și pachetul de calibrare ale fiecărei camere sursă sub formă de liste (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), iar fișierul EXIF `SerialNumber` listează numerele de serie, separate prin virgulă. Anterior, acestea nu conțineau nicio cameră sau conțineau doar numărul de serie al primei camere. A se vedea [Fișiere create de mai multe camere](#files-made-from-several-cameras).
> - **Doar un fișier `raw` conține etichetele de calibrare ale camerei.** Setul compatibil cu MicaSense (`BlackLevel`, `RadiometricCalibration`, fiecarecanal, seturile `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`) și încărcătura utilă `CalibrationJSON` descriu valorile brute ale senzorului, astfel încât acestea se află doar în fișierul de captură `raw`. Toate celelalte fișiere — fișierele de captură`debayered`, `display`, `radiance`, `reflectance` și fișierele index, cadrele de înregistrare continuă și cele din fluxuri cu mai multe camere— și fiecare produs `process()` — poartă identitatea camerei (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) și, la o captură, înregistrarea propriu-zisă a cadrului (`XMP-mapir:FrameJSON`: expunere, amplificare, temperatură, format de pixeli, ora capturii). În versiunea 1.2.0, acestea conțineau și setul raw, pe care un instrument de procesare (Pix4D, Metashape) îl aplica pixelilor care nu aveau valori raw. Procesarea identifică calibrarea unui astfel de fișier pe baza numărului de serie și a `CalBundleSha256`.
>
> **Alte modificări:**
>
> - **Rata automată de cadre a unui array se stabilește în funcție de ceea ce furnizează array-ul.** O rată care pierde mai mult de 1 % din declanșări este redusă, pe baza unei evaluări efectuate pe parcursul mai multor perioade lungi de monitorizare, nu doar una singură, iar rata de la care a fost redusă este reîncercată din când în când. O rată setată manual rămâne neschimbată.
> - **Expunerea automată este mai stabilă.** Se menține la lumină constantă, se stabilizează în câteva secunde după o schimbare, ignoră zonele foarte luminoase și restabilește funcționarea unei camere care nu mai răspunde în lumina soarelui puternică. Vezi [Cum funcționează expunerea automată](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` numără doar imaginile și înregistrările tale.** Un folder care conține fișierul `camera_cal/` al unui proiect nu mai are acele fișiere de calibrare numărate ca imagini și înregistrări, iar un proiect care le-a preluat deja le elimină.
> - **Se aplică `configure(radiometric_output=...)`.** Acum se setează produsele `Export radiance` / `Export reflectance` ale proiectului; înainte, setările proprii ale fiecărui proiect prevaleau asupra acestora, astfel încât `radiance`, `sensor-response` și `all` exportau toate radianța și reflectanța.
> - **`test_array_sync()` pornește din starea de pornire a camerelor.** Acesta activează PTP-ul camerelor pentru execuție, așteaptă ca ceasurile acestora să se sincronizeze și să se stabilizeze înainte de primul său declanșator (`ptp_lock`) — fiecare cameră rămâne sincronizată timp de 120 s, aproximativ 2 minute de la pornire (`ptp_settle_s=` îl setează), deoarece un ceas abia sincronizat prezintă o abatere de până la ~200 us timp de aproximativ 2 minute — și readuce PTP-ul fiecărei camere la starea inițială. O cameră care este încă sincronizată de la un test anterior nu mai este așteptată din nou. În trecut, lăsa PTP-ul dezactivat într-o rețea cu un singur comutator în care niciun DAQ-E nu răspundea și citea ceasul liber al fiecărei camere ca pe un ceas PTP care nu se sincroniza („PTP nu este sincronizat”, la fiecare rulare).
> - **O înregistrare `.daq` indică dacă capacul său a fost ales sau presupus.** DAQ-U, DAQ-M și DAQ-E sunt livrate fără capac, iar Chloros nu poate detecta dacă este montat unul, astfel încât un senzor conectat fără un capac specificat primește implicit `sunshine_cosine`, o *presupunere*. Fiecare înregistrare Chloros (schema 1.30) stochează acum `cap_id_source` alături de `cap_id`: `operator` atunci când ați denumit un capac (inclusiv `none`), `auto_default` atunci când acesta era implicit. Procesarea unei înregistrări al cărei condensator a fost presupus, cu setarea condensatorului DAQ al proiectului lăsată pe „Auto”, afișează un `[daq-dls] WARNING` care denumește fișierul: un DAQ-U fără condensator, sub corecția de lumină solară, indică o valoare de aproximativ 30 de ori mai mare. **Denumiți condensatorul pe care l-ați montat** atunci când vă conectați (`connect_daq_sensor(cap_id=...)` sau `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, sau din meniul derulant al GUI-ului), sau setați condensatorul DAQ al proiectului, iar avertismentul va dispărea.
> - **Nu mai apar ferestre de consolă pip pe Windows.** Pe un PC cu Python instalat, backend-ul și fiecare dintre procesele sale de prelucrare a imaginilor rulau `pip` o singură dată la pornire, iar fiecare deschidea o fereastră de consolă. Acum nu mai fac acest lucru.
> - **Înregistrările `.daq` în timp real supraviețuiesc unei întreruperi de curent.**O înregistrare este scrisă cu un jurnal și o sincronizare completă a discului la fiecare confirmare, în spatele senzorului, astfel încât un disc lent întârzie fișierul, dar niciodată citirile. O înregistrare oprită în mijlocul unei confirmări (întrerupere de curent, oprire forțată) poate lăsa fișierul `<name>.daq-journal` lângă ea:**copiați-le pe amândouă împreună.** Chloros revine la jurnalul anterior la prima citire a fișierului, astfel încât înregistrarea se deschide cu tot ce a fost confirmat înainte de oprire (se pierd aproximativ ultimele 3 secunde); în versiunea 1.2.0, o întrerupere a alimentării cu energie electrică la momentul nepotrivit putea face ca întregul fișier să devină ilizibil („imaginea discului bazei de date este defectă”).
> - **`DAQSensorSession.record_start()` pe un disc ocupat nu mai blochează senzorul**: citirile sunt puse în coadă și scrise pe un thread propriu, iar una care nu a putut fi scrisă este contabilizată în `status()` al lui `record_errors` (împreună cu `record_last_error`).
> - **`LatticeCamera` direct:**un cadru fără expunere/amplificare a blocului raportează acum expunerea și amplificarea**reale** ale camerei (soluția de rezervă citea o valoare stocată în cache care putea rămâne înghețată pe durata întregii serii de expuneri automate), iar `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` aplică aceeași eliminare a zgomotului de intrare de 5 µs de care beneficiază camerele slave conectate prin cablu la Chloros (nefiltrată, o cameră slave declanșa cadre fără a fi trimis un semnal de declanșare).
> - **Camerele declanșate prin software nu mai ignoră fiecare al doilea semnal de declanșare.** O cameră în modul de declanșare prin software pune acum în coadă un semnal de declanșare care sosește în timpul citirii sale (sau Chloros așteaptă ca aceasta să fie gata), așa cum făceau deja camerele sincronizate prin cablu.
> - **`discover_daq_sensors()` denumește un DAQ-E-S `DAQ-E-S`.** `model` era `DAQ-E` pentru fiecare senzor Ethernet; unul care nu răspunde la verificarea identității în aproximativ 2 s este încă listat, ca `DAQ-E`.
> - **Noutate: `chloros-mcp`, un server MCP pentru asistenți AI.** Claude Desktop, Claude Code, Cursor și alți clienți MCP pot procesa imagini (crea un proiect, importa capturi, alege setări, procesa, raporta rezultatele exportate) și pot controla camerele LATTICE, matricile și senzorii de lumină DAQ: conectare, previzualizare, setare expunere și culoare, captură, înregistrare. Instrumentele sale acoperă funcționalitățile acestui SDK. Se instalează odată cu SDK, iar pachetul .deb pentru Linux îl plasează în `/usr/bin`. Consultați [Serverul MCP (asistenți AI)](#mcp-server-ai-assistants).
> - **Noutate: utilitare pentru camere**, comenzile `chloros-cli lattice` pe care le apelează SDK: profil de culoare, saturație / contrast, zone luminoase, balans de alb (o singură captură, automat, raporturi fixe), presetări, modele de filtre, starea și actualizarea firmware-ului, alimentare (verificare, repaus, activare), calibrarea țintei, adaptoare de rețea (verificare, estimare, remediere), corectarea vignetării pe fișiere, o citire DLS și verificarea gazdei. Vezi [Utilitare pentru cameră](#camera-utilities).
> - **Noutate: restul CLI sub formă de apeluri SDK**, astfel încât CLI, SDK și `chloros-mcp` oferă aceleași funcționalități (CLI păstrează doar autentificarea, configurarea unui DAQ-E și serverele DAQ directe, din motive de securitate): camere deschise cu o singură comandă (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), fișiere cu profiluri de aliniere (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indici (`index_catalog()`, `compute_index()`, `capture_live_index()`), fișierul „grandmaster” PTP (`time_sync_status()` și altele similare), folderul proiectului (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) și rețetele (`run_project_recipe()`, propriul executor al CLI). Pentru senzori de lumină: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` și `light_sensor_reflectance()` (reflectanța și indicii „Ambient + Object” ai celor doi senzori); pentru o cameră: `CameraSession.status()` și `capture_repeated()`; pentru o matrice: `ArraySession.capability()` și `set_member_settings()` (expunerea, amplificarea unui element, ...). Consultați [Matrice tranzitorii, profiluri de aliniere și indici](#transient-arrays-alignment-profiles--indices), [Dosar de proiect și rețete](#project-folder--recipes) și [Sesiuni de senzori DAQ](#daq-sensor-sessions).
> - **Produse îmbunătățite ale indicilor LATTICE** (NDVI și alți indici) din `process()`, în fiecare mod de procesare.
> - **Camere LATTICE OCN: produse îmbunătățite de radianță, reflectanță și indici în lumina soarelui.** Se aplică o corecție de flotă după separarea benzilor — în timp real, în capturi și în `process()`; camerele RGN rămân neschimbate. Consultați [Ieșire radiometrică](#radiometric-output-lattice-multispectral-pipeline).
> - **Fiecare captură salvează valoarea DAQ utilizată, la fel ca interfața grafică.** `CameraSession.capture()` și `ArraySession.capture()` / `continuous()` scriu fișierul `.daq` și fișierul său `.match.json` ori de câte ori un DAQ este legat sau inclus într-un pool, inclusiv o captură doar în format brut; în versiunea 1.2.0, o captură brută sau debayered era salvată doar cu `force_daq=True`. `daq=False` o dezactivează, iar `daq_tolerance_ms=` setează toleranța de potrivire; ambele rezultate indică ce a fost salvat în `.daq_files` și `.daq_matches`. O captură așteaptă acum următoarea citire a DAQ-ului, astfel încât să poată combina citirile de pe ambele părți ale expunerii. A se vedea [Citirea DAQ-ului pe care o salvează o captură](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` are cadre în timp real și o captură continuă pe partea de server.** `grab()` și `frame_stream()` returnează cadre numpy dintr-un singur declanșator (aceeași citire ca și API-ul proiectului `ArrayHandle.grab()` / `frame_stream()`), iar `continuous()` salvează fiecare grup sincronizat în backend și returnează un [`ContinuousHandle`](#continuoushandle). Consultați [Cadre live în propriul dvs. pipeline](#live-frames-in-your-own-pipeline).
> - **Un array `grab()` este asociat unui singur declanșator** (`ArrayHandle.grab()` și noul `ArraySession.grab()`). În versiunea 1.2.0, 12 din cele 20 de-cameră `ArrayHandle.grab()` se întindeau pe două declanșatoare. Fiecare cadru conține acum `group_id`, iar o matrice `frame_stream()` generează fiecare declanșator o singură dată. A se vedea [Un declanșator pe citire](#one-trigger-per-read--group_id).
> - **Cadre pe bandă și aliniate:** `grab()` / `frame_stream()` preiau `band=`, `align=`, `bit_depth=` și `refl_max=` pe identificatorii camerei și ai matricei. Consultați [Exportarea unei singure benzi](#exporting-a-single-band--band).
> - **Noi metode și argumente `ArraySession`:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (împreună cu `CaptureResult.group_seq` și `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` leagă grupuri consecutive și se oprește la umplerea unității de stocare.
> - **`ArraySession.record()` înregistrează fără interfața grafică deschisă.** Cu vizualizarea combinată a interfeței grafice închisă, backend-ul redă vizualizarea în sine:NDVI sau indexul pe care îl transmiteți ca `expr=`. `RecorderHandle.stop()` generează `ChlorosConnectError` pentru o înregistrare care a eșuat, una care nu a inclus niciun cadru scris, și păstrează rezultatul opririi la `.result` (lăsarea unui bloc `with` nu generează o excepție: verificați `rec.result`). Consultați [`RecorderHandle`](#recorderhandle).
> - **Noi metode `DAQSensorSession`:** `enable_ptp()`, `force_disconnect()` și, pentru setările proprii ale unui DAQ-E, `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` și `update_firmware()`. `record_stop()` returnează `warning` și `gaps`. O repornire care a închis o înregistrare îi atribuie un nume, în răspunsul său sau, atunci când repornirea eșuează, în `ChlorosConnectError` (`recording_closed`; consultați [Setările dispozitivului DAQ-E](#daq-e-device-settings)). Consultați [Metodele `DAQSensorSession`](#daqsensorsession-methods).
> - **Un DAQ-E din grup care încetează să mai transmită date este reconectat de către backend**, cu setările pe care le avea, iar o înregistrare de pe acesta continuă în același `.daq`, cu intervalul marcat în fișier; `record_stop()` returnează întreruperile ca `gaps`. În versiunea 1.2.0, senzorul rămânea inactiv până când era deconectat și reconectat, ceea ce ducea la încheierea înregistrării. Consultați referința CLI [Un DAQ-E care încetează să mai transmită date](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` funcționează pe baza unui SDK instalat.** Roata (fișierele de instalareși PyPI) conține acum încărcătoarele de proiect de care are nevoie; înainte, `open_project()` genera `ImportError`, denumind `project_cameras` / `project_sensors` în orice SDK instalat prin pip. `connect_all()` conectează camerele, matricile și senzorii de lumină salvați printr-un backend (senzorii de lumină prin intermediul grupului său DAQ), iar un dispozitiv salvat pe care nu are cum să îl găsească este listat sub `errors` cu motivul respectiv, în loc să fie omis fără nicio explicație. O matrice salvată care este deja activă în backend (cea a interfeței grafice, sau ale lui `chloros-cli project connect`) este atașată și lăsată să funcționeze, în timp ce înainte era raportată ca eroare („Camera  se află<sn> deja în matrice <id>”). A se vedea [Metodele `ChlorosProject`](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` și `process_lattice_capture()` găsesc Chloros pe Windows.** Acestea caută locația în care programul de instalare plasează backend-ul (`<install folder>\backend\`, `C:\Program Files\Chloros\backend\` în mod implicit, un folder personalizat din înregistrarea propriu-zisă a programului de instalare) și se conectează la un backend care rulează deja, fără a mai căuta unul. În versiunea 1.2.0, acestea afișau mesajul „Backend Chloros neidentificat” pe o mașină instalată, cu excepția cazului în care se trecea `backend_exe=`.
> - **Un backend care se oprește la pornire este raportat imediat.** Când `ChlorosLocal()` sau un apel smart-connect pornește un backend care se închide înainte de a răspunde, apelul generează în câteva secunde erorile `ChlorosBackendError` din `ChlorosLocal` și `ChlorosConnectError` din apelurile smart--connect, împreună cu codul de ieșire, comanda pe care a executat-o și ultimele linii afișate de backend. Versiunea 1.2.0 a așteptat expirarea timpului de așteptare de 60 de secunde la pornire și nu a specificat motivul. Vezi [Pornirea automată a backend-ului](#backend-auto-start).
> - **Backend-ul continuă să ruleze între scripturi.** La ieșirea dintr-un bloc `with ChlorosLocal(...)`, `process_folder()` / `process_lattice_capture()` care se întorc și `chloros-mcp` care se închide nu mai opresc un backend pornit de SDK; scriptul următor îl reutilizează. Acesta se oprește la `logout()` (cu excepția cazului în care aplicația Chloros este deschisă sau o execuție este în curs), la `shutdown_backend()` sau de la sine după 30 de minute dacă nimic nu îl utilizează (`CHLOROS_BACKEND_IDLE_MIN` din mediul backend-ului modifică acest lucru; `0`: niciodată).
> - **Un backend pornit de SDK este solicitat să se închidă, nu este oprit forțat.** `shutdown_backend()` permite backend-ului să elibereze camerele și senzorii de lumină și să finalizeze orice înregistrare; acesta este oprit doar dacă încă rulează după 10 secunde. Versiunea 1.2.0 îl oprea imediat (pe Windows, o oprire forțată), ceea ce putea lăsa un senzor de lumină DAQ conectat la acel backend fără răspuns. Un backend pe care SDK-ul l-a găsit în execuție este lăsat în continuare să funcționeze.
> - **Pe Linux, pornirea automată nu rulează niciodată un `backend_server.py` din folderul curent.** În absența unui backend instalat, un script pornit dintr-un folder care conținea unul îl rula; acum este pornit doar backend-ul instalat (sau, pentru `ChlorosLocal`, cel indicat de `backend_exe=`).
> - **`configure(indices=...)`**adaugă fiecare presetare la proiect pentru benzile camerei respective, inclusiv un proiect nou, și afișează un avertisment (`UserWarning`) pentru un nume pe care nu îl poate adăuga.**`import_images()`** raportează înregistrările `.daq` ca `scan_count` și nu mai afișează avertismente privind un folder de captură LATTICE.
> - **`read_image_audit_tags()` funcționează după o instalare prin pip.** Citește singur metadatele imaginii; în versiunea 1.2.0 returna `{}` fără componentele interne ale instalării pentru desktop din calea Python.
> - **`capture()`, imediat după `connect_array()`**, așteaptă până la 20 de secunde pentru primul grup de cadre al matricei, în loc să eșueze cu codul HTTP 503 „Matricea încă se încălzește”.
> - **`status()` raportează o blocare.** Noul câmp `stalled` este True în timp ce bucla de captură rulează, dar nu publică nimic, iar `actual_fps` este atunci 0. În versiunea 1.2.0, `actual_fps` își păstra ultima valoare în timpul un blocaj, astfel încât un array blocat afișa `actual_fps > 0`, în timp ce capturile eșuau cu eroarea 503 „No fresh synced frame group”.
> - **Mai puține seturi de cadre incomplete și erori 503 la captură:** intrarea de declanșare a fiecărei camere slave este acum stabilizată.
> - **Transportul comprimat pe 12 biți** (`connect_array(packed="auto")`) crește rata de cadre a matricei cu senzor complet cu aproximativ o treime. A se vedea [Transportul comprimat pe 12 biți](#packed-12-bit-transport-packed).
> - **Expunerea matricei sub o zonă de interes (ROI):** o zonă de interes mică nu mai limitează expunerea unui element la perioada de funcționare liberă a senzorului (4,9 ms la 768 × 304). Consultați [Expunerea elementelor matricei](#exposure-on-array-members).
> - **Expunerea automată nu mai reduce rata de cadre automată a matricei.** O matrice sincronizată prin cablu funcționează la rata permisă de rețea și de citirea camerelor; expunerea automată umple perioada oferită de această rată, apoi crește amplificarea. O expunere manuală mai lungă decât această perioadă încetinește declanșarea pentru a se adapta la ea, în loc să fie refuzată. A se vedea [Expunerea pe elementele matricei](#exposure-on-array-members).
> - **`read_nodes()`**adaugă `frame_values` și `notes` în timp ce expunerea automată este activă (vezi [`read_nodes()` în timpul expunerii automate](#read_nodes-under-auto-exposure--frame_values-and-notes)). Când camera refuză o expunere sau un câștig,**`set_settings()`** indică intervalul pe care îl acceptă și restabilește modul automat pe care îl dezactivase pentru acea valoare; `set_settings(target_brightness_auto=True)` este nou.
> - **`orient=`** pe `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` și `ArrayHandle.capture()` / `capture_smart()` rotește produsele în funcție de orientarea camerei. Consultați [Metodele `ArraySession`](#arraysession-methods).
> - **`DLS()` necesită `mac_address` specific senzorului.** Nu mai există un MAC implicit (acesta era adresa unei unități); `DLS()` fără unul generează `DLSError`. Consultați [magnitudinile benzii DAQ-M](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Programul de instalare pentru Windows** oferă opțiunile „Activați căile lungi Windows” și „Excludeți backend-ul Chloros din scanarea Microsoft Defender”, ambele bifate implicit. Consultați [Pasul 1](#step-1--install-the-chloros-platform-package).
> - **Pachetul .deb pentru Linux** instalează Python SDK pentru sistemul `python3` (anterior, acesta doar plasa fișierul wheel pe disc), mărește bufferele de recepție ale socket-urilor din kernel de care au nevoie fluxurile de date de la camere și adaugă un serviciu backend care rulează sub contul dvs. de utilizator. Pachetul Jetson este `chloros_1.3.0_arm64_jp6.deb`, pentru JetPack 6, cu PyTorch CUDA de la JetPack. Consultați [Pasul 1](#step-1--install-the-chloros-platform-package) și [Pasul 2](#linux-deb).

---

## Arhitectura în 60 de secunde

SDK-ul este un strat subțire de Python peste backend-ul Chloros (același server Flask pe care îl folosesc interfața grafică pentru desktop și CLI). Pentru automatizare, importați `chloros_sdk` și apelați metode de nivel înalt; în fundal, fiecare apel devine o cerere HTTP către backend-ul local pe portul 5000 — `http://127.0.0.1:5000/api/...` (în mod deliberat nu `localhost`, care se rezolvă mai întâi ca `::1` pe Windows și costă ~2 s pe cerere către un backend exclusiv IPv4). Backend-ul deține parcul de echipamente — camere, senzori DAQ, profiluri de aliniere, buffere de cadre — astfel încât scripturile SDK-ului să poată coexista cu interfața grafică fără a intra în conflict pentru porturile seriale sau lățimea de bandă a plăcilor de rețea.

Există trei interfețe pe care le veți utiliza:

1. **`ChlorosLocal` + funcții libere** (`process_folder`, `process_lattice_capture`) — Flux de procesare a imaginilor. Rulați un folder întreg prin calibrare / debayer / export de index dintr-o singură apelare Python.
2. **Mânerele Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Deschide o sesiune backend persistentă pentru hardware-ul activ. Același flux „smart-prep” ca și în GUI: sondă de rețea, selectare automată a nivelului, PTP, inițializare AE, configurare declanșator GPIO.
3. **`ChlorosProject` / `open_project`** — Încarcă un proiect salvat (folderul conținând `cameras.json` + `sensors.json` + `project.json`), conectează simultan camerele, matricile și senzorii de lumină ai acestuia și controlează capturile folosind identificatori denumiți.

Suprafețele 1 și 2 **pornesc automat un backend local** dacă nu există deja unul în așteptare (același fișier binar inclus în pachet pe care îl lansează GUI-ul/interfața CLI) — astfel încât un script simplu funcționează dintr-un shell nou, fără a fi necesar să porniți mai întâi un backend: `ChlorosLocal` (și `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` și `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` și `analyze_array_network()` nu pornesc niciodată: au nevoie de un backend care rulează. Treceți `auto_start_backend=False` pentru a renunța (de exemplu, atunci când se face referire la un backend la distanță, care nu este niciodată inițializat). Consultați [Pornirea automată a backend-ului](#backend-auto-start). Surface 3 se comportă diferit: `open_project()` nu acceptă parametrul `auto_start_backend`, iar `connect_all()` nu pornește niciodată un backend — acesta interoghează o dată `http://127.0.0.1:5000` și, dacă nu primește niciun răspuns, revine la controlul direct (fără backend) al dispozitivului `lattice_sdk`. Acest lucru necesită runtime-ul SDK-ului Arena și OpenCV (`cv2`) care să poată fi importat în Python, lucru pe care pachetul SDK nu îl instalează; fără acestea, fiecare cameră și matrice salvate sunt raportate ca erori, cu motivul respectiv. Senzorii de lumină ai unui proiect nu au o cale fără backend: fără un backend în execuție, fiecare senzor salvat este raportat ca eroare, cu motivul respectiv (vezi [`SensorHandle`](#sensorhandle)). Doar `proj.process()` și `stream(..., overlays=True)` construiesc în mod leneș un `ChlorosLocal()` (care pornește automat).

Toate trei sunt autentificate-gated: rulați `chloros-cli login` o singură dată pe mașină sau autentificați-vă prin intermediul GUI-ului de pe desktop. Ce excepție generează o apelare fără o sesiune validă depinde de interfață (vezi [Excepții](#exceptions)): `ChlorosAuthenticationError` din `ChlorosLocal` (și `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` din apelurile smart-connect și sesiunile acestora, `status: "error"` cu o excepție `error_code` provenită de la `analyze_array_network()`, precum și o intrare sub `errors` în raportul `connect_all()`.

Cerințe:
- Python 3.7+ (conform specificațiilor pachetului; dezvoltat/testat pe 3.10)
- Chloros Desktop instalat local (fișierul binar al backend-ului este inclus în programul de instalare)
- Autentificare activă în Chloros+ la **orice nivel plătit de Chloros+ (Copper sau superior)**; nivelul gratuit**Iron**nu are acces la SDK/CLI. Această cerință este impusă**de partea serverului****:** fiecare cerere marcată cu SDK/CLI trebuie să includă atât o sesiune activă, cât și un plan plătit. Un plan inferior nivelului Copper primește codul de eroare `403` împreună cu `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` din `ChlorosLocal`, `ChlorosConnectError` din apelurile smart-connect). Un apelant deconectat primește în schimb `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` din `ChlorosLocal`, `ChlorosConnectError` din apelurile smart-connect) — cele două sunt distincte deoarece rulararea din nou a `chloros-cli login` rezolvă problema `401`, dar nu poate rezolva problema `403`, care necesită o actualizare a planului.
- Utilizarea offline este acceptată în perioada de grație a planului: nivelul este citit din cache-ul de validare a serverului (5 minute) sau din cache-ul licenței semnate și asociate dispozitivului (30 de zile pentru planurile lunare, până la expirarea abonamentului pentru cele anuale). La expirarea acestei perioade de grație, planul trece la nivelul gratuit, iar accesul la SDK/CLI se întrerupe până când dispozitivul reușește să se conecteze la server măcar o dată. Eroarea `chloros-cli status` (`GET /api/license-status`) rămâne accesibilă la nivelul gratuit, astfel încât motivul să fie vizibil.
- Windows 10/11 pe 64 de biți, **Ubuntu 22.04 LTS sau o versiune mai recentă**, sau Jetson (JetPack 6). Ubuntu 20.04**nu** este acceptat: dependențele lui `.deb` derivă din ceea ce leagă backend-ul, inclusiv `libc6 (>= 2.35)`, iar Focal livrează glibc 2.31.

---

## Instalare

Python SDK este un strat subțire de Python peste backend-ul Chloros. Cu excepția cazului în care rulați un backend pe o altă mașină (consultați [Modul backend la distanță](#remote-backend-mode-pip-only-host-via-tunnel)), aveți nevoie de **pachetul Chloros pentru desktop instalat local** (programul de instalare pentru Windows sau Linux `.deb`), care include senzori DAQ de lumină. Pachetul conține fișierul binar al backend-ului, mediul de execuție Arena SDK pentru camerele LATTICE și pachetele de calibrare.

Cele mai recente descărcări: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Pasul 1 — Instalați pachetul platformei Chloros

#### Windows (.exe)

1. Descărcați `Chloros Installer 1.3.0.exe` de pe pagina de descărcări.
2. Rulați programul de instalare și urmați instrucțiunile expertului. Calea de instalare implicită este `C:\Program Files\Chloros\`.
3. Lansați Chloros cel puțin o dată și conectați-vă cu contul dvs. Chloros+.

Programul de instalare permite ca backend-ul (`chloros-backend.exe`, nimic altceva) să treacă prin Firewall-ul Windows pe fiecare profil de rețea: intrare UDP, pentru fluxurile de la camere, PTP, mDNS și fluxul DAQ-E, și intrare TCP numai din subrețeaua locală, pentru conexiunea pe care un DAQ-E o stabilește înapoi către computer în timpul unei actualizări de firmware ([`update_firmware()`](#daq-e-device-settings)). Programul de dezinstalare elimină ambele reguli. API-ul HTTP al backend-ului rămâne pe `127.0.0.1` în orice caz.

Grupul **Setări de sistem** al expertului are două opțiuni, ambele bifate implicit:

| Opțiune | Ce face | Dezactivare instalare silențioasă |
| --- | --- | --- |
| **Activare căi lungi Windows** | Permite ca folderele de proiect să se afle la un nivel mai adânc decât limita de 260 de caractere a căilor Windows. Cu opțiunea pentru căi lungi dezactivată (setarea implicită a Windows), un folder de proiect prea adânc pentru produsele sale este respins înainte ca orice operațiune să fie procesată — consultați [Depanare](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Excluderea backend-ului Chloros din scanarea Microsoft Defender** | Oprește scanarea în timp reala fiecărui fișier scris de o captură de cameră. La rate de captură ridicate, scanarea consumă aproximativ 30 % din rată: măsurat la ROI 512 pe o matrice de 9 camere, captura RAW continuă a rulat la 52,7 capturi/s cu scanare și la 77,7 capturi/s cu backend-ul exclus. | `/MERGETASKS="!defenderexclusion"` |

O captură continuă raportează încărcarea Defender ca `defender_cores` (vezi [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Instalați cu `apt install ./…`, nu cu `dpkg -i`: apt instalează și cerințele Python SDK, pe care pachetul le recomandă, nu le impune (a se vedea [Pasul 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Pentru Jetson Orin / Orin NX / Orin Nano pe JetPack 6 (L4T R36). PyTorch-ul său este versiunea CUDA a JetPack 6, astfel încât procesarea GPU se execută pe GPU-ul Orin; utilizează bibliotecile CUDA și cuDNN instalate de JetPack 6 (`sudo apt install nvidia-jetpack` pe o imagine care nu le conține).

#### Linux: tampoane de recepție și serviciul backend

Ambele pachete .deb măresc la 32 MB tampoanele de recepție ale socket-urilor din kernel, necesare pentru fluxurile camerelor LATTICE (`/etc/sysctl.d/60-chloros-gige.conf`). La setările implicite ale kernel-ului, pachetele camerei sunt pierdute în kernel, iar un array pierde cadre și rata de cadre; `chloros-cli lattice network` raportează starea bufferelor atunci când acestea sunt pe terminate. Consultați secțiunea [Gazde Linux — buffere de recepție socket](cli-reference.md#linux-hosts--socket-receive-buffers) din referința CLI.

Pe o mașină fără interfață grafică, rulați backend-ul ca serviciu **sub contul dvs. de utilizator**, astfel încât un script să găsească unul în execuție (`connect_all()` nu pornește niciodată un backend):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Backend-ul rulează apoi ca și cum ați fi dumneavoastră: capturile pe care le scrie sunt ale dumneavoastră și utilizează `chloros-cli login`-ul dumneavoastră. `chloros-backend.service`-ul root este încă instalat pentru mașinile care l-au activat, dar fiecare fișier pe care îl scrie un backend root aparține utilizatorului root. Consultați secțiunea [Rularea backend-ului ca serviciu](cli-reference.md#running-the-backend-as-a-service-headless) din referința CLI.

Un backend rulat de serviciu este partajat și nu este oprit niciodată de un client: scripturile SDK și `chloros-cli` îl utilizează așa cum îl găsesc (`ChlorosLocal.shutdown_backend()` oprește doar un backend pe care `ChlorosLocal` l-a pornit el însuși), iar `GET /api/status` denumește unitatea ca `service`. Reporniți-l cu `sudo systemctl restart chloros-backend@$USER.service`. Actualizarea pachetului oprește serviciile backend în timpul instalării, astfel încât un script care rulează își pierde camerele și senzorii, apoi le pornește din nou pe toate cele activate; dezinstalarea pachetului le oprește și le dezactivează pe cele activate. Doar un singur backend rulează la un moment dat, așa că activați o singură unitate: a doua refuză să pornească și rămâne `failed` (stare de ieșire 98, nu se repornește), iar mesajul său `systemctl status` explică motivul. Pentru a comuta între unități, dezactivați-o pe cea pe care nu o utilizați.</id></sn>

### Pasul 2 — Instalarea Python SDK

**Programul de instalare Chloros include un pachet „wheel” SDK corespunzător.** Fiecare program de instalare pentru Windows și fiecare pachet .deb pentru Linux plasează pe disc un fișier `chloros_sdk-X.Y.Z-py3-none-any.whl` care corespunde exact versiunii GUI / CLI / backend. Nu este necesar să urmăriți PyPI pentru a rămâne sincronizat.

#### Windows

Programul de instalare rulează automat fișierul `pip install` cu pachetul inclus, utilizând Python-ul sistemului (se preferă lansatorul `py.exe`, dar se recurge la `python -m pip` dacă acesta nu este disponibil). Nu este necesară nicio acțiune — `import chloros_sdk` funcționează în mediul dvs. Python după o instalare reușită. Dacă pe sistem nu există Python, programul de instalare omite în mod silențios acest pas, iar GUI + CLI continuă să funcționeze.

#### Linux (.deb)

Ambele pachete .deb (x86_64 și Jetson) instalează SDK-ul pentru sistemul `python3`, astfel încât `import chloros_sdk` funcționează după instalare, fără pip și fără conexiune la rețea. Pachetele (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` și încărcătoarele de proiecte) se plasează în `/usr/lib/python3/dist-packages`, iar comanda `chloros-mcp` a SDK-ului (serverul [MCP](#mcp-server-ai-assistants)) în `/usr/bin`, toate aparținând pachetului: o actualizare le înlocuiește, iar dezinstalarea pachetului le elimină.

Cerințele proprii ale SDK-ului, `python3-numpy` și `python3-requests`, sunt **recomandate, nu obligatorii** (interfața grafică, CLI-ul și backend-ul sunt autonome): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` pe un Jetson) le instalează, în timp ce `dpkg -i` nu. Rezultatul instalării raportează versiunea SDK pe care `python3` o importă, orice cerință care lipsește, precum și un `chloros_sdk` aflat în altă parte pe calea de acces a `python3` care suprascrie copia pachetului (cum ar fi unul instalat anterior cu `pip install --user`).

Într-un virtualenv, care nu vede pachetele sistemului, instalați fișierul wheel inclus în pachet în acesta:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pentru implementările Jetson izolate (air-gapped), acest proces se desfășoară complet offline — SDK-ul și fișierul wheel se află deja pe disc.

#### PyPI public

Pentru gazdele care utilizează doar pip (fără pachetul desktop Chloros instalat; SDK-ul controlează atunci un backend pe o altă mașină, consultați [Modul backend la distanță](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI conține cea mai recentă versiune stabilă. Mențineți SDK-ul la aceeași versiune cu backend-ul Chloros pe care îl controlează (consultați [Versiuni și compatibilitate](#versioning--compatibility)). Programul auxiliar DAQ-M `DLS` necesită, de asemenea, `bleak`: `pip install "chloros-sdk[camera]"` (sau `pip install bleak`).

#### Verificare

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Este necesar un abonament Chloros+.** Toate apelurile SDK necesită o autentificare Chloros+ activă. Rulați `chloros-cli login user@example.com 'YourPassword'` o singură dată pe fiecare mașină; datele de autentificare sunt stocate în cache în `~/.chloros/`.

### Am nevoie de pachetul pentru desktop?

Pachetul pip **nu** este suficient pentru majoritatea fluxurilor de lucru. Iată ce necesită fiecare interfață SDK:

| Interfață SDK | Are nevoie de pachetul pentru desktop? | De ce |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Da**(pentru a porni unul)**/ Nu** (un backend deja accesibil) | Se utilizează un backend care rulează deja (cel al aplicației desktop sau un alt script pornit); în caz contrar, se pornește automat cel instalat: `/usr/lib/chloros/chloros-backend` (Linux) sau `<install folder>\backend\chloros-backend.exe` (sistem Windows; `C:\Program Files\Chloros\backend\` în mod implicit, un folder de instalare personalizat este citit din înregistrarea programului de instalare). Treceți `backend_exe=` pentru a-i da un nume la alegere. |
| `chloros-mcp` ([server MCP](#mcp-server-ai-assistants)) | **Da**(pentru a porni unul)**/ Nu** (un backend deja accesibil) | Construit pe `ChlorosLocal`, deci la fel ca rândul de mai sus. |
| `read_image_audit_tags` | **Nu** | Citește metadatele proprii ale imaginii procesate (numai biblioteca standard Python). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Da**(local)**/ Nu**(la distanță) | Clienți HTTP puri prin backend. Backend local → este necesar un pachet pentru desktop. Backend la distanță → `backend_url=`**prin intermediul unui tunel** (vezi Modul Backend la distanță — backend-urile livrate se leagă doar la loopback). |
| `ChlorosProject` / `open_project` | **Da** | Gestionează proiectele salvate prin intermediul backend-ului. |
| Clase LATTICE directe (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Da** | Necesită runtime-ul Arena SDK, care este inclus în pachetul pentru desktop, și OpenCV (`cv2`) în Python, pe care nici pachetul pentru desktop, nici SDK nu îl instalează (`pip install opencv-python`). În caz contrar, `CAMERA_AVAILABLE` este `False` la import. `DLS` are nevoie și de `bleak`. |

### Modul backend la distanță (gazdă numai cu pip, prin tunel)

> **Backend-ul instalat nu este accesibil prin rețeaua LAN.** Acesta ascultă
> numai pe loopback (`127.0.0.1` / `::1`), astfel încât `backend_url="http://<lan-ip>:5000"`
> **nu poate accesa un Chloros instalat**. Pentru a rula un backend pe o altă
> mașină, redirecționați manual portul loopback al acestuia și configurați SDK-ul să utilizeze
> tunelul:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Gazdele fără monitor / CI / robotice pot păstra o mașină cu instalarea completă a desktopului ca „server Chloros” și `pip install chloros-sdk` peste tot în rest — dar transportul între ele se face prin tunelul configurat de utilizator de mai sus, nu printr-o URL LAN directă.

> **`ChlorosLocal` pe o gazdă care rulează doar pip.** `ChlorosLocal(backend_url=BACKEND)` se conectează la un backend la care poate accesa fără o instalare desktop: caută un fișier binar local al backend-ului doar atunci când trebuie să pornească unul (nu răspunde nimeni și apare `auto_start_backend=True`). Până la versiunea 1.2.0, căuta mai întâi fișierul binar și genera eroarea `ChlorosBackendError` („Backend Chloros neidentificat…”) pe o gazdă care rulează doar pip, chiar și atunci când backend-ul era accesibil. Dosarele pe care le specificați (`process_folder`, `import_images`) sunt verificate pe această mașină și citite de backend, așa că, în cazul unui backend la distanță, acestea trebuie să indice același folder pe ambele, cum ar fi o unitate partajată.

### Flux de lucru exclusiv DAQ (gazdă care rulează doar pip)

`pip install chloros-sdk` nu poate controla singur un senzor DAQ: un senzor DAQ este întotdeauna controlat printr-un backend Chloros cu `connect_daq_sensor()`:

- **Senzor pe această mașină:** instalați aici pachetul desktop Chloros. `connect_daq_sensor()` își pornește backend-ul dacă nu rulează niciunul.
- **Senzor pe o altă mașină:** acea mașină are nevoie de pachetul desktop; accesați backend-ul acesteia prin tunel în [Modul Backend la Distanță](#remote-backend-mode-pip-only-host-via-tunnel). Backend-ul deschide senzorul pe propria sa mașină, astfel încât un DAQ-U să fie conectat la acel computer, un DAQ-M să se afle în raza de acțiune Bluetooth a acestuia, iar un DAQ-E să se afle într-o rețea la care acesta are acces.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

În orice caz, backend-ul aplică planul „Chloros+”: autentificați-vă pe mașina care rulează backend-ul (`chloros-cli login` sau aplicația desktop) cu orice nivel plătit (Copper sau superior). Un backend deconectat și un plan inferior nivelului Copper generează ambele codul `ChlorosConnectError`, care conține motivul specific al backend-ului (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Ghid de pornire rapidă

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Indexul API-urilor de nivel superior

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Prelucrarea imaginilor — `ChlorosLocal`

Clasa principală a pipeline-ului. Pornește backend-ul la construire (dacă niciunul nu răspunde și `auto_start_backend=True`), creează / configurează proiecte, monitorizează progresul, returnează rezumate după execuție.

### Constructor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metode

| Metodă | Descriere |
| --- | --- |
| `create_project(project_name, camera=None)` | Creează un proiect nou (opțional cu un șablon de cameră precum `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Deschide un proiect salvat (folderul acestuia sau numele acestuia din folderul de proiecte) și îl face proiectul deschis. Căile unui proiect mutat sunt redirecționate către folderul în care se află acum; `relink` denumește folderul în care se află acum celelalte fișiere lipsă. Returnează răspunsul backend-ului cu `source_files`; generează `ChlorosProjectFilesMissingError` atunci când nu se găsește niciunul. Consultați **Un proiect mutat** mai jos. |
| `import_images(folder_path, recursive=False)` | Importă imagini RAW/TIF/JPG/DNG **și `.daq` înregistrări de la senzorul de lumină**. Returnează `count` (imagini) și `scan_count` (înregistrări). Avertizează doar dacă folderul nu conține niciuna dintre acestea. |
| `export_light_sensor(daq=True, csv=True)` | Scrie fișierele calibrate `.daq` + `.csv` pentru fiecare înregistrare a senzorului de lumină din proiect, în `<project>/Light Sensor/`. Consultați [Înregistrări ale senzorului de lumină](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Setați butoanele de procesare. `indices` sunt nume de presetări (`NDVI`, `GNDVI`, `OSAVI`, …), fiecare fiind adăugat la proiect pentru benzile camerei respective; un nume care nu este o presetare sau pe care benzile nu îl pot forma (`NDRE` necesită RE și NIR, pe care nicio cameră nu le oferă), primește un nume de tip `UserWarning` și nu are imagine index. `daq_log_path` denumește o înregistrare a senzorului de lumină (un fișier de tip `.daq` sau `.csv`, sau un dosar care le conține) cu care se compară fiecare cadru, atât în LATTICE, cât și în Survey3, în locul înregistrărilor importate odată cu imaginile; dacă o cale lipsește sau nu conține nicio înregistrare, se utilizează cele importate. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Rulează pipeline-ul. Returnează `{"status": "complete", "async": False}`, plus o cheie `summary` atunci când backend-ul furnizează una — a se vedea [Rezumatul și sfaturile post-execuție](#post-run-summary--hints) — `export_check_failure` atunci când exporturile execuției nu au putut fi verificate și `image_failure` atunci când s-a pierdut o imagine. `wait=False` returnează `{"status": "started", "async": True}` simultan. |
| `get_config()` / `get_status()` / `status()` | Verificați starea backend-ului. |
| `logout()` | Deconectare: șterge datele de autentificare stocate în cache și eliberează slotul de dispozitiv al acestui computer (`device_deregistration` în rezultat). O execuție de procesare în curs nu este oprită: aceasta se finalizează în modul în care a început (`run_in_progress: true` și `run_mode` în rezultat), iar următoarea execuție utilizează procesarea standard. |
| `shutdown_backend()` | Oprește backend-ul dacă această instanță l-a pornit; un backend găsit în execuție este lăsat să funcționeze în continuare. Lăsarea unui bloc `with` sau colectarea de gunoi a instanței nu îl oprește: backend-ul continuă să ruleze pentru următorul script și se oprește singur după 30 de minute dacă nu este utilizat de nimic. Backend-ului i se cere să se oprească, așa că eliberează camerele și senzorii de lumină și finalizează mai întâi orice înregistrare; este oprit doar dacă încă rulează după 10 secunde. |
| `discover_cameras()` | Descoperă camerele LATTICE **prin backend-ul acestei instanțe** (`/api/camera/discover`). Returnează o listă de dicționare (`serial`, `model`, `ip`, …) — aceeași structură pe care o văd GUI-ul și CLI-ul. Listă goală dacă nu se găsește niciuna sau dacă backend-ul este inaccesibil. |
| `camera_capture(output_dir, format="tiff", **settings)` | Capturează un singur cadru**prin backend**(pornit automat de acest identificator), astfel încât acesta să beneficieze de aceeași pregătire ca și GUI-ul/CLI-ul (implicit 12 biți, reutilizare din pool, metadate de calibrare încorporate). Rezolvați ținta cu `serial=` sau `device_index=`; treceți `exposure`/`gain`/`pixel_format`/`preset` ca `**settings`. Returnează dicționarul de metadate vechi (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Generează cadre de previzualizare compuse prin suprapunere dintr-o cameră comună — client MJPEG ușor prin ruta `/api/camera/<serial>/stream-annotated` a backend-ului (zebra / grilă / reticul / histogramă / peaking / punct desenat pe partea de server). `decode=True` generează matrice BGR și necesită OpenCV (`cv2`), pe care SDK-ul nu îl instalează; `False` generează octeți JPEG neprelucrați. De asemenea, accesibil la nivel de proiect ca `ChlorosProject.stream(overlays=True)`. |

**O execuție în curs își păstrează proiectul.** În timp ce o execuție de procesare este în curs — a acestui script, a aplicației, a `chloros-cli process` sau a `chloros-mcp` — backend-ul refuză să creeze, să deschidă, să importe sau să modifice setările de procesare ale unui proiect (409 HTTP `PROCESSING_IN_PROGRESS`): `create_project()` și `import_images()` generează `ChlorosProcessingError`, iar `configure()` generează `ChlorosConfigurationError`, iar `process()` cu un mod diferit de cel curent generează `ChlorosProcessingError` (o execuție păstrează modul în care a început), fiecare cu mesajul backend-ului. O captură salvată în proiect se alătură acestuia la sfârșitul execuției (răspunsul său, `project_add`, indică acest lucru). Așteptați finalizarea rulării (`get_status()`), apoi continuați.**Un proiect mutat.** `project.json`-ul unui proiect denumește fiecare fișier sursă (fiecare imagine și înregistrare a senzorului de lumină) după calea sa completă, astfel încât mutarea folderului proiectului — sau a întregului folder „proiecte” pe o altă unitate — face ca acestea să rămână denumite după vechea locație. Fiecare deschidere remediază ce poate pe cont propriu: un fișier care se afla în folderul proiectului este căutat în folderul în care se află proiectul acum, iar fiecare cale găsită acolo este redirecționată și salvată (`project.json` este mai întâi copiat ca `project.json.before-relink-<date>-<time>`). O cale este redirecționată numai către un fișier care se află efectiv acolo. Fișierele care se aflau în afara folderului proiectului — o sesiune hub, o importare prin „Add Folder” — sunt găsite doar cu `relink=<the folder they are in now>` (sau orice folder de deasupra acestuia). `open_project()` returnează `source_files`: `total`, `found`, `missing`, `missing_roots` (folderele în care se aflau fișierele lipsă), `missing_outside_project` și `relink` (ce a fost redirecționat, de unde și unde). Un proiect din care lipsesc unele fișiere se deschide cu restul; unul în care nu se găsește niciun fișier nu se deschide, iar `open_project()` generează `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). `project open --relink` și `chloros-mcp` ale CLI`open_project` (`relink_folder`) fac același lucru.

Utilizați-l ca manager de context pentru o curățare garantată:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Înregistrări ale senzorului de lumină — calibrate `.daq` + `.csv`

Un DAQ-U / DAQ-M / DAQ-E poate fi înregistrat **fără** pachetul său de calibrare. Asta este
ceea ce fac în mod implicit înregistratoarele publice [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): înregistrează valorile brute ale senzorilor și marchează
fișierul astfel încât Chloros să preia calibrarea din fabrică a senzorului **după numărul de serie** — mai întâi din cache-ul local, apoi din MAPIR Cloud — și o aplică la import.

Chloros scrie rezultatul înapoi sub forma a două produse per înregistrare, sub
`<project>/Light Sensor/`:

| Produs | Ce este |
| --- | --- |
| `<name>_calibrated.daq` | Arhiva reprocesabilă — aceeași schemă ca o înregistrare în timp real, declarând acum pachetul care a generat-o. Reimportarea acesteia **nu** o calibrează a doua oară. |
| `<name>_calibrated.csv` | Iradianța spectrală în W/m²/nm pe grila de lungimi de undă proprie a senzorului, un rând pentru fiecare citire, plus coloane fotometrice (putere totală, lux fotopic/scotopic, PPFD și defalcarea sa în albastru/verde/roșu, lungimea de undă de vârf). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Numai pentru senzori fără pachet (DAQ-A).** Numărul brut de impulsuri spectrale ale senzorului — *nu* iradianța. A se vedea mai jos. |

`<name>` este numele propriu al fișierului de înregistrare. Două înregistrări din dosare diferite pot împărți
o singură valoare — citirile pe captură (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) salvate în
aceeași milisecundă de două camere — și fiecare primește totuși propriile produse: prima păstrează
`<name>`, iar următoarea adaugă folderele care o diferențiază **după** nume
(`<name>__<capture>_daq_calibrated.daq`), astfel încât data și ora înregistrării rămân în prim-plan.

`process()` efectuează acest export ca una dintre etapele sale. **Nu** necesită imagini:
un senzor de lumină zburând pe cont propriu reprezintă un flux de lucru de primă clasă, iar un astfel de proiect are zero
imagini prin natura sa.

**Înregistrările DAQ-A se exportă ca numere brute.** Familia DAQ-A este anterioară sistemului de pachete pe serie
și nu are niciun pachet de preluat — este calibrată pe teren în raport cu o
țintă de reflectanță, motiv pentru care nu a avut niciodată nevoie de unul. Aceste înregistrări se exportă
sub prefixul `_raw`, nu `_calibrated`: un nume de fișier diferit, nu un indicator
în interiorul fișierului, deoarece informația trebuie să rămână intactă atunci când este trimisă prin e-mail doar ca nume simplu. Antetul
Antetul `.csv` indică `raw spectral sensor counts (NOT irradiance)` și avertizează că
valorile sunt comparabile **în cadrul** fișierului — exact ceea ce utilizează calibrarea bazată pe țintă
pentru — și nu între senzori. Coloanele fotometrice dependente de putere (putere totală,
lux fotopic/scotopic, PPFD) returnează **NULL** în loc să fie integrate din numărări.

Un DAQ-U / DAQ-M / DAQ-E al cărui pachet pur și simplu nu a putut fi preluat este totuși **omisi**,
nu este scris în format brut: în acest caz, pachetul există, iar „reconectarea și reprocesarea” reprezintă un sfat real.

Înregistrările vechi **v1.01 / v1.02** (un DAQ-A-SD le scrie) nu conțin o epocă pentru fiecare citire,
ci doar ora de scriere a fișierului. Programul de potrivire imagine↔flux descendent încă le respinge — potrivirea unui
cadru cu un moment de scriere ar fi o eroare invizibilă — dar exportatorul le citește, iar
fișierul CSV afișează `clock=daq_created_on`, astfel încât produsul precizează pe ce ceas se află.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

O înregistrare al cărei pachet de calibrare nu poate fi preluat (offline sau un senzor fără
calibrare în fișier) este raportată sub `skipped` **cu motivul**. Nu este niciodată
salvat ca fișier „calibrat” care conține numărătoarea brută — conectați-vă la internet și
rulați din nou, iar exportul se va finaliza.

### Apeluri de progres

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Rezumat și sfaturi după rulare

La finalizare, `process()` preia `GET /api/processing-summary` și atașează corpul ca `result["summary"]`. Preluarea se face pe baza principiului „best-effort” și nu blochează niciodată o returnare reușită — dacă rezumatul nu este disponibil, `process()` revine la forma simplă `{"status": "complete", "async": False}`. Fiecare intrare din `summary["hints"]` — propoziții complete cu remedierea sugerată, de exemplu, de ce o execuție a produs zero rezultate — este, de asemenea, reemise ca un `UserWarning` în Python, astfel încât execuțiile cu 0 rezultate se autodiagnostică chiar dacă nu inspectați niciodată dicționarul:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` este partea lizibilă de către mașină:

| Cheie | Ce numără |
| --- | --- |
| `models` | Grupuri de camere în execuție. |
| `images_in_groups` | Imagini sursă din aceste grupuri. |
| `targets_found` | Ținte de reflectanță detectate. |
| `images_calibrated` | Imagini pentru care execuția a generat un produs de reflectanță. |
| `images_failed` | Imagini care au eșuat și nu au fost exportate. |
| `exported_files` | **Fișiere de produse imagine generate de execuție**: fiecare produs (debayering, previzualizare, radianță, reflectanță, indice, randări țintă), în orice format. |
| `reflectance_files` | Produsele de reflectanță dintre acestea (`Reflectance_Calibrated_Images`), indiferent dacă au fost calibrate de o țintă sau de iradianța senzorului de lumină (DAQ). O execuție fără țintă care totuși a generat date de reflectanță menționează acest lucru într-un indiciu; doar unei execuții care nu a generat nimic i se semnalează că îi lipsesc rezultatele de reflectanță. |
| `clamp_masks_written` | [Măștile de limitare a amestecării](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) menționate lângă ele — sunt elemente însoțitoare, nu produse, deci nu se regăsesc în `exported_files` (CLI-ul le numără separat în același mod). |
| `captures_left_out` | Cadrele brute din folderul `captures` al proiectului pe care această execuție nu le-a capturat. O execuție așteaptă la început ceea ce a fost capturat înaintea ei, deci această valoare este 0, cu excepția cazului în care o captură a fost salvată în timpul execuției sau timpul de așteptare a expirat. `summary["captures_left_out_note"]` indică ce trebuie făcut cu acestea. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Înregistrări ale senzorului de lumină, numărate separat în mod intenționat — provin dintr-o etapă diferită și există pentru rulări fără imagini deloc, așa că includerea lor ar face ca o rulare doar cu DAQ să pară că a exportat imagini. |

Alături de acestea: `summary["output_dirs"]` (fiecare director în care s-a scris),
`summary["light_sensor_export"]`, `summary["stopped"]` (valabil atunci când utilizatorul a întrerupt
rularea, astfel încât numărările parțiale să nu fie interpretate ca o rulare finalizată cu producție insuficientă) și
`summary["groups"]` (defalcarea pe grupuri).

`exported_files` este înregistrat de pipeline **pe măsură ce scrie**, nu este extras ulterior din
obiectele de imagine ale proiectului. Strategiile paralele și GPU își construiesc propriile obiecte de imagine
(în subprocese de lucru pentru căile GPU), astfel încât scanarea veche raporta
`0 file(s) written` pentru fiecare astfel de execuție și apoi emitea indicația „zero-exports” — în cazul execuțiilor
în care totul funcționase corect. Dacă creați un script bazat pe acest număr, o execuție paralelă corectă
raportează acum un număr diferit de zero.

Omisiunile senzorului de lumină raportează motivul stabilit efectiv de cititor pentru fiecare fișier — o
schemă ilizibilă, un pachet lipsă, o eroare de scriere — **deduplicat**, astfel încât douăzeci de fișiere
omise dintr-un singur motiv sunt interpretate ca un singur motiv, în loc de douăzeci de repetări ale acestuia.

> **`process()` nu generează o eroare atunci când o execuție nu produce imagini.** Acesta este unul dintre cele două puncte în care SDK-ul și
> CLI-ul diferă în mod deliberat: `chloros-cli process` tratează „produsele au fost solicitate, dar niciunul nu a fost
> scris” ca pe un eșec și se închide cu un cod de ieșire diferit de zero, în timp ce SDK-ul se întoarce la normal și raportează
> starea prin intermediul `summary` / hints. Dacă pipeline-ul dvs. ar trebui să se oprească la o execuție goală, verificați-l
> personal — inspectați `summary` (sau numărați fișierele din folderul proiectului) în loc să vă bazați pe
> absența unei excepții. Cauzele obișnuite sunt un folder de intrare care nu a fost recunoscut ca
> captură și produse omise ca fiind inaplicabile pentru camerele prezente (de exemplu, radianța de la camerele exclusiv RGB
>).
>
> `process()` nu se declanșează nici atunci când exporturile unei rulări nu au putut fi verificate: emite o avertizare și setează
> `result["export_check_failure"]`. Produsele sunt scrise. Pentru a opri procesul în acest moment, verificați acea cheie. O execuție care
> a pierdut o imagine se comportă la fel: afișează un avertisment și setează `result["image_failure"]`, iar `chloros-cli process`
> se închide cu o valoare diferită de zero în acest caz.

### Funcții de utilitate

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Fiecare rulează în cadrul propriului `with ChlorosLocal(...)`; un backend pe care a trebuit să-l pornească continuă să ruleze când se întoarce, pentru următoarea apelare, și se oprește automat după 30 de minute dacă nu este utilizat de nimeni. Niciuna dintre ele nu acceptă `auto_start_backend`, `radiometric_output` sau `daq_log_path`; utilizați `ChlorosLocal` și `configure()` pentru acestea. `process_lattice_capture()` nu acceptă `camera`, `debayer` sau `ppk`: omite detectarea panoului și utilizează debayerul standard.

### Valori acceptate

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Ieșire radiometrică (pipeline multispectral LATTICE)

Nivelul de export multispectral LATTICE (M3C/M3M) al pipeline-ului `process` — `reflectance` (implicit), `radiance`, `sensor-response` sau `all` (fiecare mod aplicabil pentru fiecare imagine) — stabilește **„Export radianță”**și**„Export reflectanță”****: `radiance` exportă doar radianța, `reflectance` doar reflectanța, `sensor-response` niciuna (doar produsele debayered și de previzualizare), iar `all` le exportă pe amândouă. Produsele debayered și de previzualizare nu pot fi modificate la acest nivel. `configure()` are un cuvânt-cheie dedicat pentru aceasta:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Soluția avansată de rezervă — scrierea produselor prin `custom_settings` — funcționează în continuare, dar rețineți că aceasta înlocuiește întregul bloc de setări (vezi avertismentul de mai jos). Scrie cheile `"Export <product>"`: setările proprii ale unui proiect, `"Export radiance"` / `"Export reflectance"` ale unui proiect au prioritate față de cheia mai veche `"Radiometric output"`, astfel încât acea cheie, luată singură, nu modifică nimic.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (implicit) împarte radianța camerei la **semnalul descendent DAQ corelat temporal**, determinat automat dintr-un fișier înregistrat `.daq` (DAQ-U/M/E)**sau un `.csv` nativ DAQ-M**găsit alături de imagini; orice pachet de calibrare per cameră sau DAQ care lipsește local este**preluat automat din AWS** la prima utilizare. Interfața CLI expune acest lucru conformcomutatoare de tip produs pe `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**A cui este citirea?**O captură salvată de un `.daq` reprezintă citirea camerei (camerelor) pe care o descrie — numită într-un fișier sidecar `<file>.daq.cameras.json` aflat lângă ea (capturi unice, sincronizate și în serie; o serie de capturi brute menționează citirea fiecărui element în manifestul său) — și se potrivește**numai cu imaginile de la acele camere**. Prin urmare, o cameră autonomă fără o citire proprie nu mai împrumută DAQ-ul altei camere; asociați-i un DAQ sau înregistrați una. Înregistrarea proprie a unui DAQ (`chloros-cli daq pool-record` sau o înregistrare inițiată în interfața grafică), sau orice fișier `.daq` fără acel fișier sidecar (inclusiv capturile mai vechi), servește în continuare toate camerele.**Camerele OCN în lumina soarelui.** Fiecare pachet de calibrare OCN este creat sub lumină de bandă îngustă, cu un LED pe bandă, în timp ce lumina soarelui amestecă benzile. Chloros 1.3.0 aplică o corecție globală bazată pe lumina soarelui după separarea benzilor, ceea ce îmbunătățește produsele OCN de radianță, reflectanță și indice în lumina soarelui. Aceasta se aplică oriunde are loc separarea benzilor — `process()`, capturi, `grab()` / `frame_stream()` și vizualizarea în timp real — pentru pachetele OCN care o suportă; camerele RGN rămân neschimbate. Imaginile OCN capturate cu o versiune anterioară beneficiază și ele de această funcție atunci când sunt procesate pe un computer care păstrează calibrarea cu care au fost realizate (Chloros păstrează calibrarea camerei odată ce aceasta a fost conectată); în caz contrar, ele sunt procesate ca și până acum. Prin urmare, valorile de radianță, reflectanță și indice OCN diferă de cele din versiunea 1.2.0.

> `custom_settings` **înlocuiește** întregul bloc de setări calculate (acesta ocolește, conform proiectării, celelalte cuvinte-cheie și validarea din `configure()`). Când îl utilizați, includeți fiecare cuvânt-cheie `Project Settings` care vă interesează, așa cum se arată în exemplul de mai sus.

#### Citirea pixelilor de indice ( NDVI și ceilalți indici de vegetație)

**Un raster de indici cu numere întregi reprezintă o extindere a contrastului pentru fiecare imagine în parte, nu o hartă în intervalul −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` și `JPG (8-bit)` — inclusiv fișierul TIFF de 8 biți în care este salvată o captură de 8 biți — stochează

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

unde `IndexMin` / `IndexMax` sunt **valorile finite minime și maxime ale indexului propriu al acelui cadru**. DN 0 nu este −1, iar scala completă nu este +1.

Chloros marchează ambele puncte finale, astfel încât maparea să rămână inversabilă:

| Etichetă | Semnificație |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — convenția de mai jos |
| `Chloros:IndexMin` | valoarea indexului stocată ca DN 0 |
| `Chloros:IndexMax` | valoarea indexului stocată ca scală completă |

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

> **`IndexMin == IndexMax`** înseamnă că fiecare pixel al acelui cadru avea aceeași valoare de index. Rasterul este umplut la jumătatea scalei și nu există nimic cu care să se împartă — se ia punctul final.

> **Lipsa etichetelor `Chloros:Index*` înseamnă că nu există o mapare specificată.** Un export LUT este o redare colorată ai cărei pixeli sunt culori în gradient corelate cu fereastra de prag a LUT-ului dvs., iar un cadru fără pixeli cu indice finit nu are un interval de specificat. Nu presupuneți existența unei mapări în niciunul dintre cazuri — aceeași regulă ca și în cazul lipsei unei etichete `PixelScale`.**Deoarece întinderea se aplică pe fiecare imagine în parte, două cadre dintr-un zbor nu sunt comparabile ca DN**— fiecare are propriile sale puncte finale, astfel încât același NDVI se încadrează în valori DN diferite în imagini diferite. Pentru lucrări cantitative la nivelul unui zbor (statistici zonale, praguri fixe, serii temporale), exportați**`TIFF (32-bit, Percent)`**: rasterele index sunt scrise acolo ca**valori index** de tip float32, fără a fi necesară rescalarea sau adăugarea de etichete. În ciuda denumirii formatului, produsele index nu sunt ×100 — acea scalare se aplică doar reflectanței de tip întreg.

#### Masca de limitare a separării spectrale — pe ce pixeli a fost limitată o bandă

Separarea spectrală 3×3 poate face ca o bandă să devină negativă — acolo unde lumina conține abia acea bandă sau unde scena se află în afara zonei descrise de matricea de separare a camerei. Aceasta nu este o valoare reală, așa că exportul o fixează la 0 — iar vizualizarea în timp real afișează același 0. O bandă poate fi limitată la câțiva pixeli sau la aproape toți; masca înregistrează ambele situații. `Chloros:Band{R,G,B}UnmixClampFraction` indică cât dintr-un cadru s-a întâmplat acest lucru; masca de limitare indică **care pixeli**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— într-un folder `clamp_masks/`**lângă** folderul produsului, niciodată în interiorul acestuia (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — pentru fiecare produs de radianță, reflectanță și indice în care cel puțin un pixel a fost limitat. `uint8` cu o singură bandă, pe propria grilă de pixeli a produsului (orientare, ROI, binning și deformarea de aliniere a matricei incluse). Fiecare mască specifică camera produsului său (`Make` / `Model` / serial), ora capturii și etichetele obiectivului, și enumeră benzile pe care le reprezintă biții săi (`XMP-Camera:BandName`, bitul 0 primul); marcajele `ImageDescription` și `Software` o identifică ca mască de fixare, iar aceasta nu conține nicio calibrare.
- **Biți**: bitul 0 = banda din canalul R al produsului (`Chloros:BandRName`), bitul 1 = G (`BandGName`), bitul 2 = B (`BandBName`). Un bit setat înseamnă că valoarea pixelului din banda respectivă a fost limitată la 0 — tratați-l, precum și orice indice construit pe baza acestuia, ca având un nivel scăzut de încredere.
- **`Chloros:UnmixClampMask`** este înscris pe fiecare produs care a rulat procesul de separare: calea măștii relativă la propriul folder al produsului (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, întotdeauna `/`-separat — alăturați-o folderului imaginii pentru a deschide masca), sau `none` când niciun pixel nu a fost limitat. Lipsa etichetei înseamnă că produsul nu a rulat niciodată procesul de separare (camere RGB sau monocrome, niveluri raw/debayered).
- Rezultatele Capture îl includ ca **`clamp_mask`** al cadrului (calea măștii, sau `None` când nimic nu este limitat) — în rezultatele `CameraSession.capture()`, cadrele `ArraySession.capture()`în cadrele `CaptureResult` și în răspunsurile de captură HTTP. Versiunea offline listează măștile ca `clamp_masks`.
- Stiva multibandă a unui index live (`/api/camera/cli-array-index` cu `save_multiband`) primește o mască pentru fiecare cameră sursă, în propriul `clamp_masks/` al folderului de ieșire — consultați referința CLI`lattice index`. Un videoclip nu are mască: rezultatul compilării offline raportează în schimb fracția de clampare a fiecărui cadru.

Deoarece niciun dosar de produs nu conține o mască, la niciun nivel de adâncime, un dosar de produs poate fi transmis software-ului de fotogrammetrie așa cum este — inclusiv subdosarele — iar acesta preia doar imaginile. Transmiteți-i dosarul de produs, nu folderul de deasupra acestuia. Funcția de import a Chloros omite măștile.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect pentru camerele LATTICE

Sesiuni backend persistente pentru hardware în timp real. Se utilizează aceleași puncte finale ca și interfața grafică, astfel încât comportamentul este identic în SDK, CLI și GUI.

### O singură cameră — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

Expunerea, amplificarea și ținta setate aici sunt păstrate ca fiind ale tale: controlul automat al expunerii de la Chloros (inclusiv expunerea măsurată de DAQ) nu le modifică. `cam.set_settings(exposure_auto="Continuous")` (sau `gain_auto=`) returnează expunerea (amplificarea), iar `cam.set_settings(target_brightness_auto=True)` restabilește ținta la sistemul automat de țintire al lui Chloros.

#### `connect_camera()` Semnătură

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Se întoarce cu fluxul pornit.** Backend-ul așteaptă, în limite (aproximativ 15 s pentru pornirea fluxului, apoi 10 s pentru primul său buffer complet — nu pentru primul cadru de previzualizare, care urmează după aproximativ 1,5 s), așa că un `status()` apare imediat după ce se afișează `streaming`. `connect_stream` al sesiunii este blocul `stream` al răspunsului: `state` `streaming`, `no_frames` (fluxul a pornit și nu a transmis nimic — așa indică `note` și, pentru o cameră care utilizează pachete jumbo, să se reconecteze cu o dimensiune a pachetului de 1500 de octeți), `starting` sau `start_failed`, cu `waited_s`. `None` pentru o cameră care era deja conectată. O previzualizare (`get_camera_preview`, `/api/camera/jpeg/<serial>`) solicitată imediat după așteaptă, limitat (4 s), primul cadru de previzualizare al camerei în loc să răspundă „Niciun cadru disponibil încă”; o cameră al cărei flux nu a transmis nimic (`no_frames` sau armată pentru un declanșator hardware) răspunde totuși imediat.
- **Dimensiunea pachetului o poți stabili tu.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` deschide camera cu pachete de 1500 de octeți, fără a fi testată (`chloros-cli lattice cam-connect --packet-size standard` trimite același lucru).
- **O singură conexiune la un moment dat menține grupul de camere**: deschiderea, testarea dimensiunii pachetului, o primă descărcare de calibrare (aproximativ 70 s pentru o cameră nouă pe acest computer), începerea fluxului. O cerere care necesită o cameră între timp (`read_nodes`, `set_settings`, o captură, o a doua conexiune) așteaptă cel mult 5 s, apoi eșuează cu codul HTTP 503 `CAMERA_POOL_BUSY`, indicând camera care se conectează (`ChlorosConnectError`); întreabă din nou peste câteva momente. Listările nu așteaptă niciodată: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` și `CameraSession.status()` răspund imediat. O cameră în curs de conectare este listată sub răspunsul grupului `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — și `elapsed_s`), iar `status()` indică `connecting: True` cu `phase`; celelalte camere sunt citite în timp real când grupul este liber, altfel se răspunde pe baza ultimei citiri (`live: False`).

#### Metodele `CameraSession`

| Metodă | Descriere |
| --- | --- |
| `status(timeout=15.0)` | Starea în timp real a camerei, așa cum o citește `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` sau `stopped`), `device` (model, serie, firmware, temperatură, `is_mono`) și setările sale curente. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Citește nodurile GenICam; returnează `{nodes, errors, enums, device, was_in_pool, in_array_id}`, plus `frame_values` și `notes` în timp ce se execută expunerea automată (vezi mai jos). |
| `set_settings(**kwargs)` | Scrie noduri după nume descriptiv (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Un `exposure_time` (`gain`) reprezintă o expunere manuală (amplificare): atunci când se introduce fără `exposure_auto` (`gain_auto`), modul automat trebuie mai întâi dezactivat, astfel încât valoarea să nu fie respinsă de un mod automat activ. Rezultatul, `applied`, păstrează ceea ce a înregistrat camera; o scriere refuzată de cameră se află în `errors` și nu este păstrată. Eroarea specifică intervalul pe care camera îl acceptă în prezent (o valoare maximă `ExposureTime` urmează rata de cadre a camerei), iar modul automat care a fost dezactivat doar pentru valoarea respinsă este reactivat; `restored` îl listează (de exemplu, `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Capturează un **singur** cadru. Returnează un `CaptureResult`: o listă de dicționare cu metadate ale cadrelor (câte unul pentru fiecare nivel salvat), cu `.skipped`, `.failed`, `.daq_files` și `.daq_matches`. (Captura în rafală/multi-cadru a fost eliminată — apelați `capture()` într-o buclă dacă aveți nevoie de o serie.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Capturi consecutive (`interval_s=0`) sau la intervale de timp de la această cameră, pentru capturi `count` și/sau `duration_s` (cel puțin una, astfel încât să se încheie), așa cum face `ArraySession.capture_repeated()` pentru un șir. Returnează lista de `CaptureResult`-uri; se oprește după o captură care raportează un disc plin. `on_capture(i, result)` se execută după fiecare; celelalte cuvinte-cheie sunt redirecționate către `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Eliberare din grup, camera este readusă la starea în care a fost găsită la conectare (cablarea declanșatorului, PTP, setări); `keep_settings=True` își salvează setările în propria memorie, iar `clear_settings=True` resetează acea memorie (vezi [Memoria camerei](#camera-memory--keep_settings--clear_settings)); returnează `camera_memory` atunci când se solicită oricare dintre acestea. O cameră care este membru al unui array eliberează întregul array. Nu efectuează nicio operațiune dacă ne-am atașat la o sesiune deja deschisă. Returnează odată ce backend-ul confirmă eliberarea sau când nu mai este nimic de eliberat (camera a dispărut deja sau nu rulează niciun backend); o eliberare care a depășit timpul de așteptare sau care a fost refuzată de backend generează `ChlorosConnectError`, deoarece camera ar putea fi încă reținută, iar următoarea conectare ar intra în conflict cu aceasta. Ieșirea din blocul `with` nu generează niciodată o eroare. |

Atribute: `serial`, `device` (înregistrarea de descoperire a camerei) și `backend_url`.

`capture()` controale de export (același model ca și matricea + GUI):

- `processing` / `levels` — `processing="all"` salvează toate tipurile de export aplicabile; `levels=["raw","radiance"]` salvează doar acelea (suprascrie `processing`). Omiteți ambele pentru setarea implicită a backend-ului.
- `daq` — citirea DAQ a capturii utilizate este salvată implicit ca fișier auxiliar `.daq`, incluzând o captură doar în format brut, astfel încât cadrul să poată fi reprocesat ulterior în reflectanță/indice. `daq=False` o omite atunci când niciun nivel nu o utilizează. `force_daq=True` este forma veche a valorii implicite. Fără DAQ, fără `.daq`.
- `daq_tolerance_ms` — cât de departe de expunere poate fi citirea și să se încadreze încă în toleranță (implicit 500). Se schimbă doar verdictul.

#### `read_nodes()` în regimul de expunere automată — `frame_values` și `notes`

În timp ce `ExposureAuto` (`GainAuto`) rulează, nodul `ExposureTime` (`Gain`) citește **ultima valoare scrisă**, nu ceea ce face camera în acel moment. `nodes` păstrează în continuare acea valoare a nodului. Când solicitați `ExposureTime` sau `Gain` de la o cameră conectată al cărei mod automat nu este `Off`, răspunsul adaugă:

- `frame_values` — valoarea conținută de cel mai recent cadru al camerei, sub numele nodului (`ExposureTime` în µs, `Gain` în dB);
- `notes` — o propoziție pentru fiecare astfel de nod, care menționează ambele valori.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Ambele chei lipsesc atunci când modul automat este `Off` (nodul reprezintă atunci) și pentru o cameră care nu este conectată la backend. Pentru un tablou, `ArraySession.status()["member_ae"]` furnizează simultan valorile cadrelor fiecărui element.

#### Limita maximă de expunere măsurată de DAQ (HTTP)

Cu un senzor de lumină DAQ conectat, Chloros limitează expunerea × amplificarea camerei în funcție de lumina naturală măsurată, astfel încât un panou de referință alb să nu fie supraexpus. Limita se aplică numai cât timp o țintă de calibrare se află în câmpul vizual al camerei respective (văzută în cadrul acesteia în ultimele 10 secunde); în absența unei ținte în câmpul vizual, se activează în schimb expunerea automată obișnuită a camerei, cu propria protecție împotriva supraexpunerii, astfel încât restul scenei să nu fie menținut mai întunecat decât este necesar. Nu există un wrapper SDK; citiți sau modificați această setare pentru fiecare cameră în parte folosind `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (corpul POST `{"enabled": false}` o dezactivează pentru camera respectivă). Răspunsul conține `enabled`, `cap_us` (limita maximă în timp real, `None` când funcția este inactivă, inclusiv atunci când nu se află nicio țintă de calibrare în câmpul vizual), `k_exp_g` (constanta cu care se calculează limita superioară) și:

| Câmp | Semnificație |
| --- | --- |
| `k_exp_g_source` | A cărei constantă este utilizată: `camera` — cea proprie camerei, calculată pe baza punctului de laborator din pachetul său de calibrare (camere RGB); `fleet` — constanta flotei, utilizată atunci când pachetul nu conține un punct de calibrare; `override` — o suprascriere de câmp setată la nivel de backend. |
| `k_exp_g_fleet` | Constanta flotei, pentru comparație. |

### Matrice sincronizată — `ArraySession` (Smart-Prep)

`connect_array` este **punctul de pornire recomandat** pentru configurațiile cu mai multe camere. Acesta execută în fundal fluxul complet Smart-Prep prin interfața grafică (GUI):

1. **Analiza rețelei**(`/api/camera/array/recommend`) — configurează**senzorul complet** (fără binning, fără decupare ROI) și permite ca rata de cadre să scadă la nivelul oferit de acest host și de rețea, proiectată în funcție de formatele de pixeli și de modul de împachetare pe care Connect le va utiliza. Cadrele nu sunt niciodată reduse pentru a menține rata ridicată: transmiteți un `width` mai mic / `height` sau `binning=2` / `4` pentru a schimba rezoluția în favoarea ratei de cadre (lista de la punctul final`alternatives` listează aceste opțiuni împreună cu fps-urile proiectate).
2. **Selectare automată a nivelului** — `sim-capture-sim-emit` (fiecare cameră expune la același declanșator, emițând ritmat pentru a partaja cablul, rata de cadre scăzând pe măsură ce partajarea se reduce) pe orice rețea pe care camerele pot fi sincronizate pentru a partaja; `slip-emit-and-capture` (camerele se alternează pe canal, nu sunt expuse simultan) numai atunci când nu pot face acest lucru.
3. **PTP activat**implicit — marcajele de timp între camere se sincronizează pe un ceas comun cu o precizie de**~1 ms**, [ceasul principal PTP](#ptp-grandmaster--time_sync_status) al backend-ului. Expunerea simultană este asigurată de declanșatorul hardware M8 (**< 100 µs** între module), nu de PTP: PTP aliniază *marcajele de timp*, nu expunerile.
4. **Selectare automată a formatului de pixeli pentru fiecare cameră** — monocrom → `Mono12`, color → `BayerRG12`.
5. **Inițializarea AE** — înregistrează starea actuală a AE pentru fiecare cameră, astfel încât conectarea să nu reseteze expunerea în timpul funcționării.
6. **Configurarea declanșării GPIO** — `connect_array` activează fiecare cameră (`TriggerMode=On`, `TriggerSource=Line2`), astfel încâtsă acționeze camerele slave prin cablul M8. Aceasta este o etapă specifică doar pentru matrice: o singură cameră deschisă cu `LatticeCamera` funcționează în schimb în mod liber. Eliberarea matricei (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) readuce fiecare cameră la funcționarea liberă (`TriggerMode=Off`), toate celelalte setări rămânând așa cum au fost găsite la conectare: setările declanșatorului și ale liniei de sincronizare (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; linia de sincronizare a camerei principale revenită la o intrare), PTP și setările camerei modificate în timpul sesiunii — cu excepția cazului în care `keep_settings=True` / `clear_settings=True` (vezi [Memoria camerei](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Atribute: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` și `packed_reason` (ultimele patru sunt `None` pe un mâner de la `attach_array`), `connect_stream` și `backend_url`. `connect_stream` este blocul `stream` al răspunsului la conectare, forma unei singure conectări plus `members` (`{serial: {state, why?}}`): `streaming` odată ce a sosit primul grup sincronizat al matricei (backend-ul îl așteaptă, limitat la aproximativ 10 s), sau `no_frames` cu ultimul eșec de preluare al fiecărui membru ca `why` și, pentru pachetele jumbo, rezervele de 1500 de octeți din `note`. `None` pe un identificator `attach_array`.

#### Semnătura `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` reprezintă durata de așteptare a apelului pentru conectare și este trimis către
backend ca termen limită. O conexiune care nu s-a realizat până atunci este anulată: fiecare cameră pe care a
deschis-o este readusă la starea în care a fost găsită și închisă, niciun array nu este conectat, iar
apelul generează `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, indicând
etapa la care ajunsese conexiunea). Așadar, un apel care a generat o eroare nu lasă niciodată o matrice
conectat în urma sa. O matrice de 9 camere se conectează în aproximativ un minut; imediat după
pornirea camerelor, poate dura câteva minute. O matrice care este activă, dar nu are încă
un grup de cadre sincronizate, este returnată cu `connect_stream["state"]`
`no_frames`. Jurnalul backend-ului conține o linie de cronometrare pentru fiecare etapă de conectare
(`[array-connect <id>] <step>: <s> s …`) și o ultimă linie care indică cea mai lentă.

Valori `force_tier`:
- `"sim-capture-sim-emit"` — simultanitate adevărată (toate camerele declanșează pe aceeași flancă de ceas).
- `"sim-capture-ftd-stagger"` — eșalonare flexibilă în domeniul timpului (camerele emit la momente ușor decalate, astfel încât pachetele se serializează pe cablu).
- `"slip-emit-and-capture"` — captură secvențială per cameră (fără sincronizare temporală). Se selectează automat numai atunci când camerele nu pot fi sincronizate suficient de lent pentru a partaja linia simultan.

`wire_ceiling_mbps` suprascrie **bugetul de lățime de bandă susținută al gazdei** în MB/s — singura
valoare de care depinde întreaga alocare a matricei. Lăsați-l la `None` pentru a utiliza valoarea detectată automat. Reduceți-l atunci când matricea raportează cadre corupte GVSP: valoarea automată este derivată
din rata de legătură anunțată de placa de rețea, care supraestimează adaptoarele USB, benzile PCIe subțiri și
structurile partajate aglomerate — iar această supraestimare se manifestă sub forma unor cadre corupte, mai degrabă decât ca o
legătură vizibil lentă. Valoarea este stocată în blocul de captură al matricei din proiect, astfel încât o
redeschidere sau un `connect_array` ulterior o restabilește ca orice altă setare a matricei.
Consultați [Starea matricei](#array-health--which-subsystem-is-losing-frames).

#### Transport comprimat pe 12 biți (`packed=`)

La capacitate maximă a senzorilor, rata de cadre a unei matrice este determinată de viteza cu care fiecare cameră poate trimite
cadrele sale. Formatele pe 12 biți (`BayerRG12`, `Mono12`) sunt transmise în mod normal sub formă de cuvinte de 16 biți;
transportul compactat trimite aceiași 12 biți în 1,5 octeți pe pixel și
îi decomprimă pe gazdă, astfel încât fiecare cadru necesită cu 25% mai puțin timp pentru a fi transmis — aproximativ
**33% mai multe cadre pe secundă la capacitate maximă a senzorului** (măsurat: 10,7 → 14,0 fps pe o
rețea de 9 camere). **Datele salvate sunt identice**: aceleași valori, aceleași fișiere de 16 biți, aceleași metadate.

- `packed="auto"` (implicit) comprimă doar atunci când este util **și** gazda își poate
  permite decomprimarea: la conectare, backend-ul își măsoară propria viteză de decomprimare,
  numărul de nuclee CPU, încărcarea curentă și lățimea de bandă disponibilă. Un computer desktop cu nuclee disponibile
  comprimă la capacitate maximă a senzorului; o gazdă ocupată sau cu putere redusă (computere cu o singură placă,
  controlere încorporate) transmite fluxul necomprimat. ROI-urile mici și cadrele grupate au de câștigat foarte puțin.
- `packed="off"` (sau `False`) nu comprimă niciodată — pentru a exclude această opțiune în timpul diagnosticării,
  sau pe o gazdă care trebuie să-și păstreze CPU-ul pentru alte sarcini.
- `packed="on"` (sau `True`) comprimă întotdeauna formatele de 12 biți.
- Orice altă valoare generează `ValueError`. Formatele de 8 biți și 16 biți nu sunt niciodată
  comprimate.

`analyze_array_network(..., packed="auto")` ia aceeași decizie,
astfel încât fps-ul său reprezintă rata la care va rula matricea (răspunsul conține
`packed_transport` și `packed_reason`); valoarea implicită a `pixel_format` este cea
aplicată de „connect” (Mono12 / BayerRG12), nu 8 biți.

Decizia și motivul acesteia se regăsesc în sesiune:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` aplică aceeași suprascriere tuturor
matricilor dintr-un proiect (implicit `None`, ceea ce înseamnă `"auto"`: un proiect nu salvează nicio
setare de împachetare pentru fiecare matrice).
Pentru o setare implicită la nivel de mașină, setați `CHLOROS_ARRAY_PACKED12=off` (sau `on`) în
mediul backend-ului; o setare explicită `packed="on"` / `"off"` are totuși prioritate.
`/api/camera/array/status` raportează `packed_transport` și `packed_decision`.

#### Frecvențe de cadre preconizate

Frecvențe de cadre măsurate ale matricei (sincronizate hardware, 12 biți; senzor complet = 2048 × 1536). Utilizați-le pentru a verifica corectitudinea `status()["actual_fps"]`; `analyze_array_network()` estimează rata pentru propria dvs. gazdă și rețea.

| Configurație | Rata de cadre a matricei |
| --- | --- |
| Senzor complet, placă de rețea gazdă PCIe 10 GbE, transport comprimat (`packed="auto"` pe un computer desktop) | ~14 fps |
| Senzor complet, placă de rețea gazdă PCIe 10 GbE, fără comprimare (`packed="off"`) | ~10,5 fps |
| Senzor complet, 8 camere pe un adaptor USB 10 GbE | ~3,8 fps |
| Senzor complet, legătură gazdă 1 GbE | 1,2–1,7 fps |
| Binning 2 × 2 (`binning=2`), placă de rețea gazdă PCIe 10 GbE | ~30–33 fps |

- Pe o legătură PCIe 10 GbE, **camerele** reprezintă limita la senzor complet, nu rețeaua: 4 și 8 camere funcționează la aceeași rată.
- **`target_fps` reprezintă rata de declanșare, iar matricea nu funcționează niciodată mai repede.** `target_fps=1` face ca matricea să funcționeze la 1 fps; `None` (setarea implicită) o face să funcționeze la cea mai mare rată pe care o suportă gazda și rețeaua. O rată care depășește această valoare este redusă la aceasta (vezi [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` din `4294` în `status()` nu este o viteză de legătură.** Aceasta înseamnă că Windows nu a putut citi viteza adaptorului (driverul a raportat cea mai mare valoare pe 32 de biți), ceea ce este tipic pentru adaptoarele Ethernet USB. Viteza utilizabilă a unui adaptor USB pe Windows este limitată la 200 MB/s, indiferent de situație (vezi [Starea matricei](#array-health--which-subsystem-is-losing-frames)); pentru optimizarea acestuia, consultați secțiunea „Adaptoare USB 10GbE” din referința CLI, sub [Configurarea și optimizarea plăcii de rețea gazdă](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **O limită USB se aplică și pe Linux: 400 MB/s pe x86_64, 200 MB/s pe NVIDIA Jetson.** Adaptorul este identificat prin sysfs (cu recurgere la `ethtool -i`), astfel încât un adaptor USB 5 GbE sau 10 GbE este limitat la această limită maximă, și nu la viteza nominală de 500 / ~1060 MB/s. Valoarea pentru x86_64 a fost măsurată pe Linux: același adaptor care corupe cadrele la viteze peste 200 MB/s pe Windows a rulat fără probleme o configurație cu 4 camere la orice limită de până la 480 MB/s. Pe NVIDIA Jetson, același adaptor a funcționat fără erori doar la 200 MB/s și a pierdut cadre intermitent la viteze mai mari. Dacă gazda dvs. absoarbe mai mult fără cadre `gvsp_corrupt`, măriți-l cu `wire_ceiling_mbps`. Pe Linux, unui adaptor conectat la un port USB 2.0 i se alocă 30 MB/s; conectați-l la un port USB 3.

#### Supra-abonare (limita minimă per cameră)

O matrice funcționează pe orice rețea; rata de cadre este cea care dictează. Reglarea ritmului de emisie simulată alocă fiecărei camere o parte egală din lățimea de bandă sigură împotriva coliziunilor, iar toate camerele expun în continuare la același declanșator; astfel, pe o conexiune lentă, cota — și rata de cadre — este mică: camerele de 8de 12 biți pe o rețea de 1 GbE fără cadre jumbo primesc câte 6,5 MB/s fiecare, aproximativ **1 fps**(aproximativ 1,3 fps comprimate). Cota este limitată la cea mai mică rată la care camerele pot fi sincronizate (`per_cam_floor_bps`: se citește din intervalul GevSCPD al fiecărei camere la conectare și nu scade niciodată sub 1 MB/s). Dacă rețeaua necesită o viteză mai mică decât aceasta, conexiunea revine la transmisia**serială**(`slip-emit-and-capture`): camerele se alternează la viteza maximă a legăturii, astfel încât fiecare cadru ajunge la destinație, dar elementele**nu sunt expuse simultan**— nivelul răspunsului și blocatorii (`SERIAL: ...`) indică acest lucru; transmiteți `force_tier="sim-capture-sim-emit"` pentru a refuza trecerea la modul de rezervă. Dimensiunea cadrului nu influențează partajarea, astfel încât**binning-ul și ROI-ul cresc rata cadrelor, dar nu modifică faptul că o matrice se încadrează**. Doar o legătură care nu poate transporta matricea la niciun ritm, simultan sau pe rând,**se suprasolicită** — modul de eșec este pierderea de pachete GVSP, nu o rată de cadre mai mică. Soluții: mai puține camere, cadre jumbo de la un capăt la altul sau o placă de rețea mai rapidă.

- Răspunsurile `analyze_array_network()` și `/api/camera/array/connect` conțin `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` și `per_cam_floor_bps`. Când `oversubscribed` este adevărat, proiecția **setează la zero câmpurile fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`), în loc să raporteze o rată înșelătoare, de tipul „lent, dar funcțional”.
- `POST /api/camera/array/connect` acceptă un parametru de corp `pin_resolution` (**numai HTTP — nu este un kwarg SDK**; `connect_array` nu îl expune). Este modul strict: o conexiune suprasubscrisă cu `pin_resolution` setat este**refuzată categoric** cu o eroare care menționează fiecare soluție. Fără acesta, conexiunea se realizează cu un avertisment că matricea va pierde pachete. În orice caz, conexiunea păstrează dimensiunea cadrului solicitat — nu grupează și nu decupează niciodată pachetele din proprie inițiativă.
- Numai pentru testare: setați `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` în mediul backend-ului pentru a reduce refuzul la un avertisment sonor — vă conectați oricum și acceptați pierderea de pachete.

#### Starea matricei — care subsistem pierde cadre

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) transportă un bloc activ `health` pe o
matrice conectată, reevaluat pe o fereastră rulantă de **10 secunde** . Acesta împarte pierderea de cadre
în cele două cauze care necesită remedieri opuse, în loc de o singură rată „incompletă” care
nu specifică niciuna dintre ele:

| Câmp | Ce înseamnă | Ce subsistem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pe serial) | Cadrul **a sosit și era defect din punct de vedere structural**— pierdere de pachete GVSP. |**Rețea**: lățime de bandă, ritm, inelul de recepție al plăcii de rețea, MTU |
| `never_arrived_rate_pct` (pe serial) | Cadrul **nu a sosit deloc**— camera nu s-a declanșat sau nu a transmis nimic. |**Declanșare / sincronizare**: cablu M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Cea mai slabă rată a camerei pentru fiecare. | — |
| `per_cam_rate_pct` | Rata combinată de incompletitudine pe cameră (ambele cauze împreună). | — |
| `stable_for_seconds` | Cât timp a rămas fiecare cameră sub 0,01 %. | — |

Alături de `health`, același raport indică valoarea totală a alocării:

| Câmp | Ce înseamnă |
| --- | --- |
| `wire_ceiling_mbps` | Bugetul de lățime de bandă susținută al gazdei, MB/s. |
| `wire_ceiling_source` | De unde provine acea valoare, în cuvinte — de exemplu, `USB-capped 200 MB/s (was theoretical 1062; …)` sau `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` atunci când `wire_ceiling_mbps=` l-a setat. |
| `nic_is_usb` | `true` pentru un adaptor USB Ethernet. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Interpretare:** o valoare diferită de zero pentru `gvsp_corrupt_rate_pct`, cu `never_arrived_rate_pct` la 0, înseamnă
că declanșarea și sincronizarea prin cablu sunt perfecte, iar 100 % din pierderi se află pe traseul rețelei — reduceți
`wire_ceiling_mbps` și reconectați. Modelul invers indică mai degrabă cablul de sincronizare sau
linia de declanșare.

> **`target_fps` nu este factorul determinant pentru cadrele corupte.** Ritmul GevSCPD este stabilit o singură dată la
> conectare, astfel încât reducerea ratei de declanșare modifică ciclul de lucru și nu
> rata de transmisie simultană a rafalelor. O reducere măsurată de 5× a cererii nu a produs nicio îmbunătățire, în timp ce
> reducerea limitei maxime a cablului de la 240 la 200 MB/s a dus aceeași configurație de la 10,4 % cadre corupte la
> 0,00 %.

> **O matrice în funcțiune nu se poate replanifica singură pe firmware-ul LATTICE (M3C / M3M).** Nu poate
> remedia singur această problemă; deconectați-l și reconectați-l, astfel încât selectorul de timp de conectare să se replanifice în funcție de
> noua limită maximă.

Un **adaptor Ethernet USB este limitat la 200 MB/s** de către sondă, indiferent de
denumirii de pe plăcuța de identificare: tabelul de eficiență care transformă rata de legătură într-o valoare susținută este
derivat din PCIe, iar o placă de rețea USB își anunță rata de legătură Ethernet, fiind totuși limitată de
magistrala USB și de driverul acesteia. Limita este absolută, nu o fracțiune — un adaptor USB 1 GbE
atinge ~80 MB/s și nu este afectat. Se aplică pe Windows și Linux (inclusiv Jetson);
consultați [Ratele de cadre așteptate](#expected-frame-rates) pentru a vedea unde a fost măsurată.

#### Metode `ArraySession`

| Metodă | Descriere |
| --- | --- |
| `status(timeout=10.0)` | Starea în timp real a matricei care rulează: `target_fps` / `target_fps_auto` (rata de declanșare și dacă controlerul de rată o deține), `target_fps_clamped_reason` (motivul pentru care o rată fixată este menținută la un nivel mai scăzut, altfel `None`), `actual_fps` (măsurată; 0 odată ce bucla de captare a renunțat sau s-a blocat), `recommended_fps`, `streaming` (False odată ce bucla de captare a renunțat — verificați acest lucru, nu doar `last_error`; rămâne „Adevărat” chiar și după o repornire a setărilor, fapt semnalat de `restarting`), `stalled` (Adevărat cât timp bucla de captare rulează, dar nu a publicat niciun grup de cadre timp de aproximativ zece perioade de cadre, 2–30 s; `last_error` indică acest lucru abia după 60 de secunde), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` doar când matricea a fost conectată la PTP **și** fiecare membru a citit ultima dată `PtpEnable` activat; nu este același lucru cu starea blocată, pe care o măsoară `ptp_epoch`), `ptp_configured` (ceea ce a solicitat conexiunea), `ptp_off_members` / `ptp_unread_members` (membri care au citit ultima dată `PtpEnable` dezactivat / nu au răspuns), `member_recovery` (`None`, sau un membru care a rămas fără alimentare sau și-a pierdut conexiunea: `state`, `waiting`, deși nu răspunde, `reopening`, apoi `recovered`, împreună cu `members` și `why` — matricea îl redeschide când acesta răspunde din nou și îi restabilește PTP-ul de la momentul conectării, cablarea declanșatorului și a liniei de sincronizare, ritmul și setările camerei, apoi transmite din nou fără noi `connect_array`) și `ptp_epoch` (măsurat), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` înseamnă că Windows nu a putut citi viteza adaptorului — consultați [Ratele de cadre așteptate](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (dacă lanțul de radianță pe bandă este utilizabil și care element l-a întrerupt) și `band_clamp` — în timp ce o vizualizare în timp real redă matricea: proporția de pixeli iluminați pe care procesul de separare a culorilor a limitat-o la 0, pentru fiecare element (`members`: `{serial: [band0, band1, band2]}` în ordinea benzilor R, G, B ale camerei) și, pentru fiecare bandă, compoziția arată (`composite`: `{band name: share}`), împreună cu `warn`, procentul pe care interfața grafică îl marchează ca „Date de bandă cu încredere scăzută” (măsurate aproximativ o dată pe secundă; `None` când nu rulează vizualizarea în timp real), iar `member_ae` — pentru fiecare element, expunerea (`exposure_us`) și amplificarea (`gain_db`) din ultimul cadru publicat, preluate din cadrul însuși (`read_nodes` din `ExposureTime` / `Gain` returnează ultima valoare scrisă în timp ce expunerea automată este activă, nu ceea ce face în acel moment). |
| `set_member_settings(serial, **kwargs)` | Scrie setările unei camere membre, așa cum le preia `CameraSession.set_settings()` (expunere, amplificare, luminozitate țintă, ...); camera membră rămâne în matrice. Generează o eroare `ChlorosConnectError` pentru o cameră din șir care nu este membră. A se vedea [Expunerea pe membrii matricei](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Ce a stabilit funcția smart-prep (nivelul de captură, dimensiunea cadrului, `pixel_format`), blocul live `health` și bugetul de bandă al gazdei. Consultați [Starea matricei](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Modificați rata de declanșare în timp ce matricea funcționează, fără a reconecta. O valoare numerică o fixează (controlerul de rată nu o modifică); `None` o readuce la modul automat. Returnează `{requested, target_fps, target_fps_auto, clamped}` — `clamped` este True când rata a fost redusă la ceea ce gazda și rețeaua pot suporta. Consultați [Captura la intervale pentru sondaje cu drone](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Un cadru de la fiecare cameră, **toate de la un singur declanșator**, sub formă de matrice numpy (nu se scriu fișiere). Returnează un `GrabResult` (un `dict`): `{serial: array}`, sau `{serial: (array, metadata)}` cu `with_metadata=True`. `.group_id` este declanșatorul pe care îl împărtășesc cadrele; `.errors` asociază fiecare cameră omisă cu motivul (o cameră care nu poate fi deservită la acel declanșator este omisă, nefiind niciodată completată de la o altă cameră). `processing` este `raw` / `debayered` / `radiance` / `reflectance` / `display`. Aceeași citire ca și în API-ul proiectului [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — o singură implementare comună — astfel încât fiecare cuvânt-cheie de acolo (`band`, `align`, `bit_depth`, `refl_max`) funcționează aici. Necesită `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterator care returnează un singur rezultat `grab()` pe tic (numai argumente cheie), ritmat la `fps`, limitat la `duration` (secunde) și/sau `count` (`None` = nelimitat). **Fiecare set generat reprezintă un nou declanșator:** un tick al cărui cel mai recent declanșator este setul pe care l-a generat deja așteaptă următorul declanșator în loc să-l repete, astfel încât fluxul rulează la viteza mai mică dintre `fps` și rata de cadre proprie a matricei. Generează `FrameFetchError` în loc să intre în tăcere: imediat când toate camerele refuză, dintr-un motiv anume, repetarea următorului tick, după 60 de secunde de tick-uri fără cadru și după 60 de secunde fără un nou declanșator („niciun nou declanșator timp de 60 de secunde”). Aceeași implementare ca `ArrayHandle.frame_stream()`. Vezi [Cadre live în propriul tău pipeline](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Pornește captarea continuă proprie a backend-ului (modul Continuu al GUI-ului) → [`ContinuousHandle`](#continuoushandle). Salvează **fiecare** grup sincronizat pe care matricea îl publică, așa cum l-ar salva `capture()` (aceleași controale de export), până la oprire sau până la `max_captures` / `duration_s` (`None` = până la oprire). Generează `ChlorosConnectError` dacă există deja unul care rulează pentru această matrice. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Un grup de captură sincronizat. Returnează un `CaptureResult` (listă de dicționare de cadre + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Se transmite un rezultat anterior`group_seq` ca `after_seq` pentru a obține următorul grup sincronizat după acesta, în loc să aștepte unul nou. Comenzi de export mai jos. |
| `capture(..., smart=True)` | **Captură inteligentă** — așteaptă stabilizarea AE pe toate camerele, apoi declanșează. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Captură rapidă: numai date brute + valoarea DAQ atribuită (+ indicele combinat liber). Reflectă butonul „Captură cea mai rapidă” din interfața grafică. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Singură / Continuă / La intervale într-o buclă limitată. Returnează `list[CaptureResult]`.**Necesită `count` și/sau `duration_s`** pentru a se termina (SDK-ul nu are Ctrl+C). De asemenea, se termină mai devreme, după trecere, când o trecere raportează o unitate plină (o intrare `.failed` cu `disk_full`); acea trecere este ultimul rezultat. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Începe înregistrarea vizualizării live a indexului combinat în format video/GIF → `RecorderHandle`. Videoclipul este un fișier Motion-JPEG `.avi` (nu este necesar ffmpeg). Dacă nu este deschisă nicio fereastră GUI, backend-ul îl redă: `expr` (o expresie de index în denumirile benzilor matricei, de exemplu `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`) sau, implicit, NDVI-ul matricei. Un singur înregistrator compozit per matrice. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Porniți o serie de imagini raw-Bayer cu fps ridicat → `RecorderHandle`. Reprocesați offline cu `build_video()`. `orient=True`: produsele generate de `build_video()` sunt rotite în funcție de orientarea camerei (produsele fiecărei camere în parte sunt rotite după orientarea proprie, iar indexul combinat după orientarea camerei principale); cadrele raw rămân în formatul lor original, iar fișierul manifest al seriei de fotografii înregistrează orientarea fiecărui cadru. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Reprocesare offline a unei serii RAW salvate în videoclipuri calibrate. Blochează până la finalizare (`wait=True`) și returnează rezultatul compilării: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — vezi [rezultatul compilării](#the-build_video-result) de mai jos. |
| `build_video_status(job_id, timeout=15.0)` | Interoghează o sarcină de compilare offline: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Eliberează întregul array — **dacă această sesiune l-a deschis** (`connect_array`): atunci devine `force_disconnect()` și generează o excepție `ChlorosConnectError` atunci când eliberarea nu este confirmată (ieșirea din blocul `with` nu generează niciodată o eroare). În cazul unui identificator provenit de la `attach_array`, lasă matricea în funcțiune și afișează o linie pe stderr indicând acest lucru: matricea poate aparține GUI-ului sau unui alt script. Fiecare cameră este lăsată să funcționeze liber și, în rest, așa cum a fost găsită la conectare; `keep_settings=True` salvează setările sale în memoria proprie a camerei, iar `clear_settings=True` resetează memoria respectivă la setările din fabrică (vezi [Memoria camerei](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Eliberează întreaga matrice, indiferent de cine a deschis-o. Folosiți-o pe un handle atașat atunci când intenția dvs. este să închideți matricea (de exemplu, pentru a o reconecta cu setări diferite). Se întoarce odată ce backend-ul confirmă eliberarea sau când nu mai este nimic de eliberat (matricea a dispărut deja sau nu rulează niciun backend); o dezactivare care a depășit timpul de așteptare sau pe care backend-ul a refuzat-o generează `ChlorosConnectError`. Cu oricare dintre aceste indicatori, returnează ce a primit memoria fiecărei camere (`camera_memory`), altfel `None`. |

`capture()` comenzi de export (același punct final folosit de GUI/CLI):

- `processing` / `levels` — `processing="all"` (sau `levels=["raw","radiance",…]`) salvează fiecare tip de export aplicabil pentru fiecare cameră; o singură valoare `processing` salvează doar acel nivel.
- `aligned` — **activat implicit (`None`), deoarece captarea aceleiași matrice de către aplicația Chloros**(setarea de captură**Aliniată** pe matrice). Exportul non-raw al fiecărui element este aliniat la cadrul camerei principale conform [profilului de aliniere](#array-alignment) și decupat la suprapunerea comună a elementelor, astfel încât fișierele sunt aliniate la nivel de pixeli și au dimensiunea, decupajul și etichetele `Alignment*` specifice aplicației (în cadrul unei zone de interes, transformarea este recalculată în funcție de fereastră); `XMP-mapir:AlignmentJSON` indică `"applied": true` cu `crop_roi`-ul său. Fișierele Raw rămân ne-deformate la dimensiunea nativă, dar poartă transformarea în metadate. `aligned=False` salvează fiecare fișier la dimensiunea nativă, ne-deformat; `aligned=True` este setarea implicită specificată. `continuous()`, `capture(smart=True)`, `capture_repeated()` și cele la nivel de proiect, `ArrayHandle.capture()` / `capture_smart()`, au aceeași setare implicită. Indiferent dacă se aplică sau nu deformarea, fiecare fișier membru înregistrează transformarea `Alignment*` a camerei sale odată ce matricea are un profil de aliniere — cu excepția unei camere pe care profilul nu o aliniază (omise sau forțate la identitate din cauza unei rezolvări eșuate), precum și camera de referință atunci când nicio altă cameră nu este aliniată: acestea nu conțin etichete `Alignment*`. În absența unui profil pentru matrice, captura este salvată nealiniată, iar rezultatul indică acest lucru: `result.alignment` este `{"requested": True, "applied": False, "warnings": [...]}`, câte un avertisment pentru fiecare motiv pentru care deformarea nu a putut fi aplicată (lipsa profilului, elemente cu rezoluții diferite, o matrice care nu poate fi, un cadru pe care profilul nu poate fi mapat). `applied` este `True` numai atunci când fiecare export care a solicitat deformarea a primit-o. Alinierea automată a matricei creează profilul odată ce expunerea se stabilizează; `ArrayHandle.calibrate_alignment()` / `align-calibrate` din CLI creează unul la cerere. `result.oriented` indică dacă `orient=True` a rotit produsele.
- `render_index=False` — omite suprapunerea indicelui de vegetație pentru fiecare cameră; implicit, aceasta este redată acolo unde a fost configurată.
- `daq` / `daq_tolerance_ms` — citirea DAQ este salvată implicit ca fișier auxiliar `.daq`, chiar și atunci când niciun nivel selectat nu are nevoie de acesta; `daq=False` o omite în acest caz, iar `daq_tolerance_ms` setează toleranța de potrivire (implicit 500). `force_daq=True` este denumirea veche a valorii implicite. A se vedea [Valorile DAQ salvate de o captură](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — transformă **produsele**fiecărui membru (`display`, `radiance`, `reflectance`, indexul per cameră și compozitul) cu orientarea camerei, după deformarea de aliniere: cu orientarea**camerei principale**atunci când au fost aliniate în cadrul acesteia (implicit `aligned` — membrii rămân co-înregistrați), altfel cu orientarea propriilor lor camere.**Implicit: fiecare fișier membru în format nativ**, așa cum se întâmplă întotdeauna în cazul capturilor de tip matrice; `raw` și `debayered` rămân în format nativ în orice caz. Fiecare fișier înregistrează orientarea cadrului în care se află pixelii săi (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — un fișier raw, al propriului aparat foto), iar fișierul XMP `Alignment*` păstrează transformarea cadrului nativ, aplicată înaintea orientării; un fișier RAW care conține acea transformare înregistrează, de asemenea, `ChlorosAligned*` (cea a cadrului principal), adică orientarea unui produs deformat în cadrul principal. Procesarea rotește fiecare produs cu unghiul care aparține pixelilor săi: cel aliniat atunci când deformează fișierul RAW (alinierea matricei activată), iar cel al camerei atunci când nu o face. `capture_repeated()` și `capture_fastest()` o transmit mai departe; și `ArrayHandle.capture()` / `capture_smart()` la nivel de proiect o preiau, de asemenea.

**Compresia TIFF (opțiune HTTP):** `ArraySession.capture()` nu trimite nicio cheie `compression`, așa că se aplică setarea implicită a backend-ului — `POST /api/camera/array/capture` citește un parametru al corpului `compression`, `"none"` implicit: necomprimat, pixelii senzorului exact așa cum sunt citiți (~6,3 MB pe cadru la rezoluție maximă). `"deflate"` (zlib L1 fără pierderi + predictor orizontal, ~4,1 MB/cadru) scrie fișiere mai mici, dar mai lent — ambele sunt fără pierderi și se citesc identic la import. SDK-ul nu expune niciun argument pentru aceasta; soluția de rezervă este `chloros-cli lattice array-capture --compression deflate` sau HTTP brut. DEFLATE blochează GIL-ul Python, astfel încât scrierile comprimate nu se pot paraleliza între firele de scriere ale fiecărei camere — captarea susținută la rezoluție maximă la rata senzorului necesită setarea implicită necomprimată. Detalii: [Referință CLI → array-capture](cli-reference.md).**Grupuri consecutive (protocol HTTP):** punctul final returnează `group_seq`; o buclă care îl trimite înapoi ca `after_seq` la următoarea sa solicitare obține următorul grup sincronizat — inclusiv unul publicat în timp ce captura anterioară se salva — în loc să aștepte un grup publicat după cerere, iar `skipped_groups` raportează orice grup omis de buclă. `capture_repeated`, `--continuous` din CLI și modul Continuu din GUI fac toate acest lucru; trecerile de interval rămân actualizate la fiecare moment programat.**Suprascrieri de export per membru (numai HTTP):**același punct final acceptă, de asemenea, `exclude_serials` (listă — elimină membri din setul salvat; matricea se declanșează în continuare ca un singur grup sincronizat, iar membrii excluși sunt returnați în `excluded`), `serial_levels` (suprascrieri la nivel de cameră `{serial: [level tokens]}`) și `serial_index` (suprascrieri de suprapunere a indexului pe cameră `{serial: bool}`). Aceștia sunt parametri de corp cu paritate GUI și**nu sunt încă argumente cheie SDK**; membrii absenți din hărți revin la valorile la nivel de matrice `levels` / `render_index`.

##### Inspectarea camerelor omise — `CaptureResult.skipped`

`ArraySession.capture()` returnează un `CaptureResult`, care este o subclasă a `list`: iterați-l, indexați-l, `len()` — fiecare model existent continuă să funcționeze. Codul nou poate inspecta atributul `.skipped` pentru a vedea care camere au fost excluse și de ce. Cel mai frecvent caz îl reprezintă camerele RGB dintr-o matrice cu filtre mixte atunci când se solicită `processing="radiance"` sau `"reflectance"` — radianța pe pixel Bayer nu are sens pentru un senzor de bandă largă, așa că backend-ul omite acele camere în loc să genereze date fără sens.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Token-urile de motiv urmează modelul `<level>-not-applicable-to-rgb-cam` (o intrare pentru fiecare nivel omis, fiecare conținând `level`). O cameră al cărei pachet de calibrare nu este încărcat omite `radiance` / `reflectance` cu `<level>-needs-calibration (connect the camera so its cal pack loads)`, iar în absența unei citiri DAQ utilizabile a reflectanței, sare peste cu `reflectance-skipped-no-dls`. Celelalte omisiuni specifice reflectanței sunt `reflectance-skipped-no-fresh-dls` (nu este disponibilă nicio citire recentă a radiației descendente), `reflectance-skipped-bound-daq-unavailable (…)` (nu s-a putut accesa DAQ-ul asociat) și `dls-uncalibrated-band-<nm>` — banda se află în mare parte în afara intervalului calibrat radiometric al senzorului de lumină al DAQ-ului (~374–974 nm), astfel încât divizarea absolută a reflectanței bazată pe DAQ este refuzată, iar cadrul revine în mod evident la răspunsul senzorului. Dintre codurile de produs disponibile, doar F988 declanșează această situație; calea acceptată de acea cameră este fluxul de lucru cu panoul de reflectanță.

##### Fișiere care nu au putut fi scrise — `CaptureResult.failed`

O captură care a putut scrie doar o parte din fișierele sale returnează totuși: lista conține toate fișierele care **au fost** scrise, iar `.failed` enumeră restul, câte un dicționar per fișier, cu `serial` (`None` pentru fișierul combinat compozit și fișierul auxiliar `.daq` al unui array), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, sau `match` — proveniența `.daq` a fișierului `.match.json`, atunci când `.daq` a fost scris, dar fișierul său sidecar nu a fost) și `error`. Pe o unitate plină, intrarea conține și `disk_full: True`; orice captură ulterioară pe acea unitate ar eșua în același mod, motiv pentru care `capture_repeated()` se oprește acolo. Nu se generează nicio excepție pentru o captură parțială, iar un fișier din `.failed` nu a rămas pe disc pe jumătate scris. `.failed` este gol în cazul unei capturi curate.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### Citirea DAQ-ului la salvarea unei capturi — `.daq_files` și `.daq_matches`

Ori de câte ori un DAQ este asociat cu camera sau matricea, sau este inclus într-un grup, fiecare captură salvează citirea în jos pe care a utilizat-o: un fișier `.daq` în subfolderul `daq/` al folderului de capturi, împreună cu fișierele `.match.json` și `.cameras.json` (ale căror citiri sunt). Acest lucru este valabil și pentru o captură brută sau debayered, iar fișierul este același indiferent dacă captura provine din GUI, CLI sau SDK. `daq=False` o omite pentru o captură ale cărei niveluri nu utilizează DAQ-ul (un nivel `radiance`, `reflectance` sau `display` îl scrie în continuare); `daq_tolerance_ms` mută linia de toleranță pentru acea captură și niciodată valoarea citirii utilizate. Care DAQ: cel asociat camerei sau matricei (selecția senzorului de lumină din GUI, asocierea unui proiect sau `chloros-cli daq pool-bind`), altfel cel mai recent pe care îl are backend-ul. CLI-ul descrie comenzile `--no-daq` și `--daq-tolerance-ms` ([Referință CLI](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

O captură salvează valoarea citită de senzorul de lumină pe care a utilizat-o ca `.daq`, alături de un `<file>.daq.match.json` care indică modul în care valoarea citită a fost corelată cu expunerea: `method`, `gap_ms` (de la expunere la cea mai apropiată valoare DAQ) și `in_tolerance` (în interval de 500 ms, sau `daq_tolerance_ms` al capturii). Fișierul XMP al fișierului de reflectanță conține același rezultat ca `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` și `Chloros:DlsInTolerance`.

O captură așteaptă prima citire a DAQ-ului după expunere, astfel încât să poată combina citirile de pe ambele părți ale acesteia (`method` `interpolated`). Un DAQ-E trimite o citire la fiecare 0,4 s în lumina zilei și la fiecare 1-2 s în interior, astfel încât `capture()` se înregistrează cu o întârziere de până la aproximativ un interval și jumătate; expunerea în sine nu este întârziată. Nu se așteaptă un DAQ care a încetat să mai transmită. O valoare DAQ-E reprezintă media luminii pe intervalul de timp de la ultima valoare înregistrată, deci este plasată la mijlocul acelui interval: `gap_ms` este măsurat la acel punct central, iar `.match.json`'`bracket_stamps_ns` oferă propriile marcaje temporale ale citirilor. `ArraySession.continuous()` așteaptă fără a încetini captarea. Trecerile înapoi ale lui `capture_repeated()`nu așteaptă, așa că își mențin ritmul și sunt corelate cu cea mai apropiată citire de dinaintea expunerii.

O citire nu expiră niciodată, așa că niciun cadru nu se pierde din cauza unei citiri întârziate — în schimb, este etichetată:

- **Fluxul DAQ-ului s-a blocat** — se utilizează cea mai recentă citire a acestuia, iar `gap_ms` indică vechimea acesteia (`in_tolerance: false` depășește 500 ms).
- **Nu există DAQ de citit**— deconectat (`disconnect()`, `chloros-cli daq pool-disconnect`) sau DAQ-ul asociat nu poate fi accesat — ultima valoare a camerei este reutilizată și asociată cu**această** expunere: `method`, `retained`, `gap_ms` din această expunere, `in_tolerance` este recalculat, iar `retained_from_epoch_ns` indică expunerea cu care a fost asociat inițial. O cameră care nu a avut niciodată o citire nu primește totuși nicio reflectanță, iar rezultatul explică de ce (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, sau refuzul capturii).

Fișierele pe care o captură le salvează în folderul proiectului deschis în Chloros se alătură listei de fișiere a acelui proiect, la fel cum se întâmplă cu o captură din aplicație (fișierele salvate în altă parte rămân neschimbate): `CameraSession.capture()`, `ArraySession.capture()` (și `smart=True`) și `capture_cameras_transient()`. Rezultatul indică acest lucru în `.project_add` (`capture_cameras_transient()`: `project_add` din dicționar) -- `held`, `files` și `message` din backend; `held` este `True` în timp ce o execuție de procesare este în curs, iar fișierele se alătură odată ce aceasta se încheie. `None` atunci când niciun fișier nu a fost adăugat în proiect.

`CameraSession.capture()` și `ArraySession.capture()` le returnează ca `.daq_files` și `.daq_matches`, ca răspunsuri de captură HTTP (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) conțin `daq_files` și `daq_matches`, câte unul pentru fiecare `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` și `stream_fault` — eroarea de flux specifică DAQ-ului, verificată doar în cazul unei citiri în afara toleranței. Înainte de a vă baza pe reflectanța dintr-o captură, verificați `in_tolerance` în `.match.json` sau XMP, sau nivelurile DAQ-ului [`status()`](#daqsensorsession-methods).

Nivelurile `processing`:

| Nivel | Ieșire |
| --- | --- |
| `"raw"` | Bayer monocanal (camere monocrome: banda unică) direct de la senzor. |
| `"debayered"` *(implicit SDK)* | Culoare pe 3 canale: offsetul de întuneric (DSNU) din pachetul de calibrare eliminat din datele brute, urmat de demosaic cu recunoașterea marginilor realizat de OpenCV — fără LUT de liniaritate, câmp plat sau demixare (camere monocrome: 1 canal în tonuri de gri). Fișierul TIFF salvat este în ordinea canalelor **RGB**(RGB fotometric, astfel încât orice vizualizator de imagini afișează culorile reale); matricile din memorie returnate de `grab()` / `frame_stream()` sunt în ordinea**BGR** a OpenCV — inversați canalele (`frame[..., ::-1]`) pentru a compara rezultatul cu un fișier salvat. `chloros-cli process` scrie același produs ca `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm prin lanțul radiometric complet. Numai multispectral — camerele RGB sunt omise. |
| `"reflectance"` | uint16 0..32768 (compatibil cu Pix4D); necesită o asociere DAQ în timp real pentru referință absolută. Numai pentru imagini multispectrale. |
| `"display"` | Lanț complet care corespunde previzualizării din interfața grafică (CCM + WB + gamma conform profilului camerei). |
| `"all"` | **Un fișier pentru fiecare nivel aplicabil** pentru fiecare cameră (corespunzător opțiunii „Capture All” din interfața grafică / setării implicite din CLI). Fișierul `CaptureResult` returnat conține apoi un dicționar de cadre pentru fiecare `(cam, level)`, cu nivelul specificat în fiecare dicționar; nivelurile inaplicabile apar în `.skipped`. Citirea DAQ utilizată pentru captură este salvată ca un fișier auxiliar `.daq`. |

> **Notă — setarea implicită diferă de cea din CLI.** `ArraySession.capture()` are ca valoare implicită `processing="debayered"`; comanda `chloros-cli lattice array-capture` are ca valoare implicită `processing="all"`. Transmiteți explicit `processing="all"` din SDK pentru a reflecta salvarea pe mai multe niveluri din CLI/GUI.

##### Numele fișierelor de captură a matricei

Fiecare nivel este salvat într-un subfolder propriu al `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` pentru `display`, `index/`, `composite/`). Fiecare fișier denumește camera respectivă ca `SN<serial>-<filter>`: numărul de serie, urmat de partea referitoare la filtru din modelul camerei — `SN214000533-FOCN` pentru o cameră `M3C-L87-FOCN`, `SN261902142-F450` pentru un `M3M-L87-F450` — alăturat cu `-`, astfel încât numele are aceleași câmpuri separate de `_` ca și înainte. O cameră al cărei model nu poate fi citit este doar `SN<serial>`, iar capturile din versiunile anterioare nu au niciun filtru în nume. Numele fișierului depinde de câte niveluri i s-au solicitat membrului:

| Nivelul solicitat de membru | Numele fișierului | Exemplu (`reflectance`, implicit `format="tiff"`) |
| --- | --- | --- |
| **Un singur nivel** (`processing="reflectance"` sau un `levels` cu un singur element) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — fără sufix de nivel, ca în versiunile anterioare, astfel încât scripturile existente să funcționeze în continuare | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Mai multe niveluri** (`processing="all"` sau `levels` cu mai mult de un element) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[Masca de limitare](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) urmează după numele produsului, `clamp_masks/<product folder>/<product name>_clampmask.tif`: o captură de reflectanță la un singurcaptură de reflectanță cu un singur nivel generează `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, în timp ce `processing="all"` generează `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. Regula privind sufixul este stabilită pentru fiecare element în parte, în funcție de nivelurile solicitate, astfel încât un nivel omis (de exemplu, reflectanța pe o cameră RGB) nu o modifică. Citiți căile din cadrele returnate, în loc să le construiți. O captură cu o singură cameră (`CameraSession.capture()`) adaugă întotdeauna nivelul: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### PNG / JPEG: se acceptă doar afișarea și indexarea produselor

Un fișier PNG sau JPEG conține 8 biți pe canal. Solicitarea unui fișier cu un nivel pe care acesta nu îl poate stoca este respinsă înainte de capturarea oricărui conținut, cu un mesaj care specifică nivelul și formatul TIFF care îl suportă:

- **`radiance`**(float32) și**`reflectance`**(uint16, `32768` = 1,0) — și, prin urmare, `processing="all"`, care le include. SDK-ul verifică acest lucru pe argumente: `capture()` generează o excepție**`ValueError`** fără o solicitare (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` provenite de la o cameră care transmite un flux de peste 8 biți** (M3C de 12 biți, Mono12 M3M) — backend-ul le respinge, cunoscând adâncimea de biți a fiecărei camere: 400 HTTP cu `error_code` `FORMAT_CANNOT_HOLD_LEVEL`, generat ca `ChlorosConnectError` (`CameraSession` / `ArraySession`) sau `RuntimeError` (`CameraHandle` / `ArrayHandle`). Captura `debayered` pe un singur nivel a unei camere înregistrează un cadru de 8 biți pentru PNG/JPEG și nu este respinsă.

Salvați datele radiometrice ca `format="tiff"` (`ext=".tiff"`); utilizați PNG/JPEG pentru `display` și randări indexate.

#### Cum funcționează expunerea automată

- **Mai întâi expunerea, apoi amplificarea.** Când amplificarea este setată pe „Auto”, aceasta rămâne la 0 dB până când expunerea atinge cea mai mare valoare permisă.
- **O expunere manuală lasă amplificarea pe modul „Auto”** până când modificați amplificarea sau o setați pe modul manual. Revenind la modul „Auto”, limita amplificării este aceeași ca înainte.
- **Zonele luminoase mici nu întunecă scena.** O reflexie de până la 1 % din cadru este ignorată, iar în cazul panourilor unei ținte de calibrare cunoscute, se ignoră până la 0,5 % dintr-un panou.
- **Limita maximă de expunere DAQ se aplică numai cât timp o țintă de calibrare se află în câmpul vizual.** Aceasta împiedică decuparea unui panou alb; dacă nu se vede niciun panou, camera expune pentru scenă.
- **Lumina constantă rămâne stabilă, iar lumina modificată se stabilizează în câteva secunde.** La lumină constantă, expunerea și limita acesteia rămân neschimbate, iar după o schimbare majoră a luminii, imaginea se stabilizează în câteva secunde. O supraexpunere nu mai limitează definitiv expunerea, iar o modificare a zonei de interes (ROI) restabilește expunerea.
- **Ținta de luminozitate rămâne la îndemână.** Aceasta nu crește cât timp camera se află la limita de expunere și amplificarea sa nu poate crește, astfel încât o lumină care revine nu produce reflexii în cadru.
- **O cameră a cărei expunere automată nu mai răspunde este menținută la nivelul corespunzător luminii în care se află.** În lumina puternică a soarelui, o cameră ar putea coborî la expunerea sa minimă și să rămână acolo cu un cadru întunecat, toate setările sale indicând valori normale. Chloros monitorizează cadrul, menține camera la nivelul pe care l-a păstrat pentru acea lumină și o eliberează când lumina se schimbă. Un senzor de lumină întunecat sau acoperit nu afectează acest proces.

#### Expunerea pe elementele matricei

O cameră LATTICE limitează `ExposureTime` la o perioadă din rata sa de cadre. În timp ce o matrice transmite în flux, Chloros rulează fiecare element cu `AcquisitionFrameRateEnable` activat, la o rată puțin mai mare decât rata de declanșare, astfel încât limita maximă de expunere a unui element se situează puțin sub perioada de declanșare, **în orice ROI**:

- Perioada de cadru a unui element este perioada de declanșare minus o marjă de 15 % din perioadă (cel puțin 3 ms, cel mult jumătate din perioadă), iar limita maximă de expunere a acestuia este acea perioadă de cadru minus aproximativ 0,1 ms. Exemplu: un declanșator de 10 fps (100 ms) oferă o rată a membrului de aproximativ 11,8 fps și un plafon de expunere de aproximativ 85 ms.
- **O expunere manuală `ExposureTime` mai lungă decât acest plafon încetinește declanșarea pentru a se adapta la aceasta.** Chloros deschide mai întâi plafonul acelui membru pentru valoarea respectivă și lasă rata mai lentă să ruleze câteva perioade de declanșare (scrierea poate dura cu câteva secunde mai mult), apoi o scrie, iar declanșarea nu are loc mai repede decât permite expunerea. Pe o matrice cu rată automată, rata la care funcționa anterior revine odată ce fiecare element a cărui expunere manuală a încetinit-o revine la expunerea automată. O valoare pe care camera însăși nu o poate înregistra este în continuare refuzată, intervalul acceptat fiind acum indicat în mesajul de eroare, iar expunerea automată a elementului este restabilită dacă a fost dezactivată doar pentru acea scriere. Înainte de versiunea 1.3.0, o zonă de interes (ROI) mică limita fiecare element la perioada de funcționare liberă a senzorului, indiferent de rata de declanșare: 4,9 ms la 768 × 304, astfel încât niciun element nu putea fi luminat.
- Chloros nu scurtează niciodată o expunere manuală pentru a se adapta la un declanșator mai rapid. O rată automată rămâne în limitele permise de expunerile manuale; o rată fixată mai rapidă decât permite o expunere lasă acel element cu un plafon suficient de lung pentru aceasta, iar declanșările nu sunt mai rapide decât permite expunerea sa. Reduceți expunerea sau valoarea fixată atunci când fiecare element trebuie să efectueze fiecare declanșare.
- **Citire:** citirea nodurilor unui element afișează `AcquisitionFrameRateEnable` True și `AcquisitionFrameRate` la rata elementului; activarea se dezactivează când matricea se oprește.
- **Matricea deține aceste două noduri. Nu scrieți `AcquisitionFrameRateEnable` sau `AcquisitionFrameRate` pe un element al matricei** — setați rata matriceicu `target_fps` / `set_target_fps()`.

Scrieți expunerea unui element al matricei cu `arr.set_member_settings(serial, exposure_time=20000)` (cuvintele-cheie ale `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` are același efect). Elementul rămâne în matrice. Nu deschideți un element al matricei cu `connect_camera()`: connect refuză o cameră care se află deja într-o matrice, iar comanda `CameraSession` pe care ar returna-o elimină camera din matricea sa la deconectare. `ArraySession.status()["member_ae"]` raportează expunerea fiecărui elementexpunerea și amplificarea fiecărui element din cel mai recent cadru al său.

### Un singur înregistrare de proiect, pentru fiecare mod

Un dosar de proiect conține setările fiecărui dispozitiv: `cameras.json` — ale fiecărei camere (`cameras[].settings`, pentru o singură cameră și pentru fiecare element al matricei) și cele la nivel de matrice (`arrays[].capture`: format, ROI, binning, rata de declanșare, buget de cablare) — precum și `sensors.json`, setările fiecărui senzor de lumină (timp de integrare, medie pe cadru, limită). Aplicația, CLI, SDK și serverul MCP citesc și scriu toate aceleași fișiere, în același mod:

- **Un dispozitiv conectat în cadrul unui proiect este salvat în acesta și înregistrează fiecare modificare în cadrul acestuia.** „În cadrul unui proiect” înseamnă cu un proiect deschis în backend (al aplicației) sau — prin intermediul unui `ChlorosProject` (`open_project()`), care indică propriul său folder către backend (`project_path`). O cameră, o matrice sau un senzor de lumină pe care proiectul nu le conține încă sunt adăugate la fișierele sale atunci când se conectează (`connect_camera()`, `connect_array()`, conectarea senzorului de lumină, CLI, aplicația, `chloros-mcp`), împreună cu setările sale. O schimbare din orice mod (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, setările matricei, `set_cap()`, CLI-ul, comenzile aplicației) este salvată în fișierele acelui proiect imediat după aceea. O expunere, un câștig sau o țintă AE setată pe o cameră este salvată ca fiind menținută, astfel încât redeschiderea proiectului o menține din nou.
- **Deschiderea unui proiect configurează dispozitivele acestuia**, în orice mod: o cameră, o matrice sau un senzor de lumină conectat, fără setări proprii, preia setărilesalvate ale proiectului (`lattice cam-connect` / `array-connect` cu un proiect deschis, `chloros-cli project connect`, `open_project().connect_all()`, redeschiderea aplicației). O matrice conectată sub un ID nou actualizează matricea pe care proiectul a salvat-o cu aceleași camere; o matrice cu alte camere este adăugată alături de aceasta.
- **Salvarea propriu-zisă a aplicației nu suprascrie niciodată setările active ale unui dispozitiv** cu o copie mai veche a acestora și păstrează modificările specifice fiecărui membru salvate de un apelant CLI sau SDK.
- **Formatul de pixeli pe care îl setați este cel păstrat de proiect.** Un format setat în Setările matricei, prin `--pixel-format` sau prin `connect_array(pixel_format=)`, inclusiv cel de 8 biți, este salvat și reconectat exact. Dacă nu este setat nimic, fiecare cameră se conectează la setarea implicită de 12 biți (`BayerRG12`, `Mono12` pe un senzor monocrom). Numele unui format indică adâncimea de biți a acestuia; familia aparține senzorului, astfel încât o cameră color nu este niciodată setată pe un format monocrom, iar o cameră monocromă nu este setată pe un format Bayer, iar linia `[PixelFormat <serial>] ... written` indică acest lucru atunci când un format a fost mapat. Exporturile unei capturi de 8 biți sunt de 8 biți, inclusiv reflectanța: redimensionate și marcate cu `PixelScale`.
- Un dispozitiv conectat fără a fi asociat unui proiect nu înregistrează nimic.
- **O înregistrare ține cont de un proiect creat după ea; o înregistrare deschisă rămâne asociată proiectului său.**Când se creează un proiect — `create_project()`, `chloros-mcp` al lui `create_project` sau opțiunea**Proiect nou**a aplicației — sau duplicat (opțiunea**Duplicare**a aplicației, care deschide copia) în timp ce o înregistrare încă scrie în proiectul care era deschis anterior, înregistrarea continuă, iar ceea ce scrie ajunge în noul proiect, în același loc din interiorul acestuia (mai jos). Deschiderea unui proiect existent — `ChlorosProject.process()` (care își deschide proiectul în fundal), `chloros-cli process`, opțiunea**Deschide proiect** din aplicație — lasă o înregistrare încă în curs de desfășurare cu proiectul în care se înregistrează: aceasta se finalizează acolo, iar fișierele sale se alătură listei de fișiere a acelui proiect (aplicația oprește orice înregistrare înainte de a deschide un proiect).
  - `ArraySession.record()` și `ArraySession.burst()`: videoclipul sau folderul cu toate cadrele se mută integral acolo când înregistrarea se oprește. Rezultatul opririi raportează noua cale (`out_dir`, `video_path`), iar o versiune a seriei de cadre se generează pe baza acesteia. Dacă unitatea de stocare refuză mutarea, înregistrarea rămâne acolo unde a fost scrisă, iar rezultatul indică acest lucru (`relocation_error`). Cadrele brute ale unei serii oprite și `.daq` se adaugă la lista de fișiere a proiectului în care ajung; un videoclip nu este adăugat la lista de fișiere a unui proiect.
  - `ArraySession.continuous()`: fiecare captură realizată după modificare este salvată în noul proiect; cele salvate anterior rămân unde sunt.
  - `DAQSensorSession.record_start()` (și înregistrarea propriu-zisă a senzorului de lumină al aplicației): `.daq` se mută în folderul `light_sensor` al noului proiect atunci când înregistrarea se oprește; `record_stop()` returnează noua cale, iar fișierul se adaugă la lista de fișiere a acelui proiect.

  O înregistrare într-un dosar din afara proiectului deschis rămâne acolo unde este, iar `start_camera_continuous()` / sarcinile multi-stream păstrează dosarul care le-a fost atribuit.
- **În aplicație, capturile efectuate după crearea unui proiect nou sunt mutate în acesta.** Crearea sau duplicarea unui proiect menține în desfășurare toate sesiunile de captură și înregistrările existente, iar fiecare captură efectuată ulterior (Captură totală, Începe capturarea) este salvată în noul proiect și se adaugă la lista de fișiere a acestuia. Deschiderea unui proiect existent în aplicație le oprește mai întâi pe toate.
- **Un proiect nu este mutat în timp ce se înregistrează.**Mutarea proiectului deschis într-un alt director de lucru (din**Setări → Director de lucru** al aplicației; niciun apel SDK nu mută un proiect, un client HTTP care trimite date către `/api/move-project` o face) este refuzată cu codul HTTP 409 `PROJECT_RECORDING` în timp ce o înregistrare matricială sau o serie de capturi, o captură matricială continuă, o sarcină continuă/multi-stream a camerei sau o înregistrare a senzorului de lumină scrie în acesta: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Acest lucru este separat de starea în care rămân camerele la deconectare (vezi [Memoria camerei](#camera-memory--keep_settings--clear_settings)): un proiect își configurează singur camerele la deschidere.

### Memoria camerei — `keep_settings` / `clear_settings`

Când o matrice este conectată, camerele sale preiau setările matricei, iar tu le poți modifica ulterior. Când Chloros dezactivează camerele — `ArraySession.disconnect()` / `force_disconnect()`, la sfârșitul unei funcții tranzitorii, la oprirea `start_camera_continuous()` — fiecare cameră revine la starea în care se afla înainte de conectare: cablarea declanșatorului și a liniei de sincronizare, setările PTP și ale camerei — `ExposureAuto` / `GainAuto` (și o setare manuală `ExposureTime` / `Gain`), `TargetBrightness`, precum și limitele și amortizarea AE, balansul de alb, nivelul de negru, `PixelFormat`, binning și ROI, fluxul de procesare a culorilor (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, `PtpSlaveOnly` și `PtpDomainNumber` ale PTP, dimensiunea pachetului (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), ritmul (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), datele blocului (`ChunkModeActive`, fiecare `ChunkEnable` sub `ChunkSelector`), eliminarea zgomotului de la intrarea de declanșare, o matrice oferă linia de sincronizare a unui dispozitiv secundar (`LineFilterSelector` / `LineFilterWidth`) și rata de cadre de achiziție (`AcquisitionFrameRate` numai atunci când activarea sa era pornită: când este oprită, rata citește valoarea maximă permisă de ritm, nu o setare). Se scriu doar diferențele: oprirea inițială a camerei, ritmul și rata cadrelor înainte de o expunere care necesită perioada de cadru corespunzătoare.

Pentru utilizarea camerelor în afara unui proiect, două argumente scriu în **memoria proprie** a camerei — ceea ce o pornește, în orice software (pe `disconnect()` / `force_disconnect()`, fiecare funcție tranzitorie, `start_camera_continuous()` și `ChlorosProject.disconnect_all()`; MCP: aceleași argumente ca pentru `disconnect_device` și instrumentele tranzitorii; CLI-ul: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** salvează setările fiecărei camere așa cum au fost lăsate în memoria acesteia la încheierea sesiunii (UserSet2, setul de pornire). Camerele sunt lăsate în continuare să funcționeze liber (`TriggerMode` `Off`), iar cablajul liniei de sincronizare rămâne așa cum a fost găsit. O cameră fără UserSet2 salvează în UserSet1; acest computer reține acest lucru, astfel încât funcția de auto-reparare a matricei Chloros — care resetează un UserSet1 care s-a abătut de la valoarea de referință a Chloros — o lasă așa cum este.
- **`clear_settings=True`** resetează memoria fiecărei camere la setările din fabrică: linia de bază a lui Chloros, cea pe care o încorporează funcția de auto-reparare a matricei — setările implicite din fabrică cu `TriggerMode`, `Off`, `TriggerSource`, `Line2`, expunere automată, balansul de alb calibrat al unei camere RGB păstrat — în UserSet1, care a devenit setul de pornire.
- Nu ambele (`ValueError`; backend-ul răspunde cu 400). Răspunsul conține `camera_memory`, pentru fiecare cameră `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` sau `{"cleared": bool, "errors": [...]}` — `force_disconnect()` îl returnează; pentru o sarcină, acesta vine cu `stop()`.

Aplicația nu oferă aceste valori.

**Fiecare cale de ieșire restabilește camerele.**Restabilirea în starea inițială se execută pe fiecare cale de ieșire: `CameraSession.disconnect()` (camera**X**a aplicației),**X** a aplicației pe un array sau pe orice element al array-ului (întregul array este eliberat), închiderea aplicației (backend-ul are la dispoziție până la 20 de secunde pentru a repune fiecare cameră, array și înregistrare continuă în curs; o funcție tranzitorie sau un apel unic care încă rulează se oprește la următorul punct de siguranță, își repune mai întâi camerele, timp de până la 8 s, și răspunde cu HTTP 503 `SHUTTING_DOWN`; un `connect_array()` care încă rulează se oprește înainte de următoarea sa cameră și restabilește camerele pe care le-a deschis, timp de până la 10 s, și generează `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); iar una care ajunge în timpul închiderii nu deschide nimic: HTTP 503 `SHUTTING_DOWN`), precum și o comandă `chloros-cli` care este oprită forțat sau al cărei terminal se închide (sarcina sa este oprită, iar camerele sale sunt readuse la starea inițială mai întâi). Un backend care se blochează sau este oprit forțat nu poate readuce nimic la starea inițială; o repornire readuce fiecare cameră în memoria sa.**Un proiect își configurează singur camerele.** Când o matrice se conectează printr-un proiect — proiectul deschis al aplicației sau un `ChlorosProject`, care se identifică în fața backend-ului (`project_path`) — fiecare membru primește setările salvate de proiect pentru el: o expunere, un câștig sau o țintă AE menținute pentru el (considerate ca fiind ale tale, astfel încât expunerea automată nu le modifică) și modul său de balans de alb (corecția Chloros; balansul de alb propriu al camerei rămâne dezactivat), indiferent de conținutul memoriei camerei.

### Moduri de captură și dispozitive de înregistrare

Suprafața matricei reflectă panoul de captură al interfeței grafice: modurile de declanșare Singular / Continuu / Interval / Cel mai rapid, plus două dispozitive de înregistrare (video compozit în direct și rafală RAW → reprocesare offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**este bucla Continuu/Interval a SDK-ului. Deoarece nu există `Ctrl+C` pentru a o întrerupe dintr-un script,**trebuie** să transmiteți `count` și/sau `duration_s` (se oprește când se ajunge la oricare dintre ele). `interval_s` se măsoară de la începutul fiecărei treceri (la fel ca în interfața grafică). Trecerea continuă (`interval_s=0`) leagă `group_seq` → `after_seq`, astfel încât acestea sunt grupuri sincronizate consecutive la nivelul. Restul parametrilor kwargs sunt transmise direct către `capture()`.
- **`continuous`**reprezintă captura continuă proprie a backend-ului — modul Continuu al interfeței grafice.**Utilizați-l în locul lui `capture_repeated()` atunci când doriți ca fiecare declanșare să fie înregistrată pe disc la o rată de cadre ridicată:** `capture_repeated()` efectuează o singură cerere HTTP pe grup din scriptul dvs., în timp ce `continuous()` menține două capturi în curs de procesare în interiorul backend-ului, fără transfer între grupuri. Măsurat la ROI 512 pe o matrice de 9 camere: 73–77 capturi/s, comparativ cu o matrice care publică aproximativ 78. Păstrați `capture_repeated()` pentru captarea la intervale (`interval_s`) și pentru trecerile pe care le inspectați una câte una. Pe Windows, excludeți backend-ul din Microsoft Defender pentru rate ridicate (consultați [Pasul 1](#step-1--install-the-chloros-platform-package)).
- **`record`** este de *nivel de monitorizare*: înregistrează compoziția indexului combinat redată în timp real. Cu vizualizarea combinată a GUI-ului deschisă, acesta înregistrează acea vizualizare așa cum este afișată. Dacă nu este deschisă nicio vizualizare, backend-ul redă vizualizarea în sine: `expr` dacă îi transmiteți una, altfel NDVI-ul matricei (benzile roșie și NIR cele mai apropiate de 660 nm și 850 nm, așa cum le selectează vizualizarea în timp real), odată ce matricea este aliniată. Denumiți benzile în `expr` așa cum le denumește matricea: `<band>_<filter>` pentru o cameră M3C (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` pentru o cameră M3M (`Mono_850`); un nume scurt de culoare (`NIR`, `Red`) funcționează numai atunci când o bandă a matricei o conține. O fereastră GUI deschisă în timpul înregistrării este înregistrată atâta timp cât rămâne deschisă. O înregistrare care nu scrie niciun cadru eșuează cu `error` și `"no-frames: …"` (matricea poate fi încă în proces de aliniere sau de stabilizare a expunerii), iar `stop()` generează o eroare (vezi [`RecorderHandle`](#recorderhandle)). Un singur înregistrator compozit per matrice (generează o eroare dacă unul este deja în execuție).
- **`burst` → `build_video`** este de *calitate analitică*: `burst` scrie cadre brute + un manifest pentru fiecare cadru + un `.daq` pentru fiecare citire DLS distinctă sub `<output>/bursts/<base>/` la rata maximă a buclei de captură (fără lanț, fără exiftool, fără vizualizare în timp real). `build_video` sincronizează temporal fiecare cadru cu cel mai apropiat `.daq` și rulează din nou lanțul de import pentru radianță/reflectanță/index. `products` este o listă de `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (implicit: indexul combinat). `burst().stop()` inițiază automat, de asemenea, o, returnat ca `build_job` în rezultatul de oprire.

##### Rezultatul `build_video`

| Cheie | Semnificație |
| --- | --- |
| `success` | `True` numai atunci când fiecare produs solicitat a fost generat conform cererii: cel puțin o ieșire, nimic în `skipped`, și nu în `degraded`. |
| `outputs` | Căile către videoclipurile (și GIF-urile) salvate. |
| `errors` | Probleme, sub formă de text. |
| `skipped` | Câte un `{"serial", "level", "reason"}` pentru fiecare produs solicitat care **nu** a generat niciun videoclip — fie din cauza unei camere (lipsa pachetului de calibrare pentru camera respectivă, lipsa citirii DLS pentru reflectanță, eșecul lanțului de procesare) sau a indicilor combinați, care conține `"kind": "combined"` și `serial` `None` (nu există nicio pereche de benzi roșu/ NIR printre camere, niciun grup cu radianță pentru ambele benzi). Orice intrare generează `success` `False`; celelalte înregistrări video sunt totuși scrise și listate. |
| `substitutions` | Camerele sunt redate la un nivel diferit de cel solicitat, și de ce — o cameră RGB pentru care s-a solicitat radianță/reflectanță primește videoclipul de afișare, cu balansul de alb ajustat atunci când pachetul de calibrare al camerei este disponibil. Nu este o eroare. |
| `notes` | Text informativ, nu probleme — de exemplu, o cameră fără expresie de index specifică nu are videoclip index, sau videoclipul de afișare al unei camere RGB nu are balansul de alb ajustat (nu există pachet de calibrare sau acesta nu conține un balans de alb din fabrică), cu menționarea numelui camerei. Nu este o eroare. |
| `degraded` | `True` atunci când a fost scris un produs, dar acesta nu este cel solicitat: indicele combinat offline este diferența normalizată încorporată roșu/NIR, astfel încât o expresie de indice configurată nu este evaluată. Generează `success` `False`. |
| `combined` | Rezumatul indicelui combinat: benzile utilizate, intervalul LUT, cadrele generate. |
| `clamp_masks` | Căile către [măștile de limitare a amestecării](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) ale fiecăruicadru (`save_tiffs=True`), în `clamp_masks/<level>/` lângă folderul `<level>/` al fișierelor TIFF, câte una pentru fiecare fișier TIFF în care a fost limitat un pixel. Un videoclip nu conține niciuna. |
| `clamp_fractions` | Ce indică un fișier video în locul unei măști: câte un fișier `{"kind", "serial", "level", "frames"}` pentru fiecare produs video (`kind` `"combined"` pentru camerele membre ale indexului combinat), `frames` o listă de `[frame, R, G, B]` — proporția de pixeli din acel cadru pe care procesul de separare a limitat-o în banda pe care o deține canalul său R / G / B (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Returnează `ArraySession.record()` și `ArraySession.burst()`. Utilizați-l ca manager de context pentru oprirea automată la ieșirea din scope sau controlați-l manual.

| Element | Descriere |
| --- | --- |
| `job_id` | ID-ul sarcinii din backend (șir). |
| `kind` | `"composite"` (din `record`) sau `"raw"` (din `burst`). |
| `start_stats` | Dicționarul returnat de apelul `start`. |
| `result` | `None` în timpul rulării; dicționarul cu rezultatul final al opririi odată ce s-a oprit. |
| `stats(timeout=10.0)` | Statistici în timp real ale sarcinii (cadre scrise, fps realizat, timp scurs). |
| `stop(timeout=60.0)` | Oprește înregistratorul; returnează și stochează în cache rezultatul final (`.result` rămâne `None` dacă cererea de oprire a eșuat). Pentru o înregistrare `record()` care a eșuat — una care nu a scris niciun cadru (`error` `"no-frames: …"`) — se memorează rezultatul în cache la `.result`, apoi se generează o excepție la `ChlorosConnectError`. Idempotent (o a doua apelare returnează rezultatul stocat în cache, fără a genera o excepție). Ieșirea dintr-un bloc `with` oprește înregistratorul fără a genera o eroare, așa că verificați `rec.result.get("error")` după acesta. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Returnează `ArraySession.continuous()`. Utilizați-l ca manager de context — ieșirea din blocul `with` oprește captarea dacă aceasta încă rulează — sau controlați-o cu `status()` / `wait()` / `stop()`. Se execută o singură captură continuă pe matrice la un moment dat.

| Element | Descriere |
| --- | --- |
| `start_status` | Dicționarul de stare returnat de apelul de pornire. |
| `result` | `None` în timpul rulării; dicționarul de stare final odată ce capturarea s-a încheiat (setat de `stop()`, sau de către un `status()` / `wait()` care a detectat încheierea acesteia). |
| `status(timeout=10.0)` | Dicționar de stare activ (câmpurile de mai jos). |
| `wait(timeout=None, poll_s=0.5)` | Blochează până când capturarea se încheie de la sine — atingerea valorilor `max_captures` sau `duration_s`, dispariția matricei, umplerea unității de stocare, o eroare — și returnează starea finală. Cu `timeout`, returnează starea cea mai recentă după atâtea secunde, chiar dacă procesul încă rulează: verifică `running`. |
| `stop(timeout=30.0)` | Oprește capturarea și returnează starea finală a acesteia. Idempotent (o a doua apelare returnează rezultatul stocat în cache). |

Câmpuri de stare:

| Câmp | Semnificație |
| --- | --- |
| `running` | Adevărat cât timp capturarea este în curs de desfășurare. |
| `captures` | Grupuri sincronizate salvate. |
| `files` | Fișiere scrise. |
| `failed_files` | Fișiere care nu au putut fi scrise (așa cum se arată în [`CaptureResult.failed`](#fișiere-care-nu-au-putut-fi-scrise--captureresultfailed)). |
| `failed_captures` | Încercări de captură care au eșuat în totalitate, inclusiv cele pentru care s-a încercat din nou (un cod HTTP 503 în timp ce matricea nu avea un grup nou la timp). |
| `skipped_groups` | Grupurile publicate de matrice care nu au fost salvate (captura a rămas în urmă). |
| `captures_per_s` | Rata medie de salvare de la început. |
| `elapsed_s` / `started_at` | Secunde de funcționare până în prezent; ora de începere (secunde de la epoca Unix). |
| `last_group_seq` | Cel mai recent grup sincronizat salvat. |
| `last_error` | Cea mai recentă eroare sau `None`. |
| `stop_reason` | Odată încheiat: `count` (s-a ajuns la `max_captures`), `duration` (s-a ajuns la `duration_s`), `stopped` (s-a apelat `stop()`), `array_gone` (matricea a fost deconectată), `error` (20 de eșecuri consecutive sau o solicitare respinsă de backend), `disk_full` (un fișier pentru care unitatea nu avea spațiu suficient). `None` în timpul rulării. |
| `max_captures` / `duration_s` | Limitele cu care a fost pornit (`0` = niciuna). |
| `defender_cores` | Nucleele CPU utilizate de Microsoft Defender în intervalul de 10 secunde; `None` până când se măsoară unul și întotdeauna `None` pe o gazdă fără Defender. |
| `defender_warned` | Adevărat odată ce capturarea a înregistrat faptul că Defender o încetinește (la 1 nucleu sau mai mult). Excludeți backend-ul din scanarea Defender — consultați [Pasul 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Cadre în timp real în propriul dvs. pipeline

Pentru a introduce cadre sincronizate în propriul cod — învățare automată, un vizualizator personalizat, propriul înregistrator — utilizați `ArraySession.frame_stream()`. Fiecare tick generează un `GrabResult`: `{serial: numpy array}` pentru fiecare cameră, **toate dintr-un singur declanșator**, cu ID-ul acelui declanșator în `.group_id`. Este aceeași implementare ca și `ArrayHandle.frame_stream()` din API-ul proiectului, așa că tot ce se află sub [Un declanșator per citire](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) și [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) se aplică ambelor.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Fiecare set generat reprezintă un nou declanșator.** `frames.group_id` nu se repetă niciodată în cadrul unui flux; atunci când solicitați mai multe `fps` decât numărul cu care rulează matricea, fluxul așteaptă următorul declanșator, astfel încât rulează la viteza mai mică dintre cele două.
- **Camerele lipsă sunt menționate, nu sunt completate retroactiv.** O cameră care nu a putut fi deservită la acel declanșator este absentă din dicționar și listată în `frames.errors`.
- **Se încheie brusc.** În cazul unui refuz, `FrameFetchError` se va repeta la următorul ciclu, după 60 de secunde fără cadre sau după 60 de secunde fără un nou declanșator.**Ce oferă.**`frame_stream()` transferă cadre numpy întregi prin conexiunea HTTP locală și atinge o viteză maximă de aproape**170 MB/s**. Acest lucru este suficient pentru procesarea în timp real a cadrelor brute, a unui binned sau a unui array ROI, dar nu pentru fiecare declanșare la rezoluție maximă — pentru a păstra**fiecare** declanșare, salvați pe disc cu [`continuous()`](#continuoushandle). Măsurat pe o matrice de patru camere (FOCN, FRGN, FRGB, F450) cu transport compactat; fiecare tic a reprezentat un set complet de un singur declanșator:

| | Senzor complet (matrice 14,2–14,4 fps) | Binning 2 × 2 (matrice ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 seturi/s | 22,2 seturi/s |
| `frame_stream(processing="debayered")` | 2,3 seturi/s | 9,3 seturi/s |
| `frame_stream(processing="display")` | 0,8 seturi/s | 3,0 seturi/s |
| MJPEG pe cameră, 4 fluxuri la 720p | 13,5 fps fiecare; fiecare cadru corelat cu un declanșator | ~25 fps fiecare; ~45 % dintre declanșatoare prezente pe toate cele 4 |

Utilizați `processing="raw"` pentru debit și demosaic pe partea dvs.; nivelul `debayered` elimină, de asemenea, offsetul de întuneric al pachetului de calibrare, ceea ce nu se întâmplă în cazul demosaic-ului `raw`.

#### Fluxuri MJPEG per cameră (fără RTSP)

**RTSP nu este acceptat.** Interfața de streaming este MJPEG prin HTTP (`multipart/x-mixed-replace`, un fișier JPEG pe parte, fiecare parte având un `Content-Length`), o adresă URL pe cameră:

| URL | Ce servește |
| --- | --- |
| `GET /api/camera/stream/<serial>` | Previzualizare pe 8 biți, până la ~30 fps. Pentru un element al matricei, `max_h` (implicit 720, 64–2160) și `jpeg_q` (implicit 80) stabilesc dimensiunea și calitatea. |
| `GET /api/camera/<serial>/stream-annotated` | Aceeași previzualizare cu suprapunerile GUI desenate pe ea (zebra, grilă, reticul, histogramă, peaking, spot meter). `ChlorosLocal.camera_stream()` o citește pentru tine. |
| `GET /api/camera/<serial>/stream-reflectance` | Reflectanță de 8 biți pe bandă — consultați [Fluxul de reflectanță în timp real](#live-reflectance-stream-mjpeg). |

- **Antete de declanșare.** Pentru un element al matricei, fiecare parte a tuturor celor trei fluxuri conține `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` și `X-Chloros-Timestamp-Ns`. Părțile provenite de la camere diferite, dar care au același `X-Chloros-Group-Id`, constituie același declanșator: colectați părțile și grupați-le pe baza acelui antet pentru a reasambla un set. O cameră autonomă nu are grup: părțile sale de previzualizare nu conțin niciunul dintre aceste antete (componentele sale de reflectanță poartă un `X-Chloros-Group-Id` gol).
- **Fluxul de previzualizare pierde cadre sub sarcină.** Acesta transmite cel mai recent cadru și omite orice cadru pe care nu l-a putut trimite la timp, astfel încât, la rate ridicate ale matricei, nu fiecare declanșator ajunge la fluxul fiecărei camere (vezi tabelul de mai sus). Pentru fiecare declanșare, utilizați `frame_stream()` sau `continuous()`.
- **`/api/camera/stream/<serial>` se întrerupe cât timp fluxul live combinat al matricei este deschis** (`/api/camera/array/stream/<array_id>` — vizualizarea matricei din interfața grafică îl deschide). Fluxul fiecărui membru trimite apoi un cadru de rezervă „Pauză (fluxul matricei activ)”, fără antete `X-Chloros-*`, și nimic altceva până când fluxul combinat se închide; apoi se reia de la sine. `stream-annotated` și `stream-reflectance` nu se pun în pauză. Închideți vizualizarea matricei din interfața grafică atunci când un script citește fluxurile de previzualizare pentru fiecare cameră.
- **Pentru numere, nu imagini, utilizați `frame_stream()`.**Formatul JPEG este cu pierderi și pe 8 biți; `frame_stream()` returnează aceleași produse fără codec în calea de acces.

### Captură la intervale pentru inspecții cu drone

Un zbor de inspecție necesită un grup de imagini sincronizate la fiecare câțiva metri parcurși, nu fiecare cadru pe care matricea îl poate produce. Două setări controlează acest lucru:**`target_fps`**stabilește frecvența cu care camerele captează un cadru, iar**`capture_repeated(interval_s=…)`** stabilește frecvența cu care un cadru este salvat.**Camerele continuă să transmită în flux între salvări.** La fiecare interval se salvează cel mai recent grup sincronizat; cadrele intermediare nu sunt înregistrate. Expunerea automată se ajustează la fiecare cadru, astfel încât expunerea este deja stabilizată în momentul salvării. Nu există un mod în care camerele să se oprească între fotografii și să fie reactivate pentru fiecare dintre ele: în acest caz, fiecare fotografie ar trebui să aștepte pornirea transmisiunii și stabilizarea expunerii.**Setați rata de cadre la valoarea necesară pentru interval.**Dacă se lasă pe modul automat, un array rulează la cea mai mare rată pe care o suportă gazda și rețeaua. Setarea unei rate mai mici reduce încălzirea camerei, încărcarea rețelei și a procesorului gazdei, fără a modifica conținutul salvat. Alegeți o rată care să ofere**2–4 cadre pe interval**:

- Fiecare salvare așteaptă următorul cadru după ce a fost solicitată, astfel încât un grup salvat ajunge cu o perioadă de un cadru după momentul programat. O rată mai mare menține acest decalaj mic.
- Expunerea automată se ajustează o dată pe cadru, astfel încât o rată foarte scăzută încetinește viteza cu care expunerea urmărește un nor care trece.

**Calculați intervalul pe baza planului de zbor:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

De exemplu, o amprentă de 60 m cu o suprapunere înainte de 80 % înseamnă un spațiu de 12 m; la 10 m/s, aceasta corespunde lui `interval_s=1.2`, iar un `target_fps=3` fixat oferă între 3 și 4 cadre pe interval. Captarea se bazează pe timp: SDK-ul nu are un mod declanșat de distanță sau de GPS, așa că mențineți o viteză la sol constantă pe liniile de supraveghere.

**Salvați datele brute pe aeronavă și calibrați ulterior.** `interval_s` se măsoară de la începutul fiecărei salvări, astfel încât o salvare care durează mai mult decât intervalul face ca următoarea să înceapă cu întârziere, iar distanța dintre cadre să se modifice. Salvarea datelor brute împreună cu citirea senzorului de lumină (`processing="raw"`; citirea este salvată în mod implicit) menține fiecare salvare scurtă și asigură că fiecare cadru poate fi convertit ulterior în reflectanță.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**fixează rata de la început;**`arr.set_target_fps(fps)`**o modifică în timpul zborului fără a se reconecta, iar**`arr.set_target_fps(None)`** o readuce la modul automat. O rată mai mare decât cea pe care o pot suporta gazda și rețeaua este redusă la acea limită maximă — verificați `clamped` în rezultat — iar rata minimă este de 0,1 fps.
- **Nu folosiți `burst()` pentru a reduce numărul de cadre.** Acesta scrie fiecare cadru pe care îl produce matricea; urmează un `target_fps` fixat, așa că este instrumentul potrivit doar atunci când se dorește fiecare cadru.
- **`target_fps` nu repară cadrele corupte.** Reducerea acestei valori diminuează numărul de cadre generate, nu viteza de transmitere a fiecăruia; pentru cadrele corupte de GVSP, reduceți în schimb valoarea `wire_ceiling_mbps` (consultați [Starea matricei](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Conectarea la o matrice deja conectată — `attach_array`

Dacă matricea este deja activă (a fost deschisă de interfața grafică sau o sesiune SDK anterioară a apelat `connect_array`), utilizați `attach_array` pentru a obține un identificator al acesteia, în loc să vă reconectați. `connect_array` generează întotdeauna eroarea „Camera  <sn>se află deja în matrice<id>”, deoarece trimiterea comenzii POST cu `/array/connect` pentru un element din grup nu este idempotentă; `attach_array` citește `/api/camera/array/list` și face potrivirea fie după array_id, fie după seriale.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Model: scripturile SDK-ului care funcționează în comun cu interfața grafică de desktop ar trebui să încerce mai întâi `attach_array` și să recurgă la `connect_array` dacă nu există încă nicio matrice în grup.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Un array atașat este împrumutat, nu deținut.** Un identificator provenit de la `attach_array` nu dezactivează niciodată matricea: nici ieșirea din blocul `with`, nici o eliberare explicită prin `disconnect()` nu o eliberează (apelul explicit afișează o linie pe stderr indicând că a lăsat matricea în funcțiune). Aceeași regulă ca pentru `CameraSession` / `DAQSensorSession`. Așadar, utilizarea în comun cu GUI-ul este sigură, cu sau fără `with`:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Când doriți cu adevărat să o încheiați — pentru a vă reconecta cu un alt `wire_ceiling_mbps`, de exemplu — apelați `arr.force_disconnect()`. Reconectarea după un simplu `disconnect()` pe un handle atașat eșuează cu mesajul „Camera  se află<sn> deja în matrice<id>”, deoarece nimic nu a fost eliberat.

### Asistent de analiză a rețelei

Util înainte de deschiderea matricei — estimează dacă setările propuse se vor potrivi:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Nu pornește niciodată un backend: unul care nu răspunde generează eroarea `ChlorosConnectError`, în timp ce o cerere refuzată este returnată ca rezultat. O eroare `packed` generează eroarea `ValueError`.

`status` este unul dintre `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (altfel `error` — inclusiv o solicitare refuzată: autentificare neefectuată, `error_code: AUTH_REQUIRED`, sau un plan sub pragul minim al SDK, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` înseamnă că rezoluția solicitată se potrivește cu inelul RX doar la o rată de declanșare limitată — păstrează rezoluția și treci de la `target_fps=result["recommended"]["recommended_target_fps"]` la `connect_array` (vezi [Exemplul 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` apare doar atunci când cadrul solicitat nu poate fi livrat la nicio rată de cadre; connect nu reduce niciodată cadrele din proprie inițiativă, așa că aplicați `recommended` manual.

**Cum se interpretează proiecția** (același model ca în panoul „Setări matrice” din interfața grafică):

- **Burst (`frame_bytes_total`) este însumată pe cameră, la formatul real de pixeli al fiecărei camere.**Camerele**M3M**mono transmit Mono12 (2 biți/pixel) sau, cu un `pixel_format`, formatul mono al adâncimii sale (`"BayerRG8"` este `Mono8`, 1 B/px), astfel încât, fără acesta, un cadru la rezoluție maximă cu 4 camere are**~25 MB** cu trei camere mono, nu ~12,6 MB, cum ar rezulta dintr-o ipoteză de 8 biți pentru toate. Backend-ul determină formatul fiecărei camere pe baza modelului acesteia.
- **Admitanța (`burst_fits_nic_ring`) ține cont de capacitatea de evacuare**, nu se bazează pe comparația între „burst complet” și „inel”: emisia simulată se potrivește atunci când gazda golește inelul RX mai repede decât îl umplu camerele. O gazdă de 10G + camere de 1 GbE**admite** rezoluția maximă chiar și atunci când burst-ul depășește capacitatea inelului; un host de 1 GbE blochează (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` reprezintă o limită maximă conservatoare pentru recuperarea serială** — `max(readout+emit, N×emit)` cu emisia per cameră limitată la legătura de cameră de 1 GbE, independent de expunere. De exemplu, ~2,8 fps pentru o matrice de 4 camere la rezoluție maximă de 12 biți (corespunde valorilor măsurate în timpul rulării de ~2,7–3,0). Model complet: [Referință CLI→ Model de fps și rafale pentru matrice](cli-reference.md#array-fps--burst-model).
- **După o captură continuă, proiecția indică valoarea la care s-a stabilit.** O captură de 30 s sau mai mult la o rată automată înregistrează rata la care s-a stabilit pentru acea dimensiune a matricei pe acest computer; încercările scurte ale controlerului de rată la o rată mai rapidă sunt omise. De atunci încolo, linia din `array-connect`, analiza din `connect_array()` și panoul „Setări matrice” menționează acea rată atunci când aceasta este sub cea a modelului, împreună cu ceea ce a limitat-o în `cycle_limiter`: `"clock"` (matricea a funcționat la rata la care s-a stabilit, bucla de captură așteptând ceasul de declanșare), `"loop"` (bucla de captură nu mai avea timp), `"capture"` (captura a rămas în urmă și a omis grupuri) sau `"measured"` (nu se specifică care dintre acestea). Valoarea modelului se găsește în `wire_fps_max`. Matricea continuă să crească până la valoarea pe care o suportă cablul.
- **Supra-abonarea (`oversubscribed: true`) înseamnă că cablul nu poate suporta matricea la niciun ritm atins de camere** — câmpurile fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) indică 0, iar binningul nu poate remedia această problemă (aceasta reduce numărul de octeți pe cadru, nu numărul de octeți ritmați pe secundă). O conexiune lentă, în sine, nu provoacă acest lucru: în schimb, rata de cadre scade, iar dacă camerele nu pot fi ritmate suficient de lent, nivelul devine `slip-emit-and-capture` (serial). Soluțiile constau în reducerea numărului de camere, utilizarea cadrelor jumbo sau o placă de rețea mai rapidă; `max_cams_collision_safe` indică limita maximă a cablului și a camerelor. Răspunsul conține, de asemenea, `aggregate_demand_bps`, `collision_safe_ceiling_bps` și `per_cam_floor_bps`. Consultați [Supra-abonare](#over-subscription-the-per-cam-floor).

### Descoperire și listare

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Fiecare acceptă doar cuvinte-cheie `backend_url=` și `timeout=30.0` și returnează o listă de dicționare. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) acceptă și `auto_start_backend=True`; apelurile `list_*` nu pornesc niciodată un backend.

---

## Smart-AE / Smart-Capture

Matricile LATTICE rulează AE continuu în fundal imediat ce sunt conectate, dar o scenă nou orientată are nevoie de un moment pentru a converge. **Smart-capture** este soluția convenabilă integrată: interoghează expunerea fiecărei camere, așteaptă până când matricea este stabilă pe o fereastră, apoi declanșează capturarea. Este echivalentul din interfața grafică: butonul de captură „smart” al aplicației desktop apelează același punct final al backend-ului.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Când utilizați `ChlorosProject` (secțiunea următoare), aveți la dispoziție mai multe opțiuni de configurare:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Politica smart-AE este conservatoare în mod implicit. Strângeți `exposure_tolerance_pct` pentru lucrări radiometrice exigente; lărgiți-o pentru scene care se schimbă rapid, unde doriți doar „suficient de aproape”.

---

## Sesiuni de senzori DAQ

Grup persistent de backend-uri pentru senzori spectrali (DAQ-U prin USB, DAQ-M prin BLE, DAQ-E prin Ethernet). Reflectă suprafața camerei: detectare inteligentă, reutilizarea grupului, atașare idempotentă.

### Detectare inteligentă (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioritate: Ethernet → BLE → USB. Transmiteți orice indiciu explicit pentru a fixa transportul.

### Transport fixat

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` numește un singur senzor, iar numai acel senzor este deschis vreodată. Acesta este comparat cu senzorii deja deschisi, apoi se solicită numele de gazdă sau adresa IP a unui DAQ-E (un DAQ-E pe care scanarea de rețea îl vede doar după adresa IP este interogat, în mod numai citire, pentru numele său, adresa MAC și numărul de serie, astfel încât oricare dintre acestea îl găsește, de asemenea), adresa MAC BLE a unui DAQ-M și ID-ul pe care un DAQ-U sau DAQ-M îl raportează odată deschis (orice alt dispozitiv deschis pentru verificare este închis din nou). Dacă nu este găsit, `connect_daq_sensor` generează `ChlorosConnectError`, listând ceea ce a fost găsit; împreună cu o adresă (`port=` / `mac=` / `eth_host=`) care conține un senzor diferit, se generează, de asemenea, o eroare. Nu se revine niciodată la senzorul vizibil — pentru asta este destinată apelarea cu indicatorul zero.</id></sn></id></sn>

### `connect_daq_sensor()` Semnătură

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Un senzor care se află deja în bază de date este returnat așa cum este (împrumutat): `integration_time`, `frame_avg` și `enable_ae` nu i se aplică. Deconectați-l mai întâi pentru a-l deschide cu setările dvs. `cap_id` reprezintă excepția: pe un senzor împrumutat, acesta este setat cu `set_cap()`. Dacă este omis, se aplică valoarea implicită a backend-ului (consultați nota de sub tabelul cu metode).

### Metode `DAQSensorSession`

| Metodă | Descriere |
| --- | --- |
| `status(timeout=10.0)` | Rezumatul intrării în pool (starea de streaming/înregistrare, intervalul de lungimi de undă, sha de calibrare, timpul de integrare, frame_avg, starea AE). Pentru un DAQ-E al cărui flux a încetat: `reconnect` (în cazul în care reconectarea automată a backend-ului este activă: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` în timp ce transmite), `stream_lost_s`, `reconnects` și `last_reconnect`, precum și `record_gaps` pentru înregistrarea în curs; `rx_stream_fault` indică starea de reconectare până la sosirea primului spectru. |
| `latest(*, n=1, timeout=10.0)` | Returnează până la N cadre de spectru cele mai recente (fiecare `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Reluare / pauză transmisie (mânerul rămâne deschis). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Pornește înregistrarea unui fișier .daq. Returnează calea către fișier. Nu este acceptat pentru DAQ-U/M fără un pachet de calibrare AWS (DAQ-E este exceptat). |
| `record_stop()` | Oprește înregistrarea. Returnează `{path, rows, warning, gaps}`; `warning` este `None` pentru o înregistrare curată, în caz contrar, afișează diagnosticul backend-ului (de exemplu, un fișier gol sau o lacună în flux), care este, de asemenea, afișat pe stderr. `gaps` listează fiecare interval în care fluxul nu a livrat nimic: `start_utc`, `end_utc` (`None`: încă pierdut la oprire), `duration_s`, `reconnected`, `marked_in_file`. Este gol în cazul unei înregistrări continue. |
| `set_cap(cap_id, timeout=15.0)` | Setează profilul de corecție a plafonului aplicat citirilor începând cu următoarea: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"` sau `"none"` (sau `None`) pentru un senzor fără capac. Returnează, care este citit înapoi de `cap_id`. Un DAQ-E-S menține valoarea `as_recorded` indiferent de cerere (difuzorul său este încorporat). Generează `ChlorosConnectError` pentru un profil care se încarcă. CLI-ul: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Numai pentru DAQ-E: reactivează PTP pentru o unitate al cărei PTP este dezactivat (`status()["rx_ptp_enabled"]` este `False`; setarea este salvată pe unitate, astfel încât rămâne dezactivat până la reactivare). Returnează starea actualizată. Generează `ChlorosConnectError` pentru un DAQ-U / DAQ-M sau când unitatea nu confirmă. Chloros nu reactivează niciodată o unitate de la sine, deoarece dezactivarea poate fi intenționată. CLI-ul: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Numai pentru DAQ-E: setările proprii ale unității. `firmware` (`running`, `bundled`, `update_available`), `ip` și `ip_mode` (`dhcp` / `linklocal` / `static`: ceea ce rulează acum), `network` (setarea salvată: `mode`, plus `ip` / `mask` / `gateway` în regim static), `network_change_pending` (valabil până când unitatea rulează setarea salvată, una salvată cu `set_network(..., reboot=False)` inclusă), `auto_stream`, `ptp`. `firmware["update_available"]` este `None` atunci cândversiunea unității nu a putut fi citită. În timp ce fluxul unității nu ajunge la acest computer, acesta este citit pe o conexiune nouă, iar dicționarul adaugă `stream_silent` (`True`), `stream_fault` și `reconnect`; o unitate care nu răspunde generează o eroare. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E numai: DHCP (`"auto"`) sau o adresă statică. Repornește unitatea pentru a aplica setarea și o reconectează; această sesiune continuă să funcționeze, iar `address` urmărește unitatea. `reboot=False` doar salvează setarea. Consultați [Setările dispozitivului DAQ-E](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Numai DAQ-E: transmite în flux la pornire fără software conectat (salvat pe unitate). Returnează setarea confirmată. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Numai pentru DAQ-E: repornește unitatea și o reconectează cu setările pe care le avea. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Numai pentru DAQ-E: actualizează firmware-ul inclus în această versiune Chloros pe un dispozitiv care rulează o versiune mai veche, apoi reconectează-l (aproximativ un minut; nu-l opri). Returnează `updated`, `fw_before`, `fw_after`, `bundled` și un `warning` atunci când unitatea a revenit lent sau a revenit cu o altă versiune; `updated` este `False` pentru o unitate care are deja versiunea actuală. Generează `ChlorosConnectError` pentru o unitate a cărei versiune nu poate fi citită. CLI: `pool-update-firmware`. |
| `disconnect()` | Eliberare din pool. Fără-op pentru mânerele atașate, dar care nu sunt deținute. Pe un senzor deschis de această sesiune este `force_disconnect()` și generează `ChlorosConnectError` când eliberarea nu este confirmată (ieșirea din blocul `with` nu generează niciodată o excepție). |
| `force_disconnect(timeout=30.0)` | Eliberarea senzorului indiferent de cine l-a deschis — de exemplu, un senzor lăsat în pool de un script care s-a blocat, la care un script nou ajunge doar ca un identificator atașat al cărui `disconnect()` nu face nimic. Închiderea senzorului altui client încheie orice înregistrare pe care acesta o efectuează. Se întoarce odată ce backend-ul confirmă eliberarea sau când nu mai este nimic de eliberat (senzorul nu se află în pool sau nu rulează niciun backend); o eliberare care a depășit timpul de așteptare sau care a fost refuzată de backend generează `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Atribute: `sensor_id`, `model`, `transport`, `address` (urmează unui DAQ-E pe care `set_network()` l-a redirecționat), `info` și `backend_url`.

> **Denumiți capacul.** DAQ-U, DAQ-M și DAQ-E sunt livrate fără condensator, iar Chloros nu poate vedea dacă este montat unul: un senzor conectat fără `cap_id` primește `sunshine_cosine`, o presupunere pe care înregistrările sale indică că au făcut-o (`cap_id_source=auto_default`). Un DAQ-U fără capac, sub acea corecție, afișează valori de aproximativ 30 de ori mai mari, așa că transmiteți `cap_id="none"` pentru un senzor fără capac (sau `set_cap("none")`), iar profilul capacului pe care l-ați montat în caz contrar.

### DAQ-E-S — difuzor fix și senzor de înclinare

**DAQ-E-S** este un DAQ-E cu două diferențe importante pentru software. Tot restul — detectarea, conectarea, PTP, înregistrarea, setările dispozitivului de mai jos — funcționează exact ca în cazul unui DAQ-E.**Difuzorul său este permanent, așa că nu necesită nicio corecție a capacului.** Difuzorul de tip Sunshine este încorporat și se afla pe unitate atunci când a fost măsurată calibrarea din fabrică, astfel încât calibrarea indică deja iradianța reală descendentă. Chloros își setează limita la `as_recorded` (fără profil) indiferent de cerință: o limită denumită astfel (`sunshine_cosine`, `fov_*`, `none`) este ignorat, fiind afișat un mesaj, iar înregistrările sale stochează `cap_id_source=model`, astfel încât avertismentul privind capacul presupus nu se declanșează niciodată în cazul său. Nu aplicați manual o limită datelor sale: `sunshine_cosine`, de exemplu, ar indica o valoare de aproximativ 11 ori mai mare.

**Este echipat cu un accelerometru pe 3 axe.** Răspunsul cosinus al difuzorului de lumină solară este valabil doar cât timp acesta este orizontal, astfel încât un senzor înclinat raportează o iradianță eronată — iar reflectanța moștenește eroarea — fără alte simptome. Fiecare citire DAQ-E-S conține, prin urmare, orientarea sa:

| Câmp | Semnificație |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Unghiuri în grade. `None` atunci când firmware-ul marchează unghiul ca fiind nedefinit (cădere liberă, o axă defectă). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Accelerație în mili-g. `mag_mg` departe de ~1000 înseamnă că unitatea era în mișcare și unghiurile nu sunt relevante pentru acea probă. |
| `cal_applied` | Set: axele și unghiurile sunt în sistemul de referință al difuzorului (se aplică calibrarea de montare a unității). Clear: sistemul de referință brut al plăcii, care poate afișa o înclinare de până la ~2,3°, care nu este reală. |
| `sample_age_ms` | Vechimea eșantionului accelerometrului la momentul trimiterii spectrului. |
| `temp_raw` | **Tendința** necalibrată a temperaturii cipului — doar diferențe, niciodată o citire a termometrului. |

`DAQSensorSession.latest()` le returnează sub cheia `imu` a fiecărui cadru (`None` pe un DAQ-U, DAQ-M sau DAQ-E simplu). Un `.daq` înregistrat de la un DAQ-E-S stochează aceleași valori în coloanele sale `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) lângă fiecare spectru, astfel încât cititorul să poată distinge un nor de un trepied lovit. La toate celelalte modele, aceste coloane sunt goale.

Fluxul de date privind atitudinea raportează în propriile câmpuri din `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), separate de cele spectrale, astfel încât o întrerupere a accelerometrului să nu fie niciodată interpretată ca o lacună în înregistrare. `discover_daq_sensors()` și `model` ale sesiunii indică `DAQ-E-S`.

### Setări dispozitiv DAQ-E

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Adresa statică trebuie să se afle în rețeaua acestui computer.** `set_network` generează `ChlorosConnectError` pentru un dispozitiv a cărui subrețea nu include adresa acestui computerpe rețeaua senzorului: după repornire, nimic nu va putea accesa senzorul pentru a reveni la setările inițiale. Treceți la `force=True` numai atunci când acest computer va fi redirecționat sau rutate către noua subrețea.
- **`set_network`, `reboot` și `update_firmware` repornesc senzorul** și generează `ChlorosConnectError` în timp ce acesta înregistrează, și atunci când acesta nu revine în termen de `comeback_timeout` secunde. ID-ul senzorului și numele de gazdă `daq-e-<id>.local` nu se modifică niciodată.
- **O înregistrare începută chiar înainte de repornire este încheiată de acesta și i se atribuie un nume.** Când senzorul revine, dicționarul returnat conține `recording_closed` (calea către fișier) și un `warning` care indică acest lucru. Când senzorul nu revine sau transferul de firmware eșuează, `ChlorosConnectError` indică acest lucru în schimb: mesajul său se încheie cu avertismentul, iar `e.recording_closed` / `e.warning` sunt setate (ambele `None` atunci când nicio înregistrare nu a fost încheiată). Reporniți înregistrarea odată ce senzorul revine.
- **Doar senzorul în sine este reconectat.** După repornire, unitatea trebuie să răspundă acolo unde se afla -- și, după `set_network("auto")`, și sub numele său `.local` -- cu numele, numele de gazdă și modelul, de la adaptorul de rețea (MAC) pe care îl avea; `address` devine adresa IP care a răspuns. Un răspuns sub numele respectiv este valabil doar cu acea adresă MAC, astfel încât de la un computer care accesează senzorul prin intermediul unui router sau din rețeaua NAT implicită a WSL2, un senzor mutat de DHCP nu este reconectat (`ChlorosConnectError`; reconectați-l la noua sa adresă).
- **O actualizare de firmware necesită reconectarea senzorului.** `update_firmware()` solicită unitatea, iar aceasta deschide o conexiune TCP înapoi către acest computer pentru a descărca imaginea, pe un port ales de sistemul de operare. Backend-ul ascultă pe adaptorul care comunică cu unitatea (pe fiecare adaptor, doar atunci când nu poate identifica care anume) și deservește doar acea unitate: orice altă conexiune este refuzată (`[DAQ-FW] … refused a firmware connection from <ip>` în ieșirea backend-ului) în timp ce acesta rămâne în așteptare. Un firewall care blochează conexiunea eșuează actualizarea cu `device never connected back on TCP <port>`. Programul de instalare Windows adaugă regula necesară (vezi [Pasul 1](#windows-exe)); pe un alt firewall, permite traficul TCP de intrare către backend din subrețeaua senzorului.
- **În WSL2, utilizați rețeaua oglindită** (`networkingMode=mirrored` în `.wslconfig`): în modul NAT implicit, fiecare adresă statică este refuzată (adresa proprie a WSL nu se află în rețeaua senzorului), fluxul nu ajunge la WSL, iar actualizarea firmware-ului nu se poate finaliza.
- Interfața grafică (GUI) menține propria conexiune cu un senzor: o repornire din script încheie sesiunea GUI cu acesta. Interfața grafică are aceleași setări (fila „Light Sensors”, setările senzorului, „Network”).

### Descoperire — găsirea unei adrese la care să vă conectați

`discover_daq_sensors()` scanează porturile USB / BLE / ETH în căutarea senzorilor pe care *ai putea* să îi deschizi. Este echivalentul DAQ al `discover_lattice_cameras()` și singura modalitate de a obține **adresa MAC BLE a unui DAQ-M** — un DAQ-E are un nume de gazdă, iar un DAQ-U un port COM, dar adresa MAC nu este imprimată pe dispozitiv și nici listată de sistemul de operare.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Câmp | Descriere |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Port COM / MAC BLE / nume gazdă — transmite către `connect_daq_sensor` ca `port=` / `mac=` / `eth_host=`. |
| `display` | Etichetă lizibilă pentru om. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, sau `None`. Un senzor Ethernet este `DAQ-E-S` sau `DAQ-E`, așa cum se identifică atunci când este solicitat, în mod numai pentru citire, în timpul scanării (`DAQ-E` atunci când nu răspunde la timp). Un port serial USB afișează `DAQ-U` atunci când cipul său de punte este unul dintre cele utilizate de DAQ-U (FTDI, CH340, CP210x, Prolific) — un indiciu din descrierea portului, nu o sondă — iar `None` pentru orice alt dispozitiv USB, care este totuși listat, astfel încât îl puteți transmite ca `port=`. Un port serial fără ID de furnizor USB și fără un astfel de cip de punte (un UART de pe placa de bază, cum ar fi `/dev/ttyS*` dLinux, o legătură serială Bluetooth, Intel AMT) nu poate fi un DAQ-U și nu este listat. |
| `extra` | Detalii specifice fiecărui tip de transport (numele anunțat BLE; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Valorile goale sunt omise. |

| Parametru | Implicit | Descriere |
| --- | --- | --- |
| `transports` | toate cele trei | Secvență (sau șir CSV) care limitează scanarea. Merită specificat când știi ce vrei — BLE este partea lentă. |
| `scan_timeout` | 5 | Fereastra de scanare pe transport, în secunde; backend-ul limitează valoarea între 1 și 20. |
| `timeout` | 60,0 | Limita maximă HTTP pentru întregul apel (la fel ca în restul SDK-ului). |
| `auto_start_backend` | `True` | Pornește un backend local dacă nu rulează niciunul. Nu pornește niciodată pentru un `backend_url` la distanță. |

> **Senzorii deja deschisi în pool nu apar.** Un periferic BLE conectat încetează să mai transmită anunțuri, iar un port COM deschis nu poate fi detectat, așa că lista de descoperire afișează ceea ce este *disponibil pentru conectare*. Este de așteptat un rezultat gol imediat după ce ați conectat ceva — utilizați `list_daq_sensors()` pentru ceea ce dețineți deja. Transporturile a căror scanare nu poate rula (nu este instalat bleak / zeroconf) sunt omise în loc să genereze o eroare, astfel încât o mașină fără Bluetooth primește în continuare răspunsurile USB și ETH.

### Listare

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Care DAQ alimentează ce cameră — `daq_bindings()`

Reflectanța unei capturi și `.daq` pe care o salvează utilizează DAQ-ul asociat acelei camere sau matrice, altfel se folosește cel mai recent DAQ disponibil în backend. Selecția senzorului de lumină din GUI și `chloros-cli daq pool-bind` o configurează; la fel și:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` returnează `{"cameras": {serial: b}, "arrays": {array_id: b}}`: fiecare `b` conține `binding` (valoarea setată), `source` (`project`, din `sensors.json` al proiectului sau `runtime`), `known` / `known_as` (indiferent dacă cheia camerei sau a matricei denumește ceva cunoscut de Chloros, și cum: `connected`, `project` — `cameras.json` al proiectului deschis — sau, pentru o cameră, `seen`: deschis anterior pe acest computer; `known: False` pentru o cheie care nu denumește nimic) și `resolved` (senzorul de pool la care este asociat acum: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). O legare la rulare are prioritate față de cele din proiectpentru sesiunea respectivă; fișierul de proiect nu este modificat. Deconectarea unei camere, a unui array sau a unui DAQ anulează toate legăturile cu acel hardware, indiferent de cine le-a stabilit (membrii unui array sunt legați de array-ul respectiv).

`set_daq_bindings()` leagă doar o cameră care este conectată, se află în proiectul deschis sau a fost deschisă anterior pe acest computer, precum și o matrice care este conectată sau se află în proiectul deschis (sau o cheie deja legată). Orice altă cheie generează `ChlorosConnectError`, denumind-o (`400`, `code`, `UNKNOWN_BINDING_TARGET`, cu tastele `unknown`), iar **nimic** din acel apel nu este asociat. `None` șterge orice cheie.

### Reflectanța mediului ambiant + a obiectului — `light_sensor_reflectance()`

Două DAQ-uri grupate, unul îndreptat spre cer (mediu ambiant) și unul spre țintă (obiect), furnizează reflectanța țintei și indicii de vegetație pe baza celor mai recente citiri: perechea „Mediu ambiant + Obiect” din interfața grafică și `chloros-cli daq reflectance`, fără a mai deschide senzorii a doua oară. Ambele trebuie să fie conectate și să transmită date în flux continuu.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Benzi.** Reflectanța fiecărei benzi este iradianța integrată a benzii obiectuluiintegrată a obiectului față de cea a mediului ambiant. `daq_band_presets(filter_code=None)` listează seturile de benzi: câte unul pentru fiecare filtru LATTICE M3C (`RGN`, `OCN`, `NGB`, modelul de benzi propriu al camerei, astfel încât un NDVI DAQ și un NDVI al camerei reprezintă aceeași mărime) și `GENERIC` în bandă largă. `suggested` este presetarea corespunzătoare `filter_code`; în caz contrar, se utilizează filtrul primei camere conectate (`suggested_from` indică care anume).
- **Returnează** `bands` (pe bandă: `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (cele încorporateîncorporate), `custom` (fiecare cu `value` sau `valid: False` și `error`), `invalid_bands`, `reflectance_type`, `preset` și cele două valori ale `timestamps` (`ts_ns`). O bandă pe care senzorul de mediu nu a putut-o măsura (în afara intervalului său de măsurare, la pragul de zgomot) este invalidă, cu menționarea motivului, iar fiecare indice care are nevoie de aceasta este omis, nefiind niciodată calculat față de zero.
- **Aparent sau relativ.** Indicii dependenți de scară (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) necesită un raport absolut, astfel încât sunt generați numai atunci când ambii senzori sunt calibrați în funcție de putere (fiecare având un `calibration_bundle_sha`): `"Apparent Reflectance"`. În caz contrar, rezultatul este `"Relative Reflectance"`, conținând doar indicii invarianți la scară.
- **Două grile de lungimi de undă diferite** sunt comparate în funcție de suprapunerea lor: punctele senzorului ambiental din interiorul razei de acțiune a obiectului, spectrul obiectului interpolat pe acestea.
- **Doi senzori, niciodată unul singur.** Denumirea unui singur senzor ca fiind ambii (același ID, indiferent de ortografie acceptată de pool) generează `ChlorosConnectError`: citirea unui senzor asupra lui însuși este 1,0 pe fiecare bandă, ceea ce nu măsoară nimic. Nici backend-ul nu acceptă acest lucru (`POST /api/daq/indices` răspunde cu `400`, `code` la `SAME_SENSOR`, în timp ce `ambient_sensor_id` și `object_sensor_id` trimise de SDK sunt un singur senzor din grup, de exemplu ID-ul și portul acestuia).

### Utilizare comună cu CLI-ul și alte scripturi

Grupul de senzori DAQ al backend-ului este partajat de fiecare client CLI și SDK de pe mașină. Dacă un senzor se află deja în grup — deschis de `chloros-cli daq pool-connect` sau de un alt script — apelarea `connect_daq_sensor(port="COM3")` din Python returnează un identificator împrumutat: comanda `disconnect()` nu produce nicio acțiune, astfel încât scriptul tău nu smulge senzorul de sub clientul care l-a deschis la ieșirea din scope. Același lucru este valabil și pentru `sensor_id=` (tastat cu `:` sau `-`, ambele variante sunt acceptate), pentru un DAQ-E identificat prin adresa IP atunci când a fost deschis prin numele de gazdă (sau invers), precum și pentru detectarea inteligentă cu indicatorul zero, atunci când backend-ul răspunde cu un senzor pe care îl deținea deja — un bloc `with` eliberează întotdeauna doar un senzor pe care l-a deschis el însuși.

Interfața grafică Chloros **nu** face parte din acest pool: panoul său DAQ gestionează propria conexiune la senzor, astfel încât `connect_daq_sensor` nu împrumută niciodată un senzor pe care interfața grafică îl are deschis. Nu utilizați același senzor atât din interfața grafică, cât și dintr-un script în același timp. Un DAQ-U sau DAQ-M pe care interfața grafică îl deține în mod normal nu se deschide, dar un DAQ-E acceptă o a doua sesiune: conexiunea scriptului modifică setările de achiziție ale unității din cadrul GUI, iar `disconnect()` oprește fluxul unității, ceea ce pune capăt vizualizării live a GUI și oricărei înregistrări pe care o efectuează. Închideți mai întâi senzorul din GUI.

### PTP Grandmaster — `time_sync_status()`

Backend-ul rulează PTP Grandmaster-ul de la care camerele LATTICE și unitățile DAQ-E preiau marcajele de timp. Acesta pornește la cerere: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` și `connect_all()` îl pornesc dacă nu rulează — inclusiv în cazul unei conexiuni care eșuează ulterior, cum ar fi `connect_all()` care încearcă să se conecteze la un DAQ-E care nu există — la fel procedează și `test_array_sync()` și fiecare apel care deschide camerele pentru o comandă (`capture_cameras_transient()`, `start_multi_camera_stream()`, apelurile de aliniere, `capture_live_index()`).

- **Unde se anunță.** Pe adaptoarele de rețea cu cablu conectate la camere (inclusiv o adresă link-locală `169.254.x.x`), altfel pe adaptorul rutei implicite, dacă acesta este conectat prin cablu. Niciodată pe Wi-Fi sau pe un adaptor virtual (Tailscale, WSL): nu există nicio cameră sau DAQ-E în spatele acestora. În absența unui astfel de adaptor activ, acesta rulează anunțând pe nimic (`health` `NO_SOCKETS`) și se leagă de adaptorul pe care îl activează următoarea cameră sau DAQ conectată.
- **După o întrerupere a conexiunii.** Când adaptorul camerei își pierde conexiunea și o recuperează (în urma smulgerii unui cablu, înlocuirii unui comutator sau re-enumerării unui adaptor USB), „grandmaster”-ul își reconstruiește singur socket-urile, fără a fi necesară o repornire, și verifică din nou fiecare DAQ-E conectat, reactivând unul dintre cele rămase blocate. De asemenea, se reconstruiește atunci când toate camerele și DAQ-E-urile pe care le deservea rămân inactive timp de 15 secunde.
- **Ajunge la cablu?** Citiți `time_sync_status()` (`GET /api/time-sync/status`). `state` indică doar că firul de execuție al grandmaster-ului rulează; `health` indică dacă pachetele sale sunt transmise: `OK` (nu s-a detectat nicio eroare), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (unele adaptoare prezintă defecțiuni) sau `PEERS_SILENT` (camerele și dispozitivele DAQ pe care le deservea au încetat să mai răspundă), cu o mențiune în `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` numără doar mesajele care au fost transmise; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` și `last_delay_req_age_s` descriu cele care nu au fost trimise, iar `rebinds` / `last_rebind` reconstrucțiile. `sync_sends_slow` numără sincronizările a căror transmisie a depășit 200 us, iar `sync_send_max_us` este cea mai lungă; timpul de origine al fiecărui Follow_Up înregistrează cel mult 200 us din transmisia sa. Grandmasterul rulează într-un proces propriu (`own_process`, `process_pid`), cu prioritate ridicată pe Windows (`process_priority`), astfel încât un backend ocupat nu poate întârzia timpii pe care îi înregistrează și nu poate sincroniza ceasurile tuturor camerelor simultan; dacă acel proces nu poate fi pornit, acesta rulează în interiorul backend-ului, ca și până acum (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funcție | Ce face |
| --- | --- |
| `time_sync_status()` | Starea, integritatea, identitatea ceasului, prioritățile și contoarele de mesaje ale grandmasterului (mai sus). `chloros-cli time-sync status`. |
| `time_sync_peers()` | Slave-urile PTP pe care le detectează (camere, DAQ-E): `peers`, fiecare `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Fiecare cameră matricială conectată: `PtpStatus`, `PtpOffsetFromMaster` și `PtpMeanPathDelay` (o cameră al cărei firmware nu are noduri PTP are doar numărul de serie). `time-sync cameras`. |
| `restart_time_sync()` | Oprește și repornește grandmaster-ul; PTP-ul fiecărui DAQ-E din grup este verificat din nou după (`daq_ptp_rechecked`). Generează `ChlorosConnectError` atunci când nu revine. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Prioritățile sale PTP (0-255; valoarea mai mică câștigă alegerea de master principal), pentru o rețea cu un alt master PTP. Necesită ca acesta să fie în funcțiune. `time-sync set-priority`. |

Fiecare preia `backend_url=` și `timeout=` și pornește un backend local care nu rulează.

---

## Utilitare pentru camere

Ce face `chloros-cli lattice` în afară de conectare și captură, ca funcții simple în `chloros_sdk` (modulul `chloros_sdk.camera_utils`). Fiecare apelează ruta backend-ului folosită de comanda CLI, cu regulile CLI-ului, astfel încât cele două se comportă în mod similar. Acestea acceptă un **număr de serie** al camerei, nu o sesiune, și aceleași cuvinte-cheie `backend_url=` / `timeout=` ca și apelurile smart-connect; un backend care nu rulează este pornit, așa cum face `connect_camera()`. Un refuz generează `ChlorosConnectError` cu motivul indicat de backend.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funcție | Ce face |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Profilul de culoare al ecranului: `raw`, `linear`, `natural`, `enhanced` sau `custom_temp`, precum și temperatura sa de culoare (limitată la 2000-10000 K; citită atunci când profilul este `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Afișează saturația și contrastul ecranului, fiecare între 0 și 200 % (100 = neschimbat); `reset=True` le readuce pe amândouă la 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Reduce luminozitatea celor mai luminoși și mai albi pixeli ai afișajului: -100..0 (trunchiat spre zero, apoi limitat; 0 = oprit). |
| `camera_white_balance_once(serial, settle_s=None)` | Echilibrare a albului într-o singură etapă: camera transmite un flux video scurt, converge și returnează `red_ratio` / `blue_ratio` (`ratios_read` False când nu le poate raporta). Păstrează balansul de alb pe care l-a determinat; toate celelalte setări scrise de „open” sunt readuse la valorile inițiale. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (continuu) sau `"off"`; sau rapoarte de echilibru fixe `red` / `blue`, care dezactivează mai întâi modul automat. Un raport pe care camera îl refuză este denumit sub `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Presetările camerei și aplicarea uneia dintre ele: camera păstrează setările corespunzătoare denumirilor presetărilor (răspunsul este `settings`, așa cum le citește camera ulterior, de exemplu rata de cadre la care funcționează; `None` pentru un câmp pe care presetarea nu îl numește), iar toate celelalte setări scrise de programul de control revin la valorile inițiale. `"default"` resetează camera la setările implicite din fabrică (setările din fabrică cu expunere automată liberă, limita de transfer a legăturii și corecția defectelor dezactivate, balansul de alb calibrat al unei camere RGB păstrat), fără a fi salvate în memoria sa: răspunsul conține `factory_defaults: True` și toate câmpurile din `settings`. În `camera_presets()`, `default` este `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Modelele de filtre și detaliile unui model (benzi, numărul de condiție al răspunsului spectral). Modelul de filtru al unei camere este citit din șirul său de model; acesta nu este setat manual. |
| `camera_firmware_status(serial=None)` | Firmware-ul fiecărei camere: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` și `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Instalează firmware-ul acestei versiuni Chloros pe o cameră: resetare la setările din fabrică, instalare, repornire, rescrie setările de bază Chloros; calibrarea camerei este păstrată. Câteva minute; camera trebuie să rămână alimentată. `force=True` reinstalează firmware-ul pe o cameră existentă. |
| `camera_power_probe(serial)` | Nodurile de alimentare și termice ale camerei, precum și temperatura acesteia. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Modul de repaus cu consum redus de energie al camerei (rămâne în stare de repaus până la trezire: limita de transfer a legăturii este activată (`DeviceLinkThroughputLimitMode` `On`) la valoarea minimă, modul de economisire a energiei este activat, dacă camera dispune de unul), cu temperatura înainte și după; trezirea readuce legătura la starea în care se afla la intrarea în repaus (limita, modul și întârzierea pachetelor `GevSCPD`, pe care camera o menține la 0 cât timp limita este activă; înregistrarea este păstrată pe acest computer, astfel încât o trezire din partea unui backend ulterior o găsește și ea), sau, pentru o cameră pe care acest computer nu a pus-o în repaus, dar care pare a fi în repaus (limita sa activată la pragul minim), cu limita dezactivată și un `GevSCPD` lăsat la 0 la 80 (setările din fabrică); o limită setată peste valoarea minimă este lăsată așa cum este. Se păstrează doar starea de alimentare; toate celelalte setări ale căror valoare a fost modificată sunt readuse la valorile inițiale. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Capturează cadre până când o țintă de calibrare MAPIR apare în câmpul vizual și ajustează coeficienții pe canal din panourile acesteia: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Adaptorii Ethernet ai acestui computer pentru GigE Vision (MTU, viteza de legătură, cadre jumbo) și orice `issues`; `estimate=True` adaugă rata de cadre și lățimea de bandă pe care le-ar obține camerele `cameras`. |
| `fix_network_settings()` | Aplică setările adaptorului GigE Vision la adaptorul de cameră al acestui computer. Modifică setările adaptorului și funcționează numai când Chloros rulează ca administrator; în caz contrar, afișează un mesaj de eroare cu jurnalul a ceea ce a găsit. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Corectează vignetarea imaginilor dintr-un folder, fiecare cu propria hartă de câmp plat a obiectivului: `processed`, `lens_models` (obiectivul pentru care a fost corectat fiecare fișier), `skipped` (cu motivele corecției). Fără `lens_model`, obiectivul fiecărui fișier provine din câmpul „Model” din datele EXIF; un fișier `lens_model` se aplică fișierelor al căror model nu menționează nicio cameră, iar un fișier al cărui model contrazice acest lucru este omis. Numai fișiere. |
| `read_dls(mac, filter=None, camera_serial=None)` | O citire de la un DLS (senzor Bluetooth de lumină descendentă): spectrul, iradianța pe bandă și magnitudinile de bandă prin răspunsul propriu al camerei atunci când se specifică `camera_serial`. `discover_daq_sensors(transports=["ble"])` identifică adresa MAC. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Hardware-ul gazdei pentru camere și configurația recomandată de acesta (`report_text`), precum și CPU-ul / GPU-ul / mediul de execuție PyTorch al sistemului. |

- **Unele au nevoie de cameră doar pentru ele.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` și `update_camera_firmware()` deschid camera. În timp ce Chloros o deține (conectată în aplicație, cu `connect_camera()`, ca membru al unui array sau utilizată în prezent de o altă comandă, cum ar fi un test de matrice sau o captură care își deschide camerele), backend-ul refuză cu codul HTTP 409 `CAMERA_HELD` (atât în `code`, cât și în `error_code`), indicând modul de eliberare a acestuia. Setările de culoare, zonele luminoase, `camera_power_probe()` și citirile funcționează pe o cameră conectată.
- **Culoarea, zonele luminoase și balansul de alb sunt destinate camerelor RGB.** Pe o cameră monocromă cu o singură bandă (M3M), funcțiile de setare nu modifică nimic și returnează `{"skipped": True, "reason": ...}`, la fel ca și CLI. Acestea sunt setări de afișare: vizualizarea în timp real și capturile la nivel de afișare, niciodată radianța sau reflectanța.

---

## Matrice tranzitorii, profiluri de aliniere și indici

Ce face `chloros-cli lattice` fără o conexiune persistentă, la fel ca funcțiile din `chloros_sdk` (modulul `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` și `index`. Fiecare trimite cererea pe care o trimite comanda CLI, cu valorile implicite ale CLI.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Acestea deschid singure camerele.** Fiecare comandă își deschide camerele, își îndeplinește sarcina și le închide, așa că are nevoie de camere pe care Chloros nu le deține (conectate în aplicație, cu `connect_camera()` sau ca element al unui array); backend-ul refuză cu codul HTTP 409 `CAMERA_HELD` și indică modul de eliberare a unei camere. O deschidere tranzitorie durează ceva timp (aproximativ 40 de secunde pentru câteva camere). Pentru capturi repetate, `connect_array()` o singură dată este calea rapidă și salvează fiecare tip de export, în timp ce o captură tranzitorie salvează o imagine debayered. `calibrate_alignment_profile()` este excepția: pe o matrice conectată a acestor camere, rezolvă problema pornind de la propriile cadre ale matricei (`used_live_array`).
- **`serials`** sunt camerele care trebuie deschise, prima declanșând comanda; `None` deschide toate camerele din rețea.
- **`settings`** este un dicționar de setări de suprascriere pentru camere, câmpurile setate de indicatorii CLI: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` și `preset` (o presetare a camerei după nume). O expunere sau un câștig specificat dezactivează controlul automat, la fel ca și indicatorii.
- **`gpu`**: `True` forțează warp-ul GPU, iar `False` pe cel al CPU-ului (`--gpu` / `--no-gpu`); `None` lasă backend-ul să aleagă.
- **`keep_settings`**/**`clear_settings`** (fiecare funcție care deschide camerele, implicit `False`): la final, fiecare cameră este readusă la starea în care a fost găsită; `keep_settings=True` salvează setările sesiunii în memoria proprie a camerei (funcționare liberă), `clear_settings=True` resetează memoria respectivă la setările din fabrică — vezi [Memoria camerei](#camera-memory--keep_settings--clear_settings). Răspunsul (pentru o sarcină, al lui `stop()`) conține `camera_memory`.
- **Căile sunt cele ale backend-ului**: cele relative sunt transformate în absolute față de directorul dvs. de lucru atunci când backend-ul se află pe această mașină. Funcțiile fișierului de profil de aliniere (`alignment_profile_info()`, `reorder_alignment_profile()`) citesc fișierul de pe acest computer, la fel cum o face și CLI.

| Funcție | Ce face |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Înregistrează continuu de pe o singură cameră până la oprire. Returnează un `CliJob`. Cadrele unei rulări `tiff` constituie fișierul `raw`, un tablou de capturi ale acelei camere (un plan de mozaic la valorile proprii ale senzorului, necomprimate); `png` și `jpg` conțin 8 biți, astfel încât aceste serii reprezintă cadre demosaicate pentru vizualizare. Un DAQ legat sau grupat scrie un fișier `.daq` pentru fiecare cadru în folderul seriei`daq/` (`daq_files` în statisticile sale). Fiecare cadru este scris o singură dată împreună cu metadatele complete (model, ora capturii, obiectiv, benzi, calibrare pe un cadru brut); `stop()` se returnează o dată ce fiecare cadru deja capturat a fost scris (până la 25 s), iar abia atunci camera este oprită. O cameră lăsată în stare de armare pentru un declanșator hardware (`TriggerMode` `On`) este setată pe modul „free-run” pentru sesiune (răspunsul de start `trigger` indică ce s-a găsit). La oprire, camera este readusă la starea în care a fost găsită — setările de declanșare și toate setările modificate în timpul rulării — cu excepția cazului în care se utilizează `keep_settings=True`, care salvează setările rulării în memoria proprie a camerei (funcționare liberă), sau `clear_settings=True`, care resetează memoria respectivă. O sesiune care nu înregistrează niciun cadru se încheie automat după 10 s cu `stop_reason` `no_frames`, iar motivul este indicat în `last_error`, iar comanda `stop()` returnează acest motiv ca `error`. Cadrele pierdute deoarece dispozitivele de scriere nu au putut ține pasul sunt numărate, comanda `drop_reason` indicând motivul. Pentru o cameră pe care o deține Chloros, utilizați `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Transmite grupuri de cadre sincronizate la `fps`, pentru grupuri `count` sau până la oprire. Returnează un `CliJob` (acesta `start_reply` are `n_cams` și firul `pacing`); statisticile sale finale numără grupurile preluate, scrise și abandonate, fișierele pe care scriitorul nu le-a putut scrie și grupurile pentru care o cameră nu a livrat nimic. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | O captură sincronizată, o imagine debayered per cameră: `results` (`filepath`, `serial`, `frame_id`) și `pacing`. Apare un grup fără o cameră, indicând mai întâi câte fișiere au fost salvate. `lattice multi-capture` fără matrice conectată. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` capturi sincronizate la o expunere fixă: `sync_groups`, `master`, un rând `group_timestamps` per declanșare (o cameră care a ratat o captură lipsește din rândul respectiv) și `sync`: decalajul fiecărei a doua camere față de camera principală, declanșare cu declanșare (`offsets_us`: pe cameră `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), pe UN singur ceas (`clock_domain`: `ptp` sau `device` — PTP dezactivat, ceasul de funcționare propriu al fiecărei camere, astfel încât decalajul său corespunde momentului pornirii, iar doar jitterul măsoară sincronizarea). Un declanșator ale cărui marcaje combină ceasul dispozitivului și PTP (PTP dezactivat sau încă în curs de sincronizare) este omis și luat în calcul (`groups_mixed_domain`), la fel ca unul în care un ceas PTP este încă la o distanță mai mare de `agree_within_us` față de ceasul principal (`groups_clocks_disagree`); `notes` indică acest lucru. `clock_offset_mean_us` / `clock_offset_jitter_us` reprezintă media și jitterul maxim pe cameră. `success` este fals, împreună cu `error`, atunci când s-a ratat un declanșator, când mai puțin de două declanșatoare au toate camerele pe același ceas sau când ceasurile nu se sincronizează niciodată (`sync.problems`); rândurile sunt returnate în orice situație. Pornește grandmasterul PTP dacă acesta nu rulează (`time_sync` indică `grandmaster_running` și `grandmaster_started_by_this_test`), activează PTP-ul fiecărei camere (așa cum face `connect_array()`) și, înainte de primul declanșator, așteaptă ca ceasul PTP al fiecărei camere să se sincronizeze cu grandmasterul și să se stabilizeze: `PtpStatus` Slave pe un singur grandmaster, la o abatere de maximum 100 us față de acesta și stabil timp de 3 s, iar fiecare cameră să se sincronizeze astfel, fără întrerupere, timp de 120 s — aproximativ 2 minute de la pornire. Un ceas care tocmai s-a sincronizat prezintă o abatere de până la ~200 us timp de aproximativ 2 minute după ce afișează starea „Slave”, iar propriul `PtpOffsetFromMaster` al camerei nu indică acest lucru; 120 s aduc camerele la o abatere de ~10 us față de un ceas stabilizat. O cameră care este încă sincronizată de la un test anterior care a lăsat PTP activat, așa cum a observat acest backend, începe numărătoarea de la acel moment de sincronizare, astfel încât un test efectuat imediat după altul nu mai așteaptă din nou; o cameră pornită de test sau una găsită cu PTP activat, pe care acest backend nu a văzut-o sincronizată (pornită în altă parte sau înainte de o repornire a backend-ului), așteaptă cele 120 s, iar într-un set care conține ambele camere, cele care tocmai s-au sincronizat așteaptă în continuare timpul lor. O cameră care iese din starea de Slave, depășește 100 us sau urmează un alt grandmaster pentru 3 citiri consecutive (1,5 s) în timpul așteptării își reia numărătoarea; una sau două astfel de citiri sunt înregistrate (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), iar numărătoarea continuă, iar un ID de grandmaster care nu poate fi citit este considerat necunoscut, nu ca o schimbare. Repornirea simultană a numărării tuturor camerelor (în decurs de o secundă) indică faptul că timpul „grandmaster”-ului se modifică, nu al camerelor: jurnalul backend-uluiindică `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` îl listează în `grandmaster_steps`, iar o așteptare care se epuizează apoi își începe eroarea cu `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Limita nu este extinsă pentru acesta. `ptp_settle_s` (0-600; `0` pentru lipsa așteptării de stabilizare) setează timpul de stabilizare pentru execuție; o altă valoare este refuzată (400 HTTP). Așteptarea este limitată la timpul de stabilizare plus 90 s (210 s în mod implicit, cel puțin 90 s), iar apelul așteaptă backend-ul `timeout` s: în mod implicit 240 plus timpul de stabilizare (360 s), la fel ca `test_array_sync` al lui `chloros-mcp`. `ptp_lock` are `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (cât timp fiecare cameră a fost blocată și stabilă împreună), `found_settled` (camerele erau deja blocate pentru timpul de stabilizare când au fost găsite) și, pentru fiecare cameră, `status`, `offsets_ns`, `slave_s` (cât timp a fost sincronizată; Niciuna: nu este sincronizată) și `remaining_s` (Niciuna atunci când PTP-ul niciunei camere nu se activează). Jurnalul backend-ului conține o linie la fiecare 5 s cu starea fiecărei camere, cât timp a fost sincronizată și cât timp mai are. Fiecare cameră se pornește cu PTP dezactivat; de acolo, ceasurile se sincronizează în aproximativ 10 s. Ulterior, fiecare cameră este readusă la starea în care a fost găsită în timpul testului — PTP-ul, expunerea automată și amplificarea sunt menținute dezactivate pe durata testului (`ExposureAuto` / `GainAuto` și o variantă manuală `ExposureTime` / `Gain`), iar restul, cu excepția cazului în care `keep_settings=True` — adică o cameră lăsată cu PTP activat (`PtpEnable=True` sau o execuție cu `keep_settings=True`) rămâne blocată, iar odată ce acest backend a constatat că aceasta a rămas blocată pentru timpul de stabilizare, un test ulterior nu mai așteaptă aceasta. Dacă ceasurile nu se stabilizează, `success` este fals înainte de orice declanșare, iar `error` indică ce trebuie făcut: o cameră care nu s-a sincronizat niciodată (`not_slave`) înseamnă că grandmasterul nu o accesează (`time_sync_status()`); una care a continuat să se deconecteze este identificată prin durata cât a fost sincronizată (`remaining_s`); ceasurile încă converg (`far`): verificați `time_sync_status()`, iar pentru a le menține sincronizate înainte de test, activați PTP și lăsați-l activat (`PtpEnable=True` pe fiecare cameră sau `keep_settings=True`; testul restabilește setările inițiale, așa că rămâne activ), apoi rulați-l. `lattice multi-test`. |
| `detect_array_master(line=None)` | Camerele de pe linia de sincronizare și un master implicit: `master_serial`, `slave_serials`, `line` au fost detectate. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Ce gamă variată de modele de camere (denumiri de produse, `"LATT-M3C-L87-FOCN"`) ar apărea în rețeaua acestui computer: `tier`, `achievable_fps_max`, bugetul de cabluri (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` și `fix_suggestions`. În mod implicit, modelele camerelor detectate; nicio cameră nu este deschisă. Pentru camerele pe care le dețineți, cu setările pe care le-ar utiliza o conexiune, consultați [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Rezolvați un profil de aliniere pe baza cadrelor live și salvați-l. `method`: `orb`, `akaze`, `phase`, `checkerboard` (împreună cu `checkerboard="9x6"`, colțurile interioare) sau `manual` (împreună cu `control_points={serial: [[x, y], ...]}`, inclusiv cele ale camerei de referință). `model`: `translation`, `rigid`, `affine` sau `homography`. Opțiunile mai detaliate sunt opțiunile CLI: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` este `{serial: {option: value}}`. `rms_threshold_px` refuză să salveze o soluție mai slabă. Returnează `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (pentru fiecare cameră `reprojection_error`, `num_matches`) și `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | O singură imagine multibandă aliniată pe profil: `files` (fișierul TIFF multibandă cu `bands` și dimensiunea sa, plus câte una pe bandă cu `per_band` și câte una pe cameră cu `per_camera`) și `vignette_corrected`. `bands` selectează și `order` ordonează benzile după nume (o listă sau un șir de caractere separate prin virgulă). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Aliniază imagini multibandă în flux (aceleași opțiuni de aliniere ca `capture_aligned_multiband()`), pentru `count` sau până la oprire. Returnează un `CliJob`; statisticile sale finale includ `frames_written` și orice `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Un fișier de profil rezumat ca `align-info` îl afișează astfel: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (pentru fiecare cameră `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (pentru fiecare cameră: `band_names`, `wavelengths_nm`, `enabled`); `profile` reprezintă întregul fișier. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Modifică fișierul de profil pe loc: `order` reprezintă numele benzilor (fiecare mută stratul camerei sale mai sus, în această ordine), `enable` / `disable` sunt numerele de serie. Returnează noul `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Presetările index (`presets`: fiecare `name`, `channels`, `formula`) și de culoare `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Un index pentru un fișier TIFF multibandă aliniat: un `preset` (`NDVI`, `NDRE`, ...) sau un `formula` asupra simbolurilor `channel_map` care se mapează la benzi (`{"NIR": "NIR_850", "Red": 0}`, după nume sau index). Un simbol de presetaresimbolul `channel_map` omite și preia singura bandă din fișier al cărei nume începe cu acesta (`red` -> `Red_660`, `nir` → `NIR_850_RGN`), denumită de fișierul însuși (lista sa de benzi sau `Chloros:Band{R,G,B}Name` a unei capturi/produse Chloros), așa că NDVI-ul dintr-o captură de reflectanță Chloros nu are nevoie de `channel_map`; două benzi corespunzătoare unui singur simbol sunt respinse, nu sunt presupuse. `capture_level` indică conținutul benzilor: `reflectance` sau `radiance` transformă valorile în cantitative; o stivă necalibrată (cea a lui `align-apply`) oferă valori calitative; `"unknown"` citește datele din fișier, inclus fiind un fișier Chloros, `Chloros:ProcessingLevel`. Returnează `stats` (forma, intervalul de percentile 1-99, intervalul LUT, `capture_level` utilizat) și `files` scris lângă intrare (`<input stem>_<preset>.tif`, `..._LUT.png`) sau la `output`, o cale: cu o extensie de imagine, fișierele sunt denumite pornind de la rădăcina lor (`output="D:/out/ndvi_FRGN.png"` scrie `ndvi_FRGN.tif` și `ndvi_FRGN_LUT.png`), iar fără aceasta este un prefix (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); citiți căile de la `files`. Acestea conțin înregistrarea camerei sursei de intrare (marcă / model / număr de serie, GPS, obiectiv, ora capturii în UTC, identitatea de calibrare, dar niciodată calibrarea raw-DN a acesteia), `Chloros:ProcessingLevel=index`, iar indexul ca singura lor bandă (`XMP-Camera:BandName`, atât fișierul TIFF, cât și fișierul PNG LUT). Doar fișiere; fără cameră. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Captură, calibrare (`reflectance` sau `radiance`; `raw` o omite), aliniază și calculează un index într-un singur pas; `save_multiband=True` păstrează stiva calibrată, pe care `compute_index()` o poate prelua ulterior. Returnează `stats` (împreună cu `serials` ale cărui benzi le-a utilizat) și `files`. Deschide `serials` (prima declanșează declanșatorul), altfel camerele profilului a căror intrare `layer_order` este activată, mai întâi cea de referință (deschisă chiar și cu benzile sale dezactivate); o cameră ale cărei benzi sunt dezactivate nu este niciodată calibrată. `channel_map` este necesar doar pentru un simbol la care stiva nu poate răspunde, așa cum se vede în `compute_index()`: simbolul unei presetări preia singura bandă activată al cărei nume începe cu acesta, verificată înainte ca orice cameră să se deschidă (dacă nu răspunde niciuna sau răspund mai multe, acestea sunt refuzate, fiind numite); un `formula` personalizat este necesar pentru fiecare simbol. `index.tif` și LUT-ul conțin camera din stivă, timpul de captură (expunerea camerei de referință, UTC) și obiectivul, iar ambele denumesc indexul ca fiind singura lor bandă (`XMP-Camera:BandName`, de exemplu `NDVI`: fișierul PNG color indică indexul, nu canalele sale de culoare). `lattice index --live`. |

### `CliJob`

Handle-ul returnat de cele trei fluxuri. Sarcina rulează în backend, care deține controlul asupra camerelor până la oprirea sa.

| Element | Descriere |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (cadre salvate / scrise, eliminate, `fps`, `elapsed`, eșecuri de scriere, `stop_reason`; o sarcină continuă include, de asemenea, `frames_grabbed` și `frames_queued` — fiecare cadru capturat este scris, pus în coadă, eliminat sau a eșuat — și `drop_reason`, `writers` și `write_ms`, timpul mediu necesar pentru scrierea unui cadru) și `output_dir`. `state` este `running`, `ended` (s-a oprit singur și nu a fost încă oprit), `stopping` (un proces de oprire își scrie coada de așteptare: statistici în timp real și `stopped_by`) sau `finished` (statisticile sale finale, `error` și `stopped_by` ale procesului oprit). Înregistrarea unei sarcini oprite este păstrată timp de 10 minute (cel mult ultimele 32), pentru fiecare client: un alt script sau `chloros-mcp` citește aceeași stare și aceleași statistici finale. O sarcină `chloros-cli` a fost pornită (`lattice continuous`, `multi-stream`, `align-stream`) este oprit de backend atunci când CLI-ul nu mai raportează timp de 30 de secunde — a fost oprit, mașina sa fiind în stare de repaus — cu `stop_reason`, `client_gone` și `stopped_by` pe backend; citirea statisticilor sale de aici nu îl menține activ. O sarcină pornită din SDK rulează până când este oprită. Fiecare pornire trimite `timeout` ca termen limită al backend-ului (`client_deadline_s`): o cameră care se deschide după acesta (una încă în curs de pornire) este repusă și închisă, iar nicio sarcină nu este pornită, iar apelul generează `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). O cameră deținută de o sarcină este refuzată oricărei alte deschideri (`connect_camera()`, o altă pornire, o captură tranzitorie) cu codul HTTP 409 `CAMERA_HELD`, specificând sarcina și persoana care a inițiat-o. |
| `stop(timeout=30.0)` | Oprește-o și închide-i camerele: `final_stats`, `output_dir` și `error` atunci când oprirea nu s-a desfășurat corect. Un al doilea apel returnează același răspuns, la fel ca și o oprire inițiată de un alt client: una trimisă în timp ce prima încă își scrie coada de așteptare (până la 25 de secunde; după acest interval se generează o eroare, iar statisticile finale urmează pe `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Așteaptă până se termină de la sine (când se atinge `count`, unitatea este plină sau apare o eroare), apoi `stop()` pentru a colecta valorile finale. `timeout` generează `TimeoutError` în timp ce încă rulează; o sarcină fără `count` se termină doar atunci când este oprită. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | ID-ul și tipul său (`continuous`, `multi-stream`, `align-stream`), locul unde scrie, răspunsul rutei de pornire și dacă `stop()` a fost executat. |

Utilizați-l ca manager de context pentru a-l opri la ieșirea din domeniul de aplicare.

---

## Dosarul proiectului și rețetele

**Dosarul proiectului** este dosarul în care se creează proiectele noi, partajat de aplicația Chloros, CLI și SDK (modulul `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Un folder configurat care nu mai există revine la valoarea implicită (`Documents/MAPIR/Chloros Projects`), la fel ca aplicația. O cale relativă specificată pentru `set_project_folder()` este preluată din directorul dvs. de lucru atunci când backend-ul se află pe această mașină.

**O rețetă** este lista de acțiuni în format YAML / JSON pe care `chloros-cli project run PATH RECIPE` o execută pentru un proiect salvat. `run_project_recipe()` o execută cu propriul runner al CLIpropriu (instalat odată cu SDK), astfel încât o rețetă se comportă la fel indiferent dacă este executată de:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** este un fișier `.yaml` / `.yml` / `.json` sau rețeta însăși: un dicționar cu `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) și, opțional, `overrides` și `stop_on_error` (implicit true: execuția se oprește la prima acțiune care eșuează). Acțiunile sunt cele din [Recipe DSL](cli-reference.md#recipe-dsl) din referința CLI; un `output` relativ este rezolvat în funcție de directorul de lucru al scriptului tău.
- **Returnează** `ok` (toate acțiunile au reușit), `actions` (câte), `log` (ieșirea programului de execuție; `echo=True` o afișează, de asemenea, pe măsură ce rulează) și `validation`. Un fișier de rețetă care lipsește sau nu poate fi analizat generează `ChlorosConnectError`.
- **Verificat înainte de rulare.**`validation` este `{"ok", "errors", "warnings", "steps"}`: fiecare acțiune trebuie să aibă exact un singur verb cunoscut ca cheie (`{"capture": {"name": ...}}`, nu `{"action": "capture"}` sau `{"type": "capture"}`) care denumește o cameră, o matrice sau un senzor din proiect, de tipul corespunzător. O rețetă cu o eroare**nu** execută nicio acțiune (`ok` false, erorile din `log`), indiferent de ce indică `stop_on_error`. Cheile neutilizate, precum și orice matrice sau senzor de lumină pe care rețeta îl controlează și care nu este conectat în prezent, sunt `warnings`. O rețetă nu conectează dispozitive: conectați mai întâi proiectul (`ChlorosProject.connect_all()`, `chloros-cli project connect`, sau instrumentul MCP `connect_project_devices`). `dry_run=True` nu execută nimic și returnează `{"recipe": ..., "validation": ...}`.
- Într-un proces se execută o singură rețetă la un moment dat. O rețetă YAML necesită PyYAML.

---

## Automatizarea proiectului — `ChlorosProject`

Un proiect Chloros salvat este un folder care conține `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` încarcă manifestul (`relink`: folderul în care se află acum fișierele sursă lipsă ale unui proiect mutat, utilizat atunci când backend-ul îl încarcă — `process()` — vezi **Un proiect mutat** sub `ChlorosLocal`), iar `connect_all` conectează online fiecare cameră, matrice și senzor de lumină salvate cu setările lor salvate — aceeași stare a hardware-ului pe care ar produce-o interfața grafică. Senzorii de lumină se conectează prin [pool-ul de senzori DAQ](#daq-sensor-sessions), așa că au nevoie de un backend care rulează (vezi [`SensorHandle`](#sensorhandle)).

### Exemplu minim

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Sau ca manager de context:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metode `ChlorosProject`

| Metodă | Descriere |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Descoperă și conectează fiecare cameră, matrice și senzor de lumină salvate. Returnează un raport de conectare pentru fiecare clasă (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, fiecare cu `connected`, `missing` și `errors`; `arrays` include și `alignment`). Cu un backend care ascultă pe `127.0.0.1:5000`, camerele și matricile salvate sunt găsite și conectate prin intermediul acestuia. În absența acestuia, acest proces le găsește și le controlează singur prin `lattice_sdk`, care necesită mediul de execuție Arena SDK și OpenCV (`cv2`) — pachetul SDK nu instalează niciunul dintre acestea; atunci când nu poate face acest lucru, fiecare cameră și matrice salvate sunt listate sub `errors` împreună cu motivul, iar `missing` rămâne gol. Un backend care rulează, dar refuză SDK-ul (deconectat: HTTP 401; un plan sub nivelul Copper: 403) nu înseamnă „fără backend”: fiecare cameră, matrice și senzor de lumină salvate sunt listate sub `errors` împreună cu motivul indicat de backend, iar nimic nu este deschis în acest proces. Fiecare apel verifică din nou backend-ul, astfel încât `connect_all()`, după autentificare sau după pornirea Chloros, se conectează la același obiect de proiect. O matrice salvată deja activă în backend cu aceleași camere (ale GUI-ului sau ale lui `chloros-cli project connect`) este atașată așa cum este, fără a se reaplica setările sale salvate, iar `disconnect_all()` îl lasă să ruleze. Senzorii de lumină trec întotdeauna prin backend (vezi [`SensorHandle`](#sensorhandle)). Nu generează niciodată un backend. |
| `disconnect_all()` | Oprește totul. Fiecare dispozitiv este eliberat mai întâi; apoi, dacă backend-ul nu a confirmat eliberarea unei camere, a unui array sau a unui senzor de lumină (timpul de așteptare pentru oprire a expirat sau a fost refuzat), se atribuie câte un nume `ChlorosConnectError` fiecăruia. Un dispozitiv deja eliminat, sau pentru care nu rulează niciun backend, rămâne inactiv. O matrice sau un senzor de lumină pe care un alt client le-a deschis rămâne în funcțiune. Ieșirea `with` a proiectului nu se declanșează niciodată. |
| `capture_all(output_dir=".")` | Un cadru de la fiecare cameră + matrice conectată, și un spectru (`SensorHandle.read()`) de la fiecare senzor de lumină conectat, preluate pe rând (mai întâi camerele, apoi matricile, apoi senzorii), nu simultan. |
| `stream(camera, overlays=False, fps=10.0)` | Generator care produce cadre BGR `numpy` de la o cameră (sau matrice) numită. `overlays=False` citește cadrele cu identificatorul`grab()` la nivelul `debayered`: prin backend atunci când proiectul este conectat printr-unul, altfel o buclă directă de captare `lattice_sdk` (matricile generează dicționare `{serial: frame}`). `overlays=True` redirecționează prin `ChlorosLocal.camera_stream()` → fluxul MJPEG al backend-ului `/api/camera/<serial>/stream-annotated`, cu blocul `ui.overlay` salvat de cameră transmis ca parametri de interogare. Necesită modul backend, OpenCV (`cv2`) și o **cameră autonomă**: o cameră în mod direct generează `RuntimeError` (backend-ul nu poate prelua o cameră deținută de acest proces), iar un array generează `NotImplementedError` (suprapune compoziția pentru fiecare cameră — transmite un element după nume). Echivalentul pentru o singură execuție: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Execută alinierea pentru fiecare matrice conectată în prezent. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Execută pipeline-ul de calibrare/indexare pe imaginile proiectului (încorporează `ChlorosLocal.process`; aceste patru sunt **singurele** argumente cheie acceptate — `indices=` etc. generează o excepție `TypeError`; setează indicii prin `ChlorosLocal.configure()`). Construiește în mod leneș un `ChlorosLocal()`, care pornește automat un backend. |

Atribute:
- `proj.cameras` — `Dict[str, CameraHandle]` indexat după nume ȘI număr de serie.
- `proj.arrays` — `Dict[str, ArrayHandle]` indexat după nume ȘI array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` indexat după nume ȘI slot_id.
- `proj.config` — `project.json["config"]` dicționar.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Niveluri de procesare.** `capture()`, `grab()` și `frame_stream()` acceptă toate același token `processing`, iar lanțul este cumulativ — fiecare nivel rulează tot ce se află deasupra lui:

| Nivel | Ieșire | Note |
| --- | --- | --- |
| `raw` | Bayer cu 1 canal, nativ pentru senzor | Fără demosaic. Suprapunerile nu sunt disponibile la acest nivel. |
| `debayered` | 3 canale color (**implicit** pentru `CameraHandle` și fiecare `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` și `stream()` au ca setare implicită `raw`): BGR în memorie (`grab()` / `frame_stream()`), salvat ca RGB prin `capture()` | În modul backend, ca captură: offsetul de întuneric (DSNU) din pachetul de calibrare aplicat asupra datelor brute, urmat de demosaicarea sensibilă la margini a OpenCV — fără LUT de liniaritate, câmp plat sau demixare. Singurul nivel care funcționează fără modul backend este demosaicarea biliniară simplă a bibliotecii camerei (fără offset de întuneric). Fișierul TIFF salvat este RGB fotometric, așa că inversați canalele unui cadru din memorie (`frame[..., ::-1]`) pentru a-l compara cu un fișier. |
| `radiance` | float32, W/m²/sr/nm, **un plan pe bandă** | Lanț radiometric complet: demosaic + separare 3×3 (multispectral) + DSNU + câmp plat + scală NIST, cu expunerea × amplificarea eliminate astfel încât valorile să fie absolute. |
| `reflectance` | uint16, 32768 = 1,0 (sau uint8, 255 = 1,0 — vezi `bit_depth`), **un plan pe bandă** | Radianța împărțită la iradianța descendentă (ρ = π·L/E). Necesită o citire DLS/DAQ — vezi nota de mai jos. |
| `display` | 8 biți, similar sRGB | Redare echivalentă cu interfața grafică: CCM + balans de alb + gamma prin profilul de culoare activ al camerei. |

Numai în modul direct (fără backend) este disponibil `debayered`: `grab()` la un alt nivel anulează
`NotImplementedError`, `capture()` ignoră `processing`, iar `frame_stream()` generează
`RuntimeError` la fiecare nivel. `reflectance` are nevoie de o citire descendentă utilizabilă — punctul final al cadrului trage
DAQ-ul grupat în slotul DLS al camerei automat, dar fără un DAQ asociat, lanțul refuză
ieșirea de reflectanță și marchează în mod sincer retrogradarea în metadatele returnate, în loc să
returneze în tăcere un produs de calitate inferioară.

> **Scala DN a reflectanței — nu o codificați rigid.** Reflectanța LATTICE utilizează `32768` = ρ 1,0 și înregistrează
> XMP `Chloros:PixelScale=32768`; reflectanța Survey3 utilizează `65535` = ρ 1,0 și nu conține
> etichete `Chloros:*`. Citește eticheta și împarte la valoarea acesteia. Este definită în domeniul uint16, așa că rămâne
> `32768` pentru fiecare format care se redimensionează (TIFF pe 16 biți, TIFF pe 8 biți, PNG/JPG pe 8 biți, procent pe 32 de biți)
> — normalizați mai întâi tipul de date stocat înapoi la uint16 (×257 din 8 biți, ×65535 din float). Singura
> excepție este un fișier dintr-o versiune anterioară 1.3.0: un fișier TIFF de 8 biți era *tăiat* atunci, nu redimensionat, deci
> nicio scală nu îl descrie și nu conține niciun tuple `PixelScale` sau MicaSense. Tratați o etichetă lipsă dintr-un
> fișier de reflectanță LATTICE ca „fără scală validă”, nu ca o valoare implicită.

**Etichete de calibrare: numai pe `raw`.** Fișierul `raw` al unei capturi conține pachetul de calibrare al camerei: încărcătura `CalibrationJSON` și etichetele compatibile cu MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), care descriu valorile brute ale senzorului. Fiecare fișier de la celelalte niveluri (`debayered`, `display`, `radiance`, `reflectance`, precum și redarea `index/`) și fiecare cadru din înregistrarea continuă / fluxul multi-cameră conține identitatea camerei (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...), precum și înregistrarea propriu-zisă a cadrului, `XMP-mapir:FrameJSON` (expunere, amplificare, temperatură, formatul pixelilor, ora capturii, potrivirea DAQ), deoarece fiecare produs `process()` poartă această identitate. Procesarea găsește calibrarea unui astfel de fișier după numărul de serie și `CalBundleSha256` în cache-ul de calibrare al Chloros, descărcând pachetul o singură dată dacă acest computer nu îl are.

#### Fișiere create din mai multe camere

Un fișier creat din benzile mai multor camere — o stivă multibandă (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), un index live creat din mai multe camere (`index.tif`, `index_LUT.png`), o imagine `composite/` a unei capturi matriciale — menționează fiecare cameră care a contribuit la aceasta:

- **Pentru-listă de camere, în ordinea benzilor** (compozitul: stratul de fundal, apoi prim-planul; un strat index, camerele de la care provin benzile sale): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (pachetul de calibrare prin care au trecut benzile fiecărei camere, altfel — un pachet `capture_aligned_multiband()` / `start_aligned_stream()`, ale cărui benzi nu trec prin niciunul — pachetul camerei pe care Chlorosl l-a stocat în cache, cel care dă numele fișierelor sale de captură), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — liste paralele, elementul *n* din fiecare descriind aceeași cameră; `-`, unde o cameră nu are nicio valoare (fără pachet de calibrare, un obiectiv necunoscut). Fiecare cameră reprezintă **un singur** element: în cazul în care pachetul său are mai multe valori pentru un câmp — certificatele lămpii cu halogen ale unui pachet — acestea sunt unite cu `;` (`F-1479-1;F-1480-1;F-1481-1`), niciodată cu virgulă, astfel încât o listă conține un singur element per cameră, indiferent de modul în care este afișată.
- **EXIF `SerialNumber`** conține aceleași numere de serie, separate prin virgulă (`214000535,214000551`).
- Acesta **nu** conține nicio identitate specifică unei singure camere (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): acestea denumesc o cameră și pachetul acesteia, iar procesarea ar determina calibrarea fișierului pe baza acestora. Un index calculat din benzile mai multor camere (`index.tif`, `index_LUT.png`) are EXIF `Model` `LATTICE` (riga), nu modelul `LATT-…` al unei singure camere.

Un fișier cu benzile unei **singure** camere — un fișier TIFF pe bandă, un cadru `capture_aligned_multiband(per_camera=True)`, o stivă sau un index live al unei singure camere — conține etichetele de identitate proprii ale acelei camere, la fel ca orice alt fișier de captură. Stiva `capture_aligned_multiband()`, fișierele pe bandă și pe cameră ale acesteia, precum și fiecare stivă `start_aligned_stream()` și fișierele sale pe bandă conțin ora de captură a grupului lor (expunerea camerei de referință, UTC), iar un cadru `per_camera` al unei camere mono este un singur plan, la `bit_depth` la fel ca stiva.**Orientarea camerei (răsucire / rotire).** Cadrele din `grab()` / `frame_stream()` sunt întotdeauna în
orientarea nativă a camerei, la fiecare nivel. Capturile salvate respectă setarea de orientare a camerei
(opțiunile Mirror H / Mirror V / rotație din interfața grafică):

- **Fișierele `raw` și `debayered` reprezintă exact ceea ce a citit senzorul**, indiferent de setare — un mozaic Bayer
  rotit sau răsturnat ar reprezenta un model diferit de filtre de culoare, iar hărțile de calibrare sunt
  pe grila propriu-zisă a senzorului.
- **Fișierele `display`, `radiance`, `reflectance` și fișierele index sunt orientate** — rotite o singură dată, după toate
  calculele de calibrare. Măștile lor de limitare se află pe aceeași grilă.
- **Fiecare fișier de captură înregistrează setarea** în XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, în sensul acelor de ceasornic, după
  inversări) și `ChlorosOrientationApplied` — `false` pe fișierele brute și debayered, `true` pe
  produsele orientate. `ChlorosProcessing` listează operațiile aplicate pixelilor acelui fișier (`none` când
  este nativ). Aceasta include fiecare cadru dintr-o înregistrare continuă (setarea proprie a camerei,
  neaplicată) și dintr-un flux multi-cameră (identitatea: cadrele sale componente sunt native, ca în cazul unei
  capturi matriciale).
- **Prelucrarea unui proiect aplică orientarea înregistrată** produselor pe care le generează dintr-un
  (mai întâi calibrarea în coordonatele senzorului, apoi deformarea de aliniere a matricei, iar la final orientarea),
  astfel încât acestea să corespundă produselor capturii live — inclusiv produsele de rezervă necalibrate, cu răspunsul senzorului și cu corecția de vignetare;
  produsul său debayerat rămâne nativ.
- **Vizualizatorul de imagini Chloros afișează un fișier brut sau debayerat transformat conform înregistrării sale** — doar pentru afișare,
  fișierul nu este modificat; un fișier fără înregistrare este afișat așa cum a fost stocat.
- **Capturile fără înregistrare sunt procesate ca native.** Înainte de versiunea 1.3.0, o captură cu o singură cameră
  își transforma singură fișierele raw și debayered; Chlorosnu poate distinge un astfel de fișier raw de unul nativ și
  nu face presupuneri.
- **Capturile de tip matrice păstrează fiecare fișier component nativ în mod implicit** și înregistrează identitatea;
  `ArraySession.capture(orient=True)` transformă produsele acestora (consultați comenzile de captură a matricei).

#### Ieșire pe bandă și matrice mixte

La nivelurile `radiance` și `reflectance`, o cameră returnează **un plan pe bandă pentru acea
cameră** — trei pentru un M3C (cele trei benzi nemixate ale acesteia, ordinea canalelor BGR), unul pentru un M3M (
filtrul său unic de bandă îngustă). Planurile replicate ale camerei mono sunt colapsate, astfel încât o **matrice mixtă
M3C/M3M generează forme de matrice diferite pentru fiecare serie**. Citiți cheia `bands` din
metadatele cadrului pentru a afla ce reprezintă fiecare plan; nu trageți concluzii pe baza numărului de canale:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Verificați întotdeauna `hdr["processing_level"] == "reflectance"` înainte de a utiliza valorile. O retrogradare
(fără citire DAQ, fără pachet de calibrare, dispunere necunoscută a benzilor) este raportată în `processing_level` și
`radiometric_refused`, nu ca o promovare — matricea pe care o primiți înapoi este totuși un cadru valid, doar că
nu este produsul pe care l-ați solicitat. O captură salvată care a fost retrogradată poartă același cod în fișierul XMP
ca `Chloros:RadiometricRefused` (de exemplu, `no-band-layout`, `no-dls-snapshot`).

#### Un singur declanșator per citire — `group_id`

**Elementele matricei sunt furnizate din grupul de cadre sincronizate al matricei, nu din propriul
buffer de captură al fiecărei camere.** Camerele unei matrice sincronizate hardware expun simultan, dar N citiri pe cameră
nu *eșantionează* simultan — citirea celui mai recent cadru stocat în bufferul fiecărei camere depășește, iar un singur cadru pierdut desincronizează setul fără ca nimeni să observe.

Furnizarea din grup este necesară, dar nu suficientă în sine: fiecare cerere încă
capturează *cel mai recent* grup la momentul respectiv, iar bucla de captură poate publica între timp.
Măsurat pe o matrice live cu 4 camere, **12 din 20 de citiri paralele s-au întins pe două grupuri** — întotdeauna
exact adiacente. Așadar, citirea este **fixată**:

1. Fiecare cadru conține `group_id`, contorul monoton de publicare al sistemului.
2. `grab()` citește camera principală (sau, dacă aceasta nu poate fi deservită, următoarea cameră care poate),
   apoi fixează camerele rămase la ID-ul grupului pe care a aterizat (`?group_id=N` pe punctul final HTTP).
3. Backend-ul păstrează grupurile recente timp de **cel puțin o jumătate de secundă**, astfel încât un grup înrudit deja
   depășit de publicări ulterioare este încă deservit de acel declanșator — suficient de mult pentru ca
   a exportului de reflectanță, chiar și pe o matrice cu 9 camere cu senzor complet. O asociere pe care backend-ul nu o mai
   deține este refuzată cu `409` și `group_aged_out: true`, în loc să primească un răspuns tacit
   cu un declanșator diferit — `grab()` recitește setul.

Remăsurat cu fixare: **20 din 20 de citiri au returnat un singur declanșator.** `grab()` face acest lucru pentru dvs.;
aveți nevoie de `group_id=` doar atunci când accesați direct punctul final HTTP.

- O cameră care nu poate fi deservită în cadrul grupului fixat — fie că lipsește din acesta, fie că refuză solicitarea
  (un `band` pe care nu îl are, un nivel pe care nu îl poate produce) — este **omise, în loc să fie completată retroactiv**
  dintr-un declanșator diferit, iar celelalte camere sunt totuși returnate.
- Dicționarul returnat de `grab()` este un `GrabResult` (o subclasă simplă a `dict`;
  `from chloros_sdk import GrabResult`): `.errors` asociază fiecare
  serial omis cu excepția generată de citirea sa, iar `.group_id` este grupul comun. Un
  `.errors` gol înseamnă că toate camerele fac parte din acest grup.
- O cameră autonomă nu are grup, raportează `group_id: None` și **respinge** un `group_id=`
  pin în loc să îl ignore.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Același contor este publicat pe **fiecare flux MJPEG per cameră** al unui membru al matricei —
previzualizarea (`/api/camera/stream/<serial>`), previzualizarea adnotată (`/stream-annotated`) și
fluxul de reflectanță (`/stream-reflectance`) — sub denumirea `X-Chloros-Group-Id` pe fiecare parte, pe lângă
`X-Chloros-Frame-Id` și `X-Chloros-Timestamp-Ns`. Fluxurile sunt continue, nu
la un moment dat, așa că sunt potrivite prin colectarea părților și gruparea pe baza acelui antet — nu există
niciun pin de setat. O cameră autonomă nu are grup. A se vedea
[Fluxuri MJPEG per cameră](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` nu tace niciodată.** Un tick care eșuează dintr-un motiv care se poate remedia de la sine
este omis — un flux de cameră nu returnează nimic pentru acesta, un flux de matrice returnează camerele care au
răspuns (posibil niciuna). Aceste motive sunt: încă nu există niciun cadru; firul de captare al camerei a fost pus în pauză pentru o
secundă sau două (`404 not streaming`, la care backend-ul răspunde ori de câte ori o altă cameră se conectează
sau se deconectează, sau o modificare a setărilor reechilibrează grupul); o repornire a backend-ului sau `5xx`; alinierea
încă în curs de stabilizare (`409 … retry shortly`); și un refuz al reflectanței deoarece DAQ-ul nu are încă nicio citire
(`no-dls-snapshot`, o citire expirată sau goală). Un refuz la următorul ciclu s-ar repeta doar — un
`band` necunoscut, `bit_depth=8` privind radianța, un pachet de calibrare fără scală de radianță, un refuz al reflectanței
**fără niciun DAQ** (niciunul asociat camerei sau matricei acesteia, niciunul conectat la
backend: `dls_source` al refuzului este `"none"`), orice alt `4xx` —
generează `chloros_sdk.FrameFetchError` (un `RuntimeError`, care conține `.status` și `.body`)
imediat. La fel se întâmplă și în cazul a **60 de secunde** de ticks omise fără niciun cadru între ele, indiferent de `fps` — suficient de
lung pentru a trece peste stabilizarea alinierii și o repornire a backend-ului, suficient de scurt încât o cameră care
a dispărut cu adevărat să întrerupă fluxul. Această eroare de renunțare este, de asemenea, un `FrameFetchError`, atât pentru o cameră, cât și pentru
un array, cu excepția proprie a ultimei citiri la `.errors` (și, pentru o cameră, ca `__cause__`). Pentru
un array, „refuzat” înseamnă că fiecare cameră a refuzat; excepția `.errors` conține motivul fiecărei camere.

**Un array `frame_stream()` generează fiecare declanșator o singură dată** (atât `ArrayHandle`, cât și `ArraySession`). Un
tick al cărui set cel mai recent este declanșatorul pe care l-a generat deja așteaptă următorul declanșator în loc să
genereze din nou același set, astfel încât fluxul rulează la viteza mai mică dintre `fps` și rata de cadre proprie a matricei, iar seturile consecutive nu împart niciodată un `group_id`. Un array care încetează publicarea generează
`FrameFetchError` („niciun declanșator nou timp de 60 s”) în loc să-și repete ultimul set. Fără aceasta, un
flux care solicita mai mult decât publica matricea — sau care pornea imediat după conectare — genera
repetări: într-o execuție măsurată, 20 de seturi conțineau 11 ID-uri de grup distincte.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Exportarea unei singure benzi — `band`

`grab()` / `frame_stream()` utilizează `band="<name>"` pentru a exporta doar una dintre benzile unei camere sub forma unui
tablou `(H, W, 1)` cu un singur canal — aceeași formă pe care o produce o cameră monocromă, astfel încât ieșirea cu o singură bandă este
tratată identic, indiferent dacă provine de la un M3M sau din divizarea unui M3C. Banda roșie a unui FRGN:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Ortografiile acceptate sunt numele complet al layout-ului (`Red_660`, `NIR_850`, `Green_550`) sau prescurtarea acestuia
(`red`, `nir`, `green`), fără a se ține cont de majuscule și minuscule (`red`, `nir`, `green`), fără a se ține cont de majuscule și minuscule. Un nume necunoscut este respins, fiind afișate
numele disponibile.

> **Selecția se face după nume, nu după numărul canalului — în mod deliberat.** „Canalul 1” este `Green_550` în
> ordinea canalelor BGR a matricei, dar *roșu* în ordinea literelor din codul filtrului(`RGN`), iar cele două
> nu coincid. Alegerea celui greșit generează un cadru perfect plauzibil din banda greșită, pe care
> nimic din aval nu îl poate detecta. Din acest motiv, nu există un index numeric.

#### Alinierea fluxurilor — `align`

`align=True` aliniază cadrul la cadrul de referință comun al matricei sale. Fiecare cameră este aliniată
prin **aceeași** rezoluție de profil și aceeași zonă de interes (ROI) decupată, astfel încât fluxurile per cameră deschise independent
să fie **aliniate reciproc** — nu este necesar să le solicitați împreună:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Cerințe și comportament:

- Camera trebuie să se afle într-o matrice **sincronizată hardware** (cablu de sincronizare M8) care are un profil de aliniere
  — co-înregistrarea presupune că elementele sunt expuse simultan, ceea ce o matrice cu funcționare liberă sau
  matrice cu funcționare liberă. În caz contrar, se refuză cu `409`.
- **Respins, niciodată nealiniat în mod tacit.** Calea de captură poate salva datele nealiniate cu un avertisment
  deoarece un fișier își păstrează etichetele de proveniență și poate fi reînregistrat ulterior; un flux live nu are
  o astfel de a doua șansă, astfel încât lipsa unui profil este o eroare, nu o soluție de rezervă.
- **Solicitarea alinierii o inițiază.** Solicitarea `align` pe o matrice fără profil declanșează
  calibrarea și returnează `409` cu ceea ce așteaptă (de ex. *„așteptarea stabilizării expunerii
  la 214000551 (1/3)”*) — reîncercați și va reuși, de obicei în câteva zeci de secunde.
  Acest lucru este important în modul fără interfață grafică: alinierea automată la conectare renunță după 30 de secunde, iar fiecare încercare
  din acest interval este omisă cât timp funcția smart-AE încă se stabilizează, astfel încât o matrice a cărei expunere
  converge lent ajungea să nu aibă niciun profil pentru întreaga sesiune și nicio modalitate de a obține
  unul fără a deschide vizualizarea combinată de pe desktop. `alignment_status` raportează, de asemenea,
  `ae_align_holdouts`, indicând camerele aflate încă în proces de stabilizare și progresul acestora.
- Atribuiți fiecărui flux al unei matrice **același `max_h`**, altfel decupajele lor vor fi plasate pe grile de pixeli
  diferite.
- Alinierea se aplică *înainte* de codificarea adâncimii de biți, astfel încât deformarea interpolează valori cu precizie deplină.

#### `bit_depth` — compromis între precizie și lățime de bandă

`grab()` și `frame_stream()` acceptă `bit_depth=16` (implicit) sau `bit_depth=8`:

| `bit_depth` | Codificare reflectanță | Cost |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, marjă până la ρ 2,0 | Precizie completă. |
| `8` | uint8 care acoperă `0..refl_max` (implicit ρ 1,0) | Jumătate din numărul de octeți; ~0,004 ρ cuantificare la limita superioară implicită. Valorile peste limita superioară sunt tăiate — vezi mai jos. |

##### Limita superioară de 8 biți — `refl_max`

`refl_max` stabilește valoarea ρ pe care o acoperă intervalul de 8 biți, astfel încât `pixel_scale = 255 / refl_max` și ρ rămân
`pixel / pixel_scale`. `grab()` și `frame_stream()` îl acceptă ca cuvânt-cheie atât la cameră, cât și la
mânerele de matrice (`?refl_max=` pe punctul final HTTP). Măriți-l pentru a obține marjă pentru
reflexiile speculare, cu pași proporțional mai mari:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | pas |
| --- | --- | --- |
| `1.0` (implicit) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Expunerea automată nu poate remedia o decupare de 8 biți, iar încercarea de a o utiliza agravează calitatea datelor.**
> ρ = π·L/E, după ce expunerea și amplificarea sunt eliminate din ecuație, deci ρ este o proprietate a scenei, nu a
> setărilor camerei — măsurată pe o matrice live, reducerea la jumătate a expunerii a modificat valoarea medie a lui ρ cu 1,3%. Creșterea
> expunerii pentru a „se potrivi” cu intervalul duce, dimpotrivă, la saturarea **senzorului**, ceea ce
> subestimează L și denaturează ρ **în jos** fără a genera nicio eroare (măsurat: ρ maxim 1,203 → 0,985 la
> expunere de 2×, medie −23%). Alegeți intervalul cu `refl_max`; menținerea senzorului în afara
> saturației este deja rolul AE, iar acest lucru protejează *validitatea* lui ρ, mai degrabă decât intervalul său.

**Citiți `reflectance_clipped_fraction`, nu indicatorul.** `reflectance_clipped_at_1_0` se declanșează la un
singur pixel — într-o scenă reală a raportat `True`, în timp ce fracția era `0.000001`, un pixel
la un milion. Fracția este cea care îți indică dacă trebuie să ridici pragul maxim.

**`bit_depth=8` se referă exclusiv la reflectanță.** Solicitarea acestuia pentru `radiance` este respinsă, nu
satisfeasă: reflectanța are un interval fix de la 0 la 1 care rezistă unei codificări pe 8 biți, în timp ce radianța nu are
un interval fix, astfel încât încadrarea ei în 8 biți necesită o extindere procentuală pentru fiecare cadru, a cărei mapare
se schimbă la fiecare cadru — o imagine, nu o măsurătoare.

#### Flux live de reflectanță (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` furnizează reflectanță pe bandă de 8 biți ca
`multipart/x-mixed-replace` — un M3C ca JPEG cu 3 canale, un M3M ca imagine în tonuri de gri. Parametri de interogare:
`fps` (implicit 10), `jpeg_q` (implicit 95), `max_h` (0 = nativ), `band` (o bandă denumită, ca
mai sus), `align` (`1` pentru co-înregistrare — aceleași reguli ca și `align` din SDK), `refl_max` (plafonul de
codificare pe 8 biți, ca mai sus). Fiecare parte conține
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` și `X-Chloros-Reflectance-Clipped`.
**Părțile provenite de la camere diferite care poartă același `X-Chloros-Group-Id` reprezintă același declanșator** —
astfel sunt reasamblate mai multe fluxuri per cameră ale unei matrice, iar acest lucru este garantat
la nivel de server, deoarece elementele matricei sunt furnizate din grupul de cadre sincronizate (vezi
[Un declanșator per citire](#un-declanșator-pe-citire--group_id)). Returnează `409` când reflectanța este
indisponibilă la momentul conectării și încheie fluxul dacă aceasta devine indisponibilă în timpul rulării, astfel încât un flux blocat
înseamnă „nu mai există reflectanță”, mai degrabă decât „încă este în regulă, doar static”.

> **Formatul JPEG este cu pierderi — acești pixeli reprezintă o măsurătoare.** Eroarea de compresie se reflectă direct în
> valorile de reflectanță și este cea mai gravă la margini, unde se află limitele materialului. Din acest motiv, calitatea
> este setată implicit la 95. Dacă aveți nevoie ca valorile să rămână intacte, utilizați
> `grab(processing="reflectance")` / `frame_stream(...)`, care returnează același produs fără
> codec în calea de acces (și `bit_depth=8` dacă constrângerea o reprezintă costul conexiunii, nu codecul).

> **Datele EXIF sunt preluate în export.** `process()` copiază blocul GPS al capturii sursă
> **și ExifIFD-ul acesteia** în fiecare produs, astfel încât exporturile conțin `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` și `CameraSerialNumber`, precum și
> georeferențiere. `FocalLength` este codul pe baza căruia Pix4D calculează distanța dintre eșantioanele la sol — fără acesta,
> reconstrucția revine la o scară extrem de eronată (într-un caz măsurat, un amplasament de 411 m
> a fost transformat într-unul de 47,8 km). Fișierul nu este în mod deliberat `-all:all`: etichetele structurale ale IFD0 perturbă
> ieșirea LATTICE, iar `ExifImageWidth`/`Height` sunt excluse deoarece descriu
> capturarea sursei, nu rasterul exportat.

Sub-indicatori ai etapei de captură (se aplică nivelurilor radiometrice — `radiance`, `reflectance`, `display`):

| Indicator | Implicit | Semnificație |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + câmp plat + separare 3x3 + scală radiometrică NIST. |
| `apply_white_balance` | `True` | LUT de balans de alb. Compatibil cu DLS atunci când un DAQ este asociat camerei. |
| `apply_index` | `False` | Evaluarea indicelui de vegetație. |
| `index_expression` | `None` | Suprascriere formulă. Non-gol → activează automat indicele. |
| `annotated` | `False` | Suprapunere decorații GUI (zebra/grilă/peaking). Nu este disponibil pentru `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Valoarea returnată este un `chloros_sdk.CapturePaths`, de tip `CapturePathMap`, nu `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` este `Dict[str, Union[str, List[str]]]`: o singură cale
> `processing` oferă fiecărui serial o singură cale, în timp ce `processing="all"` îi oferă
> **lista ordonată** a tuturor produselor salvate pentru acea cameră. `ArrayHandle.capture()` nu acceptă
> niciun `levels=` și ignoră argumentele-cheie pe care nu le cunoaște (`levels`, `daq`,
> `render_index`, `force_daq`), așa că transmiteți doar cele afișate aici (`aligned=False`
> salvează fișiere native, la fel ca în cazul `ArraySession.capture()`); utilizați
> `ArraySession.capture()` pentru acele controale. Un compozit combinat în timp real, dacă acesta era transmis în flux, ajunge sub cheia suplimentară
> `"combined"`, nu sub un serial. Codul care presupune `str` dă eroare la
> forma de listă fără ca vreun verificator de tip să obiecteze — adnotarea menționa `Dict[str, str]`
> pentru o perioadă după lansarea formei de listă, motiv pentru care există aliasul. Normalizați
> când doriți forma plată:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Ce nu s-ar fi putut scrie: `.failed`.** Harta returnată de `capture()` este o
> subclasă simplă `chloros_sdk.CapturePaths` — o subclasă simplă a `dict`, așa că tot ce este mai sus funcționează
> neschimbat — care conține și `.failed`: fișierele pe care backend-ul nu le-a putut scrie,
> aceleași intrări ca [`CaptureResult.failed`](#fișiere-care-nu-au-putut-fi-scrise--captureresultfailed)
> (`serial`, `level`, `error` și `disk_full: True` pe un disc plin). O
> captură parțială returnează totuși fiecare cale care a fost scrisă; pe un disc plin, harta este
> goală, iar `.failed` explică de ce. `.skipped` listează omisiunile pe cameră, iar
> `.group_seq` grupul sincronizat, iar `.alignment` / `.oriented` sunt cele
> ale backend-ului, la fel ca în cazul lui `CaptureResult`. `capture_smart()` le returnează ca
> `result["frames"].failed` și `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Alinierea matricei

`ArrayHandle` expune suprafața completă de aliniere. Profilurile sunt, în mod implicit, valabile doar pentru sesiune — apelați explicit `export_alignment()` pentru a le păstra.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Alinierea la conectare

`connect_all(align=...)` poate alinia automat fiecare matrice la conectare:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Se revine la `project.json["config"]["auto_align_on_connect"]` dacă nu se specifică altceva.

### `SensorHandle`

Un senzor de lumină salvat (DAQ-U, DAQ-M sau DAQ-E), deschis de `connect_all()` în [pool de senzori DAQ](#daq-sensor-sessions) — același pool pe care îl utilizează `connect_daq_sensor()` și `chloros-cli daq pool-connect`.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` așteaptă **următorul** spectru al senzorului — unul deja stocat în buffer în momentul apelului nu se ia în considerare — și returnează `(spectrum, is_saturated, integration_time, x, y, z)` sau `None` dacă nu sosește niciunul în termen de 5 s. Un flux pus în pauză este reluat. `spectrum` este exprimat în W/m²/nm atunci când senzorul este calibrat.
- `.sensor` este [`DAQSensorSession`](#sesiuni-de-senzor-daq), pentru orice altceva: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` și `.settings` sunt intrările salvate din `sensors.json`.

**Ce restabilește `connect_all`.** Fiecare senzor este deschis la adresa sa salvată — portul COM, adresa Bluetooth sau gazda DAQ-E — și, atunci când proiectul a înregistrat ID-ul senzorului, acesta trebuie să fie chiar acel senzor: o unitate diferită la acea adresă este respinsă, nefiind conectată. Se aplică timpul de integrare salvat, medierea cadrelor și expunerea automată, precum și corecția capacului:

| Cap salvat | Conectat cu |
| --- | --- |
| Un profil de cap (de ex. `fov_30`) | Profilul respectiv. |
| Niciunul selectat | `sunshine_cosine`, valoarea implicită de conectare, înregistrată conform presupunerii (`cap_id_source=auto_default`), astfel încât procesarea generează o avertizare. Senzorii sunt livrați fără capac: alegeți capacul care este montat. |
| `none` — selectat ca **Niciunul (senzor fără capac)** în interfața grafică | Fără corecție pentru capac. |

Un DAQ-E-S nu acceptă niciun profil de capac, indiferent de ce a fost salvat: difuzorul său face parte din calibrarea sa (vezi [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Un proiect salvat de un Chloros anterior a înregistrat un senzor fără capac ca opțiune implicită, așa că se reconectează cu `sunshine_cosine`; alegeți din nou **Niciuna (senzor fără capac)** în interfața grafică și salvați proiectul.

Un senzor pe care un alt client îl are deja deschis în grup (CLI, un alt script) este atașat așa cum este, fără a se reaplica setările, iar `disconnect_all()` îl lasă să funcționeze; un senzor deschis de `connect_all()` este eliberat. La fel ca în cazul `connect_daq_sensor()`, interfața grafică Chloros **nu** face parte din grup: închideți un senzor în interfața grafică înainte de a conecta proiectul dintr-un script.**Cerințe și raport.** Senzorii de lumină necesită un backend Chloros în funcțiune, conectat la un plan Chloros+ plătit (Copper sau superior), iar `connect_all()` nu pornește niciodată unul. Dacă nu rulează niciunul, fiecare senzor salvat este listat sub `report["sensors"]["errors"]`, cu mențiunea respectivă. Un senzor pe care backend-ul nu îl poate deschide este listat și el acolo, împreună cu motivul indicat de backend; `report["sensors"]["missing"]` rămâne gol pentru senzorii de lumină.

---

## Hardware direct (fără backend)

Când doriți să nu existe nicio dependență de backend pentru camerele LATTICE (CI, roboți fără cap, încorporate), importați direct `lattice_sdk` — acesta este reexportat de `chloros_sdk`. Atenție la `CAMERA_AVAILABLE`: `lattice_sdk` se află în pachetul SDK, dar necesită prezența runtime-ului Arena SDK și a OpenCV (`cv2`), iar pachetul nu instalează niciunul dintre acestea. Senzorii de lumină trec prin backend — [`connect_daq_sensor()`](#daq-sensor-sessions) sau prin `proj.sensors` al unui proiect — cu o singură excepție, helperul de magnitudine de bandă DAQ-M [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), care necesită, de asemenea, `bleak`.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Presetările și declanșatorul

Trei dintre cele patru presetări funcționează în **mod liber**: camera expune continuu, iar un
`capture()` returnează următorul cadru. `triggered` este excepția — aceasta armează
camera pentru un semnal de margine hardware pe linia 2, așa că nu captează nimic până când nu apare unul.

| Presetare | Declanșator | Se utilizează când |
| --- | --- | --- |
| `default` | funcționare liberă | utilizare generală |
| `high_speed` | funcționare liberă | 8 biți, limită de 60 fps, expunere scurtă |
| `high_quality` | funcționare liberă | 12 biți, fără limită de fps — alegerea obișnuită pentru fotografii |
| `triggered` | **pregătită, Linia 2** | camera este conectată la un cablu de sincronizare M8 și este declanșată de altceva |

Dacă alegeți `triggered` (sau setați manual `trigger_mode="On"`) fără ca nimic
să acționeze Linia 2, fiecare `capture()` va intra în timeout — în mod corect, deoarece ați cerut
camerei să aștepte. SDK-ul explică acest lucru atunci când se întâmplă; consultați
[SC_ERR_TIMEOUT în timpul capturii](#direct-hardware-backend-free).

> **Notă — Mesajele „GVSP probe” / `SC_ERR_TIMEOUT -1011` la conectare nu sunt erori.**
> La conectare, SDK-ul încearcă să negocieze**cadre jumbo** (pachete GVSP de 9000 de octeți) pentru un debit mai mare. Pe o legătură directă punct-la-punct prin placa de rețea (de exemplu, o adresă `169.254.x.x` locală de legătură), rețeaua nu poate transporta, de obicei, cadre jumbo, astfel încât această sondă depășește timpul de așteptare și înregistrează linii precum:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Aceasta este **soluția de rezervă prevăzută**: SDK-ul revine automat la pachetele standard de 1500 de octeți, iar camera continuă să se conecteze în mod normal (liniile `[chunk-enable …]` care urmează fac parte din secvența normală de conectare). Captura funcționează în continuare.
>
> Puteți omite această probă, dar **aceasta nu este doar o opțiune de reducere a jurnalelor — ci dezactivează cadrele jumbo.** Camera răspunde la ping-urile „Don't-Fragment” doar până la 1500 de octeți, indiferent cât de bună este rețeaua dvs., așa că testul de ping singur nu poate detecta niciodată cadrele jumbo; această probă este singura care poate face acest lucru. Dacă o dezactivați, camera va transmite pentru totdeauna pachete standard de 1500 de octeți, pe orice rețea:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Merită folosită doar pe o rețea despre care *știți* că nu suportă cadre jumbo, unde economisește aproximativ o secundă din timpul de conectare pentru fiecare cameră. Deoarece este vorba de un compromis real, nu doar unul cosmetic, SDK-ul menționează acum acest lucru atunci când îl utilizați:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Nu-l modificați decât dacă aveți un motiv.**Dacă rămâne activat, fiecare cameră este măsurată o dată pe sesiune de backend și din nou când legătura sa se modifică (un alt adaptor, o altă viteză de legătură sau o legătură care s-a întrerupt, cum se întâmplă la schimbarea unui switch); reconectările reutilizează rezultatul.**După o modificare în spatele unui switch care a rămas activ, reporniți backend-ul pentru a măsura din nou.** O primă conectare poate dura ocazional câteva minute pentru această verificare, iar jurnalul indică acest lucru atunci când se întâmplă.
>
> Dacă *doriți* un debit jumbo, activați jumbo end-to-end (MTU NIC 9000 + un switch care le transmite), sau fixați-l cu `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` când știți că legătura îl suportă — deși este de preferat să folosiți `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` pe bază de comandă decât să-l setați permanent, deoarece o dimensiune fixată omite testarea și oprește adaptarea la rețeaua din fața sa. **Fiecare** dispozitiv din traseu trebuie să transmită pachete jumbo — inclusiv orice splitter sau injector PoE, acesta fiind motivul obișnuit pentru care o configurație care altfel ar fi compatibilă cu pachetele jumbo nu le poate transmite.

> **`SC_ERR_TIMEOUT -1011` în timpul `capture()` / `grab*()` reprezintă o problemă diferită — aceea este o eroare reală.**
> Nota de mai sus se referă doar la `-1011` înregistrată de**sonda de timp de conectare**. Aceeași eroare generată de o**captură** înseamnă că camera s-a conectat corect, dar nu transmite nicio imagine:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Indiciul revelator este o cameră al cărei canal de *control* funcționează corect — descoperirea funcționează, setările și înregistrările `[chunk-enable …]` se efectuează cu succes — în timp ce *fiecare* cadru depășește timpul de așteptare.
>
> **Cauza obișnuită este faptul că camera este setată pentru un declanșator hardware.** Cu `trigger_mode="On"` și `trigger_source="Line2"`, camera nu emite absolut nimic până când nu apare o flancă electrică pe cablul de sincronizare M8. Dacă nu aveți niciun cablu care să alimenteze acea linie, fiecare captură așteaptă la nesfârșit. Camera nu este defectă și rețeaua funcționează corect — face exact ceea ce i s-a cerut.
>
> `CameraSettings()` și `default` / `high_speed` / `high_quality` sunt presetate pentru funcționare liberă, iar o captură care depășește timpul de așteptare în timp ce este activată afișează o explicație în loc să afișeze doar codul `-1011`. `PRESETS["triggered"]` activează Linia 2, conform specificațiilor.
>
> Pentru a forța orice cameră să funcționeze în modul liber:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Dacă totuși se produce o expirare a timpului cu `trigger_mode="Off"`, înseamnă că camera nu transmite date — trimiteți-ne jurnalul și `ip link show`.

#### Profiluri de culoare (previzualizare live RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` selectează profilul de culoare al ecranului pentru **previzualizarea live** pe camerele RGB (camerele multispectrale ignoră această setare):

| Profil | Semnificație |
| --- | --- |
| `raw` | Ocolește complet lanțul radiometric. |
| `linear` | DSNU + flat + WB, fără CCM, fără gamma. |
| `natural` | Linear + CCM măsurat + gamma sRGB, doar cu finisajul simplu (netezire cromatică + desaturare a zonelor luminoase) — setarea implicită realistă. |
| `enhanced` | `natural` plus finisajul complet „hub-parity” (eliminarea franjurilor, vibranță, contrast local CLAHE). Aspect mai bogat la aproximativ **dublul costului de finisare pe cadru**, deci o rată de cadre LIVE mai mică. |
| `custom_temp` | `natural`, dar WB fixat la `custom_cct_k` Kelvin (DLS ignorat; limitat la 2000–10000 K la nivel de backend). |

Profilul este un buton de reglare a vitezei/aspectului **destinat exclusiv previzualizării live**: capturile salvate beneficiază întotdeauna de finisajul complet, indiferent de profilul selectat, astfel încât alegerea `natural` pentru a câștiga timp de cadru nu scade calitatea fișierului salvat pe disc. Un profil necunoscut activează `ValueError`; când un backend chloros este accesibil, modificarea este trimisă și către acesta prin POST, astfel încât următorul cadru de previzualizare să o reflecte (utilizatorii SDK-ului direct fără un backend primesc totuși modificarea setărilor).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Zone de evidențiere (afișaj RGB) — `set_highlights`

`LatticeCamera.set_highlights(value)` extrage cei mai luminoși și mai albi pixeli ai **afișajului** pe camerele RGB — soluția pentru o suprafață albă strălucitoare care se afișează ca o pată albă uniformă. `value` este `-100`..`0` (limitat); `0` este dezactivat, iar redarea este atunci exact așa cum ar fi fără această setare. Setările mai puternice reduc albul ecranului (la aproximativ 60% la `-100`) și lărgesc curba de ton, astfel încât o suprafață pe care luminozitatea automată a previzualizării a împins-o spre alb pur își recuperează detaliile.

Este un control al afișajului, dar spre deosebire de saturație și contrast (care modifică doar previzualizarea în timp real), acesta modifică și ceea ce salvați: previzualizarea în timp real și capturile **la nivel de afișaj** respectă această setare, astfel încât și acele fișiere ies mai întunecate (ele poartă valoarea `Chloros:DisplayHighlights`); exporturile de radianță și reflectanță nu fac niciodată acest lucru, iar această opțiune nu poate recupera datele senzorului care au fost decupate — reduceți expunerea pentru asta. Camerele multispectrale și monocrome o ignoră. Valoarea este stocată în `settings.highlights` și trimisă prin POST către un backend chloros accesibil, exact ca `set_color_profile`; echivalentul în CLI este `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Camere mono (M3M) și `Calibration`

O cameră mono **M3M** (`M3M-<lens>-F<wavelength>`) este monobandă: un singur plan în tonuri de gri, fără mozaic Bayer, fără matrice spectrală de interferență 3×3. `Calibration` o recunoaște și expune un indicator `is_mono`. Reflectanța se aplică în continuare ca o hartă radiometrică pe bandă (demixarea este matricea identitate), dar calculele multibandă pe o singură cameră generează rezultate valabile, în loc să returneze date fără sens:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Pentru a construi un indice de vegetație folosind echipament monocromatic, combinați mai multe camere M3M la lungimi de undă diferite într-un stack multibandă aliniat (vezi [Alinierea matricei](#array-alignment)) și calculați indicele pe acel stack, în loc să îl calculați pe o singură cameră.

#### Magnitudini de bandă DAQ-M — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` se conectează la un DAQ-M prin Bluetooth folosind propria adresă MAC BLE. Este singura clasă de senzori de lumină care comunică direct cu senzorul, fără a trece prin backend, așa că are nevoie de ceea ce au nevoie clasele de camere directe (`CAMERA_AVAILABLE`: runtime-ul SDK-ului Arena și OpenCV), plus `bleak` pentru Bluetooth (`pip install "chloros-sdk[camera]"` sau `pip install bleak`; fără acesta, `DLS` generează `DLSError`). Un DAQ-M conectat în altă parte (pool-ul backend-ului, un proiect, GUI-ul) încetează să mai transmită, așa că eliberați-l mai întâi acolo. **`mac_address` este obligatoriu**: nu există o valoare implicită, deoarece fiecare unitate are propria sa valoare, iar `DLS()`, în absența acesteia, generează `DLSError` înainte de a se încerca orice conexiune. Găsiți adresa MAC cu `discover_daq_sensors(transports=["ble"])` (vezi [Descoperire](#discovery--finding-an-address-to-connect-with)) sau `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integrează un spectru DAQ-M (cea mai recentă citire atunci când `spectrum` este `None`; `DLSError` dacă nu există încă unul) prin răspunsurile celor trei canale ale camerei și returnează `{"red": …, "green": …, "nir": …}` — tastele denumesc canalele în această ordine; în cazul curbelor unei camere RGB, al treilea (`nir`) este canalul albastru.

- Fără `spectral_response`, utilizează curba standard a modelului de filtru (cea cu care a fost construit `filter_model` și `DLS`), care a fost măsurată pe o singură unitate.
- `spectral_response` utilizează în schimb curbele măsurate **proprii** ale camerei: un dicționar `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, așa cum este stocat în pachetul de calibrare al camerei respective (`color_correction.spectral_response_smoothed` — camere RGB; pachetele multispectrale nu conțin o astfel de curbă). Dacă dicționarul nu poate fi utilizat, se revine la curba de flotă.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

Comanda `chloros-cli lattice dls --camera SERIAL` din CLI face același lucru prin intermediul backend-ului, găsind curba camerei în pachetul de calibrare încărcat sau stocat în cache și indicând a cui curbă a fost utilizată. Prin HTTP, aceasta este `POST /api/dls/cli-read` cu `mac` (obligatoriu: fără acesta, răspunsul este HTTP 400, `code` `MAC_REQUIRED`) și `camera_serial` în corpul mesajului; `band_magnitudes_source` din răspuns este `camera <serial>` atunci când a fost utilizată curba proprie a camerei respective, sau `model <filter>` pentru curba flotei (o cameră fără curbă: nu este RGB sau nu are pachet încărcat sau stocat în cache).

---

## Excepții

Interceptează clasa de bază pentru a gestiona „orice problemă apărută în Chloros”:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` și `ChlorosConfigurationError` sunt atribute ale `chloros_sdk` (și ale lui `chloros_sdk.exceptions`), dar nu se află în `__all__`, așa că `from chloros_sdk import *` nu le include: importați-le după nume. Familia `ChlorosError` este derivată din `ChlorosLocal` și din funcțiile construite pe baza acesteia (`process_folder()`, `process_lattice_capture()`); suprafața smart-connect generează `ChlorosConnectError`.

Ierarhie:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Exemple end-to-end

### 1. Procesarea unui folder cu o bară de progres personalizată

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Matrice LATTICE în timp real → Reflectanță + Referință DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Campanie de captură bazată pe proiect

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Flux de cadre de la mai multe camere → Pipeline NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Script de captură directă pe hardware fără interfață grafică (fără backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Testare a capacităților înainte de conectarea unei matrice cu 4 camere

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Echivalentul unei rețete de captură (Python pur)

`run_project_recipe()` execută o rețetă CLI așa cum este (vezi [Dosarul proiectului și rețetele](#project-folder--recipes)). Scrise în Python, pașii sunt aceiași:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Serverul MCP (asistenți AI)

`chloros-mcp` permite unui asistent AI care utilizează [Model Context Protocol](https://modelcontextprotocol.io) (MCP), precum Claude Desktop, Claude Code sau Cursor, să controleze Chloros: să proceseze imagini (să creeze un proiect, să importe un folder cu capturi, să aleagă setările, să proceseze, să raporteze ce a exportat execuția) și să controleze camerele LATTICE, matricile și senzorii de lumină DAQ (conectare, previzualizare, setarea expunerii și a culorii, captură, înregistrare, actualizare firmware). Instrumentele sale acoperă funcționalitățile acestui SDK. Tu soliciți în limbaj simplu, iar asistentul apelează instrumentele serverului:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Serverul este construit pe baza acestui SDK: instrumentele de procesare apelează [`ChlorosLocal`](#image-processing--chloroslocal), iar instrumentele hardware [apelurile smart-connect](#smart-connect-for-lattice-cameras), [sesiunile DAQ](#daq-sensor-sessions), `ChlorosProject` și [utilitarele camerei](#camera-utilities), toate comunicând cu același backend pe care îl folosește aplicația Chloros. Cererile sale conțin `X-Chloros-Client: sdk`, astfel încât backend-ul aplică verificările de autentificare și de plan ale SDK-ului, iar regulile SDK-ului se aplică automat (de exemplu, funcția „Texture Aware” este refuzată pentru capturile LATTICE).

### Instalare

Serverul este livrat în pachetul `chloros-sdk`; nu mai este nevoie să instalați nimic altceva. SDK-ul instalează o comandă `chloros-mcp`, care este identică cu `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** programul de instalare Chloros instalează SDK-ul prin pip în Python, ceea ce plasează `chloros-mcp.exe` în folderul `Scripts` al acelui Python.
- **Linux (.deb):** pachetul instalează SDK-ul pentru sistemul `python3` și plasează comanda `chloros-mcp` în `/usr/bin`, astfel încât aceasta să fie disponibilă în `PATH`. Într-un virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` adaugă `chloros-mcp` la `bin` al virtualenv-ului.

Serverul utilizează doar biblioteca standard pe lângă SDK (Python 3.7+). Un client MCP îl pornește și comunică cu acesta prin stdin/stdout; nu îl rulați voi înșivă, cu excepția cazului în care folosiți `--help`.

### Configurarea clientului

**Claude Desktop:**adăugați serverul la `claude_desktop_config.json` (pe Windows, `%APPDATA%\Claude\claude_desktop_config.json`; îl puteți deschide din**Setări > Dezvoltator > Editare configurație**), apoi reporniți Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursor:** aceeași intrare `mcpServers` ca în Claude Desktop, în `~/.cursor/mcp.json` (pentru fiecare proiect; `%USERPROFILE%\.cursor\mcp.json` pe Windows) sau `.cursor/mcp.json` într-un singur proiect.

Opțiunile se introduc în `args` sau după comanda pentru Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Pentru a utiliza un anumit SDK Python în locul celui din `chloros-mcp` pe `PATH`, rulați modulul cu acel Python (calea completă a acestuia ca `command` dacă nu este primul `python3` din `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Un client pornește serverul fără shell-ul dvs., așa că, dacă acesta raportează că serverul nu a reușit să pornească, probabil că `chloros-mcp` nu se află pe `PATH` pe care îl vede clientul. Specificați în schimb calea completă (`where chloros-mcp` pe Windows, `which chloros-mcp` pe Linux, unde fișierul .deb se află la `/usr/bin/chloros-mcp`) sau rulați modulul cu versiunea de Python care conține SDK-ul.

### Opțiuni

| Opțiune | Implicit | Ce face |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Backend-ul Chloros care va fi utilizat. |
| `--no-auto-start` | dezactivat | Nu porniți niciodată un backend: utilizați unul deja în funcțiune, cum ar fi cel al aplicației Chloros, serviciul backend Linux sau un tunel către un alt computer (vezi [Modul backend la distanță](#remote-backend-mode-pip-only-host-via-tunnel); căile către foldere trebuie să indice același folder pe ambele mașini). |
| `--backend-exe PATH` | unde îl instalează Chloros | Backend-ul care trebuie pornit atunci când nu rulează niciunul, pentru un Chloros instalat într-un loc în care SDK-ul nu îl găsește. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` sau `ERROR`. Jurnalele sunt redirecționate către stderr, pe care majoritatea clienților le afișează în jurnalul MCP; stdout conține doar mesaje de protocol. |

### Instrumente

Argumentele fiecărui instrument sunt cele ale apelului SDK pe care îl încorporează; clientul le listează pe fiecare împreună cu schema completă. Căile sunt absolute.

**Prelucrare**

| Instrument | Apel SDK | Ce face |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Creează un proiect și îl setează ca proiect deschis, asupra căruia acționează instrumentele de procesare. Un nume simplu creează folderul în folderul implicit al proiectului; o cale absolută îl creează acolo. |
| `open_project` | `ChlorosLocal.open_project` | Deschide un proiect salvat și îl setează ca proiect deschis. Căile unui proiect mutat sunt redirecționate către folderul în care se află acum; `relink_folder` (absolut) denumește folderul în care se află acum celelalte fișiere lipsă. Returnează `source_files` și `image_count`. Nu s-a găsit nimic: o listă de erori care indică unde se aflau și denumește `relink_folder`; nu este deschis nimic. |
| `import_images` | `import_images` | Importă perechile `.RAW`+`.JPG` din Survey3, `.tif`443 capturi și DAQ `.daq` înregistrări; returnează `count` (imagini) și `scan_count` (înregistrări). |
| `configure_processing` | `configure` | Modifică setările transmise și nimic altceva (valori: [Valori acceptate](#supported-values)); `custom_settings` trimite întreaga configurație așa cum a fost specificată. |
| `get_project_settings` | `get_config` | Citește setările salvate ale proiectului deschis privind procesarea, exportul și indexarea. |
| `start_processing` | `process(wait=False)` | Începe procesarea proiectului deschis și se întoarce imediat. |
| `get_processing_status` | `get_status` | Raportează `state` (`idle`, `starting`, `processing` sau `finished`) și progresul; odată finalizată, se înregistrează `summary`, `export_check_failure` atunci când exporturile nu au putut fi verificate și `image_failure` atunci când imaginile au fost pierdute: celelalte produse sunt înregistrate; se raportează execuția ca eșuată. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Creează un proiect pentru un folder, îl importă, îl configurează și pornește execuția într-o singură operațiune (`lattice: true` pentru capturi LATTICE). În timp ce o execuție este în curs, aceasta se întrerupe înainte de a crea ceva, iar backend-ul refuză atunci și modificarea proiectului din orice instrument. |
| `export_light_sensor` | `export_light_sensor` | Scrie fișierele calibrate `.daq` și `.csv` pentru înregistrările senzorului de lumină ale proiectului fără a procesa imaginile, preluând calibrarea din fabrică din MAPIR Cloud atunci când aceasta nu este stocată în cache (vezi [Înregistrări ale senzorului de lumină](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Citește metadatele de audit Chloros ale unei imagini procesate (sursă, calibrare, etape aplicate). |

**Camere și matrice**

| Instrument | Apel SDK | Ce face |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Ce este conectat în prezent, în orice client, fiecare cu `connected_here`; plus sarcinile care rulează pe server. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Camere și senzori de lumină care ar putea fi conectați. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Starea în timp real a unei camere, a unui ansamblu (cu capacitatea și starea de funcționare a acestuia) sau a unui senzor de lumină. |
| `analyze_array_network` | `analyze_array_network` | Ce pot suporta această gazdă și această rețea pentru o matrice propusă. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Conectarea unei camere sau a unei rețele sincronizate (mai întâi masterul). |
| `connect_project_devices` | `open_project(...).connect_all` | Conectați fiecare dispozitiv memorat într-un proiect salvat, împreună cu setările sale. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Dezactivați o cameră, o matrice sau un senzor de lumină (vezi Comportament). `keep_settings: true` aplicat unei matrice salvează setările fiecărei camere în propria sa memorie, iar `clear_settings: true` resetează acea memorie (rezultatul său generează `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Dispozitivele proiectului, împreună. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Citește sau scrie setările camerei, după numele descriptiv sau numele nodului GenIC. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Fixarea ratei unui tablou sau revenirea la rata automată. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | O singură captură (`fastest` și `wait_for_exposure` sunt `capture_fastest` și captura cu expunere stabilizată). În absența `output_dir`, se salvează în folderul deschis al proiectului`captures`, iar fișierele sunt adăugate la proiect (puse în așteptare până la finalizarea unui ciclu de procesare); răspunsul `project_add` confirmă acest lucru. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Capturi continue sau la intervale, ca sarcină. |
| `start_array_recording` | `ArraySession.record` / `burst` | Un videoclip cu index combinat sau o serie brută, ca sarcină. |
| `build_burst_video` | `ArraySession.build_video` | Reprocesarea unei serii de imagini într-un videoclip calibrat, ca sarcină. |
| `get_job_status` / `stop_job` | | Urmărirea sau oprirea unei sarcini. |
| `get_camera_preview` | | Previzualizarea în timp real a fiecărei camere, returnată sub formă de imagine. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Un cadru (sau un set sincronizat) la un nivel de procesare, fără salvare: statistici pe plan și un fișier PNG pentru fiecare. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Alinierea unui tablou. |

**Senzori de lumină**

| Instrument | Apel SDK | Ce face |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Conectează un DAQ-U, DAQ-M sau DAQ-E (`cap_id` setează corecția capacului acestuia). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Cele mai recente citiri: marcaj temporal, saturație, intervalul spectrului și valoarea de vârf, precum și spectrul mediu pe intervale de lungime de undă (`full_spectrum` pentru fiecare punct). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Pornirea sau oprirea fluxului. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Înregistrează un fișier `.daq`, ca sarcină; `duration_s` îl oprește automat. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Corecția capacului (`none` pentru un senzor neacoperit). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Care senzor furnizează reflectanța fiecărei camere și a fiecărei matrice. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Seturile de benzi, precum și reflectanța și indicii „Ambient + Obiect” ai celor doi senzori de flux continuu. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Setările proprii ale unui DAQ-E, salvate în unitate. |

**Utilitare pentru cameră** ([Utilitare pentru cameră](#camera-utilities))

| Instrument | Apel SDK | Ce face |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` și funcțiile lor de setare | Culoarea afișată de o cameră RGB: profilul și temperatura culorii, saturația / contrastul, zonele luminoase. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Echilibrarea albului cu o singură măsurătoare sau mod automat / oprit / raporturi fixe. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Setări prestabilite ale camerei. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Modelele de filtre sau unul după nume. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Versiuni de firmware și un bliț (o sarcină implicită). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Starea de alimentare și termică; repaus sau activare. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Coeficienți proveniți de la o țintă de calibrare afișată. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Adaptorii de rețea ai camerei, o estimare și soluția (Numai pentru administrator). |
| `apply_vignette_correction` | `apply_vignette_correction` | Corecția vignetării pentru un folder de imagini. |
| `read_dls` | `read_dls` | O citire DLS. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Hardware-ul și mediul de execuție al gazdei. |

**Camere tranzitorii, profiluri de aliniere și indici** ([Matrice tranzitorii, profiluri de aliniere și indici](#transient-arrays-alignment-profiles--indices)). Fiecare instrument de aici care deschide camere utilizează `keep_settings` / `clear_settings` (implicit false): consultați [Memoria camerei](#camera-memory--keep_settings--clear_settings).

| Instrument | Apel SDK | Ce face |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Înregistrează o singură cameră sau transmite în flux mai multe camere sincronizate, ca o sarcină (`duration_s` o oprește automat). |
| `capture_cameras_transient` | `capture_cameras_transient` | O singură captură sincronizată fără conectarea unui array. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Verificarea sincronizării declanșatorului și a camerelor aflate pe linia de sincronizare. |
| `analyze_array_models` | `analyze_array_models` | Ce ar obține o matrice formată din aceste modele în această rețea. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Rezolvarea și salvarea unui fișier de profil de aliniere (`options` conține opțiunile mai detaliate ale programului de rezolvare). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | O imagine multibandă aliniată sau un flux de astfel de imagini ca sarcină. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Conținutul unui fișier de profil; ordinea benzilor și camerele activate. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Presetări și gradienturi de index; un index pentru un fișier TIFF multibandă; captură către index într-un singur pas. |

**Sincronizare temporală, dosar de proiect și rețete** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Dosar de proiect și rețete](#project-folder--recipes))

| Instrument | Apel SDK | Ce face |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | Starea de funcționare a grandmasterului PTP, a peer-urilor acestuia și starea PTP a fiecărei camere din matrice. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Reporniți-l; setați-i prioritățile. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Dosarul în care sunt create proiectele noi. |
| `run_project_recipe` | `run_project_recipe` | Execută rețeta unui proiect salvat, ca sarcină, împreună cu jurnalul acestuia. Verificat mai întâi: o rețetă care conține o eroare este respinsă și nu se execută nimic; se afișează avertismente privind matrice/senzori neconectați (nu îi conectează: mai întâi se execută `connect_project_devices`). `dry_run` încheie verificarea și nu execută nimic. |

Nu există niciun instrument pentru a deschide un proiect salvat în vederea procesării: fiecare sarcină de procesare începe cu `create_project` (sau `process_folder`).

### Cerințe

- **Chloros instalat pe același computer.** Backend-ul Chloros se ocupă de procesare. Serverul poate utiliza, în schimb, un backend accesat printr-un tunel (`--backend-url` cu `--no-auto-start`, vezi [Opțiuni](#options)).
- **Autentificat pe acel computer**, în aplicația Chloros sau cu `chloros-cli login`. Serverul nu se poate autentifica în locul dvs.: un instrument apelat în timp ce sunteți deconectat răspunde „Nu sunteți autentificat în Chloros” cu motivul indicat de backend, iar asistentul vă informează.
- **Orice nivel plătit Chloros+ (Copper sau superior).** Serverul are acces la SDK, iar nivelul gratuit Iron nu are: un instrument apelat la nivelul gratuit răspunde „Nu este disponibil în acest plan Chloros” cu motivul furnizat de backend.
- Python 3.7 sau o versiune mai recentă cu SDK și un client MCP care pornește servere locale (stdio).

### Comportament

- **Backend-ul.** Lansarea serverului sau afișarea listei de instrumente nu inițiază nimic; prima apelare a unui instrument stabilește conexiunea. Dacă aplicația Chloros (sau serviciul backend Linux) rulează, serverul utilizează backend-ul acesteia și îl lasă să ruleze. Dacă nu rulează niciun backend, serverul îl pornește pe cel instalat, fără fereastră. Când serverul se închide (ieșiți din client sau acesta repornește serverul), acesta oprește sarcinile de captură și înregistrările senzorului de lumină pe care le-a pornit și eliberează dispozitivele la care s-a conectat; backend-ul continuă să ruleze, astfel încât o procesare neterminată se desfășoară până la final, iar acesta se oprește automat după 30 de minute dacă nu este utilizat de nimeni. Cu `--no-auto-start`, serverul nu pornește niciodată un backend: o comandă a unui instrument, atunci când nu rulează niciunul, vă solicită să deschideți aplicația Chloros, iar următoarea comandă, după ce faceți acest lucru, stabilește conexiunea.
- **Un singur proiect deschis.** Backend-ul are un singur proiect deschis, partajat cu aplicația Chloros atunci când aceasta este deschisă, așa că nu lucrați la un alt proiect în aplicație în timp ce asistentul funcționează.
- **Se modifică doar setările transmise.** `configure_processing` lasă fiecare setare care nu i-a fost furnizată la valoarea sa salvată (SDK-ul `configure()` resetează debayer-ul, corecția de vignetă, calibrarea reflectanței, PPK și formatul de export la valorile implicite atunci când acestea nu sunt transmise; serverul le completează din proiect). Dacă nu poate citi setările salvate ale proiectului, nu modifică nimic și afișează un mesaj în acest sens, cu excepția cazului în care sunt specificate toate cele cinci. `indices` înlocuiește lista de indici a proiectului. Rezultatul listează ceea ce a fost trimis ca `settings_sent`, iar sub `warnings` denumește orice index pe care benzile camerei nu îl pot genera.
- **Prelucrarea se realizează după principiul „start-then-poll”.** `start_processing` returnează imediat ce începe execuția, care durează apoi de la câteva minute la câteva ore, iar asistentul continuă cu `get_processing_status`. `state` este `idle` (nu se procesează nimic), `starting` (backend-ul nu a raportat încă execuția, pentru o perioadă de până la 20 de secunde), `processing` sau `finished`, împreună cu [rezumat](#post-run-summary--hints): ce a exportat și indicații atunci când a exportat mai puțin decât se aștepta (fără imagini, nu s-a găsit ținta de calibrare). Produsele sunt salvate în folderul proiectului, la fel ca în aplicație. `start_processing` este respins în timp ce o execuție este în curs, iar un backend care nu mai răspunde este raportat ca eroare, nu ca o execuție finalizată.
- **Ce vede asistentul.** Avertismentele SDK-ului apar în rezultatul instrumentului sub `warnings`, iar eșecurile (deconectare, plan, cale incorectă) sunt afișate ca text de eroare pe care asistentul îl poate citi și transmite mai departe către dvs. O listă cu mai mult de 20 de elemente este redusă la primele 20, iar lungimea completă este indicată ca `<key>_total`. Căile trebuie să fie absolute.
- **Dispozitivele sunt partajate.** Camerele, matrice și senzori de lumină se află în backend, astfel încât asistentul poate controla unul dintre ele prin aplicația Chloros, iar modificările efectuate se afișează în aplicație. `disconnect_device` afișează ce a conectat serverul; un dispozitiv la care s-a conectat un alt client (aplicația, `chloros-cli`, un script, un alt `chloros-mcp`) este eliberat numai cu `force: true`, care este `force_disconnect`-ul SDK-ului, iar refuzul indică cine l-a conectat atunci când backend-ul știe; un dispozitiv pe care backend-ul nu îl deține deloc este raportat ca neconectat (`released: false`, `connected: false`, nimic de eliberat). O înregistrare sau un flux rulat de backend generează vizualizarea sa `get_job_status` împreună cu `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (grupuri de cadre pentru un flux multi-cameră) — și o linie `summary` („220 de cadre scrise, 55 pierdute, 0 eșuate: ...”), în timp ce rulează și, la final, cifrele finale; lista unui alt client referitoare la acesta (`other_clients_jobs`) conține aceleași informații (`list_cameras()`, `list_arrays()` și `list_daq_sensors()` conțin fiecare`connected_by`: `client`, `label`, `name`, `at`). Fiecare `chloros-mcp` se identifică în fața backend-ului ca `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, trimis ca `X-Chloros-Client-Label`; orice script SDK îl poate seta). Oprirea înregistrării senzorului de lumină al unui alt client necesită, de asemenea, `force`.
- **O activitate de lungă durată este o sarcină.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, iar orice captură, rezolvare sau index solicitat pentru `background` returnează imediat un `job_id`; `get_job_status` urmează, iar `stop_job` încheie . O execuție continuă până la oprire, cu excepția cazului în care îi transmiteți `max_captures`, `duration_s` (sau, pentru o serie, `max_frames`); `min_free_gb` refuză să pornească atunci când unitatea de stocare are prea puțin spațiu liber. Nu se aplică nicio limită, cu excepția cazului în care setați una. O înregistrare sau un flux rulat de backend (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) păstrează ID-ul sarcinii backend-ului, astfel încât un alt `chloros-mcp` — unul pornit ulterior — îl găsește după acel ID: `get_job_status` îl urmărește (și listează astfel de sarcini precum `other_clients_jobs`, de la `GET /api/camera/cli-jobs`, fiecare cu `started_by`), iar `stop_job` o oprește doar cu ajutorul lui `force: true`. Orice altă sarcină (o captură, o înregistrare în matrice sau un burst, o captură în fundal) rulează în `chloros-mcp` care a pornit-o, și numai acel server știe acest lucru: altul care a solicitat-o confirmă acest lucru. Una oprită citește `stopping` în timp ce își scrie coada de așteptare, apoi `finished` cu numărările finale, timp de 10 minute și de la orice `chloros-mcp`: cel care a inițiat procesul observă oprirea forțată a altuia în același mod. `get_job_status` nu așteaptă niciodată o oprire: în timpul unei opriri (a propriului `stop_job`, a altui clientsau oprirea care eliberează camerele unei sarcini care s-a încheiat de la sine), acesta răspunde imediat cu `stopping` și numărările în timp real, iar `stop_job` al unei sarcini care se oprește deja indică acest lucru. O sarcină care s-a încheiat precizează motivul în `stop_reason`: motivul propriu (`no_frames`, `disk_full`, `member_missing`, sau, pentru o execuție continuă a unui șir, `count`, `duration`, `array_gone`), `limit reached` doar atunci când valorile sale indică `count` (sau `max_frames` al unei rafale, `duration_s` al unei înregistrări) a fost atins, `error`, `stopped`,`duration_s`) a fost atins, `error`, `stopped` (cu `stopped_by`) atunci când un client l-a oprit, altfel `finished`.
- **Unde se salvează capturile.** În folderul `captures` al proiectului deschis, unde aplicația le salvează, cu excepția cazului în care se specifică `output_dir`.
- **Imagini.** `get_camera_preview` și `grab_frames` returnează imagini pe care asistentul le poate vizualiza, pe lângă fișierul JSON corespunzător. Imaginile `grab_frames` au contrastul ajustat pentru vizualizare; statisticile acestora reprezintă valorile reale.
- **Când serverul se închide**, sarcinile pe care le-a pornit se opresc, iar înregistrarea senzorului de lumină pe care a inițiat-o este închisă. Dacă serverul a pornit backend-ul, eliberează și dispozitivele la care s-a conectat înainte de a opri backend-ul; în backend-ul aplicației, acestea rămân conectate.
- **Protocol.** Numai stdio (fără punct final HTTP), numai instrumente (fără resurse sau solicitări de confirmare), rezultate sub formă de text și imagini, versiunile protocolului MCP din perioada 05.11.2024 – 25.11.2025.**Ce nu este inclus.** Autentificarea sau deconectarea (contul este al dvs.; serverul nu se poate autentifica) și oprirea unui backend pe care serverul nu l-a pornit. Pentru orice altceva ce face acest SDK în ceea ce privește procesarea și hardware-ul există un instrument. (Configurarea unui DAQ-E — numele său, tokenul de autentificare, parola OTA, cerința de semnătură, resetarea la setările din fabrică — iar serverele DAQ directe sunt exclusiv ale `chloros-cli`, incluse și în SDK: un nume sau un token greșit blochează orice instalare Chloros din unitate.)

---

## Pornirea automată a backend-ului

Punctele de intrare smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` și `discover_daq_sensors` — sunt clienți HTTP ușori care presupun că un backend ascultă pe `127.0.0.1:5000` (URL-ul implicit al interfeței smart-connect). Atunci când interfața grafică (GUI) sau interfața de linie de comandă (CLI) rulează deja, unul dintre acestea este activ. Din cadrul unui script simplu, este posibil să nu fie — așa că aceste funcții **pornesc automat binarul backend inclus în pachet** (fără fereastră, la fel cum o face `ChlorosLocal`) înainte de prima lor apelare, apoi așteaptă până la 60 de secunde până când acesta pornește. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` și `analyze_array_network()` nu pornesc niciodată unul.

Reguli:

- **Se generează întotdeauna doar un URL-ul local.** Un `backend_url` care indică către `localhost` / `127.0.0.1` / `[::1]` este eligibil; orice altă gazdă este considerată a fi mașina altcuiva și nu este niciodată generată.
- **Un backend care se închide înainte de a răspunde pune capăt așteptării.** Apelul se încheie imediat cu codul de ieșire, comanda pe care a rulat-o și ultimele linii afișate de backend, în loc să aștepte cele 60 de secunde. Codul de ieșire 98 reprezintă excepția: un alt backend deține deja portul sau fișierul de blocare (al aplicației Chloros, al serviciului backend sau al unui alt script care încă se inițializează), astfel încât apelul continuă să aștepte răspunsul celui respectiv. `ChlorosLocal` se comportă la fel.
- **Backend-ul este lăsat să ruleze pentru reutilizare** (la fel ca în CLI) — nu are loc o oprire implicită la ieșirea scriptului. Rularea din nou a scriptului reutilizează backend-ul activ.
- **Dezactivați această opțiune cu `auto_start_backend=False`** pentru oricare dintre aceste apeluri (de exemplu, când ați indicat un backend la distanță sau când gestionați singur ciclul de viață al backend-ului).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Dacă binarul inclus nu poate fi localizat sau pornit, apelul HTTP ulterior generează o eroare `ChlorosConnectError` **adaptată la platformă**, care permite acțiuni concrete, în loc de o simplă urmărire de tip „conexiune refuzată” — pe Windows, acesta vă îndrumă către aplicația desktop sau către o comandă `chloros-cli`; pe Linux (fără GUI), acesta vă îndrumă către o comandă `chloros-cli` sau către `.deb`.

---

## Mediu și antete

SDK-ul marchează fiecare apel HTTP către backend cu `X-Chloros-Client: sdk`. Backend-ul aplică regulile de licențiere SDK/CLI (este necesară autentificarea **și** un plan Chloros+ plătit), în loc de calea de acces gratuită a GUI-ului. Această setare se face automat la momentul importului — nu trebuie să faceți nimic.

Antetul este adăugat la fiecare apel `requests` / `urllib` al cărui URL conține `localhost` sau `127.0.0.1`, pe orice port — inclusiv propriile servicii locale — și înlocuiește o valoare pe care ați setat-o dumneavoastră. Apelurile către alte gazde rămân neschimbate.

Puteți suprascrie URL-ul backend-ului trecând `backend_url=` (`ChlorosLocal` acceptă `backend_url=` sau `api_url=`; `process_folder()` și `process_lattice_capture()` acceptă doar `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Un backend instalat ascultă numai pe loopback, astfel încât un `backend_url` din rețeaua LAN nu îl poate accesa; consultați [Modul backend la distanță](#remote-backend-mode-pip-only-host-via-tunnel) pentru modelul de tunel.)

---

## Versiuni și compatibilitate

- Versiunea SDK-ului este expusă ca `chloros_sdk.__version__`.
- SDK-ul leagă comportamentul de versiunea backend-ului inclus în pachet. Combinarea unui SDK mai vechi cu un backend mai nou funcționează de obicei (puncte finale compatibile cu versiunile ulterioare), dar combinarea unui SDK mai nou cu un backend mai vechi poate genera erori `404` pe punctele finale noi — actualizați aplicația desktop pentru a se potrivi.
- Interfața smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) și punctul final de analiză a rețelei returnează scheme JSON stabile; câmpurile noi sunt adăugate.

---

## Indicații pentru depanare

- **`ChlorosAuthenticationError: Login required`** (din `ChlorosLocal`; apelurile Smart-Connect generează eroarea `ChlorosConnectError: Login required. …` din același motiv) → Rulați o dată `chloros-cli login EMAIL PASSWORD` pe acest computer sau conectați-vă prin intermediul aplicației desktop Chloros.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (sau `Failed to create project: …` din `create_project()`) → Căile lungi din Windows sunt dezactivate pe acest computer (setarea implicită a Windows), așa că nicio cale nu poate depăși 260 de caractere, iar produsele se află cu până la aproximativ 130 de caractere sub folderul proiectului. Nu s-a procesat nimic. Mesajul indică cel mai lung folder de proiect care se încadrează: utilizați unul mai scurt sau activați căile lungi dWindows și reporniți Chloros. Programul de instalare Windows oferă această opțiune sub denumirea „Activați căile lungi Windows”, bifată implicit; în caz contrar, utilizați Politica de grup: *Configurare computer > Șabloane administrative > Sistem > Sistem de fișiere > Activați căile lungi Win32*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Funcția Smart-Connect apelează pornirea automată a unui backend local, așa că această eroare apare doar când binarul inclus nu poate fi găsit/pornit (de exemplu, un host care rulează doar pipfără pachet desktop). Mesajul ține cont de platformă: pe Windows, deschideți aplicația desktop sau rulați orice comandă `chloros-cli`; pe Linux, rulați o comandă `chloros-cli` (nu există interfață grafică) sau instalați `.deb`. Pentru un backend la distanță, treceți `backend_url=` (și `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** la import → `lattice_sdk` nu s-a putut încărca: runtime-ul SDK-ului Arena sau OpenCV (`cv2`), nu pot fi importate în această versiune de Python. Suprafața controlată de backend (`connect_*`, proiectată prin intermediul unui backend, `ChlorosLocal`) funcționează în continuare.
- **Array se conectează la senzorul complet, dar cu o rată de cadre redusă**→ Conform proiectării: modul automat păstrează senzorul complet și permite ca rata să scadă la ceea ce suportă cablul. Treceți la `binning=2` / `4` sau un `width` / `height` mai mic pentru mai multe cadre pe secundă, sau activați cadrele jumbo / actualizați conexiunea. Binningul și ROI**nu** acoperă suprasubscrierea agregată (`oversubscribed: true`, câmpurile fps 0): numărul prea mare de camere pentru conexiunea de rețea nu poate fi remediat prin binning/ROI — reduceți numărul de camere, activați cadrele jumbo sau treceți la o placă de rețea mai rapidă (consultați [Supra-abonare](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` raportează că inelul de recepție al plăcii de rețea este foarte mic (~0,26 MB) / porți de conectare cu „FRAMES WILL DROP”** → Inelul de recepție al plăcii de rețea a gazdei este la valoarea implicită (adesea resetat la 32 după o actualizare a driverului plăcii de rețea). Pe un adaptor Realtek USB 10GbE, setați `ReceiveBufferLen=256` și `PendingReceives=64` (cu drepturi ridicate), apoi reporniți backend-ul pentru ca acesta să recitească inelul. Procedura completă: [Referință CLI → Configurare și reglare NIC gazdă](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Gazda se blochează la repornire/oprire, ulterior apar erori WMI `Invalid class` / placa de rețea nu se activează** → Driverul USB 10GbE învechit provoacă `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Actualizați driverul adaptorului la o versiune curentă (≥ 2026) și reaplicați setările inelului de recepție. Consultați [Referință CLI → Configurare și reglare NIC gazdă](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Captura matricei eșuează cu eroarea HTTP 503 „No fresh synced frame group”** → Matricea nu a publicat niciun grup de cadre nou în timpul perioadei de așteptare. Verificați `arr.status()`: `stalled: True` (sau `streaming: False`) cu `actual_fps` 0, ceea ce înseamnă că bucla de captare a încetat să publice — deconectați și reconectați matricea. Dacă `stalled` este False și `actual_fps` este mai mare decât 0, încercați din nou. Imediat după `connect_array()`, o captură așteaptă deja până la 20 de secunde pentru primul grup.
- **`status()` afișează `link_speed_mbps: 4294`** → Windows-ul nu a putut citi viteza adaptorului; acest lucru este tipic pentru adaptoarele USB Ethernet și nu reprezintă o eroare. Consultați [Ratele de cadre așteptate](#expected-frame-rates).
- **`update_firmware()` eșuează cu `device never connected back on TCP <port>`** → Un firewall a blocat conexiunea DAQ-E către acest computer. Programul de instalare Windows adaugă regula (consultați [Setările dispozitivului DAQ-E](#daq-e-device-settings)); în cazul unui alt firewall, permiteți traficul TCP de intrare către backend din subrețeaua senzorului.
- **Reflectanța a fost refuzată** → Pentru reflectanța la scară absolută, un DAQ activ trebuie asociat cu camera (sau matricea). Asociați-l fie prin interfața grafică, fie utilizați `processing="radiance"` (W/m²/sr/nm), care nu necesită un senzor asociat.
- **Captura cu `smart=True` durează mai mult decât era de așteptat** → Convergența AE depinde de dinamica scenei. Cu `ArrayHandle.capture_smart()`, lărgiți `exposure_tolerance_pct`, scurtați `stability_window_s` sau reduceți `settle_timeout_s` pentru o declanșare mai rapidă și mai puțin stabilă.

---

## Vezi și

- [Referință CLI](cli-reference.md) — fiecare subcomandă CLI corespunde unui apel SDK.
- [Senzori de lumină DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) în manualul Chloros — senzorii, limitele și intervalul calibrat al acestora, înregistrarea, rețeaua DAQ-E.
- [Camere LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) în manualul Chloros — conectare, setări ale camerei, moduri de captură, matrice.
- Documentație online: `https://mapir.gitbook.io/chloros/api-python-sdk`
