# Zaptec Solar Charging voor Home Assistant

Automatisch laden op basis van zonneoverschot met een Zaptec Go 2 laadpaal, inclusief geforceerd laden op 3-fase.

## Hoe het werkt

Zodra de auto aangesloten wordt verschijnt er een melding op je telefoon met de vraag hoe je wilt laden. Je kiest tussen zonneladen op 1-fase of geforceerd laden op 3-fase. De keuze is ook te maken via een knop op het dashboard.

**Zonneladen (1-fase)**
Elke 30 seconden wordt de actuele zonproductie uitgelezen en wordt de laadstroom daarop aangepast tussen 6A en 16A. Bij minder dan 1380W (6A x 230V) wordt het laden gepauzeerd en automatisch hervat zodra er weer genoeg zon is.

**Geforceerd laden (3-fase)**
Laadt altijd op maximaal vermogen (16A), ongeacht de zonproductie.

## Vereisten

- Zaptec Go 2 laadpaal
- [custom-components/zaptec](https://github.com/custom-components/zaptec) integratie via HACS
- P1 slimme meter gekoppeld aan Home Assistant
- Home Assistant Companion App op Android of iOS

## Zaptec portal instellingen

Zorg dat de volgende instellingen correct zijn in de Zaptec portal:

- **Zaptec Sense (APM)**: uitgeschakeld
- **Standalone mode**: uitgeschakeld
- **Max phases** (laadpaal instelling): 3
- **Allow chargers to return to three phase charging**: ingeschakeld (vereist service-rechten bij Zaptec)

## Installatie

### 1. Helpers aanmaken

Maak de volgende helper aan via **Instellingen > Apparaten en diensten > Helpers**:

| Type | Naam | Entity ID |
|------|------|-----------|
| Schakelaar | Geforceerd laden | `input_boolean.zaptec_geforceerd_laden` |

Of voeg het onderstaande toe aan `configuration.yaml` (zie `configuration.yaml` in deze repo).

### 2. Template sensor toevoegen

Voeg de inhoud van `configuration.yaml` toe aan jouw bestaande `configuration.yaml` en herstart Home Assistant.

Pas `YOUR_INSTALL_NAME` aan naar de naam van jouw Zaptec installatie, bijvoorbeeld `mijn lader`.

### 3. Automations toevoegen

Voeg de inhoud van `automations.yaml` toe aan jouw `automations.yaml` of importeer ze via de UI.

Pas de volgende placeholders aan:

| Placeholder | Omschrijving | Voorbeeld |
|-------------|--------------|-----------|
| `YOUR_CHARGER_NAME` | Naam van jouw Zaptec laadpaal | `GNP123456` |
| `YOUR_INSTALL_NAME` | Naam van jouw Zaptec installatie | `mijn lader` |
| `YOUR_MOBILE_APP` | Naam van jouw mobiele app entity | `mobile_phone` |

De naam van je mobiele app vind je via **Instellingen > Apparaten en diensten > Companion App**.

### 4. Dashboard toevoegen

Voeg de inhoud van `lovelace.yaml` toe als handmatige kaart op jouw dashboard.

## Entiteiten

De volgende Zaptec entiteiten worden gebruikt:

| Entiteit | Omschrijving |
|----------|--------------|
| `sensor.YOUR_CHARGER_NAME_charger_mode` | Status van de laadpaal |
| `number.YOUR_INSTALL_NAME_available_current` | Beschikbare laadstroom |
| `number.YOUR_INSTALL_NAME_3_to_1_phase_switch_current` | Fase-schakeldrempel |
| `button.YOUR_CHARGER_NAME_resume_charging` | Hervat laden |
| `button.YOUR_CHARGER_NAME_stop_charging` | Stop laden |

## Technische achtergrond

De Zaptec Go 2 vereist minimaal 6A laadstroom (IEC 61851 protocol). Op 1-fase komt dit overeen met 1380W minimaal vermogen. De fase-schakeldrempel wordt ingesteld op 32 (altijd 1-fase) bij zonneladen en op 0 (altijd 3-fase) bij geforceerd laden.

Zie ook de [Zaptec documentatie over fase-wisseling](https://docs.zaptec.com/docs/3-to-1-phase-switching-with-zaptec-go-2).
