# Conectarea camerelor

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Fila „Camere” înainte de conectarea vreunui dispozitiv</p></figcaption></figure>Chloros detectează automat camerele LATTICE pe conexiune — din fila „Camere” a interfeței grafice, din `chloros-cli lattice` sau din Python SDK. Șirul de caractere care indică modelul camerei determină toate etapele ulterioare: Chloros determină profilul senzorului, configurația benzilor și calibrarea din fabrică pe baza parametrilor `DeviceUserID` + `DeviceSerialNumber` ai camerei, astfel încât **nu este necesară nicio configurare pentru fiecare cameră în parte**.

Înainte de conectare, asigurați-vă că rețeaua gazdă este configurată — adresare link-local, cadre jumbo și, pentru matrice, setările bufferului de recepție al plăcii de rețea (NIC). Aceasta este configurarea la nivel de hardware și se regăsește în manualul LATTICE: [**Configurare rețea**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Conectarea din interfața grafică

Deschideți fila **Camere**din bara laterală Chloros (filele de hardware apar odată ce backend-ul a terminat de pornit) sau utilizați meniul principal →**Conectare la cameră**. Ambele opțiuni deschid fereastra de dialog**Conectare cameră(e)**.

### Fereastra de dialog **Conectare cameră (camere)**Fereastra de dialog scanează rețeaua imediat ce se deschide („Se scanează rețeaua...”) și afișează toate camerele pe care le găsește. Fiecare rând afișează**modelul**camerei (de ex. `LATT-M3M-L41-F550`),**numărul de serie**și**adresa IP**.

* **Faceți clic pe un rând pentru a-l selecta**(evidențiere verde). Puteți selecta**mai multe camere** și le puteți conecta dintr-o singură mișcare — Chloros le conectează în ordine.
* Rândurile cu o insignă **„Conectat”** sunt deja conectate și nu pot fi selectate din nou.
* Rândurile cu o insignă **„În grup”** aparțin unui grup de camere conectat în prezent. Deconectați mai întâi grupul pentru a utiliza acea cameră independent.
* **Conectare** — conectează camera (camerele) selectată (selectate); butonul afișează un număr, de exemplu „Conectare (3)”, atunci când sunt selectate mai multe camere.
* **Rescanare** — relansează procesul de detectare.
* **Închidere** — închide fereastra de dialog.
* Dacă scanarea se încheie fără rezultate, fereastra de dialog afișează **„Nu s-au găsit camere în rețea”** — consultați secțiunea [Depanare](connecting.md#troubleshooting) de mai jos.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Fereastra de dialog „Conectare cameră(e)” — afișată aici fără camere în rețea</p></figcaption></figure>### Prima conectare: descărcarea pachetului de calibrare

La **prima conectare**a unei anumite camere la un echipament, Chloros preia pachetul de calibrare din fabrică al camerei (\~3,8 MB) direct de pe cameră prin GigE. În timp ce acest proces se desfășoară, fereastra de dialog afișează un panou verde**„Se descarcă datele de calibrare de la cameră”**cu o bară de progres pentru fiecare număr de serie — durata estimată este de aproximativ**70 de secunde** per cameră. Pachetul este stocat în memoria cache a gazdei, astfel încât conectările ulterioare ale aceleiași camere omit complet descărcarea (și nu afișează niciodată panoul).

### Analizează sistemul

Butonul **Analizează sistemul** din fereastra de dialog examinează gazda și rețeaua (eticheta afișează „Se analizează...” în timpul procesului) și generează un raport de diagnosticare:

* **Gazdă** — nuclee CPU și memorie RAM; numele și memoria GPU-ului, sau „GPU: Nu s-a detectat niciunul”.
* **Interfețe de rețea** — numele fiecărei plăci de rețea, viteza de conexiune, MTU (cu o etichetă „jumbo” acolo unde este activă), starea de activare/dezactivare și dacă se află pe un bus USB.
* **Camere**— număr de serie, model, adresă IP și**pe ce placă de rețea se află fiecare cameră**.
* **Performanță** — fps actual vs. ideal per cameră pentru formatul de pixeli, cu o linie verde „Potențial: îmbunătățire de N× posibilă” atunci când valoarea ideală o depășește pe cea actuală.
* **Avertismente și recomandări numerotate** — sau „Sistemul pare în regulă pentru numărul actual de camere” atunci când nu există nimic de remediat.

Rulați-l ori de câte ori detectarea sau redarea în flux se comportă neașteptat — acesta identifică majoritatea problemelor legate de plăcile de rețea (MTU incorect, cameră conectată la interfața greșită, limitări ale adaptorului USB) fără a părăsi fereastra de dialog.

### Conectarea unui ansamblu

Pentru a conecta două sau mai multe camere ca un **ansamblu sincronizat**, utilizați în schimb expertul de conectare a ansamblului (**Conectare ansamblu de camere**): acesta vă ghidează prin selectarea modului master/slave (precompletat de o sondă de cablare GPIO), alegerea modului de afișare (mosaic separat vs. combinat) și o scenă de setări pentru grup, cu o proiecție în timp real a numărului de cadre pe secundă (fps) și a lățimii de bandă a cablului, înainte de a confirma. Asistentul și fluxurile de lucru pentru rețea sunt descrise în secțiunea dedicată rețelelor cu mai multe camere din acest manual; echivalentul pentru CLI este „Fluxul de lucru pentru prima conectare a camerei LATTICE” din [Referința CLI](../reference/cli-reference.md).

## Conectarea din CLI și SDK

Accesul la CLI și SDK necesită un plan plătit Chloros+ și autentificarea; această restricție este aplicată la nivel de server (`401 AUTH_REQUIRED` atunci când nu sunteți autentificat, `403 PLAN_UPGRADE_REQUIRED` la nivelul gratuit).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Semnături complete, opțiuni și fluxuri de lucru de captură: [CLI Referință](../reference/cli-reference.md) § `chloros-cli lattice`, [Referință SDK](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Cum se rezolvă calibrarea la conectare

Fiecare cameră LATTICE are pachetul de calibrare din fabrică **încorporat în cameră**, iar Chloros verifică și cloud-ul MAPIR atunci când camera se conectează:

| Situație   | Ce utilizează Chloros                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**|**Cea mai recentă calibrare publicată pentru acel număr de serie** — copia din cloud are prioritate față de cea de pe cameră. Prin urmare, o cameră care a fost recalibrată sau actualizată de MAPIR se actualizează automat; nu este necesară nicio acțiune din partea utilizatorului. |
| **Offline**|**Pachetul din cameră**, așa cum este. Fluxurile de lucru complet offline continuă să funcționeze; pur și simplu nu preiau calibrările mai noi până când camera nu este conectată la internet măcar o dată (sau nu este reflasată la setările din fabrică).                                                  |

La momentul capturii, coeficienții aplicați efectiv sunt **fixați în metadatele XMP ale fiecărei imagini**. O actualizare ulterioară a calibrării nu modifică niciodată în mod silențios imaginile pe care le-ați capturat deja — reprocesarea unei capturi vechi utilizează coeficienții înregistrați în metadatele XMP ale acesteia, nu cei mai recenți din prezent.

## Depanare

* **„Nu s-au găsit camere în rețea”**— verificați configurarea link-local în [Configurare rețea](https://mapir.gitbook.io/lattice-camera/setup/network-setup): adresă statică a plăcii de rețea a gazdei `169.254.x.x/16`, camerele pe același link, nu se așteaptă DHCP/gateway. Apoi utilizați**Analizare sistem**în fereastra de conectare pentru a verifica pe ce placă de rețea este (sau nu este) vizibilă fiecare cameră.**Reanalizați** după orice modificare a cablajului sau a plăcii de rețea.
* **Un sistem care funcționa anterior refuză să se conecteze** (panoul matricei cu `FRAMES WILL DROP` / `Reduce ROI to enable`) — o actualizare a driverului plăcii de rețea a resetat în mod silențios setările inelului de recepție. Reaplicați-le sau rulați `chloros-cli lattice network --fix` dintr-un terminal cu drepturi de administrator; consultați [Configurarea rețelei](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **O cameră afișează „In Array”** — aceasta aparține unei sesiuni de matrice conectate. Deconectați matricea pentru a utiliza camera în mod autonom.
