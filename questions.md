# Vaihe 3: Service-kerros, Repository, Result Pattern ja API-dokumentaatio — Teoriakysymykset

Vastaa alla oleviin kysymyksiin omin sanoin. Kirjoita vastauksesi kysymysten alle.

> **Vinkki:** Jos jokin kysymys tuntuu vaikealta, palaa lukemaan teoriamateriaalit:
> - [Service-kerros ja DI](https://github.com/xamk-mire/Xamk-wiki/blob/main/C%23/fin/04-Advanced/WebAPI/Services-and-DI.md)
> - [Repository Pattern](https://github.com/xamk-mire/Xamk-wiki/blob/main/C%23/fin/04-Advanced/Patterns/Repository-Pattern.md)
> - [Result Pattern](https://github.com/xamk-mire/Xamk-wiki/blob/main/C%23/fin/04-Advanced/Patterns/Result-Pattern.md)

---

## Osa 1: Service-kerros

### Kysymys 1: Fat Controller -ongelma

Miksi on ongelma jos controller sisältää kaiken logiikan (tietokantakyselyt, muunnokset, validoinnin)? Anna vähintään kaksi konkreettista haittaa.

**Vastaus:**

Controller kasvaa satoja rivejä pitkäksi
Sama logiikka kopioituu useaan paikkaan

---

### Kysymys 2: Vastuunjako

Miten vastuut jakautuvat controller:n, service:n ja repository:n välillä tässä harjoituksessa? Kirjoita lyhyt kuvaus kunkin kerroksen tehtävästä.

**Controller vastaa:**
"Mitä HTTP-pyynnöllä haluttiin?"

**Service vastaa:**
"Mitä sovellus tekee?"

**Repository vastaa:**
"Miten data haetaan?"

---

### Kysymys 3: DTO-muunnokset servicessä

Miksi DTO ↔ Entity -muunnokset kuuluvat serviceen eikä controlleriin? Mitä hyötyä siitä on, että controller ei tunne `Product`-entiteettiä lainkaan?

**Vastaus:**
Service on luokka, joka sisältää sovelluksen liiketoimintalogiikan. Controller kutsuu servicea, eikä tiedä miten se toimii sisältä.

---

## Osa 2: Interface ja Dependency Injection

### Kysymys 4: Interface vs. konkreettinen luokka

Miksi controller injektoi `IProductService`-interfacen eikä suoraan `ProductService`-luokkaa? Mitä hyötyä tästä on?

**Vastaus:**
Controller riippuu vain IProductService-sopimuksesta, testauksessa voidaan korvata mock-toteutuksella, toteutus voidaan vaihtaa muuttamatta controlleria

---

### Kysymys 5: DI-elinkaaret

Selitä ero näiden kolmen elinkaaren välillä ja anna esimerkki milloin kutakin käytetään:

- **AddScoped:**
  Yksi HTTP-pyyntö, yleisin palvelut jotka käyttävät DbContextia
- **AddSingleton:**
  Sovelluksen koko elinkaari, konfiguraatio, välimuisti, yhteyspoolit
- **AddTransient:**
  Luodaan uusi joka kerta, Kevyet, tilattomat apuluokat
  
Miksi `AddScoped` on oikea valinta `ProductService`:lle?
AddScoped on oikea valinta koska yksi pyyntö = yksi operaatio, sama elinkaari kuin DbContextilla

---

### Kysymys 6: DI-kontti

Selitä omin sanoin mitä DI-kontti tekee kun HTTP-pyyntö saapuu ja `ProductsController` tarvitsee `IProductService`:ä. Mitä tapahtuu vaihe vaiheelta?

**Vastaus:**
Pyyntö saapuu, ASP.NET luo controllerin
Kontti huomaa että controller tarvitsee IProductService
Kontti luo ProductService-instanssin (ja sen riippuvuudet)
Injektoi sen controllerille
Pyyntö käsitellään, instanssit tuhotaan


---

### Kysymys 7: Rekisteröinnin unohtaminen

Mitä tapahtuu jos unohdat rekisteröidä `IProductService`:n `Program.cs`:ssä? Milloin virhe ilmenee ja miltä se näyttää?

**Vastaus:**
Sovellus kaatuu käynnistyksessä tai ensimmäisessä pyynnössä InvalidOperationException-virheellä: "No service for type IProductService has been registered."


---

## Osa 3: Repository-kerros

### Kysymys 8: Miksi repository?

`ProductService` käytti aluksi `AppDbContext`:ia suoraan. Miksi se refaktoroitiin käyttämään `IProductRepository`:a? Anna vähintään kaksi syytä.

**Vastaus:**
Service voidaan testata mockaamalla repository ilman oikeaa tietokantaa
Kyselylogiikka on yhdessä paikassa, ei kopioitu ympäri koodia


---

### Kysymys 9: Service vs. Repository

Mikä on `IProductService`:n ja `IProductRepository`:n välinen ero? Mitä tietotyyppejä kumpikin käsittelee (DTO vai Entity)?

**IProductService:**
IProductService: käsittelee DTO:ja — ottaa ja palauttaa ProductResponse, CreateProductRequest
**IProductRepository:**
IProductRepository: käsittelee entiteettejä — ottaa ja palauttaa Product

---

### Kysymys 10: Controllerin muuttumattomuus

Kun Vaihe 7:ssä lisättiin repository-kerros, `ProductsController` ei muuttunut lainkaan. Miksi? Mitä tämä kertoo rajapintojen (interface) hyödystä?

**Vastaus:**
Controller riippuu vain IProductService-rajapinnasta. Kun sisäinen toteutus muuttui, rajapinta pysyi samana — controller ei tiedä eikä välitä mitä alla tapahtuu.


---

## Osa 4: Exception-käsittely ja lokitus

### Kysymys 11: ILogger

Mikä on `ILogger` ja miksi sitä tarvitaan? Mistä lokit näkee kehitysympäristössä?

**Vastaus:**
Kirjaa tapahtumia ja virheitä sovelluksesta. Kehitysympäristössä lokit näkyvät konsolissa/terminaalissa.


---

### Kysymys 12: Odotetut vs. odottamattomat virheet

Selitä ero "odotetun" ja "odottamattoman" virheen välillä. Anna esimerkki kummastakin ja kerro miten ne käsitellään eri tavalla servicessä.

**Odotettu virhe (esimerkki + käsittely):**
Odotettu: tuotetta ei löydy — palautetaan Result.Failure tai 404
**Odottamaton virhe (esimerkki + käsittely):**
Odottamaton: tietokantayhteys katkeaa — heitetään exception, lokitetaan, palautetaan 500


---

## Osa 5: Result Pattern

### Kysymys 13: Miksi null ja bool eivät riitä?
null kertoo vain että jotain meni pieleen, muttei mitä tai miksi. Result.Failure sisältää virheviestin joka voidaan palauttaa API-vastauksessa.

Alla on kaksi esimerkkiä. Selitä miksi ensimmäinen tapa on ongelmallinen ja miten toinen ratkaisee ongelman:

```csharp
// Tapa 1: null
ProductResponse? product = await _service.GetByIdAsync(id);
if (product == null)
    return NotFound();

// Tapa 2: Result
Result<ProductResponse> result = await _service.GetByIdAsync(id);
if (result.IsFailure)
    return NotFound(new { error = result.Error });
```

**Vastaus:**


---

### Kysymys 14: Result.Success vs. Result.Failure

Miten `Result Pattern` muutti virheiden käsittelyä servicessä? Vertaa Vaihe 8:n `throw;`-tapaa Vaihe 9:n `Result.Failure`-tapaan: mitä eroa niillä on asiakkaan (API:n kutsuja) näkökulmasta?

**Vastaus:**
throw keskeyttää suorituksen odottamattomasti — kutsujan pitää tietää catchata. Result.Failure on eksplisiittinen — kutsujan on pakko käsitellä tulos, kontrollivirta pysyy selkeänä.


---

## Osa 6: API-dokumentaatio

### Kysymys 15: IActionResult vs. ActionResult\<T\>

Miksi `ActionResult<ProductResponse>` on parempi kuin `IActionResult`? Anna vähintään kaksi syytä.

**Vastaus:**
Swagger näkee palautustyypin ja generoi oikean dokumentaation
Tyyppiturvallisuus — kääntäjä varmistaa palautusarvon


---

### Kysymys 16: ProducesResponseType

Mitä `[ProducesResponseType]`-attribuutti tekee? Miten se näkyy Swagger UI:ssa?

**Vastaus:**
Kertoo Swaggerille mitä statuskoodeja endpoint voi palauttaa. Swagger UI näyttää ne dokumentaatiossa eri vastausvaihtoehtoina.


---

### Kysymys 18: Refaktorointi

Sovelluksen toiminnallisuus pysyi täysin samana koko harjoituksen ajan — samat endpointit, samat vastaukset. Mitä refaktorointi tarkoittaa ja miksi se kannattaa, vaikka käyttäjä ei huomaa eroa?

**Vastaus:**
Refaktorointi = koodin rakenteen parantaminen ilman toiminnallisuuden muuttamista. Kannattaa koska koodi on helpompi testata, ylläpitää ja laajentaa jatkossa.


---
