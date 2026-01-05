# Chloros+ Autentificare

## Chloros și Chloros (Browser) Autentificare

Meniul lateral <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> din bara laterală vă permite să vă conectați la contul dvs. Chloros+ și să deblocați funcții suplimentare.

După conectare, vor fi afișate detaliile contului dvs.:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" data-size="line"><figcaption></figcaption></figure>## CLI Autentificare

Autentificați-vă cu datele de autentificare Chloros+ pentru a activa procesarea CLI.

**Sintaxă:**

```bash
chloros-cli login <email> <password>
```

{% hint style=&quot;info&quot; %}
**Utilizatori SDK**: Python SDK oferă, de asemenea, o metodă programatică `logout()` pentru ștergerea datelor de autentificare stocate în memoria cache. Pentru detalii, consultați [documentația Python SDK](api-python-sdk.md#logout).
{% endhint %}

**Exemplu:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Caractere speciale**: Utilizați ghilimele simple în jurul parolelor care conțin caractere precum `$`, `!` sau spații.
{% endhint %}

**Ieșire:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Expirarea planului

Expirarea planului în GUI arată când licența dvs. va deveni invalidă. Pentru abonamentele lunare recurente, expirarea are loc la sfârșitul lunii. Pentru abonamentele anuale, expirarea are loc la un an după începerea abonamentului. Verificarea licenței necesită o conexiune lunară la internet, cu o perioadă de grație de 30 de zile.

### Limită de dispozitive

Fiecare plan Chloros+ oferă un număr diferit de dispozitive înregistrate. Fiecare dispozitiv pe care vă conectați cu un cont Chloros+ va fi luat în calcul la numărul de dispozitive înregistrate. Puteți redenumi și elimina un dispozitiv din pagina contului dvs. MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Planul Chloros</th><th align="center">COPPER</th><th align="center">BRONZ</th><th align="center">SILVER</th><th align="center">AUR</th></tr></thead><tbody><tr><td align="right">Dispozitive acceptate</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
