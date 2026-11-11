# Autentificare Chloros+

## Autentificare prin interfața grafică (GUI)

Meniul din bara laterală a utilizatorului <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> vă permite să vă autentificați în contul Chloros+ și să deblocați funcții suplimentare.

**Trebuie să vă autentificați o singură dată pe fiecare computer.** Interfața grafică (GUI), interfața de linie de comandă (CLI) și SDK-ul Python partajează aceeași sesiune stocată în cache — conectarea prin interfața grafică de pe desktop activează, de asemenea, CLI-ul și SDK-ul pe acel computer (și invers, prin `chloros-cli login`).

După conectare, vor fi afișate detaliile contului dumneavoastră:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Niveluri de abonament

| Abonament | `plan_id` | Tip |
| --- | --- | --- |
| Iron | `0` | Gratuit |
| Copper | `1` | Cu plată (Chloros

+) |
| Bronze | `2` | Cu plată (Chloros

+) |
| Silver | `3` | Cu plată (Chloros

+) |
| Aur | `4` | Cu plată (Chloros

+) |

Consultați [planurile și prețurile](https://cloud.mapir.camera/pricing) pentru a afla ce include fiecare nivel cu plată.

### Accesul la CLI

/SDK

necesită un nivel cu plată

Accesul la CLI

și la Python

SDK

necesită **orice nivel plătit Chloros

+ (Copper sau superior)**. Această cerință este aplicată**la nivel de server** — fiecare solicitare CLI

/SDK

trebuie să includă atât o sesiune activă, cât și un plan plătit:

| Stare HTTP

| `error_code` | Semnificație | Remediere |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Nu sunteți autentificat pe acest computer | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Sunteți conectat, dar nivelul planului este prea scăzut (nivelul gratuit Iron) | Treceți la orice plan plătit Chloros

+ |

`chloros-cli status` rămâne accesibil în cadrul planului gratuit, astfel încât puteți vedea oricând planul dvs. actual și motivul pentru care accesul este refuzat.

### Limite privind echipamentele conectate pentru fiecare plan

Fiecare plan limitează numărul de camere LATTICE și de senzori de lumină DAQ care pot fi conectați simultan în timp real:

| Plan | Camere LATTICE | Senzori de lumină DAQ |
| --- | --- | --- |
| Iron (gratuit / fără autentificare) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Autentificare CLI



Autentificați-vă cu datele de autentificare Chloros

+ pentru a activa procesarea CLI

. Pe Linux

(fără GUI), aceasta este singura modalitate de a vă activa licența.

**Sintaxă:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Utilizatori SDK**: Python

SDK

oferă, de asemenea, o metodă programatică `logout()` pentru a șterge datele de autentificare stocate în cache. Consultați [Referința SDK

](reference/sdk-reference.md) pentru detalii.
{% endhint %}

**Exemplu:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caractere speciale**: Folosiți ghilimele simple în jurul parolelor care conțin caractere precum `$`, `!` sau spații.
{% endhint %}

**Rezultat:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Stocarea datelor de autentificare

Datele de autentificare și configurația stocate în cache sunt păstrate în folderul `.chloros` din directorul de bază al utilizatorului pe **toate platformele**:

| Platformă | Calea către cache-ul de date de autentificare |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

Instalarea unei versiuni mai noi peste una existentă păstrează datele dvs. de autentificare, atât pe Windows

(programul de instalare), cât și pe Linux

(`.deb`, inclusiv `chloros-cli update`). Dezinstalarea Chloros

— sau eliminarea pachetului Linux

— șterge datele de autentificare stocate în cache, astfel încât va trebui să vă autentificați din nou după o instalare nouă.

### Expirarea abonamentului și perioada de grație offline

Data de expirare a abonamentului afișată în interfața grafică indică momentul în care licența dvs. va deveni invalidă. Pentru abonamentele lunare recurente, expirarea are loc la sfârșitul lunii; pentru abonamentele anuale, aceasta are loc la un an de la începerea abonamentului.

Chloros

vă validează licența online, dar funcționarea offline este acceptată în cadrul unei perioade de grație:

* Validările reușite pe server sunt stocate în cache timp de **5 minute**, astfel încât utilizarea normală generează foarte puține solicitări de licență.
* O memorie cache a licenței semnate și asociate unui anumit dispozitiv acoperă perioade offline mai lungi: **30 de zile pentru planurile lunare**și**până la data de expirare a abonamentului (cel mult 365 de zile) pentru planurile anuale**.
* La expirarea perioadei de grație, planul revine la nivelul gratuit „Iron” până când dispozitivul reușește să acceseze serverul de licențe o singură dată; accesul se reia la următoarea verificare reușită.

### Limita de dispozitive

Fiecare plan Chloros

+ oferă un număr diferit de dispozitive înregistrate. Fiecare dispozitiv pe care vă conectați cu un cont Chloros

+ este luat în calcul la numărul de dispozitive înregistrate. Puteți redenumi și elimina un dispozitiv de pe pagina contului dvs. MAPIR

Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Planul Chloros</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Dispozitive compatibile</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Numărul exact de dispozitive permise pentru contul dvs. este afișat pe pagina contului dvs. MAPIR Cloud. Deconectarea de pe un dispozitiv eliberează în mod sigur locul acestuia, iar un dispozitiv deja înregistrat se poate reconecta oricând, chiar și atunci când contul a atins limita de dispozitive.
