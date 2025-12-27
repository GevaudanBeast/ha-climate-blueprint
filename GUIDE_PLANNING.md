# Guide d'Utilisation - Planning Horaire

## 📅 Vue d'ensemble

Le planning horaire vous permet de définir des **presets différents selon l'heure de la journée**, en utilisant les entités `schedule` de Home Assistant.

**Disponible dans tous les blueprints** :
- ✅ Thermostat Heat v3.7
- ✅ Room Thermostat v2.10
- ✅ X4FP Bathroom v7.17
- ✅ X4FP Room v7.14

---

## 🎯 Principe de Fonctionnement

### Logique Générale

Le planning horaire est **actif uniquement si l'alarme est désarmée** (présence à la maison).

```
┌─────────────────────────────────────────────┐
│ Alarme ARMÉE (absent)                       │
│ ❌ Planning ignoré                          │
│ ✅ Utilise preset_when_armed (eco/away)    │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│ Alarme DÉSARMÉE (présent)                   │
│ ✅ Planning actif si configuré              │
│ 📅 Matin 06h-08h → Comfort                  │
│ 📅 Journée 08h-17h → Eco                    │
│ 📅 Soirée 17h-22h → Comfort                 │
│ 📅 Nuit 22h-06h → Eco                       │
└─────────────────────────────────────────────┘
```

### Ordre de Priorité

**Tous les blueprints** :
1. ☀️ **Été** → OFF (ou ECO selon configuration)
2. 🪟 **Fenêtre ouverte** → OFF (ou preset fenêtre)
3. 🌞 **Solar Optimizer** → COMFORT (si actif)
4. 🔒 **Alarme ARMÉE** → preset alarme (ignore planning)
5. ⭐ **Planning actif** → preset du planning (si alarme désarmée)
6. 🏠 **Comportement par défaut** → preset désarmée OU lumière/thermique

---

## 📋 Configuration Étape par Étape

### Étape 1 : Créer les Schedules dans Home Assistant

1. Allez dans **Paramètres** → **Automatisations & Scènes** → **Helpers**
2. Cliquez sur **+ Créer un Helper**
3. Sélectionnez **Schedule**
4. Configurez votre planning :

**Exemple : Planning Matin**
- **Nom** : `Chauffage Matin`
- **ID entité** : `schedule.chauffage_matin`
- **Configuration** :
  - ✅ Lundi : 06:00 - 08:00
  - ✅ Mardi : 06:00 - 08:00
  - ✅ Mercredi : 06:00 - 08:00
  - ✅ Jeudi : 06:00 - 08:00
  - ✅ Vendredi : 06:00 - 08:00
  - ⬜ Samedi
  - ⬜ Dimanche

**Répétez** pour les autres périodes :
- `schedule.chauffage_journee` : Lun-Ven 08:00-17:00
- `schedule.chauffage_soiree` : Tous les jours 17:00-22:00
- `schedule.chauffage_nuit` : Tous les jours 22:00-06:00

### Étape 2 : Configurer l'Automatisation

1. Ouvrez votre automatisation de chauffage
2. Cliquez sur **⋮** → **Modifier**
3. Descendez jusqu'à la section **Planning horaire**
4. Configurez :

```yaml
📅 Planning Matin (optionnel):
  - Entité : schedule.chauffage_matin
  - Preset Matin : comfort

📅 Planning Journée (optionnel):
  - Entité : schedule.chauffage_journee
  - Preset Journée : eco

📅 Planning Soirée (optionnel):
  - Entité : schedule.chauffage_soiree
  - Preset Soirée : comfort

📅 Planning Nuit (optionnel):
  - Entité : schedule.chauffage_nuit
  - Preset Nuit : eco
```

5. **Sauvegardez** l'automatisation

### Étape 3 : Tester

1. Activez le **mode Trace** sur l'automatisation
2. Activez/désactivez un schedule manuellement
3. Vérifiez dans **Traces** :
   - ✅ Trigger ID = `schedule_change`
   - ✅ Variable `schedule_preset` = preset attendu
   - ✅ Action `set_preset_mode` = preset appliqué
4. Consultez le **Logbook** : message `📅 Planning → PRESET`

---

## 💡 Exemples de Configuration

### Exemple 1 : Planning Semaine de Travail

**Contexte** : Maison vide en journée du lundi au vendredi.

**Schedules** :
```yaml
schedule.chauffage_matin:
  Lun-Ven: 06:00-08:00

schedule.chauffage_journee:
  Lun-Ven: 08:00-17:00

schedule.chauffage_soiree:
  Tous les jours: 17:00-22:00

schedule.chauffage_nuit:
  Tous les jours: 22:00-06:00
```

**Configuration Blueprint** :
```yaml
morning_preset: comfort    # Réveil confortable
day_preset: eco           # Économie pendant travail
evening_preset: comfort   # Retour à la maison
night_preset: eco         # Nuit économique
```

**Résultat** :
- 06:00 → COMFORT (réveil)
- 08:00 → ECO (départ travail)
- 17:00 → COMFORT (retour maison)
- 22:00 → ECO (sommeil)

---

### Exemple 2 : Weekend Différent

**Contexte** : Grasse matinée le weekend.

**Schedules** :
```yaml
schedule.chauffage_matin_semaine:
  Lun-Ven: 06:00-08:00

schedule.chauffage_matin_weekend:
  Sam-Dim: 08:00-10:00

schedule.chauffage_journee:
  Sam-Dim: 10:00-22:00  # Journée entière
  Lun-Ven: 17:00-22:00  # Seulement soirée
```

**Configuration Blueprint** :
```yaml
morning_preset: comfort
day_preset: comfort      # Confort toute la journée weekend
evening_preset: comfort
night_preset: eco
```

---

### Exemple 3 : Télétravail

**Contexte** : Télétravail certains jours.

**Schedules** :
```yaml
schedule.chauffage_teletravail:
  Mercredi: 08:00-17:00
  Vendredi: 08:00-17:00

schedule.chauffage_standard:
  Lun-Mar-Jeu: 08:00-17:00
```

**Configuration Blueprint** :
```yaml
# Utiliser schedule.chauffage_teletravail pour day_preset
day_preset: comfort  # Confort en télétravail
```

---

### Exemple 4 : Chauffage Économique la Nuit

**Contexte** : Réduire le chauffage fortement la nuit.

**Schedules** :
```yaml
schedule.chauffage_nuit_profonde:
  Tous les jours: 23:00-05:00
```

**Configuration Blueprint** :
```yaml
night_preset: frost_protection  # Hors-gel seulement
# OU
night_preset: eco  # Eco standard
```

---

## 🔧 Configuration par Blueprint

### Thermostat Heat v3.7

**Presets disponibles** :
- `eco` : Mode économique
- `comfort` : Mode confort
- `comfort-1` : Confort -1°C
- `comfort-2` : Confort -2°C
- `frost_protection` : Hors-gel
- `boost` : Mode boost
- `none` : Aucun preset

**Exemple** :
```yaml
morning_preset: comfort
day_preset: eco
evening_preset: comfort
night_preset: frost_protection  # Hors-gel la nuit
```

---

### Room Thermostat v2.10

**Presets disponibles** :
- `none` : Aucun preset
- `eco` : Mode économique
- `comfort` : Mode confort
- `home` : Mode présence
- `away` : Mode absence
- `boost` : Mode boost

**Exemple** :
```yaml
morning_preset: comfort
day_preset: eco
evening_preset: home    # Mode home en soirée
night_preset: eco
```

---

### X4FP Bathroom v7.17

**Spécificité** : Le planning **remplace** la gestion par lumière si actif.

**Presets disponibles** :
- `eco` : Mode économique
- `comfort` : Mode confort
- `comfort-1` / `comfort-2`
- `away` : Mode absence
- `boost` : Mode boost

**Priorité** :
1. Alarme armée → `preset_away`
2. ⭐ **Planning actif** → preset du planning (ignore la lumière)
3. Lumière ON/OFF → `preset_heat` / `preset_idle`

**Exemple** :
```yaml
morning_preset: comfort   # Chauffe le matin (ignore lumière)
day_preset: eco          # Eco en journée (ignore lumière)
evening_preset: comfort  # Confort soirée (ignore lumière)
night_preset: eco        # Eco nuit (ignore lumière)
```

---

### X4FP Room v7.14

**Spécificité** : Le planning **remplace** le contrôle thermique si actif.

**Presets disponibles** :
- `eco` : Mode économique
- `comfort` : Mode confort
- `comfort-1` / `comfort-2`
- `away` : Mode absence
- `boost` : Mode boost

**Priorité** :
1. Alarme armée → `preset_away`
2. ⭐ **Planning actif** → preset du planning (ignore thermique)
3. Contrôle thermique → `preset_heat` / `preset_idle` selon température

**Exemple** :
```yaml
morning_preset: comfort   # Force comfort le matin
day_preset: eco          # Force eco en journée
evening_preset: comfort  # Force comfort en soirée
night_preset: eco        # Force eco la nuit
```

---

## ✅ Vérification et Tests

### Checklist Post-Configuration

- [ ] Schedules créés dans Home Assistant
- [ ] Schedules configurés dans l'automatisation
- [ ] Presets configurés pour chaque période
- [ ] Mode Trace activé
- [ ] Test manuel : activer/désactiver un schedule
- [ ] Vérifier trace : trigger `schedule_change`
- [ ] Vérifier logbook : message `📅 Planning → PRESET`
- [ ] Test avec alarme armée : planning ignoré
- [ ] Test avec alarme désarmée : planning actif
- [ ] Validation sur 24h complètes

### Commandes de Test

**Vérifier l'état d'un schedule** :
```yaml
État : {{ states('schedule.chauffage_matin') }}
```

**Forcer l'activation d'un schedule** :
```yaml
service: schedule.turn_on
target:
  entity_id: schedule.chauffage_matin
```

**Désactiver un schedule** :
```yaml
service: schedule.turn_off
target:
  entity_id: schedule.chauffage_matin
```

---

## 🐛 Dépannage

### Problème : Le planning ne se déclenche pas

**Diagnostic** :
1. Vérifiez que l'alarme est **désarmée**
   - Le planning est ignoré si alarme armée
2. Vérifiez l'état du schedule :
   - Outils de développement → États → `schedule.xxx`
   - État doit être `on` pendant la période active
3. Vérifiez les **Traces** :
   - Automatisation → Traces
   - Cherchez `trigger.id = schedule_change`
4. Vérifiez la variable `schedule_preset` dans la trace

**Solutions** :
- Rechargez les automatisations : Outils dev → YAML → Rechargement
- Vérifiez que le schedule est bien configuré avec les bonnes heures
- Assurez-vous que l'entité schedule est bien sélectionnée dans le blueprint

---

### Problème : Le preset ne change pas

**Diagnostic** :
1. Vérifiez que le preset existe sur votre thermostat :
   ```yaml
   climate.xxx:
     preset_modes: [eco, comfort, ...]
   ```
2. Consultez les **Logs** système :
   - Paramètres → Système → Logs
   - Cherchez erreurs `climate.set_preset_mode`

**Solutions** :
- Si le preset n'est pas supporté, le blueprint utilisera les **températures fallback**
- Vérifiez le message logbook : doit indiquer le fallback si preset absent

---

### Problème : Le planning est actif même quand l'alarme est armée

**Diagnostic** :
C'est impossible par design ! Le code vérifie toujours :
```yaml
schedule_preset: >-
  {% if not is_away %}  ← Vérifie alarme désarmée
    ...
  {% else %}
    none
  {% endif %}
```

**Vérifications** :
1. Assurez-vous d'avoir la **bonne version** du blueprint (v3.7, v2.10, v7.17, v7.14)
2. Rechargez les automatisations
3. Vérifiez la trace : `is_away` doit être `false` pour que le planning soit actif

---

## 📊 Exemple Complet

Voici une configuration complète pour une maison avec travail en semaine :

### Schedules à créer

```yaml
schedule.chauffage_matin:
  Lundi-Vendredi: 06:00-08:00
  Samedi-Dimanche: 08:00-10:00

schedule.chauffage_journee:
  Lundi-Vendredi: 08:00-17:00
  Samedi-Dimanche: 10:00-22:00

schedule.chauffage_soiree:
  Lundi-Vendredi: 17:00-22:00

schedule.chauffage_nuit:
  Tous les jours: 22:00-06:00
```

### Configuration Blueprint

```yaml
name: Thermostat Salon Planning
description: Chauffage salon avec planning semaine/weekend

blueprint: Thermostat Heat v3.7

# Entités de base
room_name: Salon
climate_entity: climate.thermostat_salon

# Alarme
alarm_entity: alarm_control_panel.alarme
preset_when_armed: eco
preset_when_disarmed: comfort

# Planning horaire
schedule_morning: schedule.chauffage_matin
morning_preset: comfort

schedule_day: schedule.chauffage_journee
day_preset: eco

schedule_evening: schedule.chauffage_soiree
evening_preset: comfort

schedule_night: schedule.chauffage_nuit
night_preset: eco

# Fenêtres
window_sensors:
  - binary_sensor.fenetre_salon
delay_open_min: 2
delay_close_min: 2

# Été
summer_entity: input_boolean.ete
summer_behavior: off_in_summer

# Tick
tick_minutes: "10"
```

### Comportement Attendu

| Jour | Heure | Alarme | Schedule Actif | Preset Final |
|------|-------|--------|----------------|--------------|
| Lundi | 06:00 | Off | matin | **comfort** |
| Lundi | 08:00 | Off | journée | **eco** |
| Lundi | 09:00 | **On** | journée (ignoré) | **eco** (alarme) |
| Lundi | 17:00 | **On** | soirée (ignoré) | **eco** (alarme) |
| Lundi | 18:00 | Off | soirée | **comfort** |
| Lundi | 22:00 | Off | nuit | **eco** |
| Samedi | 08:00 | Off | matin | **comfort** |
| Samedi | 10:00 | Off | journée | **eco** |
| Samedi | 22:00 | Off | nuit | **eco** |

---

## 💡 Astuces et Bonnes Pratiques

### 1. Nommer Clairement les Schedules

✅ **Bon** :
- `schedule.chauffage_matin`
- `schedule.chauffage_weekend_matin`
- `schedule.chauffage_teletravail`

❌ **Mauvais** :
- `schedule.schedule_1`
- `schedule.test`
- `schedule.temp`

### 2. Grouper par Logique

Créez des schedules différents pour différentes logiques :
- **Semaine/Weekend** : 2 schedules matin (semaine vs weekend)
- **Télétravail** : Schedule spécifique jours télétravail
- **Vacances** : Schedule vacances scolaires

### 3. Utiliser les Preset Modes Existants

Vérifiez les presets supportés par votre thermostat avant de configurer :
```yaml
# Outils de développement → États
climate.xxx:
  preset_modes: [eco, comfort, boost]
```

Utilisez SEULEMENT ces presets dans le blueprint !

### 4. Tester Progressivement

1. Configurez **un seul** schedule d'abord (matin)
2. Testez 24h
3. Ajoutez les autres progressivement
4. Validez chaque ajout

### 5. Surveiller les Logs

Consultez régulièrement le **Logbook** :
```
📅 Planning → COMFORT (06:00)
📅 Planning → ECO (08:00)
🔒 Alarme armée → ECO (09:00)
```

---

## 📚 Ressources

- [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md) : Guide de migration depuis anciennes versions
- [DIAGNOSTIC_ALARME.md](DIAGNOSTIC_ALARME.md) : Diagnostic problèmes alarme
- [CHECKLIST_TESTS.md](CHECKLIST_TESTS.md) : Checklist de tests
- [CHANGELOG.md](CHANGELOG.md) : Historique des versions

---

**Bon chauffage intelligent ! 🔥**
