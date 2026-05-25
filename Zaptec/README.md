# Zaptec laadaansturing voor Home Assistant

Automatiseert het laden van een elektrisch voertuig via een Zaptec lader, met keuze tussen **zonneladen (1-fase)** en **geforceerd laden (3-fase)**. Bij zonneladen wordt de laadstroom continu aangepast aan het beschikbare zonne-overschot.

---

## Functies

- Notificatie op je telefoon zodra de auto wordt aangesloten, met directe keuze tussen zonneladen of geforceerd laden
- Zonneladen op 1-fase: laadstroom schaalt automatisch mee met het zonne-overschot (6A minimum, 16A maximum)
- Geforceerd laden op 3-fase met 16A voor maximale snelheid
- Automatische reset naar zonneladen bij loskoppelen
- Na volledig laden: schakelaar naar 3-fase 16A voor snelle bijlading van klimaatbeheersing (voorverwarmen/koelen)

---

## Vereisten

- Home Assistant met [HACS](https://hacs.xyz/)
- [Zaptec integratie via HACS](https://github.com/custom-components/zaptec) (`custom-components/zaptec`)
- Zaptec lader met faseschakeling (Zaptec Go of vergelijkbaar)
- Omvormer met vermogenssensor in Home Assistant (bijv. SolarEdge, Fronius, Enphase)
- Home Assistant Companion app op Android of iOS
- Template sensor voor het geschatte zonne-overschot (zie hieronder)

---

## Installatie

### 1. Zonne-overschot sensor aanmaken

Voeg onderstaande template sensor toe aan je configuratie. Pas de entiteitsnamen aan naar jouw situatie.

```yaml
template:
  - sensor:
      - name: Geschat zonne overschot
        unit_of_measurement: W
        state: >
          {{ [states('sensor.YOUR_SOLAR_POWER') | float(0)
             - states('sensor.YOUR_HOUSE_CONSUMPTION') | float(0), 0] | max }}
```

> Heb je geen aparte huisverbruikmeting? Dan kun je de teruglevering van je P1-meter gebruiken als benadering. Let op: dit leidt tot een feedbackloop waarbij de teruglevering naar nul wordt geregeld. Een aparte huisverbruikmeting geeft nauwkeurigere resultaten. Zie [Bekende beperkingen](#bekende-beperkingen).

### 2. Placeholders vervangen

Vervang in `zaptec_laadaansturing.yaml` de volgende placeholders door jouw eigen entiteitsnamen:

| Placeholder | Omschrijving | Voorbeeld |
|---|---|---|
| `YOUR_CHARGER_NAME` | Prefix van je Zaptec lader entiteiten | `mijn_zaptec` |
| `YOUR_INSTALL_NAME` | Prefix van je Zaptec installatie entiteiten | `mijn_adres` |
| `YOUR_MOBILE_APP` | Naam van je HA Companion app notificatieservice | `mobile_app` |
| `YOUR_SOLAR_POWER` | Vermogenssensor van je omvormer | `sensor.solaredge_ac_power` |

De Zaptec integratie maakt doorgaans twee soorten entiteiten aan: entiteiten op **lader-niveau** (prefix van de lader zelf) en entiteiten op **installatie-niveau** (prefix van het installatie-adres). Controleer in Home Assistant welke entiteiten beschikbaar zijn na het instellen van de integratie.

### 3. Bestand plaatsen

Kopieer `zaptec_laadaansturing.yaml` naar je `packages/` map.

Controleer of packages zijn ingeschakeld in je `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages/
```

### 4. Herstarten

Herstart Home Assistant en controleer of de entiteiten en automations beschikbaar zijn.

---

## Werking

```
Auto aangesloten
      │
      ▼
Notificatie op telefoon
      │
      ├── Zonneladen  ──► 1-fase, stroom = zonne-overschot / 230V
      │                   (min. 6A, max. 16A, bijgewerkt bij elke wijziging)
      │
      └── Geforceerd  ──► 3-fase, 16A vast
            │
            ▼
      Laden voltooid ──► 3-fase 16A (voor klimaatbeheersing)
            │
            ▼
      Auto losgekoppeld ──► reset naar zonneladen
```

### Faseschakelaar

De Zaptec faseschakelaar werkt via een stroomdrempel op `number.YOUR_INSTALL_NAME_3_to_1_phase_switch_current`:

- Waarde `0`: 3-fase laden actief
- Waarde `32`: 1-fase laden actief (de lader schakelt automatisch bij overschrijding van de drempel)

Na het instellen van de fase wordt 5 seconden gewacht voordat de laadstroom wordt ingesteld, zodat de lader de fasewisseling kan verwerken.

### Stroomberekening bij zonneladen

```
beschikbare_ampere = max(min(floor(overschot_watt / 230), 16), 6)
```

Het zonne-overschot wordt gedeeld door 230V (1-fase spanning) en naar beneden afgerond. Het resultaat wordt begrensd tussen 6A (Zaptec minimum) en 16A (aansluiting maximum).

---

## Bekende beperkingen

**Feedbackloop bij gebruik van P1-teruglevering**
Als je `sensor.geschat_zonne_overschot` baseert op de teruglevering van je P1-meter, ontstaat een feedbackloop: zodra de auto begint te laden daalt de teruglevering, waarna de laadstroom wordt verlaagd, de teruglevering stijgt, de stroom weer omhoog gaat, enzovoort. Het systeem zoekt een evenwicht maar reageert traag en onnauwkeurig.

De meest stabiele oplossing is een onafhankelijke meting van het huisverbruik exclusief de auto, bijvoorbeeld via een energiemonitor of de huisverbruikmeting van je omvormer.

**SolarEdge update frequentie**
SolarEdge sensoren werken standaard met een lage updatefrequentie. Dit beperkt hoe snel de laadstroom reageert op wijzigingen in de zonneproductie.

**3-fase zonneladen niet mogelijk**
Bij een maximale zonneopbrengst van ongeveer 3700W is 3-fase laden op zonne-energie niet haalbaar: de minimale 3-fase laadstroom van 6A per fase vereist al 3 x 6 x 230 = 4140W. Daarom werkt zonneladen altijd op 1-fase.

---

## Bestanden

| Bestand | Omschrijving |
|---|---|
| `zaptec_laadaansturing.yaml` | HA package met input_boolean, template sensor en alle automations |

---

## Licentie

MIT. Vrij te gebruiken en aan te passen.