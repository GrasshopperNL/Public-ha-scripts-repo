# Zaptec Solar Charging voor Home Assistant

Automatisch laden op basis van zonneoverschot met een Zaptec Go 2 laadpaal, inclusief geforceerd laden.

Twee varianten beschikbaar:

- **Met fasewisseling** (`automations_phase_switching.yaml`): schakelt automatisch tussen 1-fase en 3-fase via de Zaptec fase-schakeldrempel. Vereist een P1 slimme meter.
- **Zonder fasewisseling** (`automations.yaml`): eenvoudiger, laadt altijd op 1-fase bij zonneladen. Gebruikt SolarEdge integratie voor zonproductie.

## Hoe het werkt

Zodra de auto aangesloten wordt verschijnt er een melding op je telefoon met de vraag hoe je wilt laden. Je kiest tussen zonneladen of geforceerd laden. De keuze is ook te maken via een knop op het dashboard.

**Zonneladen (1-fase)**
De actuele zonproductie wordt uitgelezen via de SolarEdge integratie. Er wordt 400W gereserveerd voor huishoudelijk verbruik. De laadstroom wordt aangepast tussen 6A en 16A op basis van het overschot. Bij minder dan 1380W (6A x 230V) wordt de laadstroom op minimaal 6A gehouden.

**Geforceerd laden**
Laadt altijd op maximaal vermogen (16A), ongeacht de zonproductie.

## Vereisten

- Zaptec Go 2 laadpaal
- [custom-components/zaptec](https://github.com/custom-components/zaptec) integratie via HACS
- SolarEdge integratie in Home Assistant
- Home Assistant Companion App op Android of iOS

## Installatie

### 1. Helpers aanmaken

Maak de volgende helper aan via **Instellingen > Apparaten en diensten > Helpers**:

| Type | Naam | Entity ID |
|------|------|-----------|
| Schakelaar | Geforceerd laden | `input_boolean.geforceerd_laden` |

### 2. Automations toevoegen

Voeg de inhoud van `automations.yaml` toe aan jouw `automations.yaml` of importeer ze via de UI.

Pas de volgende placeholders aan:

| Placeholder | Omschrijving | Voorbeeld |
|-------------|--------------|-----------|
| `YOUR_CHARGER_NAME` | Naam van jouw Zaptec laadpaal | `GNP123456` |
| `YOUR_INSTALL_NAME` | Naam van jouw Zaptec installatie | `mijn_lader` |
| `YOUR_MOBILE_APP` | Naam van jouw mobiele app entity | `mobile_phone` |

De naam van je mobiele app vind je via **Instellingen > Apparaten en diensten > Companion App**.

### 3. Dashboard toevoegen

Voeg de inhoud van `lovelace.yaml` toe als handmatige kaart op jouw dashboard.

Pas `YOUR_INSTALL_NAME` en `YOUR_CHARGER_NAME` aan naar jouw eigen entiteitnamen.

## Entiteiten

De volgende Zaptec entiteiten worden gebruikt:

| Entiteit | Omschrijving |
|----------|--------------|
| `sensor.YOUR_CHARGER_NAME_charger_mode` | Status van de laadpaal |
| `sensor.YOUR_CHARGER_NAME_charge_power` | Huidig laadvermogen |
| `number.YOUR_INSTALL_NAME_available_current` | Beschikbare laadstroom |

Aanvullend voor de variant met fasewisseling:

| Entiteit | Omschrijving |
|----------|--------------|
| `number.YOUR_INSTALL_NAME_3_to_1_phase_switch_current` | Fase-schakeldrempel |
| `button.YOUR_CHARGER_NAME_resume_charging` | Hervat laden |
| `button.YOUR_CHARGER_NAME_stop_charging` | Stop laden |

## Technische achtergrond

De Zaptec Go 2 vereist minimaal 6A laadstroom (IEC 61851 protocol). Op 1-fase komt dit overeen met minimaal 1380W. Bij de variant zonder fasewisseling wordt 400W gereserveerd voor huishoudelijk verbruik voordat de resterende productie omgerekend wordt naar laadstroom.

Zie ook de [Zaptec documentatie over fase-wisseling](https://docs.zaptec.com/docs/3-to-1-phase-switching-with-zaptec-go-2).