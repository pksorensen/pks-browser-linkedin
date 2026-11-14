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

1. Hent filen fra **[opskriftssiden](https://agentics.dk/browser/recipes/pksorensen/pks-browser-linkedin)** — tryk *Hent opskriften*.
   (Eller direkte fra [GitHub](https://raw.githubusercontent.com/pksorensen/pks-browser-linkedin/main/recipes/linkedin-gemte.json), højreklik → Gem som.)
2. Læg den i `%USERPROFILE%\.agentics\browser-tray\recipes\`
3. Højreklik på tray-ikonet. Menuen bygges forfra hver gang den åbnes, så opskriften er der med det samme — ingen genstart.

**Behold filnavnet.** Trayen nøgler vandmærket (`linkedin-gemte.seen.json`) på filnavnet, så en omdøbning starter høsten forfra.

Første gang beder trayen om din godkendelse af trinnene. Det er med vilje: en opskrift kan nå en browser hvor du er logget ind, og godkendelsen hænger på trinnenes hash — ændrer filen sig, bliver du spurgt igen.

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

Push til `main`. Der er ingen udgivelse at vente på: hent filen igen, eller brug Kør-knappen på agentics.dk når opskriftssiden er klar.
