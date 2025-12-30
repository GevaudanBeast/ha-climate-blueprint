# 🔍 Diagnostic: Problème de Priorité Alarme

## ❌ Problème Rapporté

Les thermostats restent en mode CONFORT même quand l'alarme est armée.

**Comportement attendu**: Alarme armée → Thermostat passe en ECO (ou preset configuré)
**Comportement observé**: Alarme armée → Thermostat reste en CONFORT

---

## 🎯 Diagnostic du Problème

### Analyse de la Trace

La trace montre:
```json
"is_away": false
```

Cela signifie que la logique de détection d'alarme a évalué l'alarme comme **PAS ARMÉE**.

### Logique de Détection

```yaml
is_away: >-
  {% if alarm_id and alarm_id != '' %}
    {% set st = states(alarm_id) | lower %}
    {{ st.startswith('armed') }}
  {% else %}
    false
  {% endif %}
```

Cette logique retourne `true` uniquement si l'état de l'alarme commence par "armed" :
- ✅ `armed_away` → détecté
- ✅ `armed_home` → détecté
- ✅ `armed_night` → détecté
- ❌ `arming` → **PAS** détecté (état transitoire)
- ❌ `disarmed` → **PAS** détecté
- ❌ `pending` → **PAS** détecté

---

## ✅ Solution: Checklist de Vérification

### 1. **CRITIQUE: Recharger les Automatisations** ⚠️

Après avoir mis à jour les blueprints, vous **DEVEZ** recharger les automatisations dans Home Assistant.

#### Comment Recharger les Automatisations:

**Option A: Via l'Interface**
1. Ouvrir **Outils de développement** → **YAML**
2. Cliquer sur **"Recharger Automatisations"**

**Option B: Via Service**
1. Outils de développement → Services
2. Service: `automation.reload`
3. Cliquer "APPELER LE SERVICE"

**Option C: Redémarrer Home Assistant**
1. Paramètres → Système → Redémarrer

⚠️ **IMPORTANT**: Sans cette étape, vos automatisations utilisent encore l'ANCIENNE version des blueprints !

---

### 2. Vérifier l'État de l'Alarme

#### Étapes:

1. **Outils de développement** → **États**
2. Chercher votre alarme (ex: `alarm_control_panel.maison`)
3. Noter l'état EXACT

#### États Attendus:

| État | Détecté comme Away? | Comportement |
|------|---------------------|--------------|
| `disarmed` | ❌ NON | Planning ou preset désarmé |
| `arming` | ❌ NON | Comptage avant armement |
| `armed_away` | ✅ OUI | Preset armé (eco) |
| `armed_home` | ✅ OUI | Preset armé (eco) |
| `armed_night` | ✅ OUI | Preset armé (eco) |
| `pending` | ❌ NON | Délai d'entrée avant désarmement |
| `triggered` | ❌ NON | Alarme déclenchée |

#### ⚠️ Cas Particulier: État `arming`

Si vous armez l'alarme et testez IMMÉDIATEMENT pendant la période de comptage (ex: 30 secondes), l'alarme est dans l'état `arming`, **PAS** `armed_away`.

**Solution**: Attendre que l'état passe à `armed_away` ou `armed_home` avant de tester.

---

### 3. Tester à Nouveau

#### Procédure de Test:

1. **Recharger les automatisations** (voir étape 1)
2. **Vérifier que l'alarme est bien armée**:
   - Outils de développement → États
   - Chercher `alarm_control_panel.maison`
   - État doit être `armed_away` ou `armed_home`
3. **Ouvrir les Traces** de l'automatisation
4. **Attendre 5-10 secondes** (le tick ou un autre trigger va déclencher l'automatisation)
5. **Vérifier dans les traces**:
   - Variable `is_away` doit être `true` ✅
   - Variable `target_preset` doit être `eco` (ou votre preset_when_armed)
   - Action: `set_preset_mode` = `eco`
6. **Vérifier le thermostat**:
   - Preset = `eco` ✅

---

## 🐛 Si le Problème Persiste

Si après avoir rechargé les automatisations et vérifié l'état de l'alarme, le problème persiste:

### Vérifications Supplémentaires:

#### A) Vérifier la Configuration de l'Entité Alarme

Dans l'automatisation:
1. Paramètres → Automatisations → [Votre automatisation]
2. Section "Alarme / Présence"
3. Vérifier que l'entité est bien `alarm_control_panel.maison`

#### B) Vérifier les Presets Configurés

1. Paramètres → Automatisations → [Votre automatisation]
2. Vérifier:
   - **Preset quand ARMÉE**: doit être `eco` (ou votre choix)
   - **Preset quand DÉSARMÉE**: doit être `comfort` (ou votre choix)

#### C) Tester Manuellement la Détection

Créer une automatisation de test:

```yaml
alias: "TEST - Détection Alarme"
trigger:
  - platform: state
    entity_id: alarm_control_panel.maison
action:
  - service: logbook.log
    data:
      name: "TEST Alarme"
      message: >-
        État alarme: {{ states('alarm_control_panel.maison') }}
        État lower: {{ states('alarm_control_panel.maison') | lower }}
        Starts with armed: {{ states('alarm_control_panel.maison') | lower | regex_search('^armed') is not none }}
```

Cette automatisation va logger chaque changement d'état de l'alarme avec les détails de détection.

---

## 📊 Exemple de Trace Correcte

Voici ce que vous devriez voir dans une trace quand l'alarme est armée:

```yaml
variables:
  is_away: true                                    # ✅ Alarme détectée comme armée
  alarm_id: "alarm_control_panel.maison"
  schedule_id: "calendar.chauffage_suite_parentale"
  schedule_preset: "eco"                           # Planning calculé (mais ignoré car is_away=true)
  target_preset: "eco"                             # ✅ Preset armé appliqué
  current_preset: "comfort"

action:
  - service: climate.set_preset_mode
    data:
      preset_mode: eco                             # ✅ Passage en ECO
```

---

## 🎯 Résumé des Actions

1. ✅ **RECHARGER** les automatisations dans Home Assistant
2. ✅ **VÉRIFIER** l'état de l'alarme (`armed_away` ou `armed_home`)
3. ✅ **TESTER** à nouveau en consultant les traces
4. ✅ **VÉRIFIER** que `is_away` = `true` dans les traces

---

## 💡 Note sur l'État `arming`

Par conception, l'état `arming` (comptage avant armement) n'est **PAS** détecté comme "away". Cela évite de couper le chauffage prématurément si vous annulez l'armement.

Si vous voulez détecter l'état `arming` comme "away", modifiez la logique:

```yaml
# Détection STANDARD (actuelle)
{{ st.startswith('armed') }}

# Détection AGRESSIVE (inclut arming)
{{ st.startswith('arm') }}
```

⚠️ Pas recommandé, car cela forcera eco même pendant le comptage d'armement.
