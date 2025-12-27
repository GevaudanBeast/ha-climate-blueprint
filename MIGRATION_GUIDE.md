# Guide de Migration - Blueprints HVAC v3.6 / v2.9 / v7.16 / v7.13

## 🎯 Pourquoi recréer les automatisations ?

Les anciennes configurations contenaient des paramètres obsolètes qui n'existent plus dans les nouvelles versions :
- ❌ `eco_flag_entity` (supprimé)
- ❌ `solar_enable` (supprimé)
- ❌ `solar_behavior` (supprimé)
- ❌ `solar_setpoint` (supprimé)

**Les nouvelles versions utilisent** :
- ✅ Détection automatique des attributs Solar Optimizer
- ✅ Configuration simplifiée
- ✅ Détection robuste avec `.lower()` (insensible à la casse)

## 📋 Procédure de Migration

### Étape 1 : Sauvegarder vos configurations actuelles

1. Allez dans **Paramètres** → **Automatisations & Scènes**
2. Pour chaque automatisation concernée :
   - Cliquez sur les **3 points** → **Modifier en YAML**
   - **Copiez** tout le contenu dans un fichier texte (backup)
   - Notez les paramètres importants :
     - `climate_entity`: l'entité climate
     - `room_name`: le nom de la pièce
     - `alarm_entity`: l'entité alarme
     - `summer_entity`: l'entité mode été
     - `window_sensors`: les capteurs de fenêtres
     - Températures configurées
     - Presets configurés

### Étape 2 : Supprimer les anciennes automatisations

1. Pour chaque automatisation :
   - Cliquez sur les **3 points** → **Supprimer**
   - Confirmez la suppression

### Étape 3 : Importer les nouveaux blueprints (si pas déjà fait)

Vérifiez que vous avez bien les dernières versions :

1. **Paramètres** → **Automatisations & Scènes** → **Blueprints**
2. Vérifiez les versions :
   - Thermostat Heat : **v3.6**
   - Room Thermostat : **v2.9**
   - X4FP Bathroom : **v7.16**
   - X4FP Room : **v7.13**

Si vous avez des versions plus anciennes :
1. Supprimez l'ancien blueprint (3 points → Supprimer)
2. Cliquez sur **Importer un Blueprint**
3. Collez l'URL correspondante (voir README.md)

### Étape 4 : Créer les nouvelles automatisations

Pour chaque automatisation à recréer :

1. Allez dans **Paramètres** → **Automatisations & Scènes**
2. Cliquez sur **Créer une automatisation**
3. Choisissez le blueprint approprié
4. Configurez les paramètres (utilisez votre backup comme référence)

---

## 🔧 Configurations Recommandées par Blueprint

### 1. Thermostat Heat v3.6

```yaml
# Paramètres essentiels
room_name: "Nom de la pièce"
climate_entity: climate.votre_thermostat

# Fenêtres (optionnel)
window_sensors:
  - binary_sensor.fenetre_1
delay_open_min: 2
delay_close_min: 2

# Alarme
alarm_entity: alarm_control_panel.votre_alarme
preset_when_armed: eco        # Preset quand alarme ARMÉE
preset_when_disarmed: comfort # Preset quand alarme DÉSARMÉE

# Été (optionnel)
summer_entity: input_boolean.ete
summer_behavior: off  # ou eco

# Températures fallback
comfort_temp: 21
eco_temp: 18.5
frost_temp: 7
boost_temp: 23

# Tick
tick_minutes: "10"  # ou "0" pour désactiver
```

### 2. Room Thermostat v2.9

```yaml
# Paramètres essentiels
room_name: "Nom de la pièce"
climate_entity: climate.votre_clim

# Fenêtres (optionnel)
window_sensors:
  - binary_sensor.fenetre_1
delay_open_min: 2
delay_close_min: 2

# Alarme
alarm_entity: alarm_control_panel.votre_alarme
preset_when_armed: away       # ou none si pas de preset away
preset_when_disarmed: none    # ou home si preset home existe

# Été/Hiver
summer_entity: input_boolean.ete
temp_cool_summer: 24  # Température en mode COOL (été)
temp_heat_winter: 20  # Température en mode HEAT (hiver)

# Solar Optimizer (optionnel)
solar_entity: switch.solar_optimizer

# Tick
tick_minutes: "10"
```

### 3. X4FP Bathroom v7.16

```yaml
# Paramètres essentiels
room_name: "Salle de Bain"
climate_entity: climate.x4fp_sdb

# Lumière
light_entity: light.sdb
light_mode: toggle  # ou preset

# Fenêtres (optionnel)
window_sensors:
  - binary_sensor.fenetre_sdb

# Alarme
alarm_entity: alarm_control_panel.votre_alarme

# Été
summer_entity: input_boolean.ete
summer_behavior: off  # ou eco

# Solar Optimizer (optionnel)
solar_entity: switch.solar_optimizer
allow_solar_when_away: true  # Solar Optimizer actif même en Away

# Tick
tick_minutes: "10"
```

### 4. X4FP Room v7.13

```yaml
# Paramètres essentiels
room_name: "Nom de la pièce"
climate_entity: climate.x4fp_piece

# Contrôle thermique
temp_sensor: sensor.temperature_piece
setpoint_entity: input_number.consigne_piece
hysteresis: 0.3
temp_min: 16
temp_max: 25

# Fenêtres (optionnel)
window_sensors:
  - binary_sensor.fenetre_1

# Alarme
alarm_entity: alarm_control_panel.votre_alarme

# Été
summer_entity: input_boolean.ete
summer_behavior: off

# Solar Optimizer (optionnel)
solar_entity: switch.solar_optimizer
allow_solar_when_away: true

# Tick
tick_minutes: "10"
```

---

## ✅ Vérification Post-Migration

Après avoir recréé chaque automatisation :

### 1. Vérifier la configuration YAML

1. Ouvrez l'automatisation → **3 points** → **Modifier en YAML**
2. Vérifiez qu'il n'y a **AUCUN** de ces anciens paramètres :
   - `eco_flag_entity`
   - `solar_enable`
   - `solar_behavior` (sauf pour X4FP)
   - `solar_setpoint`

### 2. Tester avec la Checklist

Utilisez **CHECKLIST_TESTS.md** pour tester chaque automatisation :

```bash
# Exemple de test pour Thermostat Heat
1. Activer le mode Trace sur l'automatisation
2. Armer l'alarme (armed_away)
3. Attendre 5 secondes
4. Vérifier :
   ✓ L'automatisation s'est déclenchée
   ✓ Variable is_away = true
   ✓ Action set_preset_mode = eco
   ✓ Le thermostat affiche preset = eco
   ✓ Message logbook : "🔒 Away → ECO"
```

### 3. Vérifier les Traces

Pour chaque test :
1. Ouvrez **Traces** de l'automatisation
2. Vérifiez :
   - ✅ Trigger ID correspond à l'action (alarm_change, season_change, etc.)
   - ✅ Variables calculées correctement
   - ✅ Action exécutée
   - ✅ Pas d'erreur

---

## 🐛 Résolution de Problèmes

### Problème : "L'automatisation ne se déclenche pas"

**Solution** :
1. Vérifiez que l'entité alarme existe : **Outils de développement** → **États**
2. Vérifiez l'état exact (doit être `armed_away`, `armed_home`, ou `disarmed`)
3. Activez le mode **Trace** et provoquez un changement d'état

### Problème : "Le preset ne change pas"

**Solution** :
1. Vérifiez que votre climate supporte le preset :
   ```yaml
   climate.votre_thermostat:
     preset_modes: [eco, comfort, ...]  # doit contenir le preset
   ```
2. Si pas de support preset, le blueprint utilisera les températures fallback
3. Vérifiez les logs : **Paramètres** → **Système** → **Logs**

### Problème : "Comportement erratique"

**Solution** :
1. Rechargez les automatisations : **Outils de développement** → **YAML** → **Rechargement des automatisations**
2. Si ça persiste : **Redémarrez Home Assistant**
3. Vérifiez qu'il n'y a pas de **double automatisation** sur le même climate

---

## 📊 Checklist de Migration Complète

- [ ] Sauvegarde de toutes les configurations actuelles
- [ ] Suppression des anciennes automatisations
- [ ] Import des nouveaux blueprints (v3.6, v2.9, v7.16, v7.13)
- [ ] Recréation automatisation Thermostat Heat
- [ ] Recréation automatisation Room Thermostat
- [ ] Recréation automatisation X4FP Bathroom
- [ ] Recréation automatisation X4FP Room
- [ ] Vérification YAML (pas de paramètres obsolètes)
- [ ] Rechargement des automatisations
- [ ] Tests avec CHECKLIST_TESTS.md
- [ ] Vérification traces et logbook
- [ ] Validation complète sur 24h

---

## 💡 Conseils

1. **Migrez une automatisation à la fois** : testez avant de passer à la suivante
2. **Gardez vos backups** pendant quelques jours
3. **Utilisez le mode Trace** systématiquement pendant la phase de test
4. **Consultez le logbook** pour voir les actions effectuées en temps réel

---

## 📞 Besoin d'aide ?

Si vous rencontrez des problèmes après la migration :
1. Vérifiez les **Traces** de l'automatisation
2. Consultez les **Logs** système
3. Ouvrez une [issue GitHub](https://github.com/GevaudanBeast/ha-climate-blueprint/issues)
