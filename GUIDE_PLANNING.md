# Guide d'Utilisation - Planning Horaire

## 📅 Vue d'ensemble

Le planning horaire vous permet de définir **deux presets différents selon les périodes définies dans un planning**, en utilisant une entité `schedule` de Home Assistant.

**Principe** :
- ✅ **Planning ON** (pendant vos périodes définies) → Preset Confort
- ⏸️ **Planning OFF** (hors de vos périodes) → Preset Éco

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
│ 📅 Schedule ON → Preset Confort             │
│ 📅 Schedule OFF → Preset Éco                │
└─────────────────────────────────────────────┘
```

### Un Seul Schedule, Plusieurs Périodes

Vous créez **UN SEUL** schedule dans Home Assistant qui contient **toutes vos périodes de confort**.

**Exemple** : Schedule `Chauffage Confort`
```
Lundi-Vendredi:
  06:00-08:00  ← ON (confort matin)
  17:00-22:00  ← ON (confort soirée)

Samedi-Dimanche:
  08:00-22:00  ← ON (confort journée complète)

Le reste du temps → OFF (éco)
```

Le blueprint vérifie simplement :
- **Schedule ON** → Applique `preset_schedule_on` (ex: comfort)
- **Schedule OFF** → Applique `preset_schedule_off` (ex: eco)

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

### Étape 1 : Créer le Schedule dans Home Assistant

1. Allez dans **Paramètres** → **Automatisations & Scènes** → **Helpers**
2. Cliquez sur **+ Créer un Helper**
3. Sélectionnez **Schedule**
4. Configurez votre planning :

**Exemple : Planning Semaine de Travail**
- **Nom** : `Chauffage Confort`
- **ID entité** : `schedule.chauffage_confort`
- **Configuration** :

**Lundi** :
- ✅ 06:00 - 08:00 (matin)
- ✅ 17:00 - 22:00 (soirée)

**Mardi** :
- ✅ 06:00 - 08:00
- ✅ 17:00 - 22:00

**Mercredi** :
- ✅ 06:00 - 08:00
- ✅ 17:00 - 22:00

**Jeudi** :
- ✅ 06:00 - 08:00
- ✅ 17:00 - 22:00

**Vendredi** :
- ✅ 06:00 - 08:00
- ✅ 17:00 - 22:00

**Samedi** :
- ✅ 08:00 - 22:00 (toute la journée)

**Dimanche** :
- ✅ 08:00 - 22:00 (toute la journée)

**Résultat** :
- Le schedule sera **ON** pendant : 06h-08h et 17h-22h en semaine, 08h-22h le weekend
- Le schedule sera **OFF** le reste du temps (22h-06h en semaine, 22h-08h le weekend)

### Étape 2 : Configurer l'Automatisation

1. Ouvrez votre automatisation de chauffage
2. Cliquez sur **⋮** → **Modifier**
3. Descendez jusqu'à la section **Planning horaire**
4. Configurez :

```yaml
📅 Planning Horaire (optionnel):
  - Entité : schedule.chauffage_confort

Preset quand Planning ACTIF (ON):
  - comfort

Preset quand Planning INACTIF (OFF):
  - eco
```

5. **Sauvegardez** l'automatisation

### Étape 3 : Tester

1. Activez le **mode Trace** sur l'automatisation
2. Activez/désactivez le schedule manuellement
3. Vérifiez dans **Traces** :
   - ✅ Trigger ID = `schedule_change`
   - ✅ Variable `schedule_preset` = preset attendu
   - ✅ Action `set_preset_mode` = preset appliqué
4. Consultez le **Logbook** : message `📅 Planning → COMFORT` ou `📅 Planning → ECO`

---

## 💡 Exemples de Configuration

### Exemple 1 : Planning Semaine de Travail

**Contexte** : Maison vide en journée du lundi au vendredi.

**Schedule** :
```yaml
schedule.chauffage_confort:
  Lundi-Vendredi:
    - 06:00-08:00  # Matin avant travail
    - 17:00-22:00  # Soirée après travail
  Samedi-Dimanche:
    - 08:00-22:00  # Toute la journée
```

**Configuration Blueprint** :
```yaml
schedule_entity: schedule.chauffage_confort
preset_schedule_on: comfort   # Confort pendant les périodes ON
preset_schedule_off: eco      # Éco le reste du temps
```

**Résultat** :
- **Lundi 06:00** → Schedule ON → COMFORT (réveil)
- **Lundi 08:00** → Schedule OFF → ECO (départ travail)
- **Lundi 17:00** → Schedule ON → COMFORT (retour maison)
- **Lundi 22:00** → Schedule OFF → ECO (sommeil)
- **Samedi 08:00** → Schedule ON → COMFORT
- **Samedi 22:00** → Schedule OFF → ECO

---

### Exemple 2 : Télétravail Certains Jours

**Contexte** : Télétravail mercredi et vendredi.

**Schedule** :
```yaml
schedule.chauffage_confort:
  Lundi-Mardi-Jeudi:
    - 06:00-08:00
    - 17:00-22:00
  Mercredi-Vendredi:  # Jours de télétravail
    - 06:00-22:00  # Confort toute la journée
  Samedi-Dimanche:
    - 08:00-22:00
```

**Configuration Blueprint** :
```yaml
schedule_entity: schedule.chauffage_confort
preset_schedule_on: comfort
preset_schedule_off: eco
```

---

### Exemple 3 : Économies Nocturnes Importantes

**Contexte** : Réduire fortement le chauffage la nuit.

**Schedule** :
```yaml
schedule.chauffage_confort:
  Tous les jours:
    - 06:00-23:00  # Journée complète
```

**Configuration Blueprint** :
```yaml
schedule_entity: schedule.chauffage_confort
preset_schedule_on: comfort
preset_schedule_off: frost_protection  # Hors-gel la nuit (23h-06h)
```

---

### Exemple 4 : Weekend Grasse Matinée

**Contexte** : Se lever plus tard le weekend.

**Schedule** :
```yaml
schedule.chauffage_confort:
  Lundi-Vendredi:
    - 06:00-08:00
    - 17:00-22:00
  Samedi-Dimanche:
    - 09:00-23:00  # Lever plus tard
```

**Configuration Blueprint** :
```yaml
schedule_entity: schedule.chauffage_confort
preset_schedule_on: comfort
preset_schedule_off: eco
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
schedule_entity: schedule.chauffage_confort
preset_schedule_on: comfort
preset_schedule_off: eco
```

**Alternative hors-gel nocturne** :
```yaml
schedule_entity: schedule.chauffage_jour  # ON uniquement 06h-23h
preset_schedule_on: comfort
preset_schedule_off: frost_protection  # Hors-gel la nuit
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
schedule_entity: schedule.chauffage_confort
preset_schedule_on: comfort
preset_schedule_off: eco
```

**Alternative avec preset home** :
```yaml
schedule_entity: schedule.chauffage_confort
preset_schedule_on: home     # Mode présence pendant périodes actives
preset_schedule_off: eco
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
schedule_entity: schedule.salle_de_bain_confort
preset_schedule_on: comfort   # Chauffe selon planning (ignore lumière)
preset_schedule_off: eco      # Éco hors planning (ignore lumière)
```

**Usage typique** : Créer un schedule avec les périodes de douche (matin 06h-08h, soir 18h-20h) pour garantir une salle de bain chaude sans dépendre de la lumière.

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
schedule_entity: schedule.chambre_confort
preset_schedule_on: comfort   # Force comfort selon planning
preset_schedule_off: eco      # Force éco hors planning
```

**Usage typique** : Définir des périodes de confort garanti (matin, soirée) sans dépendre du contrôle thermique automatique.

---

## ✅ Vérification et Tests

### Checklist Post-Configuration

- [ ] Schedule créé dans Home Assistant avec toutes les périodes
- [ ] Schedule configuré dans l'automatisation
- [ ] Preset ON et OFF configurés
- [ ] Mode Trace activé
- [ ] Test manuel : activer/désactiver le schedule
- [ ] Vérifier trace : trigger `schedule_change`
- [ ] Vérifier logbook : message `📅 Planning → COMFORT` ou `📅 Planning → ECO`
- [ ] Test avec alarme armée : planning ignoré
- [ ] Test avec alarme désarmée : planning actif
- [ ] Validation sur 24h complètes

### Commandes de Test

**Vérifier l'état du schedule** :
```yaml
État : {{ states('schedule.chauffage_confort') }}
```

**Forcer l'activation du schedule** :
```yaml
service: schedule.turn_on
target:
  entity_id: schedule.chauffage_confort
```

**Désactiver le schedule** :
```yaml
service: schedule.turn_off
target:
  entity_id: schedule.chauffage_confort
```

**Vérifier le preset actuel** :
```yaml
Preset : {{ state_attr('climate.xxx', 'preset_mode') }}
```

---

## 🐛 Dépannage

### Problème : Le planning ne se déclenche pas

**Diagnostic** :
1. Vérifiez que l'alarme est **désarmée**
   - Le planning est ignoré si alarme armée
2. Vérifiez l'état du schedule :
   - Outils de développement → États → `schedule.xxx`
   - État doit être `on` pendant la période active, `off` en dehors
3. Vérifiez les **Traces** :
   - Automatisation → Traces
   - Cherchez `trigger.id = schedule_change`
4. Vérifiez la variable `schedule_preset` dans la trace
   - Doit être égal à `preset_schedule_on` ou `preset_schedule_off`

**Solutions** :
- Rechargez les automatisations : Outils dev → YAML → Rechargement
- Vérifiez que le schedule est bien configuré avec les bonnes heures
- Assurez-vous que l'entité schedule est bien sélectionnée dans le blueprint
- Testez manuellement en activant/désactivant le schedule

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
3. Vérifiez dans la trace que le preset demandé est bien dans `preset_modes`

**Solutions** :
- Si le preset n'est pas supporté, le blueprint utilisera les **températures fallback** (Thermostat Heat uniquement)
- Choisissez un preset supporté par votre thermostat
- Vérifiez le message logbook : doit indiquer le fallback si preset absent

---

### Problème : Le planning est actif même quand l'alarme est armée

**Diagnostic** :
C'est impossible par design ! Le code vérifie toujours :
```yaml
schedule_preset: >-
  {% if not is_away and schedule_id and schedule_id != '' %}
    ...
  {% else %}
    none
  {% endif %}
```

**Vérifications** :
1. Assurez-vous d'avoir la **bonne version** du blueprint (v3.7, v2.10, v7.17, v7.14)
2. Rechargez les automatisations
3. Vérifiez la trace : `is_away` doit être `false` pour que le planning soit actif
4. Consultez le logbook : doit montrer preset alarme si alarme armée

---

### Problème : Le schedule change d'état mais le preset ne suit pas

**Diagnostic** :
1. Vérifiez que le trigger `schedule_change` se déclenche dans les traces
2. Vérifiez la priorité : une condition supérieure peut être active
   - Été actif → OFF prioritaire
   - Fenêtre ouverte → OFF prioritaire
   - Solar Optimizer → COMFORT prioritaire
3. Consultez les messages logbook pour voir quelle condition est active

**Solutions** :
- Si une condition prioritaire est active, c'est normal
- Attendez que la condition prioritaire se termine
- Vérifiez l'ordre de priorité dans la documentation

---

## 📊 Exemple Complet

Voici une configuration complète pour une maison avec travail en semaine :

### Schedule à créer

```yaml
Nom: Chauffage Confort Maison
ID: schedule.chauffage_confort

Configuration:
  Lundi:
    - 06:00-08:00
    - 17:00-22:00
  Mardi:
    - 06:00-08:00
    - 17:00-22:00
  Mercredi:
    - 06:00-08:00
    - 17:00-22:00
  Jeudi:
    - 06:00-08:00
    - 17:00-22:00
  Vendredi:
    - 06:00-08:00
    - 17:00-22:00
  Samedi:
    - 08:00-22:00
  Dimanche:
    - 08:00-22:00
```

### Configuration Blueprint (Thermostat Heat v3.7)

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

# ⭐ Planning horaire
schedule_entity: schedule.chauffage_confort
preset_schedule_on: comfort   # Pendant les périodes définies
preset_schedule_off: eco      # En dehors des périodes

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

| Jour | Heure | Alarme | Schedule | Preset Final | Raison |
|------|-------|--------|----------|--------------|--------|
| Lundi | 06:00 | Off | **ON** | **comfort** | Planning ON |
| Lundi | 08:00 | Off | **OFF** | **eco** | Planning OFF |
| Lundi | 09:00 | **On** | OFF | **eco** | Alarme (ignore planning) |
| Lundi | 17:00 | **On** | ON | **eco** | Alarme prioritaire |
| Lundi | 18:00 | Off | **ON** | **comfort** | Planning ON |
| Lundi | 22:00 | Off | **OFF** | **eco** | Planning OFF |
| Samedi | 08:00 | Off | **ON** | **comfort** | Planning ON |
| Samedi | 22:00 | Off | **OFF** | **eco** | Planning OFF |

---

## 💡 Astuces et Bonnes Pratiques

### 1. Nommer Clairement le Schedule

✅ **Bon** :
- `schedule.chauffage_confort`
- `schedule.chauffage_maison`
- `schedule.periodes_presence`

❌ **Mauvais** :
- `schedule.schedule_1`
- `schedule.test`
- `schedule.temp`

### 2. Définir Toutes les Périodes dans UN Schedule

**Principe** : Un schedule peut contenir plusieurs plages horaires par jour.

**Exemple** : Pour avoir confort matin ET soir
```
Lundi:
  - 06:00-08:00  ← Première plage
  - 17:00-22:00  ← Deuxième plage
```

Pas besoin de créer deux schedules séparés !

### 3. Utiliser les Preset Modes Existants

Vérifiez les presets supportés par votre thermostat avant de configurer :
```yaml
# Outils de développement → États
climate.xxx:
  preset_modes: [eco, comfort, boost]
```

Utilisez SEULEMENT ces presets dans le blueprint !

### 4. Tester Progressivement

1. Créez d'abord un schedule simple (ex: 06h-22h tous les jours)
2. Testez 24h avec `preset_schedule_on: comfort` et `preset_schedule_off: eco`
3. Affinez ensuite les plages horaires
4. Validez chaque modification

### 5. Surveiller les Logs

Consultez régulièrement le **Logbook** :
```
📅 Planning → COMFORT (06:00)
📅 Planning → ECO (08:00)
🔒 Alarme armée → ECO (09:00)
📅 Planning → COMFORT (17:00)
```

### 6. Comprendre la Différence avec le Preset par Défaut

Sans planning configuré :
```
Alarme OFF → preset_when_disarmed (ex: comfort permanent)
```

Avec planning configuré :
```
Alarme OFF + Schedule ON → preset_schedule_on (ex: comfort)
Alarme OFF + Schedule OFF → preset_schedule_off (ex: eco)
```

Le planning permet donc d'alterner automatiquement entre deux presets selon les périodes définies.

---

## 📚 Ressources

- [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md) : Guide de migration depuis anciennes versions
- [DIAGNOSTIC_ALARME.md](DIAGNOSTIC_ALARME.md) : Diagnostic problèmes alarme
- [CHECKLIST_TESTS.md](CHECKLIST_TESTS.md) : Checklist de tests
- [CHANGELOG.md](CHANGELOG.md) : Historique des versions

---

**Bon chauffage intelligent ! 🔥**
