# Intégration Solar Optimizer

Guide complet pour utiliser les blueprints Climate avec Solar Optimizer pour optimiser l'autoconsommation de votre installation solaire.

---

## Qu'est-ce que Solar Optimizer ?

**Solar Optimizer** est un système qui utilise le surplus de production solaire photovoltaïque pour chauffer l'eau ou les radiateurs, maximisant ainsi l'autoconsommation et réduisant la facture électrique.

### Principe de fonctionnement

```
Production solaire > Consommation
        ↓
Surplus disponible
        ↓
Solar Optimizer active le chauffage
        ↓
Surplus consommé au lieu d'être injecté
        ↓
Autoconsommation maximisée
```

### Avantages

- Autoconsommation jusqu'à 90%+
- Économies sur la facture électrique
- Valorisation du surplus solaire
- Chauffage gratuit en journée
- Réduction de l'injection réseau

---

## Blueprints compatibles

| Blueprint | Support SO | Priorité | Autorisation Away |
|-----------|------------|----------|-------------------|
| Thermostat Heat | ❌ | - | - |
| Room Thermostat | ✅ | Prioritaire sur Away | Non |
| **X4FP Bathroom** | ✅ | **Priorité absolue** | **Oui** |
| **X4FP Room** | ✅ | **Priorité absolue** | **Oui** |

**Recommandé :** X4FP Bathroom et X4FP Room pour usage optimal avec SO.

---

## Configuration Solar Optimizer

### Prérequis

1. **Installation solaire photovoltaïque** avec monitoring
2. **Solar Optimizer configuré** dans Home Assistant :
   - Add-on Solar Optimizer
   - Ou routeur solaire tiers
   - Ou custom automation
3. **Switch d'état** qui indique quand SO chauffe activement

### Types de Solar Optimizer supportés

#### 1. Solar Optimizer (add-on Home Assistant)

**Installation :**
- Add-on disponible dans le store HA
- Configuration via interface
- Crée automatiquement les entités

**Entités créées :**
```
switch.solar_optimizer_enable_xxx
sensor.solar_optimizer_power_xxx
sensor.solar_optimizer_surplus
```

**Configuration blueprint :**
```yaml
Solar Optimizer – SWITCH: switch.solar_optimizer_enable_chauffage_salon
```

#### 2. Routeur solaire matériel

**Exemples :**
- MyEnergi Eddi
- Victron Energy
- Fronius Ohmpilot
- Solar-Log
- SMA Home Manager

**Configuration :**
Créez un template switch qui surveille l'état :

```yaml
template:
  - switch:
      - name: "SO Actif Salon"
        unique_id: so_actif_salon
        state: >-
          {{ states('sensor.eddi_mode_salon') == 'boost' }}
        turn_on:
          service: script.so_enable_salon
        turn_off:
          service: script.so_disable_salon
```

**Configuration blueprint :**
```yaml
Solar Optimizer – SWITCH: switch.so_actif_salon
```

#### 3. Custom automation

Créez votre propre logique :

```yaml
input_boolean:
  so_chauffe_salon:
    name: SO Chauffe Salon

automation:
  - alias: "SO - Démarrer chauffage si surplus"
    trigger:
      - platform: numeric_state
        entity_id: sensor.surplus_solaire
        above: 1000  # Watts
        for: "00:05:00"
    action:
      - service: input_boolean.turn_on
        target:
          entity_id: input_boolean.so_chauffe_salon

  - alias: "SO - Arrêter chauffage si plus de surplus"
    trigger:
      - platform: numeric_state
        entity_id: sensor.surplus_solaire
        below: 500  # Watts
        for: "00:05:00"
    action:
      - service: input_boolean.turn_off
        target:
          entity_id: input_boolean.so_chauffe_salon
```

**Configuration blueprint :**
```yaml
Solar Optimizer – SWITCH: input_boolean.so_chauffe_salon
```

---

## Configuration dans les blueprints

### Room Thermostat

```yaml
Nom de la pièce: Salon
Entité climate: climate.clim_salon
Alarme: alarm_control_panel.alarmo
Solar Optimizer – ACTIVE switch: switch.solar_optimizer_salon
Consigne Hiver: 21°C
Fréquence ré-application: 10 min
```

**Ordre de priorité :**
1. Fenêtres ouvertes → OFF
2. **Solar Optimizer actif → Laisser SO piloter** ⭐
3. Alarme armée → Preset away
4. Mode normal → Heat/Cool selon saison

### X4FP Bathroom

```yaml
Nom de la pièce: Salle de Bain
Entité climate: climate.x4fp_seche_serviette
Lumière: light.sdb
Solar Optimizer – SWITCH: switch.solar_optimizer_seche_serviette
Autoriser SO en Away: true
Preset quand SO chauffe: comfort
Quand lumière ON: force_comfort
Quand lumière OFF: force_eco
Preset Away: away
```

**Ordre de priorité :**
1. Été → OFF/ECO
2. Fenêtres ouvertes → Away
3. **Solar Optimizer actif → Comfort** ⭐ (priorité absolue)
4. Alarme armée → Away (sauf si SO actif ET autorisé)
5. Lumière → Comfort/Eco

### X4FP Room

```yaml
Nom de la pièce: Bureau
Entité climate: climate.x4fp_bureau
Capteur température: sensor.temp_bureau
Consigne: input_number.consigne_bureau
Hystérésis: 0.5°C
Solar Optimizer – SWITCH: switch.solar_optimizer_bureau
Autoriser SO en Away: false
Preset quand SO chauffe: comfort
Preset quand on chauffe: comfort
Preset quand on n'a plus besoin: eco
```

**Ordre de priorité :**
1. Été → OFF/ECO
2. Fenêtres ouvertes → Away
3. **Solar Optimizer actif → Comfort** ⭐ (priorité absolue)
4. Alarme armée → Away (bloque SO si non autorisé)
5. Contrôle thermique → Heat/Idle

---

## Paramètre "Autoriser SO en Away"

### Description

Ce paramètre détermine si Solar Optimizer peut chauffer quand l'alarme est armée (mode Away).

**Disponible sur :**
- X4FP Bathroom
- X4FP Room

**Non disponible sur :**
- Room Thermostat (SO toujours prioritaire)

### Comportement

#### Autorisé (true)

```yaml
Autoriser SO en Away: true
```

| État SO | Alarme | Action |
|---------|--------|--------|
| ON | Armée | **SO chauffe** (Comfort) ⭐ |
| ON | Désarmée | SO chauffe (Comfort) |
| OFF | Armée | Preset Away |
| OFF | Désarmée | Blueprint normal |

**Usage :** Profiter du surplus même en absence.

**Exemple :**
```
11h00 - Personne à la maison (alarme armée)
Production solaire : 4000W
Surplus : 2000W
→ SO active le chauffage → COMFORT
→ Surplus consommé au lieu d'être perdu
```

#### Non autorisé (false - défaut)

```yaml
Autoriser SO en Away: false
```

| État SO | Alarme | Action |
|---------|--------|--------|
| ON | Armée | **Preset Away** (SO bloqué) |
| ON | Désarmée | SO chauffe (Comfort) |
| OFF | Armée | Preset Away |
| OFF | Désarmée | Blueprint normal |

**Usage :** Sécurité, économie maximale en absence.

**Exemple :**
```
11h00 - Personne à la maison (alarme armée)
→ Preset AWAY (hors-gel 7°C)
→ Même si surplus disponible, pas de chauffage
```

### Quel choix ?

| Situation | Recommandation |
|-----------|----------------|
| Absence courte (journée) | **Autoriser** (true) |
| Absence longue (vacances) | **Ne pas autoriser** (false) |
| Sécurité prioritaire | **Ne pas autoriser** (false) |
| Autoconsommation maximale | **Autoriser** (true) |
| Surplus important régulier | **Autoriser** (true) |

**Conseil :** Commencez avec `false`, puis testez `true` selon vos besoins.

---

## Preset quand SO chauffe

### Description

Quel preset appliquer quand Solar Optimizer est actif.

**Disponible sur :**
- Room Thermostat (limité)
- X4FP Bathroom
- X4FP Room

### Options

| Preset | Description | Usage |
|--------|-------------|-------|
| **comfort** | Chauffage confort maximum | **Recommandé** (défaut) |
| comfort-1 | Confort -1°C | Si surplus limité |
| comfort-2 | Confort -2°C | Si surplus très limité |
| eco | Éco | Maintien minimum |
| boost | Boost (si supporté) | Chauffage intensif |
| **none** | Blueprint en retrait | **SO pilote seul** |

### Preset "comfort" (recommandé)

```yaml
Preset quand SO chauffe: comfort
```

**Comportement :**
- SO ON → Blueprint applique COMFORT
- Chauffage au maximum
- Valorisation surplus maximale

**Logs :**
```
⚡ SO ACTIF (switch.solar_optimizer_xxx) → COMFORT
```

### Preset "none" (avancé)

```yaml
Preset quand SO chauffe: none
```

**Comportement :**
- SO ON → Blueprint se met en retrait
- SO pilote directement le radiateur comme il veut
- Blueprint ne touche à rien

**Logs :**
```
⚡ SO ACTIF → Blueprint en retrait (preset none)
```

**Usage :**
- Si votre Solar Optimizer gère lui-même les presets
- Si vous voulez un contrôle total par SO
- Avec add-on Solar Optimizer avancé

**Attention :** Le blueprint ne fait plus RIEN tant que SO est actif.

---

## Scénarios d'utilisation

### Scénario 1 : Sèche-serviettes avec SO

**Configuration :**
```yaml
Blueprint: X4FP Bathroom
Entité: climate.x4fp_seche_serviette
Lumière: light.sdb
SO Switch: switch.solar_optimizer_seche_serviette
Autoriser SO en Away: true
Preset quand SO chauffe: comfort
Lumière ON: force_comfort
Lumière OFF: force_eco
```

**Journée type :**
```
07h00 - Douche (lumière ON)
→ COMFORT (blueprint)

08h00 - Départ maison (alarme armée, lumière OFF)
→ AWAY (hors-gel)

10h00 - Production solaire démarre
→ SO détecte surplus
→ SO ON → COMFORT (même si Away autorisé)
→ Sèche-serviettes chauffe avec surplus gratuit

14h00 - Nuages, plus de surplus
→ SO OFF → AWAY (hors-gel)

18h00 - Retour maison (alarme désarmée)
→ Lumière OFF → ECO

19h00 - Douche (lumière ON)
→ COMFORT (blueprint)
```

**Résultat :**
- 4h de chauffage gratuit en journée (10h-14h)
- Confort le matin et soir
- Économie en absence

### Scénario 2 : Chauffage salon avec contrôle thermique

**Configuration :**
```yaml
Blueprint: X4FP Room
Entité: climate.x4fp_salon
Capteur température: sensor.temp_salon
Consigne: input_number.consigne_salon (20°C)
Hystérésis: 0.5°C
SO Switch: switch.solar_optimizer_salon
Autoriser SO en Away: false
Preset quand SO chauffe: comfort
```

**Journée type :**
```
08h00 - Départ maison (alarme armée)
→ AWAY (hors-gel 7°C)
→ Température descend lentement

11h00 - Production solaire, surplus disponible
→ SO veut chauffer
→ Mais Away et "Autoriser SO en Away" = false
→ AWAY maintenu (SO bloqué)

18h00 - Retour maison (alarme désarmée)
→ Température = 17°C, Consigne = 20°C
→ 17°C ≤ 19.5°C → COMFORT (chauffe)

19h00 - Température = 20.6°C
→ 20.6°C ≥ 20.5°C → ECO (maintien)
```

**Résultat :**
- Sécurité en absence (hors-gel)
- Contrôle thermique précis à la maison
- Pas d'utilisation SO en absence (choix)

### Scénario 3 : Cumulus + Radiateurs

**Stratégie :**
1. SO chauffe d'abord le cumulus (priorité)
2. Si surplus restant → Radiateurs

**Configuration cumulus (prioritaire) :**
```yaml
# Via Solar Optimizer add-on
Priority: 1
Power: 2000W
```

**Configuration radiateur 1 (secondaire) :**
```yaml
Blueprint: X4FP Room
SO Switch: switch.solar_optimizer_chambre
Autoriser SO en Away: true
Preset quand SO chauffe: comfort
# Priority: 2 (dans SO add-on)
```

**Configuration radiateur 2 (tertiaire) :**
```yaml
Blueprint: X4FP Room
SO Switch: switch.solar_optimizer_bureau
Autoriser SO en Away: true
Preset quand SO chauffe: comfort-1  # Moins gourmand
# Priority: 3 (dans SO add-on)
```

**Résultat :**
- Surplus optimal réparti
- Cumulus chargé en priorité
- Radiateurs si surplus restant

---

## Monitoring et optimisation

### Dashboard Lovelace

**Exemple de carte :**

```yaml
type: entities
title: Solar Optimizer
entities:
  - entity: sensor.surplus_solaire
    name: Surplus disponible
  - entity: switch.solar_optimizer_seche_serviette
    name: SO Sèche-serviettes
  - entity: switch.solar_optimizer_salon
    name: SO Salon
  - entity: sensor.autoconsommation_pct
    name: Autoconsommation
  - entity: sensor.production_solaire
    name: Production
```

### Graphiques

```yaml
type: history-graph
title: Production et Chauffage SO
entities:
  - entity: sensor.production_solaire
    name: Production
  - entity: sensor.consommation_totale
    name: Consommation
  - entity: switch.solar_optimizer_seche_serviette
    name: SO Actif
hours_to_show: 24
```

### Statistiques

**Energy Dashboard :**
- Ajoutez vos entités de production solaire
- Ajoutez vos radiateurs comme consommateurs
- Suivez l'autoconsommation

---

## Dépannage

### SO ne s'active jamais

**Vérifications :**
1. Switch SO bien configuré dans le blueprint
2. L'entité existe et est accessible
3. SO fonctionne (testez manuellement le switch)
4. Surplus solaire suffisant

**Test :**
```yaml
# Outils de développement → États
# Activez manuellement le switch SO
# Consultez le logbook du blueprint
```

### SO actif mais blueprint ne réagit pas

**Vérifications :**
1. Le switch est bien ON
2. L'automatisation blueprint est activée
3. Consultez le mode trace

**Attendu dans trace :**
```
Condition: solar_is_heating = true
Action: Preset COMFORT
Stop: SO chauffe activement
```

### SO et Away : conflit

**Symptôme :**
SO devrait chauffer mais ne le fait pas car alarme armée.

**Solution :**
```yaml
Autoriser SO en Away: true
```

### Blueprint override SO

**Symptôme :**
SO veut chauffer, mais blueprint applique autre chose.

**Cause :**
Ordre de priorité non respecté.

**Vérification :**
Consultez le mode trace, SO doit être en position 3 (après Été et Fenêtre).

**Solution :**
- Mettez à jour le blueprint vers v7.2+
- Vérifiez la configuration

---

## Liens utiles

- [Retour au README principal](../README.md)
- [X4FP Bathroom](x4fp_bathroom.md)
- [X4FP Room](x4fp_room.md)
- [Room Thermostat](room_thermostat.md)
- [Troubleshooting général](troubleshooting.md)

---

## Ressources externes

- [Solar Optimizer Add-on](https://github.com/tribp/Solar-Optimizer-HA-Addon) (exemple)
- [Home Assistant Energy Management](https://www.home-assistant.io/docs/energy/)
- [Forum HA - Solar Routing](https://community.home-assistant.io/)
