# pks-browser-linkedin

Browser-opskrifter til LinkedIn, kørt **lokalt** af [Agentics browser-tray](https://agentics.dk/products/pks-agent-browser)
på en profil du selv har logget ind i. Intet login og ingen cookie forlader maskinen.

**[Åbn opskrifterne på agentics.dk →](https://agentics.dk/browser/recipes/pksorensen/pks-browser-linkedin)** — siden læser dette repo
direkte og viser for hver opskrift hvad trinene gør, hvilke værter den rører, og
om den kan køre uovervåget. Alt på siden er låst til én commit, også filen du
henter, så et push imellem ikke kan bytte opskriften ud under dig.

## Opskrifter

| Fil | Hvad den gør | Profil | Status |
| --- | --- | --- | --- |
| [`recipes/linkedin-gemte.json`](recipes/linkedin-gemte.json) | Høster dine gemte opslag | `linkedin` | **UDKAST — ikke efterprøvet** |

> `linkedin-gemte.json` er et udgangspunkt, ikke en færdig opskrift. Selektoren
> `a[href*='/feed/update/']` er valgt ud fra hvordan LinkedIn plejer at linke et
> opslag, og **ingen del af den er målt på en rigtig side.** Kør den, se hvad
> `linkedin-gemte.result.json` indeholder, ret selektoren, og ret så navnet så
> det ikke længere siger UDKAST.

## Installér en opskrift

1. Gå til **[opskriftssiden](https://agentics.dk/browser/recipes/pksorensen/pks-browser-linkedin)** og tryk *Installér i trayen*. Trayen henter opskriften
   og viser en kvittering: hvilken commit den kom fra, hvilke værter den rører, om den
   handler eller kun læser, og hvad aftrykket er. Siger du ja, ligger den i mappen og
   står i menuen med det samme. **Kør nu** er det samme, plus at den kører bagefter.
2. Sker der ingenting, kender maskinen ikke `agentics-browser://` endnu — start trayen
   én gang, og prøv igen. Den melder sig som den der håndterer adressen ved hver start.

I hånden, hvis du vil se filen først:

1. Tryk *Hent opskriften* på [opskriftssiden](https://agentics.dk/browser/recipes/pksorensen/pks-browser-linkedin) — eller direkte fra
   [GitHub](https://raw.githubusercontent.com/pksorensen/pks-browser-linkedin/main/recipes/linkedin-gemte.json), højreklik → Gem som.
2. Læg den i `%USERPROFILE%\.agentics\browser-tray\recipes\`
3. Højreklik på tray-ikonet. Menuen bygges forfra hver gang den åbnes, så opskriften er der med det samme — ingen genstart.

**Behold filnavnet.** Trayen nøgler vandmærket (`linkedin-gemte.seen.json`) på filnavnet, så en omdøbning starter høsten forfra.

Trayen beder altid om din godkendelse af trinnene. Det er med vilje: en opskrift kan nå en
browser hvor du er logget ind. For en fil du selv har lagt i mappen hænger godkendelsen på
trinnenes aftryk — ændrer filen sig, bliver du spurgt igen. Kommer opskriften fra en knap på
en webside, bliver du spurgt **hver gang**, og det ja bliver aldrig en stående tilladelse til
at køre den uovervåget. Et klik på en webside er ikke det samme som et klik i din egen menu.

## Krav

```powershell
irm https://agentics.dk/install/pks-agent-browser-tray.ps1 | iex
```

Dertil en profil der heder `linkedin` i trayens `settings.json`, og at du har logget ind i den én gang via menuen (*Log ind i profil 'linkedin'…*).

## Kør fra kommandolinjen

```powershell
$tray = "$env:LOCALAPPDATA\Agentics\bin\pks-agent-browser-tray.exe"
& $tray recipes                                          # hvad ligger der, og kan det køres
& $tray run --name linkedin-gemte *> run.log; cat run.log
```

Omdirigeringen er ikke pynt: trayen er bygget som `WinExe` og har ingen konsol, så uden `*> fil` forsvinder både resultatet og fejlen.

## Hvad du skal vide om LinkedIn

- **Arkivet er gammelt.** Ved en måling 31/10-2026 var 86 % af de gemte opslag ældre end tre måneder; den brugbare skive var 69 fra seneste måned. Dagligt er rigeligt — timevis er spild.
- **Gemte opslag ligger på `/my-items/saved-posts/`,** og siden er virtualiseret: kort fjernes fra DOM'en når man scroller forbi, så `scrollCollect` opsamler pr. runde frem for til sidst.

## Rettelser

Push til `main`. Der er ingen udgivelse at vente på: hent filen igen, eller tryk **Kør nu** på opskriftssiden — den læser `main` og er låst til den commit du ser.

## Målt 15/11 mod et levende login

Begge opskrifter her er efterprøvet gennem Agent Share-broen, ikke gættet. To
ting er værd at kende, fordi de begge fejler **tavst**:

**Selektér kun på `msg-*`.** LinkedIn sender hashede klassenavne ved siden af de
semantiske (`LoFCGJDWoyvzKDjVwOwOvXAfNPYWHyPSpo`), og de skifter.

**Trådlisten har flere `<li>` end kort.** Målt: 18 `li.msg-conversation-listitem`,
men kun **10** af dem bærer `.msg-conversation-listitem__link`. De otte er
pladsholdere. En løkke der tæller `<li>` og klikker på indeks ville derfor
forskydes med op til otte og åbne de forkerte tråde — uden at fejle. Derfor
peger både opsamlingen og klikket i `linkedin-beskeder` på **linket**, ikke på
`<li>`'et.

### Hvad `linkedin-beskeder` giver, og hvad den ikke giver

Pr. besked, målt over et vindue på 20:

| Felt | Udfyldt | Noter |
| --- | --- | --- |
| `urn` | 20/20 | `data-event-urn`. **Samtale-id'et ligger inde i URN'en**, så beskeder fra flere tråde kan ligge i én liste uden grænsemarkør. |
| `tekst` | 20/20 | |
| `afsender` | 16/20 | |
| `klokkeslaet` | 16/20 | 8 tegn — ingen dato |
| `dato` | 2/20 | Overskriften ligger **inde i** den første beskeds `<li>` |
| `indgaaende` | 6/20 | `""` = udgående |

De fire manglende afsendere er ikke et hul i opskriften: det er gruppefortsættelser,
hvor LinkedIn udelader navn og klokkeslæt fordi beskeden hører til samme gruppe som
den forrige. **Udfyldningen hører i trayen**, som læser listen i rækkefølge og bærer
den sidst sete afsender og dato frem. Motoren forbliver deklarativ — et felt-udtræk
pr. element kan ikke se sin forgænger. Tomme felter er derfor `""` og ikke en
manglende nøgle: det er netop dét der skelner "ingen afsender her" fra "feltet
findes ikke i denne høst".

`occurredAt` **rekonstrueres**. Der er intet `datetime`-attribut nogen steder på
beskedrækken, så tidsstemplet er klokkeslæt plus en relativ dato-overskrift ("I dag",
"15. nov"). Minut-præcist i bedste fald. Lad være med at love mere.

Høsten tager det **synlige vindue** — både trådlisten og beskedlisten er
virtualiserede. Det er med vilje for v1: kartoteket kører to gange dagligt, og
URN-dedupe gør en gentagelse gratis.

### Kapløbet efter et klik — målt, fordi det fejler med rigtige data

LinkedIns beskedside er en enkeltsides-app. Et klik på en anden samtale skifter
**ikke** URL'en synligt for en opskrift, og den forrige samtales beskedrækker
bliver stående i DOM'en imens. Målt 15/11 ved at klikke og aflæse hvert 25. ms:

```
t=0 ms      20 rækker   (forrige tråd)
t=0 ms      20 rækker   (synkront EFTER click — intet sker)
t=1399 ms    7 rækker   (ny tråd)
```

**Næsten halvandet sekund.** En `assert` på beskedrækken bliver sand med det
samme — på det gamle indhold — og høsten tager derfor den forrige tråd igen,
med rigtige beskeder og rigtige URN'er. Intet fejler, intet ser forkert ud, og
resultatet er at den sidste tråd aldrig bliver læst.

Derfor står der et `wait` på 2500 ms efter klikket. Det er ikke dovenskab: siden
stiller intet signal til rådighed, og et ophold er det ærlige svar. Bæltet
ovenpå er gratis — hver besked-URN bærer sit eget samtale-id, så en forvekslet
tråd kan opdages **efter** høsten i stedet for at blive opdaget aldrig.

### Sammenføjningen

`hoveder` opsamles på trådruden `div.msg-thread`, ikke på profillinket. Ruden er
den eneste node der rummer **både** linket og beskedernes URN'er, så hver
hovedrække bærer et `urn` med samtale-id'et i og kan sættes sammen med sine
beskeder. Et profillink opsamlet for sig havde ingen nøgle, og i en gruppesamtale
uden profillink ville rækken udeblive og forskyde alle senere hoveder én tråd.

Prisen er at `text` på en hovedrække er hele samtalens tekst. Læs `visitkort`
(tre linjer: navn, headline, +grad) og `profil`, og se bort fra `text`.

### Den læser dine beskeder

`linkedin-beskeder` klikker hver samtale op, og LinkedIn **markerer dem som
læst** på din rigtige konto. Det er en bivirkning af den eneste vej ind, ikke en
fejl — men det er et valg, og det skal stå her frem for at overraske.
