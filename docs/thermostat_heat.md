# Blueprint : Thermostat Chauffage Simple (v3.0)

## Description

Blueprint pour thermostat de chauffage avec gestion automatique basée sur l'état de l'alarme (armée = ECO, désarmée = CONFORT). Idéal pour le chauffage électrique simple, radiateurs avec thermostat intégré, ou chaudières.

**Fichier :** `blueprint_hvac_thermostat_heat.yaml`
**Version :** 3.0
**Auteur :** LaCaseHome

---

## Cas d'usage

- Chauffage électrique avec thermostat intégré
- Radiateurs pilotés par un thermostat programmable
- Chaudière avec thermostat d'ambiance
- Convecteurs électriques avec module climate

---

## Fonctionnalités principales

### 1. Gestion par alarme (Eco/Confort)
- **Alarme armée** (armed_away, armed_home, armed_night) → Preset ECO (configurable)
- **Alarme désarmée** → Preset CONFORT (configurable)
- Triggers STATE explicites et fiables

### 2. Détection fenêtres/portes
- Arrêt automatique du chauffage (OFF) quand fenêtre ouverte
- Délai configurable avant pause (défaut : 2 min)
- Délai configurable avant reprise (défaut : 2 min)
- Support de plusieurs capteurs

### 3. Mode Été
- Arrêt automatique du chauffage en été
- Basé sur un indicateur (calendar, input_boolean, sensor)
- Comportements : OFF ou Ne rien faire

### 4. Fallback température complet
- Si le thermostat ne supporte pas un preset → température de secours
- Températures configurables pour :
  - Confort (défaut 21°C)
  - Eco (défaut 18.5°C)
  - Hors-gel (défaut 7°C)
  - Boost (défaut 23°C)

### 5. Tick périodique
- Ré-application automatique des réglages
- Fréquences : Désactivé, 5, 10, 15 minutes
- Optimisé pour minimiser les appels inutiles

---

## Configuration

### Paramètres obligatoires

| Paramètre | Type | Description |
|-----------|------|-------------|
| **Nom logique de la pièce** | Texte | Nom utilisé dans les logs (ex: "Salon") |
| **Thermostat** | climate.* | Entité du thermostat à piloter |
| **Indicateur Été** | entity | Indicateur ON = Été (calendar/input_boolean/sensor) |

### Paramètres optionnels

#### Fenêtres
| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| **Capteurs fenêtre(s)/porte(s)** | binary_sensor.* | [] | Liste des capteurs (ON = ouvert) |
| **Délai avant PAUSE (min)** | Nombre | 2 | Minutes avant d'arrêter le chauffage |
| **Délai avant REPRISE (min)** | Nombre | 2 | Minutes avant de reprendre le chauffage |

#### Alarme & Presets
| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| **Alarme / Présence** | alarm_control_panel.* | - | Alarme pour mode Away |
| **Preset quand ARMÉE** | Select | eco | Preset à appliquer (armed_*) |
| **Preset quand DÉSARMÉE** | Select | comfort | Preset à appliquer (disarmed) |

Presets disponibles : `eco`, `comfort`, `comfort-1`, `comfort-2`, `frost_protection`, `boost`, `none`

#### Températures fallback
| Paramètre | Défaut | Min | Max | Step |
|-----------|--------|-----|-----|------|
| **Température Confort (°C)** | 21 | 16 | 24 | 0.5 |
| **Température Eco (°C)** | 18.5 | 12 | 22 | 0.5 |
| **Température Hors-gel (°C)** | 7 | 5 | 15 | 0.5 |
| **Température Boost (°C)** | 23 | 20 | 28 | 0.5 |

#### Comportement Été
| Paramètre | Type | Défaut | Options |
|-----------|------|--------|---------|
| **Comportement en Été** | Select | off_in_summer | "Mettre sur OFF" / "Ne rien faire" |

#### Tick
| Paramètre | Type | Défaut | Options |
|-----------|------|--------|---------|
| **Fréquence ré-application** | Select | 10 | Désactivé / 5 min / 10 min / 15 min |

---

## Ordre de priorité (actions)

L'automatisation applique les règles dans cet ordre :

1. **Été** (si activé et policy = OFF) → Thermostat OFF
2. **Fenêtre ouverte** → Thermostat OFF
3. **Mode HEAT** → Force le mode heat (thermostat chauffage)
4. **Preset ou Fallback** :
   - Si preset supporté → Application du preset
   - Si preset non supporté → Température de secours

---

## Exemples de configuration

### Exemple 1 : Configuration basique

```yaml
Nom de la pièce: Salon
Thermostat: climate.thermostat_salon
Indicateur Été: input_boolean.ete
Alarme: alarm_control_panel.alarmo
Preset quand ARMÉE: eco
Preset quand DÉSARMÉE: comfort
```

**Comportement :**
- Alarme armée → Preset ECO
- Alarme désarmée → Preset COMFORT
- Été → Thermostat OFF
- Pas de gestion fenêtres

### Exemple 2 : Avec fenêtres et températures personnalisées

```yaml
Nom de la pièce: Chambre
Thermostat: climate.thermostat_chambre
Indicateur Été: input_boolean.ete
Capteurs fenêtre:
  - binary_sensor.fenetre_chambre_1
  - binary_sensor.fenetre_chambre_2
Délai avant PAUSE: 3 min
Délai avant REPRISE: 5 min
Alarme: alarm_control_panel.alarmo
Preset quand ARMÉE: eco
Preset quand DÉSARMÉE: comfort
Température Confort: 19°C
Température Eco: 17°C
Fréquence ré-application: 10 min
```

**Comportement :**
- Fenêtre ouverte pendant 3 min → OFF
- Fenêtre fermée pendant 5 min → Reprise
- Alarme armée → ECO (17°C si preset non supporté)
- Alarme désarmée → COMFORT (19°C si preset non supporté)

### Exemple 3 : Thermostat sans presets (fallback systématique)

Si votre thermostat ne supporte pas les presets eco/comfort :

```yaml
Nom de la pièce: Bureau
Thermostat: climate.radiateur_bureau
Preset quand ARMÉE: eco
Preset quand DÉSARMÉE: comfort
Température Confort: 21°C
Température Eco: 18°C
```

Le blueprint détectera automatiquement que les presets ne sont pas supportés et appliquera directement les températures de secours.

---

## Triggers (déclencheurs)

L'automatisation se déclenche sur :

1. **Fenêtre ouverte** → Après délai configuré
2. **Fenêtre fermée** → Après délai configuré
3. **Changement saison** (Été/Hiver) → Immédiat
4. **Alarme armée** (armed_away, armed_home, armed_night) → Immédiat
5. **Alarme désarmée** → Immédiat
6. **Tick périodique** → Toutes les N minutes (si activé)

---

## Logs et monitoring

Le blueprint écrit dans le **Logbook** avec le nom de la pièce :

### Types de logs

| Emoji | Message | Signification |
|-------|---------|---------------|
| ☀️ | Été → OFF | Mode été activé, thermostat arrêté |
| 🪟 | Fenêtre ouverte → OFF | Fenêtre détectée ouverte, pause |
| 🔥 | Mode HEAT activé | Thermostat forcé en mode chauffage |
| 🔒 | Away → ECO | Alarme armée, preset ECO appliqué |
| 🏠 | Présent → COMFORT | Alarme désarmée, preset COMFORT appliqué |
| 🔒/🏠 | preset XXX indisponible → fallback YY°C | Preset non supporté, température de secours appliquée |

### Consulter les logs

**Dans Logbook :**
1. Allez dans **Logbook** (📖)
2. Recherchez le nom de votre pièce
3. Consultez les actions effectuées

**Mode Trace :**
1. Allez dans l'automatisation
2. Cliquez sur **⋮** → **Trace**
3. Consultez le déroulement pas à pas

---

## Prérequis techniques

### Entités requises

- **climate.*** : Thermostat avec au minimum mode `heat` et `off`
- **Indicateur Été** :
  - `input_boolean.*` (ON = été)
  - `calendar.*` (ON/in session = été)
  - `sensor.*` ou `binary_sensor.*` (ON = été)

### Entités optionnelles

- **binary_sensor.*** : Capteurs fenêtre/porte (device_class: window ou door, ON = ouvert)
- **alarm_control_panel.*** : Système d'alarme (Alarmo, Manual Alarm, etc.)

### Presets supportés (si disponibles)

Le thermostat peut supporter un ou plusieurs de ces presets :
- `eco`
- `comfort`
- `comfort-1`
- `comfort-2`
- `frost_protection`
- `boost`
- `none` (aucun preset)

Si le preset configuré n'existe pas, le blueprint applique automatiquement la température de secours correspondante.

---

## Dépannage

### Le preset n'est pas appliqué

**Vérification :**
```yaml
# Outils de développement → États
# Recherchez votre climate.*
# Attribut "preset_modes" doit contenir le preset configuré
```

**Solution :**
- Si le preset n'existe pas → Le fallback température sera utilisé automatiquement
- Vérifiez les logs pour voir "preset XXX indisponible → fallback"

### Le thermostat ne passe pas en mode HEAT

**Causes possibles :**
- Thermostat en mode manuel/off
- Thermostat qui ne supporte pas le mode heat

**Solution :**
- Vérifiez `hvac_modes` dans les attributs de l'entité climate.*
- Le blueprint force le mode heat, consultez les logs

### Fenêtre : le chauffage ne se coupe pas

**Vérifications :**
1. Le binary_sensor est bien ON quand ouvert
2. Le délai configuré est respecté
3. L'automatisation est activée

**Test :**
```yaml
# Outils de développement → États
# Changez manuellement le binary_sensor en ON
# Attendez le délai configuré
# Consultez le logbook
```

### Alarme : pas de changement de preset

**Vérifications :**
1. L'alarme est bien dans l'état armed_* ou disarmed
2. Le preset configuré existe sur le thermostat
3. Consultez le mode trace

---

## Scénarios avancés

### Utilisation sans alarme

Si vous n'avez pas d'alarme :
- Laissez le champ "Alarme / Présence" vide
- Le blueprint utilisera le preset désarmé (confort) par défaut
- Créez un input_boolean pour simuler présence/absence :

```yaml
input_boolean:
  presence_maison:
    name: Présence maison
    icon: mdi:home-account
```

Utilisez cet input_boolean comme "alarme" :
- ON = disarmed (confort)
- OFF = armed (eco)

### Mode hors-gel seul (vacances)

Pour forcer le hors-gel :
1. Armez l'alarme
2. Configurez "Preset quand ARMÉE" = `frost_protection`
3. Température Hors-gel = 7°C (ou selon besoin)

### Boost temporaire manuel

Le blueprint ne gère pas directement le boost, mais vous pouvez :
- Désactiver temporairement l'automatisation
- Activer manuellement le preset boost sur le thermostat
- Réactiver l'automatisation après

---

## Compatibilité

### Thermostats testés

- Thermostats Zigbee (Tuya, Sonoff, Moes, etc.)
- Netatmo Thermostat
- Honeywell Evohome
- Thermostats ESPHome
- Generic Thermostat (HA)

### Home Assistant

- Version minimum : **2023.8**
- Testé jusqu'à : **2024.11**

---

## Changelog

### v3.0
- Triggers alarme STATE explicites et fiables (armed_away, armed_home, armed_night)
- Fallback température COMPLET pour tous les presets
- Tick optimisé + logs conditionnels
- Support preset "none"
- Amélioration robustesse

### v2.x
- Support comportement été configurable
- Amélioration gestion fenêtres

### v1.x
- Version initiale

---

## Liens utiles

- [Retour au README principal](../README.md)
- [Guide d'installation](../INSTALLATION.md)
- [Troubleshooting général](troubleshooting.md)
- [Autres blueprints](../README.md#blueprints-disponibles)
