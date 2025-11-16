# Blueprint : Thermostat/Climatisation Pièce (v2.0)

## Description

Blueprint universel pour pièce avec thermostat ou climatisation réversible. Gère automatiquement la bascule Été/Hiver (cool/heat), l'intégration avec Solar Optimizer et le mode Away basé sur l'alarme.

**Fichier :** `blueprint_hvac_room_thermostat.yaml`
**Version :** 2.0
**Auteur :** LaCaseHome

---

## Cas d'usage

- Climatisation réversible (chaud/froid)
- Pompe à chaleur air/air
- Radiateurs avec thermostat programmable
- Systèmes multi-split
- Thermostats compatibles modes heat ET cool

---

## Fonctionnalités principales

### 1. Bascule automatique Été/Hiver
- **Hiver** → Mode HEAT + consigne hiver (défaut 21°C)
- **Été** → Mode COOL + consigne été (défaut 24°C)
- Basé sur un indicateur (calendar, input_boolean, sensor)

### 2. Gestion Away/Home via alarme
- **Alarme armée** (armed_*) :
  - Preset "away" (si supporté)
  - Sinon mode OFF
- **Alarme désarmée** :
  - Preset "home" (si supporté)
  - Sinon preset "none" (si supporté)

### 3. Solar Optimizer (prioritaire)
- Si SO actif → Laisser SO piloter (priorité absolue)
- Compatible avec tous les Solar Optimizer
- Surveille l'état du switch SO

### 4. Détection fenêtres/portes
- Arrêt automatique (OFF) en mode heat ET cool
- Délai configurable avant pause (défaut : 2 min)
- Délai configurable avant reprise (défaut : 2 min)
- Support de plusieurs capteurs

### 5. Tick périodique
- Ré-application automatique preset home + mode/consigne saisonniers
- Fréquences : Désactivé, 5, 10, 15 minutes

---

## Configuration

### Paramètres obligatoires

| Paramètre | Type | Description |
|-----------|------|-------------|
| **Nom logique de la pièce** | Texte | Nom utilisé dans les logs (ex: "Salon") |
| **Entité climate** | climate.* | Thermostat ou climatisation à piloter |

### Paramètres optionnels

#### Fenêtres
| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| **Capteurs fenêtre(s)/porte(s)** | binary_sensor.* | [] | Liste des capteurs (ON = ouvert) |
| **Délai avant PAUSE (min)** | Nombre | 2 | Minutes avant d'arrêter |
| **Délai avant REPRISE (min)** | Nombre | 2 | Minutes avant de reprendre |

#### Saison & Alarme
| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| **Indicateur Été** | entity | "" | ON = Été (calendar/input_boolean/sensor) |
| **Alarme / Présence** | alarm_control_panel.* | "" | Alarme pour mode Away |

#### Solar Optimizer
| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| **Solar Optimizer – ACTIVE switch** | switch/input_boolean/binary_sensor | "" | Switch qui indique si SO chauffe (ON = actif) |

#### Consignes par défaut
| Paramètre | Défaut | Min | Max | Step | Description |
|-----------|--------|-----|-----|------|-------------|
| **Consigne Hiver (°C)** | 21 | 16 | 24 | 0.5 | Température en mode heat |
| **Consigne Été (°C)** | 24 | 20 | 28 | 0.5 | Température en mode cool |

#### Tick
| Paramètre | Type | Défaut | Options |
|-----------|------|--------|---------|
| **Fréquence ré-application** | Select | 10 | Désactivé / 5 min / 10 min / 15 min |

---

## Ordre de priorité (actions)

L'automatisation applique les règles dans cet ordre :

1. **Fenêtres ouvertes** → OFF (prioritaire même sur SO)
2. **Solar Optimizer actif** → Laisser SO piloter (stop)
3. **Alarme armée (Away)** :
   - Preset "away" si supporté
   - Sinon OFF
4. **Alarme désarmée (Home)** :
   - Preset "home" si supporté
   - Sinon preset "none" si supporté
5. **Mode normal** :
   - Mode : `cool` (été) ou `heat` (hiver)
   - Température : consigne été ou hiver

---

## Exemples de configuration

### Exemple 1 : Climatisation réversible basique

```yaml
Nom de la pièce: Salon
Entité climate: climate.clim_salon
Indicateur Été: input_boolean.ete
Consigne Hiver: 21°C
Consigne Été: 24°C
Fréquence ré-application: 10 min
```

**Comportement :**
- Hiver → Mode HEAT à 21°C
- Été → Mode COOL à 24°C
- Pas de gestion Away ni SO

### Exemple 2 : Avec alarme et fenêtres

```yaml
Nom de la pièce: Chambre
Entité climate: climate.clim_chambre
Indicateur Été: input_boolean.ete
Alarme: alarm_control_panel.alarmo
Capteurs fenêtre:
  - binary_sensor.fenetre_chambre
Délai avant PAUSE: 2 min
Délai avant REPRISE: 3 min
Consigne Hiver: 20°C
Consigne Été: 25°C
```

**Comportement :**
- Fenêtre ouverte 2 min → OFF
- Alarme armée → Preset "away" (ou OFF si non supporté)
- Alarme désarmée → Preset "home" (ou "none" si non supporté)
- Hiver/Été → Heat 20°C / Cool 25°C

### Exemple 3 : Avec Solar Optimizer

```yaml
Nom de la pièce: Bureau
Entité climate: climate.radiateur_bureau
Indicateur Été: input_boolean.ete
Alarme: alarm_control_panel.alarmo
Solar Optimizer – ACTIVE switch: switch.solar_optimizer_bureau
Consigne Hiver: 21°C
Fréquence ré-application: 10 min
```

**Comportement :**
- SO actif (ON) → Blueprint en retrait, SO pilote
- SO inactif (OFF) + Alarme armée → Preset "away"
- SO inactif (OFF) + Alarme désarmée → Preset "home" + Heat 21°C
- **Priorité SO sur Away**

---

## Intégration Solar Optimizer

### Configuration

Le blueprint surveille un **switch d'état** qui indique si Solar Optimizer chauffe activement :

**Exemple :**
```yaml
Solar Optimizer – ACTIVE switch: switch.solar_optimizer_enable_heating
```

### Comportement

| État SO | Alarme | Action blueprint |
|---------|--------|------------------|
| ON | Armée | SO pilote (prioritaire) |
| ON | Désarmée | SO pilote (prioritaire) |
| OFF | Armée | Preset "away" |
| OFF | Désarmée | Preset "home" + mode saisonnier |

### Logs

Quand SO est actif :
```
⚡ SO actif → laisser SO piloter
```

---

## Triggers (déclencheurs)

L'automatisation se déclenche sur :

1. **Fenêtre ouverte** → Après délai configuré
2. **Fenêtre fermée** → Après délai configuré
3. **Changement saison** (Été/Hiver) → Immédiat
4. **Alarme armed/disarmed** → Immédiat
5. **Solar Optimizer ON/OFF** → Immédiat
6. **Tick périodique** → Toutes les N minutes (si activé)

**Note :** Les triggers optionnels (été/alarme/SO) utilisent un template pour gérer les valeurs vides.

---

## Logs et monitoring

### Types de logs

| Emoji | Message | Signification |
|-------|---------|---------------|
| 🪟 | Fenêtre ouverte → OFF (était en heat/cool) | Fenêtre détectée ouverte, pause |
| ⚡ | SO actif → laisser SO piloter | Solar Optimizer actif, blueprint en retrait |
| 🔒 | Alarme armée → Preset AWAY activé | Preset away appliqué |
| 🔒 | Alarme armée → OFF (pas de preset 'away') | Preset away non supporté, thermostat arrêté |
| 🏠 | Alarme désarmée → Preset HOME appliqué | Preset home appliqué |
| 🏠 | Alarme désarmée → Preset NONE (pas de 'home') | Preset home non supporté, none appliqué |
| 🌡️ | Mode HEAT/COOL activé (Été/Hiver) | Bascule saisonnière |
| 🌡️ | Consigne XX°C appliquée (Été/Hiver) | Température appliquée |

### Consulter les logs

1. **Logbook** (📖) → Filtrer par nom de pièce
2. **Mode Trace** → Automatisation → ⋮ → Trace

---

## Prérequis techniques

### Entités requises

- **climate.*** : Doit supporter au minimum :
  - Modes : `heat`, `cool`, `off`
  - Service : `climate.set_hvac_mode`
  - Service : `climate.set_temperature`

### Entités optionnelles

- **Indicateur Été** : input_boolean, calendar, sensor, binary_sensor (ON = été)
- **binary_sensor.*** : Capteurs fenêtre/porte (ON = ouvert)
- **alarm_control_panel.*** : Système d'alarme
- **switch.*** / input_boolean / binary_sensor : Solar Optimizer active switch

### Presets supportés (si disponibles)

Le thermostat peut supporter :
- `away` : Mode absent (éco renforcé)
- `home` : Mode présent (confort)
- `none` : Aucun preset

Si les presets n'existent pas, le blueprint applique directement les consignes de température.

---

## Dépannage

### Le mode ne bascule pas Été/Hiver

**Vérifications :**
1. L'indicateur Été est bien configuré
2. L'état est bien ON (été) ou OFF (hiver)
3. Le climate supporte les modes heat ET cool

**Test :**
```yaml
# Outils de développement → États
# Vérifiez l'état de votre indicateur été
# Changez-le manuellement
# Consultez le logbook
```

### SO ne prend pas la priorité

**Vérifications :**
1. Le switch SO est bien configuré
2. L'état est bien ON quand SO chauffe
3. Consultez le mode trace pour voir l'ordre d'exécution

**Log attendu :**
```
⚡ SO actif → laisser SO piloter
```

### Preset away/home non appliqué

**Cause :** Le thermostat ne supporte pas ces presets.

**Vérification :**
```yaml
# Outils de développement → États
# climate.votre_thermostat
# Attribut "preset_modes" doit contenir "away" et "home"
```

**Solution :**
- C'est normal si le thermostat ne supporte pas
- Le blueprint applique OFF (away) ou les consignes normales (home)
- Consultez les logs pour confirmation

### Fenêtre : le thermostat ne se coupe pas

**Vérifications :**
1. Le binary_sensor est bien ON quand ouvert
2. Le délai configuré est respecté
3. Le mode actuel est heat ou cool (pas off ou autre)

---

## Scénarios avancés

### Utilisation sans indicateur Été

Si vous laissez le champ "Indicateur Été" vide :
- Le blueprint considère qu'on est **toujours en Hiver**
- Mode HEAT uniquement
- Consigne Hiver appliquée

### Forcer mode cool toute l'année

Configurez un input_boolean toujours ON :
```yaml
input_boolean:
  toujours_ete:
    name: Toujours Été
    initial: on
```

Puis utilisez-le comme indicateur Été.

### Mode away manuel sans alarme

Créez un input_boolean pour simuler présence/absence :
```yaml
input_boolean:
  presence:
    name: Présence
    icon: mdi:home-account
```

Créez une alarme virtuelle basée sur ce boolean :
```yaml
# Utilisez une automation ou un template
```

Ou utilisez directement un template alarm_control_panel.

---

## Compatibilité

### Thermostats/Clims testés

- Climatisations réversibles (Daikin, Mitsubishi, Toshiba, etc.)
- Pompes à chaleur air/air
- Thermostats Zigbee réversibles
- Climatiseurs IR (via Broadlink, etc.)
- ESPHome climate

### Solar Optimizer

Compatible avec :
- Solar Optimizer (add-on Home Assistant)
- Routeurs solaires tiers
- Tout switch/boolean qui indique "chauffe maintenant"

### Home Assistant

- Version minimum : **2023.8**
- Testé jusqu'à : **2024.11**

---

## Changelog

### v2.0
- Support Solar Optimizer (prioritaire sur Away)
- Bascule automatique été/hiver (cool/heat)
- Preset away/home/none
- Gestion fenêtres en heat ET cool
- Consignes par défaut paramétrables
- Tick optimisé

### v1.x
- Version initiale

---

## Différences avec "Thermostat Heat"

| Caractéristique | Thermostat Heat | Room Thermostat |
|-----------------|-----------------|-----------------|
| Modes HVAC | Heat uniquement | Heat + Cool (réversible) |
| Bascule Été/Hiver | Non (OFF en été) | Oui (cool/heat) |
| Solar Optimizer | Non | Oui (prioritaire) |
| Gestion preset | Eco/Comfort + fallback | Away/Home |
| Consignes | Fallback complet | Hiver/Été |

**Conseil :** Utilisez "Room Thermostat" pour les climatisations réversibles, "Thermostat Heat" pour le chauffage simple.

---

## Liens utiles

- [Retour au README principal](../README.md)
- [Guide d'installation](../INSTALLATION.md)
- [Solar Optimizer Integration](solar_optimizer.md)
- [Troubleshooting général](troubleshooting.md)
- [Autres blueprints](../README.md#blueprints-disponibles)
