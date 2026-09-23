# OLF – Open Lesdoelen Formaat

**Versie 1.4** (1 september 2026)

Het Open Lesdoelen Formaat (OLF) is een open, XML-gebaseerde standaard om lesdoelen uit te wisselen tussen aanbieders van lesdoelen (educatieve uitgeverijen en andere contentleveranciers) en leerplatformen (LMS'en, planningstools, rapportsystemen).

Een OLF-bericht beschrijft per methode welke lesdoelen ze bevat, hoe die lesdoelen gestructureerd zijn (hoofdstuk, les, thema …) en aan welke **leerplandoelen** van de Vlaamse onderwijskoepels en welke **minimumdoelen/eindtermen** ze gekoppeld zijn.

OLF werd in 2014 ontwikkeld op initiatief van de Vlaamse educatieve uitgeverijen (versies 1.0 tot 1.2). Versie 1.3 breidde het schema uit met uuid's en leerplanversies. Sindsdien zijn er nieuwe leerplannen en doelenpickers bijgekomen (Zill, Op.Stap, LLinkid, GO! navigator, LeerLokaal, minimumdoelen …). Versie 1.4 brengt het formaat in lijn met dat landschap en bouwt verder op 1.2 en 1.3.

## Inhoud

- [Bestanden](#bestanden)
- [Snel aan de slag](#snel-aan-de-slag)
- [Validatie](#validatie)
- [Structuur van een bericht](#structuur-van-een-bericht)
- [Elementen](#elementen)
- [Codelijsten](#codelijsten)
- [Leerplandoelen per doelenpicker](#leerplandoelen-per-doelenpicker)
- [Minimumdoelen en eindtermen](#minimumdoelen-en-eindtermen)
- [Regels buiten het schema](#regels-buiten-het-schema)
- [Compatibiliteit en wijzigingen](#compatibiliteit-en-wijzigingen)
- [Openstaande punten](#openstaande-punten)
- [Bijdragen](#bijdragen)

## Bestanden

| Bestand | Omschrijving |
|---|---|
| `schemas/olf_1_4.xsd` | XML Schema (XSD 1.0) voor OLF 1.4 |
| `schemas/olf_1_3.xsd` | XML Schema voor OLF 1.3 (ter referentie) |
| `examples/voorbeeld-1.4.xml` | Volledig voorbeeldbericht met alle doelenpickers |
| `README.md` | Deze documentatie |

## Snel aan de slag

Een minimaal OLF-bericht:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<OLFMessage version="1.4">
  <Header>
    <Sender>Uitgeverij X</Sender>
    <SentDateTime>2026-09-23T16:51</SentDateTime>
  </Header>
  <Publishers>
    <Publisher>
      <PublisherIdentifier>0000000398731938</PublisherIdentifier>
      <PublisherName>Uitgeverij X</PublisherName>
      <PublisherEmail>support@uitgeverijx.be</PublisherEmail>
      <Methods>
        <Method>
          <MethodIdentifier>UX-FR-5</MethodIdentifier>
          <MethodName><![CDATA[Frans & Co 5]]></MethodName>
          <LearningObjectives>
            <LearningObjective>
              <LearningObjectiveIdentifier>UX-FR-5-001</LearningObjectiveIdentifier>
              <LearningObjectiveDescription><![CDATA[De leerlingen tonen bereidheid om te luisteren in het Frans.]]></LearningObjectiveDescription>
              <LearningObjectiveCodes>
                <LearningObjectiveCode type="GO">5.25*</LearningObjectiveCode>
                <LearningObjectiveCode type="VVKBAO" codetype="uuid" version="0bb40237-487c-4c9c-9608-1e30debadb9f">0b71fcdc-53f5-476e-8528-3e7f0a839a86</LearningObjectiveCode>
              </LearningObjectiveCodes>
              <Qualifications>
                <Qualification codetype="code">WT 1.11</Qualification>
              </Qualifications>
            </LearningObjective>
          </LearningObjectives>
        </Method>
      </Methods>
    </Publisher>
  </Publishers>
</OLFMessage>
```

Een uitgebreider voorbeeld staat in [`examples/voorbeeld-1.4.xml`](examples/voorbeeld-1.4.xml).


## Structuur van een bericht

```
OLFMessage @version
├── Header
│   ├── Sender
│   └── SentDateTime
└── Publishers
    └── Publisher (1..n)
        ├── PublisherIdentifier
        ├── PublisherName
        ├── PublisherEmail
        └── Methods
            └── Method (1..n)
                ├── MethodIdentifier
                ├── MethodName                 (optioneel)
                ├── TargetAudiences            (optioneel)
                │   └── TargetAudience (0..n)
                └── LearningObjectives
                    └── LearningObjective (1..n)
                        ├── LearningObjectiveIdentifier   (verplicht sinds 1.4, uniek per Method)
                        ├── LearningObjectiveDescription  (optioneel)
                        ├── LearningObjectiveCategories   (optioneel)
                        │   └── LearningObjectiveCategory @type (0..n)
                        ├── LearningObjectiveCodes        (optioneel)
                        │   └── LearningObjectiveCode @type @codetype @version (0..n)
                        ├── LearningObjectiveUri          (optioneel, sinds 1.3)
                        └── Qualifications                (optioneel)
                            └── Qualification @codetype (0..n)
```

De volgorde van de kindelementen binnen `Header`, `Publisher`, `Method` en `LearningObjective` is vrij. Binnen `OLFMessage` komt `Header` altijd vóór `Publishers`.

## Elementen

### OLFMessage

Root element van het bericht.

| Attribuut | Verplicht | Waarden |
|---|---|---|
| `version` | ja | `1`, `1.0`, `1.1`, `1.2`, `1.3`, `1.4` |

Gebruik voor nieuwe berichten `version="1.4"`. Het attribuut `release` uit de voorbeelden van de oude specificatie is niet (meer) geldig.

### Header

Metadata over het bericht zelf.

| Element | Verplicht | Type | Omschrijving |
|---|---|---|---|
| `Sender` | ja | tekst | Tekstuele aanduiding van de afzender |
| `SentDateTime` | ja | `JJJJ-MM-DDTHH:MM` | Tijdstip van verzending, lokale Belgische tijd, bv. `2026-09-23T16:51` |

### Publisher

Gegevens over één aanbieder. Een bericht kan meerdere aanbieders bevatten, al zal dat in de praktijk zelden voorkomen.

| Element | Verplicht | Type | Omschrijving |
|---|---|---|---|
| `PublisherIdentifier` | ja | tekst | Unieke identifier van de aanbieder. Gebruik bij voorkeur het [ISNI](https://isni.org/)-nummer (16 tekens). |
| `PublisherName` | ja | tekst | Naam van de aanbieder |
| `PublisherEmail` | ja | e-mailadres | Contactadres voor afnemers bij vragen of problemen |
| `Methods` | ja | – | Groepeert één of meer `Method`-elementen |

Bekende ISNI-nummers:

| Uitgeverij | ISNI |
|---|---|
| Averbode | `0000000398729934` |
| Die Keure | `0000000398724324` |
| Pelckmans | `0000000398727787` |
| Plantyn | `0000000398727947` |
| Van In | `0000000398731938` |
| Zwijsen | `000000039873168X` |

### Method

Gegevens over één methode (lesmethode, leerpakket, cursus …).

| Element | Verplicht | Type | Omschrijving |
|---|---|---|---|
| `MethodIdentifier` | ja | tekst | Aanbiederspecifieke identifier van de methode |
| `MethodName` | nee | tekst | Naam van de methode. Gebruik een CDATA-sectie als de naam speciale tekens bevat. Sterk aanbevolen. |
| `TargetAudiences` | nee | – | Groepeert `TargetAudience`-elementen |
| `TargetAudience` | nee (0..n) | code | Doelgroep, zie [codelijst 1](#codelijst-1-indeling-onderwijs) |
| `LearningObjectives` | ja | – | Groepeert één of meer `LearningObjective`-elementen |

### LearningObjective

Gegevens over één lesdoel.

| Element | Verplicht | Type | Omschrijving |
|---|---|---|---|
| `LearningObjectiveIdentifier` | ja | tekst | *Verplicht sinds 1.4.* Aanbiederspecifieke identifier van het lesdoel, uniek binnen de `Method`. Hou de identifier stabiel over berichten heen: afnemers gebruiken hem om een lesdoel bij te werken in plaats van het opnieuw aan te maken. |
| `LearningObjectiveDescription` | nee | tekst | Omschrijving van het lesdoel, bij voorkeur in een CDATA-sectie |
| `LearningObjectiveCategories` | nee | – | Groepeert `LearningObjectiveCategory`-elementen |
| `LearningObjectiveCodes` | nee | – | Groepeert `LearningObjectiveCode`-elementen |
| `LearningObjectiveUri` | nee | URI | *Sinds 1.3.* Permanente link naar het lesdoel bij de aanbieder |
| `Qualifications` | nee | – | Groepeert `Qualification`-elementen |

### LearningObjectiveCategory

Plaatst het lesdoel in de structuur van de methode. Omdat die structuur sterk verschilt per aanbieder, is ze niet vast in het formaat gegoten: elk niveau wordt als een generieke categorie meegegeven, getypeerd met [codelijst 2](#codelijst-2-categorisering-lesdoelen). De afnemer bepaalt zelf hoe de categorieën getoond worden.

```xml
<LearningObjectiveCategory type="CHAPTER">Hoofdstuk 1</LearningObjectiveCategory>
```

| Attribuut | Verplicht | Waarden |
|---|---|---|
| `type` | ja | codelijst 2 |

### LearningObjectiveCode

Koppelt het lesdoel aan een leerplandoel. Hoe je dit element invult, hangt af van de doelenpicker; zie [Leerplandoelen per doelenpicker](#leerplandoelen-per-doelenpicker).

| Attribuut | Verplicht | Waarden | Omschrijving |
|---|---|---|---|
| `type` | ja | [codelijst 3](#codelijst-3-leerplannen-en-doelenpickers) | Het leerplan of de doelenpicker waartoe het doel behoort |
| `codetype` | nee | [codelijst 4](#codelijst-4-codetype), standaard `full` | Wat de waarde voorstelt: een code of een uuid |
| `version` | nee | tekst | *Sinds 1.3.* Versie van het leerplan, bv. de curriculum-id bij Zill of de leerplanversie bij LeerLokaal |

### Qualification

Koppelt het lesdoel aan een minimumdoel of eindterm; zie [Minimumdoelen en eindtermen](#minimumdoelen-en-eindtermen).

| Attribuut | Verplicht | Waarden | Omschrijving |
|---|---|---|---|
| `codetype` | nee | `code`, `uuid` | *Nieuw in 1.4.* Geeft aan of de waarde de code of de uuid van het doel is |

## Codelijsten

### Codelijst 1: indeling onderwijs

| Code | Onderwijsniveau |
|---|---|
| **Kleuteronderwijs** | |
| `KO-J0` | Kleuter onthaalklas |
| `KO-J1` | Kleuter 1ste jaar |
| `KO-J2` | Kleuter 2de jaar |
| `KO-J3` | Kleuter 3de jaar |
| **Lager onderwijs** | |
| `LO-J1` … `LO-J6` | 1ste … 6de leerjaar |
| **Secundair onderwijs – jaar** | |
| `SO-J1` … `SO-J6` | 1ste … 6de jaar |
| **Secundair onderwijs – graad** | |
| `SO-G1` | 1ste graad |
| `SO-G2` | 2de graad |
| `SO-G3` | 3de graad |
| **Secundair onderwijs – doelgroep** | |
| `SO-ASTR` | A-stroom |
| `SO-BSTR` | B-stroom |
| `SO-ASO` | ASO |
| `SO-BSO` | BSO |
| `SO-TSO` | TSO |
| `SO-KSO` | KSO |

### Codelijst 2: categorisering lesdoelen

| Code | Categorie |
|---|---|
| `CHAPTER` | Hoofdstuk |
| `LEARNINGAREA` | Leergebied |
| `LEARNINGDOMAIN` | Leerdomein |
| `LESSONNUMBER` | Lesnummer |
| `LESSON` | Les |
| `THEME` | Thema |
| `BLOCK` | Blok |
| `OBJECTIVEPRIORITY` | Hoofd- of nevendoel |

### Codelijst 3: leerplannen en doelenpickers

| Code | Koepel | Leerplan / doelenpicker | Sinds |
|---|---|---|---|
| `GO` | GO! | Leerplandoelen GO! (legacy, o.a. basisonderwijs) | 1.0 |
| `GO-NAV-BAO` | GO! | GO! navigator – basisonderwijs | 1.4 |
| `GO-NAV-SO` | GO! | GO! navigator – secundair onderwijs | 1.4 |
| `VVKBAO` | Katholiek Onderwijs Vlaanderen | Basisonderwijs: legacy leerplancodes en Zill | 1.0 |
| `OPSTAP` | Katholiek Onderwijs Vlaanderen | Op.Stap (basisonderwijs) | 1.4 |
| `VVKSO` | Katholiek Onderwijs Vlaanderen | Legacy leerplancodes secundair onderwijs | 1.0 |
| `LLINKID` | Katholiek Onderwijs Vlaanderen | LLinkid (secundair onderwijs) | 1.4 |
| `VSKO` | Katholiek Onderwijs Vlaanderen | Legacy – niet meer gebruiken voor nieuwe berichten | 1.0 |
| `OVSG` | OVSG | Leerplandoelen OVSG (legacy, basisonderwijs) | 1.0 |
| `LEERLOKAAL-BAO` | OVSG | LeerLokaal – basisonderwijs | 1.4 |
| `LEERLOKAAL-SO` | OVSG | LeerLokaal – secundair onderwijs (1ste graad) | 1.4 |
| `POV` | POV | Leerplannen POV (basis- en secundair onderwijs) | 1.0 |
| `ROOMSKATH-BAO` | – | Rooms-katholieke godsdienst – basisonderwijs | 1.4 |
| `ROOMSKATH-SO` | – | Rooms-katholieke godsdienst – secundair onderwijs | 1.4 |

### Codelijst 4: codetype

| Code | Betekenis | Sinds |
|---|---|---|
| `full` | Volledige code of prefix van het doel (standaard als het attribuut ontbreekt) | 1.2 |
| `short` | Korte code van het doel, bv. `2008/038` i.p.v. `D/2008/7841/038`, of `1.1.PF1.4` bij Op.Stap | 1.2 |
| `uuid` | Unieke id van het doel zoals gepubliceerd door de koepel of doelenpicker. Meestal een standaard uuid, bij de GO! navigator een eigen formaat. | 1.3 |

De waarde `smartschool_uuid` uit OLF 1.3 is in 1.4 geschrapt.

## Leerplandoelen per doelenpicker

Elke koepel publiceert zijn leerplandoelen anders. Sommige doelen hebben een leesbare code (prefix), andere enkel een uuid. De tabel geeft per doelenpicker aan wat er verwacht wordt.

| Doelenpicker | `type` | `codetype` | Waarde | `version` |
|---|---|---|---|---|
| GO! legacy BaO | `GO` | – (`full`) | prefix van het doel | – |
| GO! navigator BaO | `GO-NAV-BAO` | `short` of `uuid` | korte code of id van het doel (eigen formaat) | – |
| GO! navigator SO | `GO-NAV-SO` | `short` of `uuid` | korte code of id van het doel (eigen formaat) | – |
| Zill | `VVKBAO` | `uuid` | id van het Zill-doel | **ja**: id van de leerplanversie (curriculum) |
| Op.Stap | `OPSTAP` | `short` of `uuid` | korte code of id van het doel | – |
| LLinkid | `LLINKID` | `uuid` | id van het doel | – |
| OVSG legacy BaO | `OVSG` | – (`full`) | prefix van het doel | – |
| LeerLokaal | `LEERLOKAAL-BAO` / `LEERLOKAAL-SO` | `short` of `uuid` | korte code of id van het doel | aanbevolen: leerplanversie |
| POV | `POV` | `uuid` | id van het doel | – |
| RK godsdienst | `ROOMSKATH-BAO` / `ROOMSKATH-SO` | `uuid` (aanbevolen) | id van het doel; prefix als aanvulling indien beschikbaar | – |

Als van een doel zowel een prefix als een uuid bekend is, geef je beide mee als afzonderlijke `LearningObjectiveCode`-elementen. De afnemer koppelt op de combinatie van `type`, `codetype` en waarde.

### GO!

**Legacy basisonderwijs** – de prefix van het doel wordt doorgegeven:

```xml
<LearningObjectiveCode type="GO">5.25*</LearningObjectiveCode>
```

**GO! navigator** (basis- en secundair onderwijs) – de korte code of de id van het doel.

De id van de GO! navigator is **geen standaard uuid** maar een eigen samengesteld formaat, bv. `1071_3e9bbf13-78db-4980-84a2-4a3cddf73331_navigator-bao`. Geef de id integraal door zoals de GO! navigator hem aanlevert; knip er geen uuid uit.

```xml
<LearningObjectiveCode type="GO-NAV-BAO" codetype="short">NL.001</LearningObjectiveCode>
<LearningObjectiveCode type="GO-NAV-BAO" codetype="uuid">1071_3e9bbf13-78db-4980-84a2-4a3cddf73331_navigator-bao</LearningObjectiveCode>

<LearningObjectiveCode type="GO-NAV-SO" codetype="short">BV1_01.01</LearningObjectiveCode>
<LearningObjectiveCode type="GO-NAV-SO" codetype="uuid">1071_3e9bbf13-78db-4980-84a2-4a3cddf73331_navigator-so</LearningObjectiveCode>
```

Het achtervoegsel is `_navigator-bao` voor het basisonderwijs en `_navigator-so` voor het secundair onderwijs.

### Katholiek Onderwijs Vlaanderen

**Zill** – de id van het doel én de id van de versie van het Zill-leerplan (`version`). Zonder versie kan de afnemer het doel niet eenduidig terugvinden.

```xml
<LearningObjectiveCode type="VVKBAO" codetype="uuid" version="0bb40237-487c-4c9c-9608-1e30debadb9f">0b71fcdc-53f5-476e-8528-3e7f0a839a86</LearningObjectiveCode>
```

`VVKBAO` zonder `codetype="uuid"` blijft een legacy leerplancode (bv. `DO.01.a`).

**Op.Stap** – de korte code of de id van het doel. Op.Stap werkt niet met een versie-id.

```xml
<LearningObjectiveCode type="OPSTAP" codetype="short">1.1.PF1.4</LearningObjectiveCode>
<LearningObjectiveCode type="OPSTAP" codetype="uuid">1408e88d-4356-494d-a930-9d83e5799329</LearningObjectiveCode>
```

**LLinkid** (enkel secundair onderwijs) – de id van het doel:

```xml
<LearningObjectiveCode type="LLINKID" codetype="uuid">72e4456f-27e2-42ca-bce0-63abec7c3737</LearningObjectiveCode>
```

**Legacy leerplannen secundair onderwijs** – korte of volledige leerplancode:

```xml
<LearningObjectiveCode type="VVKSO" codetype="short">2008/038</LearningObjectiveCode>
<LearningObjectiveCode type="VVKSO">D/2008/7841/038</LearningObjectiveCode>
```

### OVSG

**Legacy basisonderwijs** – de prefix van het doel:

```xml
<LearningObjectiveCode type="OVSG">DL-FR-LUI-03.01</LearningObjectiveCode>
```

**LeerLokaal** (basisonderwijs en 1ste graad secundair onderwijs) – de korte code of de id van het doel:

```xml
<LearningObjectiveCode type="LEERLOKAAL-SO" codetype="short">DIG 1 B.25</LearningObjectiveCode>
<LearningObjectiveCode type="LEERLOKAAL-BAO" codetype="uuid">42d38231-f900-4494-981e-da050bfd7fea</LearningObjectiveCode>
```

OVSG brengt binnenkort een nieuwe versie van LeerLokaal uit. Omdat codes tussen versies kunnen verschillen, geef je de leerplanversie mee in het attribuut `version` zodra die beschikbaar is. De waarde hieronder is illustratief; gebruik de versie-aanduiding zoals OVSG die publiceert.

```xml
<LearningObjectiveCode type="LEERLOKAAL-SO" codetype="short" version="2">DIG 1 B.25</LearningObjectiveCode>
```

### POV

Eén doelenpicker voor basis- en secundair onderwijs; de id van het doel:

```xml
<LearningObjectiveCode type="POV" codetype="uuid">00da8561-98ac-4d2a-94b1-c6ff554bc053</LearningObjectiveCode>
```

### Rooms-katholieke godsdienst

De doelen voor rooms-katholieke godsdienst worden los van de koepels aangeboden, apart voor basis- en secundair onderwijs. Niet elk doel heeft een prefix, dus de id is de betrouwbare sleutel. Een prefix mag als aanvulling meegegeven worden.

```xml
<LearningObjectiveCode type="ROOMSKATH-BAO" codetype="uuid">3d003006-642e-4a26-8f54-cfd7d1c152f3</LearningObjectiveCode>
<LearningObjectiveCode type="ROOMSKATH-BAO">DAA.1.1.1</LearningObjectiveCode>

<LearningObjectiveCode type="ROOMSKATH-SO" codetype="uuid">76cd6d85-1a86-4789-a64b-46df9c1b3d4c</LearningObjectiveCode>
<LearningObjectiveCode type="ROOMSKATH-SO">PLU1</LearningObjectiveCode>
```

## Minimumdoelen en eindtermen

Minimumdoelen en eindtermen gelden voor het hele Vlaamse onderwijs en verschillen niet per koepel. Ze worden meegegeven als `Qualification`.

Een minimumdoel kan je meegeven via zijn naam/code, via zijn uuid, of via beide. Geef met het attribuut `codetype` aan welke vorm je gebruikt, zodat de afnemer het onderscheid niet zelf moet afleiden.

Enkel de code:

```xml
<Qualifications>
  <Qualification codetype="code">WT 1.11</Qualification>
  <Qualification codetype="code">WT 1.12</Qualification>
</Qualifications>
```

Enkel de uuid:

```xml
<Qualifications>
  <Qualification codetype="uuid">d6650093-c069-4530-bb5d-1737e29794f5</Qualification>
  <Qualification codetype="uuid">8775bbc9-5264-4d20-aa4e-90f83ad1c050</Qualification>
</Qualifications>
```

Code en uuid samen mag ook; elke vorm is dan een afzonderlijk `Qualification`-element.

Berichten zonder `codetype` blijven geldig. Afnemers die zulke berichten ontvangen, herkennen een uuid aan het formaat `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

## Regels buiten het schema

Deze regels kunnen niet in XSD 1.0 afgedwongen worden. Aanbieders volgen ze; afnemers mogen berichten die ze schenden weigeren of loggen.

1. Bij `codetype="uuid"` is de waarde een uuid in kleine letters met koppeltekens: `^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$`. **Uitzondering:** bij `GO-NAV-BAO` en `GO-NAV-SO` volgt de id het eigen formaat van de GO! navigator; afnemers mogen daar geen standaard uuid verwachten.
2. Voor `LLINKID` en `POV` is `codetype="uuid"` verplicht; deze doelenpickers kennen geen betrouwbare prefix.
3. `VVKBAO` met `codetype="uuid"` (Zill) vereist het attribuut `version`.
4. Het attribuut `version` wordt enkel gebruikt waar de doelenpicker met leerplanversies werkt: Zill (verplicht bij uuid) en LeerLokaal (aanbevolen). Op.Stap gebruikt geen versie.
5. `Qualification codetype="uuid"` bevat een uuid volgens regel 1.
De verplichting en uniciteit van `LearningObjectiveIdentifier` binnen een `Method` worden sinds 1.4 wel door het schema zelf afgedwongen.

## Compatibiliteit en wijzigingen

### Achterwaartse compatibiliteit

OLF 1.4 is een uitbreiding van 1.2 en 1.3. Een bericht dat geldig was onder 1.2 of 1.3 blijft grotendeels geldig onder het 1.4-schema, met deze kanttekeningen:

- **`LearningObjectiveIdentifier` is verplicht en uniek binnen een `Method`.** Dit is de belangrijkste wijziging: berichten met lesdoelen zonder identifier, of met dubbele identifiers binnen dezelfde methode, zijn ongeldig onder 1.4.
- **`codetype="smartschool_uuid"` (1.3) is geschrapt.** Berichten die deze waarde gebruiken, zijn ongeldig onder 1.4. Gebruik `uuid` met de id van het doel zoals de koepel of doelenpicker die publiceert.
- Het root-attribuut heet `version`. De voorbeelden in de oude specificatie gebruikten `release`; dat is ongeldig.
- Lege codes en identifiers (bv. `<LearningObjectiveCode type="GO"></LearningObjectiveCode>`) worden geweigerd.
- `SentDateTime` moet een bestaande maand, dag, uur en minuut bevatten.
- `PublisherEmail` moet een domein met minstens één punt hebben.

Afnemers moeten er rekening mee houden dat nieuwe `type`-waarden kunnen binnenkomen, en onbekende waarden bij voorkeur negeren in plaats van het hele bericht te weigeren.

### Historiek

| Versie | Datum | Omschrijving |
|---|---|---|
| 1.0 | 3/7/2014 | Initiële draft (Frank Salliau) |
| 1.1 | 13/8/2014 | CDATA bij `MethodName` en `LearningObjectiveDescription`; attribuut `release` vervangen door `version`; leesbare codes voor `type`-attributen; onthaalklas toegevoegd |
| 1.2 | 10/9/2014 | `PublisherIdentifier` op basis van ISNI; attribuut `codetype` (short/full). Laatste versie met een volledige specificatie. |
| 1.3 | onbekend | Enkel schema-uitbreiding: `codetype` uitgebreid met `uuid` en `smartschool_uuid`; attribuut `version` op `LearningObjectiveCode`; element `LearningObjectiveUri` |
| 1.4 | 1/9/2026 | Nieuwe doelenpickers: `GO-NAV-BAO`, `GO-NAV-SO`, `OPSTAP`, `LLINKID`, `LEERLOKAAL-BAO`, `LEERLOKAAL-SO`, `ROOMSKATH-BAO`, `ROOMSKATH-SO`; `LearningObjectiveIdentifier` verplicht en uniek per `Method`; `codetype="smartschool_uuid"` geschrapt; attribuut `codetype` (code/uuid) op `Qualification` voor minimumdoelen; `LearningObjectiveUri` als URI getypeerd; lege codes en identifiers niet meer toegelaten; schema herschreven met benoemde types en documentatie |

## Openstaande punten

- **LeerLokaal**: het is nog niet duidelijk of de doelenpickers voor BaO en SO structureel verschillen. Het formaat van de versie-aanduiding voor de nieuwe LeerLokaal-versie ligt nog niet vast.
- **Codelijst 1** weerspiegelt de onderwijsstructuur van 2014. Een uitbreiding voor de modernisering van het secundair onderwijs (finaliteiten, 7de jaar) en voor het buitengewoon onderwijs wordt voorbereid voor een volgende versie.

## Bijdragen

Vragen, fouten en voorstellen zijn welkom via de issues van deze repository. Beschrijf bij een voorstel voor een nieuwe doelenpicker: de koepel, het onderwijsniveau, een voorbeeld van de data die de picker teruggeeft, en welke waarde (prefix of uuid) betrouwbaar is.
