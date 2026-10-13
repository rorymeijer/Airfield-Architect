# Modding Airfield Architect

Mods add **airlines**, **aircraft types**, **destinations**, **liveries** and **scenarios** to
the game, plus translations of their own texts. A mod is a plain folder of JSON files, just like the extra
languages in the `Languages` folder. No programming is needed.

A complete, working example lives next to this file in [`example_mod/`](example_mod/). In the
game, **Settings → Mods → Add example mod** copies it into your mods folder.

## Where mods go

```
<application support>/Airopuerto/
├── Saves/
├── Languages/
└── Mods/
    ├── northern-lights/        ← one folder per mod
    │   ├── mod.json            ← the manifest (required)
    │   ├── aircraft.json
    │   ├── airlines.json
    │   ├── destinations.json
    │   ├── liveries.json
    │   ├── scenarios.json
    │   └── translations/
    │       └── nl.json
    └── another-mod/
        └── mod.json
```

**Settings → Mods → Open mods folder** opens the right folder for your system. After adding or
changing files, press **Reload**. Folders whose name starts with `_` or `.` are skipped (handy
for backups).

To share a mod, zip its folder and put it anywhere you like (a website, a GitHub repository,
a forum post). Players unzip it into their own `Mods` folder.

## The Mods tab

Settings → Mods lists every mod in the folder with:

- a **switch** to turn it on or off (new mods are on);
- its place in the **load order** (arrows). When two mods add the same airline code or city,
  the mod higher in the list wins; liveries from mods lower in the list are painted last;
- its **errors** (red) and **warnings** (amber). An error skips only the item it is about; a
  mod whose `mod.json` cannot be read is shown as *Broken* and is not used at all.

Changes take effect immediately.

## mod.json

```json
{
  "id": "northern-lights",
  "name": "Northern Lights",
  "author": "Your name",
  "version": "1.0.0",
  "gameVersion": "1.0.0",
  "description": "Three Nordic airlines, four destinations, a livery and a scenario.",
  "content": {
    "aircraft": ["aircraft.json"],
    "airlines": ["airlines.json"],
    "destinations": ["destinations.json"],
    "liveries": ["liveries.json"],
    "scenarios": ["scenarios.json"],
    "translations": { "nl": "translations/nl.json" }
  }
}
```

| Field | Required | Meaning |
| --- | --- | --- |
| `id` | yes | Unique id: lower-case `a–z`, `0–9`, `-` and `_`. Saves refer to the mod by this id, so do not change it after release. |
| `name` | yes | Name shown in the game. |
| `version` | no (warning) | Your mod's version. |
| `gameVersion` | no | The game version the mod was made for. A newer number than the running game gives a warning. |
| `author`, `description` | no | Shown in the Mods tab. The description can be translated (see Translations). |
| `content` | no | Lists of files per content type (a single string is fine too). Paths are relative to the mod folder and use `/`. |

Each content file is either a list, or an object with the list under the type's name
(`{"airlines": [...]}`).

## Airlines

```json
{
  "code": "AN",
  "name": "Aurora Nordic",
  "type": "network",
  "fleet": ["regionalJet", "narrowbody"],
  "homeRegion": "europe",
  "livery": { "tail": "#1B998B", "body": "#F7F7F7", "accent": "#7A4DCC", "engines": "#5C6370", "pattern": "swoosh" }
}
```

- `code` (required): two or three capital letters or digits. It must not clash with the game's
  airlines (PA, TJ, NC, AB, SK, MW, CF, GF, ER, OS, NX, and MY for your own airline) or another
  enabled mod.
- `name` (required).
- `type` (required): `network`, `low-cost`, `holiday` (or `leisure`), `cargo` or `regional`.
  The type sets bags per passenger, punctuality and how much travellers shop.
- `fleet` (required): aircraft classes the airline flies: `turboprop`, `regionalJet`,
  `narrowbody`, `widebody`, `jumbo`, `narrowFreighter`, `wideFreighter`, or the `id` of an
  aircraft type of the same mod (see Aircraft types), which then flies in place of its class.
  Cargo airlines fly only freighters; other airlines fly none.
- `homeRegion` (optional): `europe`, `middleEast`, `africa`, `asia`, `oceania`, `northAmerica`,
  `caribbean`, `southAmerica`. The airline then offers routes to that region when it can.
- `livery` (optional): colours as `"#RRGGBB"`. `tail` is required inside a livery; `body`
  (white), `accent` (tail colour) and `engines` (light grey) have defaults. `pattern`:
  `cheatline` (a stripe), `belly` (two-tone), `swoosh` or `full`.

Enabled airlines join the pool of airlines that send contract offers, and their aircraft use their
livery on the map, the flight board and the network map.

## Aircraft types

A mod can add its own aircraft types. Its airlines fly them in place of one of the game's
classes: the type decides the seats, the cargo hold, the ground times and the look; the class it
falls in decides which stands it fits, the runway it needs and the landing fee.

```json
{
  "id": "b727",
  "name": "Boeing 727-200",
  "size": "medium",
  "seats": 170,
  "cargo": 2.5,
  "wingspan": 32.9,
  "length": 46.7,
  "engines": 3,
  "enginePosition": "rear",
  "speed": 850,
  "turnaround": 45,
  "boarding": 35,
  "basedOn": "b737",
  "wingtip": "plain",
  "proportions": { "body": 1.0, "engines": 1.1 },
  "colors": { "fuselage": "#F2F2F2", "engines": "#8A8F96" }
}
```

| Field | Required | Meaning |
| --- | --- | --- |
| `id` | yes | Lower-case `a–z`, `0–9`, `-`, `_`; used in the airlines' `fleet`. Not a class name. |
| `name` | yes | Shown on the flight board, in the flight details and in the Mods tab. |
| `size` | yes | `small`, `medium` or `large`: the stand size it needs. |
| `freighter` | no | `true` for a freighter (no seats; `medium` or `large`). |
| `seats` | yes (not for freighters) | small 10–130, medium 90–260, large 180–900. |
| `cargo` | no | Tonnes of cargo (belly hold, or the main deck of a freighter), 0–150. |
| `wingspan`, `length` | yes | Metres (10–90), for the silhouette on the map. A wingspan over 65 m makes a large passenger type code F (it needs a code F stand). |
| `engines` | no | 2 (default), 3 (the third in the tail) or 4. |
| `enginePosition` | no | `wing` (default) or `rear` (on the fuselage, with a T-tail). |
| `propellers` | no | `true` for a turboprop; a small type with propellers falls in the turboprop class. |
| `speed` | no | Cruise speed in km/h, 250–1000 (default 800), shown with the type in the Mods tab. |
| `turnaround` | yes | Shortest ground time in minutes (15–240). |
| `boarding` | no | Minutes before departure that boarding starts (10–90; default: the class's). |
| `basedOn` | yes | The game's aircraft whose silhouette it starts from: `atr72`, `dash8`, `e190`, `a220`, `crj900`, `a320neo`, `b737`, `b737max`, `a321neo`, `b787`, `a350`, `a330`, `b777`, `b737f`, `b757f`, `b767f`, `b777f`, `a380`, `b7478` (or a class name). |
| `wingtip` | no | `plain`, `winglet`, `scimitar` or `raked` (default: as `basedOn`). |
| `proportions` | no | `body` (fuselage width) and `engines` (engine size), each 0.6–1.6. |
| `colors` | no | `fuselage` and `engines` as `"#RRGGBB"`, instead of the airline's livery. |

An airline flies at most one type per class; when two of its fleet entries fall in the same class
the first one flies those flights (with a warning). When the mod is missing later, the flights
of its airlines fall back to the plain class.

## Destinations

```json
{
  "city": "Reykjavik",
  "country": "Iceland",
  "iata": "KEF",
  "lat": 63.99,
  "lon": -22.62,
  "region": "europe",
  "schengen": true,
  "demand": { "passengersPerDay": 150, "profile": "leisure", "cargo": false }
}
```

- `city`, `country`, `iata` (three capital letters), `lat` and `lon` are required. A city or
  IATA code that the game or an earlier mod already has is an error.
- `region` (optional; worked out from the coordinates when left out).
- `schengen` (default `false`): flights to non-Schengen destinations need border control.
- `longHaul` (optional): by default destinations more than 3,800 km away are long-haul, served
  by widebodies.
- `demand` (optional): `passengersPerDay` (0–2,000; default 110) is the market size for your own
  airline's routes; `profile` is `mixed`, `business` or `leisure` (leisure destinations also
  get holiday charters); `cargo: true` adds the city to freighter routes.

## Liveries

New colours for an airline of the game or of any enabled mod, by airline code:

```json
{ "airline": "TJ", "tail": "#E4572E", "body": "#FFFFFF", "accent": "#E4572E", "engines": "#E4572E", "pattern": "swoosh" }
```

An unknown airline code is an error.

## Scenarios

Scenarios use the same structure as the game's own: a story, a goal, a deadline, starting
conditions and two bonus stars.

```json
{
  "id": "northern-lights-season",
  "title": "Northern Lights Season",
  "briefing": "Aurora Nordic opens winter routes to Tromsø and Reykjavik. …",
  "goal": "6,000 passengers in 14 days, 80% on time",
  "days": 14,
  "start": {
    "cash": 600000,
    "reputation": 60,
    "demandBoost": 1.1,
    "openCargo": false,
    "contracts": [
      {
        "airline": "AN",
        "fleet": "narrowbody",
        "weeks": 4,
        "slots": [
          { "arrival": "08:10", "ground": 50, "destination": "TOS" },
          { "arrival": "13:40", "ground": 50, "destination": "Reykjavik" }
        ]
      }
    ]
  },
  "goals": { "passengers": 6000, "onTime": 0.8 },
  "bonus": [{ "within": 10 }, { "cashKept": true }]
}
```

- `id`, `title`, `briefing` and `days` (1–3,650) are required. `goal` is the one-line goal;
  left out, the game writes one from `goals`.
- `start` (all optional): `cash`, `reputation` (0–100), `demandBoost` (0.3–3; 1.35 is the
  Summer Peak rush), `openCargo` (opens a cargo stand with loaders and handlers) and
  `contracts` already signed. A contract names an airline code, an aircraft class (`fleet`),
  `weeks` and `slots` with an `arrival` time (`"HH:MM"`), a `ground` time in minutes and a
  `destination` (city or IATA code, from the game or any enabled mod).
- `goals` (at least one of): `passengers` (handled since the start), `cargo` (tonnes),
  `reputation`, `cash`, `passengerStands` (open), `bestDayPassengers` (a single day). `onTime`
  (0–1) is checked together with the others. All goals must be met before the deadline.
- `bonus`: up to two of `{"within": days}`, `{"cashKept": true}`, `{"neverRed": true}`,
  `{"cash": amount}`. Missing ones are filled with "finish within 70% of the time" and "end
  with more cash than you started with".

Enabled scenarios appear at the bottom of the scenario list in **New game**.

## Translations

The mod's texts (scenario title, briefing and goal, the description) are written in English and
can be translated per language. A translation file maps the English text to the translation:

```json
{
  "strings": {
    "Northern Lights Season": "Noorderlichtseizoen"
  }
}
```

List the files under `content.translations` by language code (`nl`, `de`, `fr`, `es`, `it`,
`pt`, `pl`, `zh`, `ru`, `ja`, `ko` or the code of a language from the `Languages` folder).
Airline and city names are not translated.

## Saves and missing mods

A saved game records which mods it used, with the names and codes of their airlines and the
coordinates of their cities. When such a game is loaded without one of those mods, the game
shows a warning and loads anyway: flights and contracts of the missing airlines keep flying under
their saved names, and their cities stay on the map. A modded scenario whose mod is missing keeps
running until its deadline but cannot be won.

## Determinism

Mods are read before a game is created or loaded, in load order, and every mod's files in the
order its `mod.json` lists them. The result never depends on the order the file system lists
folders in, so the same mods in the same order always give the same game. Airline ids are
derived from the mod id and the airline code, so they stay the same between sessions.

## Common messages

| Message | Cause |
| --- | --- |
| No mod.json in this folder | The folder has no manifest. |
| Invalid JSON near line N | A comma, quote or bracket is missing or extra. |
| Missing field "x" | A required field is missing or empty. |
| "v" is not a valid value for "x" | Wrong type, unknown name or out of range. |
| Duplicate airline code / destination | The code or city already exists in the game or an earlier mod. |
| Unknown airline / destination | A livery or scenario refers to something no enabled mod provides. |
| Unknown aircraft type | A `fleet` entry is not one of the classes above or an aircraft type of the mod, or `basedOn` names no aircraft of the game. |
| Duplicate aircraft type | Two aircraft types of the mod have the same `id` (or an `id` is a class name). |
| N seats do not fit a … aircraft | The seats are outside the range of the size. |
| Made for game version x | The mod needs a newer game; it may still work. |

---

## Samenvatting (Nederlands)

Mods voegen **maatschappijen**, **bestemmingen**, **kleurstellingen** en **scenario's** toe, als
JSON-bestanden in een eigen map per mod in `Airopuerto/Mods` (naast `Saves` en `Languages`).
Elke mod heeft een `mod.json` met `id`, `name`, `author`, `version`, `gameVersion`,
`description` en de lijst met inhoudsbestanden. Het voorbeeld in [`example_mod/`](example_mod/)
laat alles zien; in het spel zet **Instellingen → Mods → Voorbeeldmod toevoegen** het in je
modsmap.

- **Maatschappijen**: code (2–3 tekens), naam, type (`network`, `low-cost`, `holiday`, `cargo`,
  `regional`), vloot, thuisregio en kleuren.
- **Toesteltypes**: naam, formaat (`small`/`medium`/`large`), stoelen, vracht, spanwijdte en
  lengte (meters), motoren (aantal en positie), snelheid, omdraai- en instaptijd, en het toestel
  uit het spel waarop het silhouet is gebaseerd (`basedOn`) met eigen verhoudingen en kleuren.
  De maatschappijen van de mod vliegen het in plaats van een klasse; zonder de mod vallen die
  vluchten terug op de klasse.
- **Bestemmingen**: stad, land, IATA-code, coördinaten, regio, Schengen en vraag (passagiers per
  dag, zakelijk/vakantie, vracht).
- **Kleurstellingen**: nieuwe kleuren voor een bestaande of gemodde maatschappij.
- **Scenario's**: verhaal, doel, aantal dagen, startsituatie (kas, reputatie, contracten) en twee
  bonussterren, net als de scenario's van het spel.
- **Vertalingen**: de teksten van de mod per taal.

In **Instellingen → Mods** zet je mods aan of uit, kies je de laadvolgorde (hoger wint bij
dubbele codes) en zie je per mod duidelijke fouten en waarschuwingen. Opgeslagen spellen
onthouden welke mods ze gebruikten; ontbreekt er een, dan krijg je een waarschuwing en laadt het
spel toch, met de opgeslagen namen van de maatschappijen. Een mod delen gaat door de map (als zip)
te delen, bijvoorbeeld via GitHub of een website.
