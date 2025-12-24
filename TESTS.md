# Guide de Test des Blueprints - Détections Robustes

Ce document contient tous les tests à effectuer pour valider que les détections d'état sont robustes et insensibles à la casse.

## 🎯 Objectif

Vérifier que **TOUTES** les détections fonctionnent quelle que soit la casse retournée par Home Assistant :
- `'on'`, `'On'`, `'ON'`
- `'armed_away'`, `'Armed_away'`, `'ARMED_AWAY'`
- etc.

## 📋 Prérequis

1. Avoir importé les blueprints (v3.6, v2.9, v7.16, v7.13)
2. Avoir créé les automatisations basées sur ces blueprints
3. Avoir les entités configurées (alarme, été, Solar Optimizer, lumière)

## 🧪 Scripts de Test

Copiez les scripts ci-dessous dans votre fichier `scripts.yaml` ou via l'interface Home Assistant.

### Test 1 : Alarme (States avec différentes casses)

```yaml
test_alarm_lowercase:
  alias: "Test Alarme - lowercase (armed_away)"
  sequence:
    - service: input_text.set_value
      target:
        entity_id: input_text.test_alarm_state
      data:
        value: "armed_away"
    - service: logbook.log
      data:
        name: "Test Alarme"
        message: "État alarme forcé à : armed_away (lowercase)"

test_alarm_capitalized:
  alias: "Test Alarme - Capitalized (Armed_away)"
  sequence:
    - service: input_text.set_value
      target:
        entity_id: input_text.test_alarm_state
      data:
        value: "Armed_away"
    - service: logbook.log
      data:
        name: "Test Alarme"
        message: "État alarme forcé à : Armed_away (Capitalized)"

test_alarm_uppercase:
  alias: "Test Alarme - UPPERCASE (ARMED_AWAY)"
  sequence:
    - service: input_text.set_value
      target:
        entity_id: input_text.test_alarm_state
      data:
        value: "ARMED_AWAY"
    - service: logbook.log
      data:
        name: "Test Alarme"
        message: "État alarme forcé à : ARMED_AWAY (UPPERCASE)"

test_alarm_disarmed:
  alias: "Test Alarme - Désarmée (disarmed)"
  sequence:
    - service: input_text.set_value
      target:
        entity_id: input_text.test_alarm_state
      data:
        value: "disarmed"
    - service: logbook.log
      data:
        name: "Test Alarme"
        message: "État alarme forcé à : disarmed"
```

### Test 2 : Été (States avec différentes casses)

```yaml
test_summer_lowercase:
  alias: "Test Été - lowercase (on)"
  sequence:
    - service: input_boolean.turn_on
      target:
        entity_id: input_boolean.test_summer
    - delay:
        seconds: 1
    - service: developer_tools.set_state
      data:
        entity_id: input_boolean.test_summer
        state: "on"
    - service: logbook.log
      data:
        name: "Test Été"
        message: "État été forcé à : on (lowercase)"

test_summer_capitalized:
  alias: "Test Été - Capitalized (On)"
  sequence:
    - service: input_boolean.turn_on
      target:
        entity_id: input_boolean.test_summer
    - delay:
        seconds: 1
    - service: developer_tools.set_state
      data:
        entity_id: input_boolean.test_summer
        state: "On"
    - service: logbook.log
      data:
        name: "Test Été"
        message: "État été forcé à : On (Capitalized)"

test_summer_uppercase:
  alias: "Test Été - UPPERCASE (ON)"
  sequence:
    - service: input_boolean.turn_on
      target:
        entity_id: input_boolean.test_summer
    - delay:
        seconds: 1
    - service: developer_tools.set_state
      data:
        entity_id: input_boolean.test_summer
        state: "ON"
    - service: logbook.log
      data:
        name: "Test Été"
        message: "État été forcé à : ON (UPPERCASE)"

test_summer_off:
  alias: "Test Été - OFF"
  sequence:
    - service: input_boolean.turn_off
      target:
        entity_id: input_boolean.test_summer
    - service: logbook.log
      data:
        name: "Test Été"
        message: "État été forcé à : off"
```

### Test 3 : Solar Optimizer (States avec différentes casses)

```yaml
test_solar_lowercase:
  alias: "Test Solar - lowercase (on)"
  sequence:
    - service: switch.turn_on
      target:
        entity_id: switch.test_solar_optimizer
    - delay:
        seconds: 1
    - service: developer_tools.set_state
      data:
        entity_id: switch.test_solar_optimizer
        state: "on"
    - service: logbook.log
      data:
        name: "Test Solar Optimizer"
        message: "État SO forcé à : on (lowercase)"

test_solar_capitalized:
  alias: "Test Solar - Capitalized (On)"
  sequence:
    - service: switch.turn_on
      target:
        entity_id: switch.test_solar_optimizer
    - delay:
        seconds: 1
    - service: developer_tools.set_state
      data:
        entity_id: switch.test_solar_optimizer
        state: "On"
    - service: logbook.log
      data:
        name: "Test Solar Optimizer"
        message: "État SO forcé à : On (Capitalized)"

test_solar_uppercase:
  alias: "Test Solar - UPPERCASE (ON)"
  sequence:
    - service: switch.turn_on
      target:
        entity_id: switch.test_solar_optimizer
    - delay:
        seconds: 1
    - service: developer_tools.set_state
      data:
        entity_id: switch.test_solar_optimizer
        state: "ON"
    - service: logbook.log
      data:
        name: "Test Solar Optimizer"
        message: "État SO forcé à : ON (UPPERCASE)"

test_solar_off:
  alias: "Test Solar - OFF"
  sequence:
    - service: switch.turn_off
      target:
        entity_id: switch.test_solar_optimizer
    - service: logbook.log
      data:
        name: "Test Solar Optimizer"
        message: "État SO forcé à : off"
```

### Test 4 : Lumière X4FP Bathroom (States avec différentes casses)

```yaml
test_light_lowercase:
  alias: "Test Lumière - lowercase (on)"
  sequence:
    - service: light.turn_on
      target:
        entity_id: light.test_bathroom
    - delay:
        seconds: 1
    - service: developer_tools.set_state
      data:
        entity_id: light.test_bathroom
        state: "on"
    - service: logbook.log
      data:
        name: "Test Lumière"
        message: "État lumière forcé à : on (lowercase)"

test_light_capitalized:
  alias: "Test Lumière - Capitalized (On)"
  sequence:
    - service: light.turn_on
      target:
        entity_id: light.test_bathroom
    - delay:
        seconds: 1
    - service: developer_tools.set_state
      data:
        entity_id: light.test_bathroom
        state: "On"
    - service: logbook.log
      data:
        name: "Test Lumière"
        message: "État lumière forcé à : On (Capitalized)"

test_light_uppercase:
  alias: "Test Lumière - UPPERCASE (ON)"
  sequence:
    - service: light.turn_on
      target:
        entity_id: light.test_bathroom
    - delay:
        seconds: 1
    - service: developer_tools.set_state
      data:
        entity_id: light.test_bathroom
        state: "ON"
    - service: logbook.log
      data:
        name: "Test Lumière"
        message: "État lumière forcé à : ON (UPPERCASE)"

test_light_off:
  alias: "Test Lumière - OFF"
  sequence:
    - service: light.turn_off
      target:
        entity_id: light.test_bathroom
    - service: logbook.log
      data:
        name: "Test Lumière"
        message: "État lumière forcé à : off"
```

## 🏗️ Entités de Test à Créer

Ajoutez ces entités dans votre `configuration.yaml` :

```yaml
# Entités de test pour les blueprints
input_text:
  test_alarm_state:
    name: "Test - État Alarme"
    initial: "disarmed"

input_boolean:
  test_summer:
    name: "Test - Été"
    initial: off

switch:
  - platform: template
    switches:
      test_solar_optimizer:
        friendly_name: "Test - Solar Optimizer"
        value_template: "{{ states('input_boolean.test_solar_state') }}"
        turn_on:
          service: input_boolean.turn_on
          target:
            entity_id: input_boolean.test_solar_state
        turn_off:
          service: input_boolean.turn_off
          target:
            entity_id: input_boolean.test_solar_state

input_boolean:
  test_solar_state:
    name: "Test - Solar State"
    initial: off

light:
  - platform: template
    lights:
      test_bathroom:
        friendly_name: "Test - Lumière Salle de Bain"
        value_template: "{{ states('input_boolean.test_light_state') }}"
        turn_on:
          service: input_boolean.turn_on
          target:
            entity_id: input_boolean.test_light_state
        turn_off:
          service: input_boolean.turn_off
          target:
            entity_id: input_boolean.test_light_state

input_boolean:
  test_light_state:
    name: "Test - Light State"
    initial: off
```

## 📊 Matrice de Test

### Blueprint: Thermostat Heat (v3.6)

| Test | État Alarme | Résultat Attendu | ✅/❌ |
|------|-------------|------------------|-------|
| 1 | `armed_away` | Mode ECO | |
| 2 | `Armed_away` | Mode ECO | |
| 3 | `ARMED_AWAY` | Mode ECO | |
| 4 | `armed_home` | Mode ECO | |
| 5 | `disarmed` | Mode CONFORT | |

### Blueprint: Room Thermostat (v2.9)

| Test | État Alarme | État Été | État Solar | Résultat Attendu | ✅/❌ |
|------|-------------|----------|------------|------------------|-------|
| 1 | `armed_away` | `off` | `off` | Preset Away | |
| 2 | `Armed_away` | `off` | `off` | Preset Away | |
| 3 | `disarmed` | `off` | `off` | Preset Home + Mode heat | |
| 4 | `disarmed` | `on` | `off` | Mode cool | |
| 5 | `disarmed` | `On` | `off` | Mode cool | |
| 6 | `disarmed` | `off` | `on` | SO actif | |
| 7 | `disarmed` | `off` | `On` | SO actif | |

### Blueprint: X4FP Bathroom (v7.16)

| Test | État Alarme | État Été | État Solar | État Lumière | Résultat Attendu | ✅/❌ |
|------|-------------|----------|------------|--------------|------------------|-------|
| 1 | `disarmed` | `off` | `off` | `on` | Mode CONFORT | |
| 2 | `disarmed` | `off` | `off` | `On` | Mode CONFORT | |
| 3 | `disarmed` | `off` | `off` | `OFF` | Mode ECO | |
| 4 | `armed_away` | `off` | `off` | `off` | Mode AWAY | |
| 5 | `Armed_away` | `off` | `off` | `off` | Mode AWAY | |
| 6 | `disarmed` | `on` | `off` | `off` | Mode été (OFF ou ECO) | |
| 7 | `disarmed` | `On` | `off` | `off` | Mode été (OFF ou ECO) | |
| 8 | `disarmed` | `off` | `on` | `off` | SO actif → CONFORT | |
| 9 | `disarmed` | `off` | `On` | `off` | SO actif → CONFORT | |

### Blueprint: X4FP Room (v7.13)

| Test | État Alarme | État Été | État Solar | Résultat Attendu | ✅/❌ |
|------|-------------|----------|------------|------------------|-------|
| 1 | `armed_away` | `off` | `off` | Mode AWAY | |
| 2 | `Armed_away` | `off` | `off` | Mode AWAY | |
| 3 | `disarmed` | `off` | `off` | Contrôle thermique | |
| 4 | `disarmed` | `on` | `off` | Mode été (OFF ou ECO) | |
| 5 | `disarmed` | `On` | `off` | Mode été (OFF ou ECO) | |
| 6 | `disarmed` | `off` | `on` | SO actif → CONFORT | |
| 7 | `disarmed` | `off` | `On` | SO actif → CONFORT | |

## 🔍 Procédure de Test Manuelle

### Méthode 1 : Utiliser les Scripts (Recommandé)

1. **Installer les scripts** ci-dessus dans Home Assistant
2. **Installer les entités de test** dans `configuration.yaml`
3. **Redémarrer** Home Assistant
4. **Activer le mode trace** sur l'automatisation à tester
5. **Exécuter un script de test** (ex: `test_alarm_capitalized`)
6. **Vérifier dans le trace** :
   - La variable `is_away` / `is_summer` / `solar_active` / `is_light_on`
   - L'action effectuée
   - Le logbook
7. **Remplir la matrice** avec ✅ ou ❌

### Méthode 2 : Test Manuel via Developer Tools

1. **Ouvrir Developer Tools** → **States**
2. **Sélectionner l'entité** (ex: `alarm_control_panel.maison`)
3. **Modifier manuellement l'état** :
   - Cliquer sur l'entité
   - Dans "State", entrer : `Armed_away` (avec majuscule)
   - Cliquer "Set State"
4. **Vérifier l'automatisation** :
   - Ouvrir **Traces**
   - Vérifier que le trigger s'est déclenché
   - Vérifier la valeur de `is_away` dans les variables
   - Vérifier l'action effectuée

### Méthode 3 : Test via Template dans Developer Tools

1. **Ouvrir Developer Tools** → **Template**
2. **Tester les détections** :

```jinja2
{# Test détection alarme avec différentes casses #}
{% set alarm_id = 'alarm_control_panel.maison' %}

{# Simuler état Armed_away #}
{% set test_state = 'Armed_away' %}
{% set st = test_state | lower %}
État original: {{ test_state }}
État avec .lower(): {{ st }}
startswith('armed'): {{ st.startswith('armed') }}

{# Simuler état armed_home #}
{% set test_state = 'armed_home' %}
{% set st = test_state | lower %}
État original: {{ test_state }}
État avec .lower(): {{ st }}
startswith('armed'): {{ st.startswith('armed') }}

{# Test détection été #}
{% set test_state = 'On' %}
{% set st = test_state | lower %}
État original: {{ test_state }}
État avec .lower(): {{ st }}
in ['on','true','open']: {{ st in ['on','true','open'] }}
```

## 📝 Rapport de Test

Une fois les tests effectués, remplissez ce rapport :

```markdown
# Rapport de Test - Blueprints v3.6 / v2.9 / v7.16 / v7.13

Date: ___________
Testeur: ___________
Version Home Assistant: ___________

## Thermostat Heat v3.6
- [ ] Alarme `armed_away` → ECO
- [ ] Alarme `Armed_away` → ECO
- [ ] Alarme `disarmed` → CONFORT

## Room Thermostat v2.9
- [ ] Alarme `Armed_away` → Preset Away
- [ ] Été `On` → Mode cool
- [ ] Solar `On` → SO actif

## X4FP Bathroom v7.16
- [ ] Lumière `On` → CONFORT
- [ ] Alarme `Armed_away` → AWAY
- [ ] Été `On` → Mode été
- [ ] Solar `On` → SO actif

## X4FP Room v7.13
- [ ] Alarme `Armed_away` → AWAY
- [ ] Été `On` → Mode été
- [ ] Solar `On` → SO actif

## Problèmes Identifiés
[Décrire ici tout problème rencontré]

## Recommandations
[Suggestions d'amélioration]
```

## 🐛 Débogage

Si un test échoue, vérifiez :

1. **Le mode trace** de l'automatisation :
   - Variables calculées
   - Conditions évaluées
   - Actions exécutées

2. **Les logs** dans le logbook :
   - Messages de l'automatisation
   - Changements d'état du climate

3. **L'état réel** de l'entité :
   - Developer Tools → States
   - Vérifier la casse exacte de l'état

4. **Le code du blueprint** :
   - Vérifier que `.lower()` est bien présent
   - Vérifier la version du blueprint (doit être v3.6, v2.9, v7.16, v7.13)

## ⚠️ Notes Importantes

- **`developer_tools.set_state`** ne fonctionne que pour forcer temporairement un état. L'entité reviendra à son état réel rapidement.
- Pour des tests durables, utilisez les **entités template** définies dans `configuration.yaml`
- Les tests doivent être effectués **un par un** avec au moins 5 secondes d'intervalle pour laisser les automatisations se déclencher
- Activez toujours le **mode trace** avant de lancer les tests

## 📞 Support

Si vous rencontrez des problèmes :
1. Vérifiez que les blueprints sont bien aux versions indiquées
2. Consultez les traces d'exécution
3. Partagez les logs dans une issue GitHub avec :
   - La version du blueprint
   - Le test qui échoue
   - La trace complète
   - L'état de l'entité au moment du test
