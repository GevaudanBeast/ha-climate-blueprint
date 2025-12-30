# 📊 Tableau des Priorités - Blueprints HVAC

## Vue d'ensemble

Chaque blueprint applique des règles dans un **ordre de priorité strict**. Une règle de priorité plus haute **arrête l'exécution** et ignore toutes les règles suivantes.

---

## 🔥 Blueprint: **Thermostat Chauffage** (blueprint_hvac_thermostat_heat.yaml)

**Version**: v3.8
**Usage**: Thermostat simple chauffage uniquement

| Priorité | Condition | Action | Comportement | Stop? |
|----------|-----------|--------|--------------|-------|
| **1** | ☀️ **ÉTÉ** (indicateur été = ON + politique OFF) | `hvac_mode: off` | Thermostat complètement éteint | ✅ STOP |
| **2** | 🪟 **FENÊTRE OUVERTE** (capteur = on pendant délai) | `hvac_mode: off` | Pause chauffage | ✅ STOP |
| **3** | 🔥 **MODE HEAT** | `hvac_mode: heat` | Force mode chauffage | ❌ Continue |
| **4a** | 🔒 **ALARME ARMÉE** (armed_*) | `preset: preset_when_armed` (défaut: eco) | Preset économique | ✅ STOP |
| **4b** | 📅 **PLANNING ACTIF** (alarme désarmée + calendar ON) | `preset: preset_schedule_on` (défaut: comfort) | Preset planning actif | ✅ STOP |
| **4c** | 📅 **PLANNING INACTIF** (alarme désarmée + calendar OFF) | `preset: preset_schedule_off` (défaut: eco) | Preset planning inactif | ✅ STOP |
| **5** | 🏠 **DÉFAUT** (alarme désarmée + pas de planning) | `preset: preset_when_disarmed` (défaut: comfort) | Preset par défaut | ✅ STOP |

### Résumé en cascade:
```
ÉTÉ? → OFF (stop)
  ↓ NON
FENÊTRE? → OFF (stop)
  ↓ NON
MODE ≠ HEAT? → HEAT
  ↓
ALARME ARMÉE? → PRESET_ARMED (stop)
  ↓ NON
PLANNING CONFIGURÉ?
  ├─ OUI → CALENDAR ON? → PRESET_SCHED_ON (stop)
  │         └─ NON → PRESET_SCHED_OFF (stop)
  └─ NON → PRESET_DISARMED (stop)
```

---

## 🌡️ Blueprint: **Pièce Thermostat/Clim** (blueprint_hvac_room_thermostat.yaml)

**Version**: v2.11
**Usage**: Thermostat avec bascule été/hiver automatique + Solar Optimizer

| Priorité | Condition | Action | Comportement | Stop? |
|----------|-----------|--------|--------------|-------|
| **1** | 🪟 **FENÊTRE OUVERTE** | `hvac_mode: off` | Pause (heat ou cool) | ✅ STOP |
| **2** | ⚡ **SOLAR OPTIMIZER ACTIF** (jour + SO actif) | Aucune | SO pilote le thermostat | ✅ STOP |
| **3a** | 🔒 **ALARME ARMÉE** (preset away disponible) | `preset: away` | Mode absence | ✅ STOP |
| **3b** | 🔒 **ALARME ARMÉE** (preset away indisponible) | `hvac_mode: off` | Arrêt complet | ✅ STOP |
| **4a** | 📅 **PLANNING ACTIF** (alarme désarmée + calendar ON) | `preset: preset_schedule_on` | Preset planning | ❌ Continue |
| **4b** | 📅 **PLANNING INACTIF** (alarme désarmée + calendar OFF) | `preset: preset_schedule_off` | Preset planning | ❌ Continue |
| **4c** | 🏠 **ALARME DÉSARMÉE** (pas de planning) | `preset: home` (si disponible) | Mode présence | ❌ Continue |
| **5** | 🌡️ **BASCULE ÉTÉ/HIVER** (alarme désarmée) | `mode: cool/heat` + `temp: setpoint` | Consigne saisonnière | ❌ FIN |

### Résumé en cascade:
```
FENÊTRE? → OFF (stop)
  ↓ NON
JOUR + SO ACTIF? → SO pilote (stop)
  ↓ NON
ALARME ARMÉE? → AWAY ou OFF (stop)
  ↓ NON
PLANNING?
  ├─ OUI → PRESET_SCHED (on/off)
  └─ NON → PRESET HOME
  ↓
BASCULE ÉTÉ/HIVER → cool/heat + consigne
```

---

## ⚡ Blueprint: **X4FP Pièce Thermique** (blueprint_hvac_X4FP_room.yaml)

**Version**: v7.15
**Usage**: Fil pilote avec contrôle thermique (hystérésis) + Solar Optimizer

| Priorité | Condition | Action | Comportement | Stop? |
|----------|-----------|--------|--------------|-------|
| **1a** | ☀️ **ÉTÉ** (politique OFF) | `hvac_mode: off` | Arrêt complet | ✅ STOP |
| **1b** | ☀️ **ÉTÉ** (politique ECO) | `preset: preset_idle` (défaut: eco) | Maintien eco | ✅ STOP |
| **2** | 🪟 **FENÊTRE OUVERTE** | `preset: preset_window` (défaut: away) | Pause | ✅ STOP |
| **3a** | ⚡ **SO ACTIF** (jour + SO actif + preset ≠ none) | `preset: solar_preset` (défaut: comfort) | SO chauffe | ✅ STOP |
| **3b** | ⚡ **SO ACTIF** (preset = none) | Aucune | SO pilote (mode passif) | ✅ STOP |
| **4** | 🔒 **ALARME ARMÉE** (SO ne chauffe pas) | `preset: preset_away` (défaut: away) | Mode absence | ✅ STOP |
| **5a** | 📅 **PLANNING ACTIF** (alarme désarmée + calendar ON) | `preset: preset_schedule_on` | Planning actif | ✅ STOP |
| **5b** | 📅 **PLANNING INACTIF** (alarme désarmée + calendar OFF) | `preset: preset_schedule_off` | Planning inactif | ✅ STOP |
| **6a** | 🌡️ **THERMIQUE** (temp ≤ consigne - hystérésis) | `preset: preset_heat` (défaut: comfort) | Demande de chauffe | ✅ STOP |
| **6b** | 🌡️ **THERMIQUE** (temp ≥ consigne + hystérésis) | `preset: preset_idle` (défaut: eco) | Arrêt chauffe | ✅ STOP |
| **7** | 🔄 **PAS D'ACTION** | Aucune | Maintien état actuel | ✅ FIN |

### Résumé en cascade:
```
ÉTÉ? → OFF ou ECO (stop)
  ↓ NON
FENÊTRE? → PRESET_WINDOW (stop)
  ↓ NON
JOUR + SO ACTIF? → SO pilote (stop)
  ↓ NON
ALARME ARMÉE? → PRESET_AWAY (stop)
  ↓ NON
PLANNING?
  ├─ OUI → CALENDAR ON/OFF → PRESET_SCHED (stop)
  └─ NON ↓
THERMIQUE CONFIGURÉ?
  ├─ OUI → T° ≤ consigne-hys? → HEAT (stop)
  │        └─ T° ≥ consigne+hys? → IDLE (stop)
  └─ NON → Pas d'action
```

**⚠️ Note importante**: Le planning **remplace** complètement le contrôle thermique quand actif !

---

## 💡 Blueprint: **X4FP Bathroom** (blueprint_hvac_X4FP_bathroom.yaml)

**Version**: v7.19
**Usage**: Fil pilote avec déclenchement par lumière + planning autorisant/bloquant

| Priorité | Condition | Action | Comportement | Stop? |
|----------|-----------|--------|--------------|-------|
| **1a** | ☀️ **ÉTÉ** (politique OFF) | `hvac_mode: off` | Arrêt complet | ✅ STOP |
| **1b** | ☀️ **ÉTÉ** (politique ECO) | `preset: preset_idle` (défaut: eco) | Maintien eco | ✅ STOP |
| **2** | 🪟 **FENÊTRE OUVERTE** | `preset: preset_window` (défaut: away) | Pause | ✅ STOP |
| **3a** | ⚡ **SO ACTIF** (jour + SO actif + preset ≠ none) | `preset: solar_preset` (défaut: comfort) | SO chauffe | ✅ STOP |
| **3b** | ⚡ **SO ACTIF** (preset = none) | Aucune | SO pilote (mode passif) | ✅ STOP |
| **4** | 🔒 **ALARME ARMÉE** (SO ne chauffe pas) | `preset: preset_away` (défaut: away) | Mode absence | ✅ STOP |
| **5** | 📅 **PLANNING OFF** (alarme désarmée + calendar OFF) | `preset: preset_idle` (défaut: eco) | **Bloque** la lumière | ✅ STOP |
| **6a** | 💡 **LUMIÈRE ON** (alarme désarmée + planning autorisant) | `preset: preset_heat` (défaut: comfort) | Chauffe | ✅ STOP |
| **6b** | 💡 **LUMIÈRE OFF** (alarme désarmée + planning autorisant) | `preset: preset_idle` (défaut: eco) | Arrêt | ✅ STOP |
| **7** | 🔄 **PAS D'ACTION** | Aucune | Maintien état | ✅ FIN |

### Résumé en cascade:
```
ÉTÉ? → OFF ou ECO (stop)
  ↓ NON
FENÊTRE? → PRESET_WINDOW (stop)
  ↓ NON
JOUR + SO ACTIF? → SO pilote (stop)
  ↓ NON
ALARME ARMÉE? → PRESET_AWAY (stop)
  ↓ NON
PLANNING OFF? → ECO (bloque lumière) (stop)
  ↓ NON (planning ON ou pas configuré)
LUMIÈRE?
  ├─ ON → HEAT
  └─ OFF → IDLE
```

**⚠️ Spécificité Bathroom**: Le planning **autorise/bloque** le chauffage par lumière (ne le remplace pas directement).

---

## 🎯 Résumé Général des Priorités

### Ordre universel:
1. **☀️ ÉTÉ** (absolu)
2. **🪟 FENÊTRE** (sécurité)
3. **⚡ SOLAR OPTIMIZER** (si configuré - économies)
4. **🔒 ALARME ARMÉE** (absence)
5. **📅 PLANNING** (si alarme désarmée)
6. **🌡️ THERMIQUE / LUMIÈRE** (logique par défaut)

### Points clés:

#### ✅ ALARME = PRIORITAIRE sur PLANNING
- Si alarme armée → Planning **ignoré**
- Si alarme désarmée → Planning **actif**

#### ✅ CALENDAR = ON/OFF
- State `on` = événement en cours → `preset_schedule_on`
- State `off` = pas d'événement → `preset_schedule_off`

#### ✅ SO = SUPER-PRIORITAIRE
- Solar Optimizer actif → **Ignore** alarme (si `allow_solar_in_away: true`)
- SO → **Ignore** planning
- SO → **Ignore** thermique/lumière

#### ✅ FENÊTRE & ÉTÉ = ABSOLU
- Fenêtre ouverte → **Toujours** OFF ou AWAY
- Été (si politique OFF) → **Toujours** OFF

---

## 🧪 Comment Tester les Priorités

### Test 1: Alarme prioritaire sur Planning
1. Calendar avec événement actif (state = `on`)
2. Alarme désarmée → Thermostat = `preset_schedule_on` ✅
3. Armer alarme → Thermostat = `preset_when_armed` (eco) ✅
4. Planning **ignoré**

### Test 2: Planning fonctionne si alarme désarmée
1. Alarme désarmée
2. Calendar state `on` → Thermostat = `preset_schedule_on` ✅
3. Calendar state `off` → Thermostat = `preset_schedule_off` ✅

### Test 3: Fenêtre prioritaire sur tout
1. Alarme armée + SO actif + Planning actif
2. Ouvrir fenêtre → Thermostat = `OFF` ou `preset_window` ✅
3. Tout le reste **ignoré**

### Test 4: SO prioritaire sur alarme (X4FP uniquement)
1. `allow_solar_in_away: true`
2. Alarme armée
3. SO actif → Thermostat = `solar_preset` (comfort) ✅
4. Alarme **ignorée**

---

## 🐛 Problèmes Fréquents

### ❌ "L'alarme ne fonctionne pas"
**Cause**: Automations pas rechargées après mise à jour blueprint
**Solution**: Outils de développement → YAML → Recharger Automatisations

### ❌ "Le planning ne fonctionne jamais"
**Causes possibles**:
1. Alarme armée (planning ignoré si alarme armée)
2. Entité calendar mal configurée
3. State du calendar incorrect (vérifier dans États)

**Solution**:
- Vérifier alarme désarmée
- Vérifier state calendar = `on` ou `off`

### ❌ "SO ne fonctionne jamais"
**Cause**: Mauvaise entité configurée
**Solution**: Configurer le **SWITCH** que SO contrôle (ex: `switch.solar_optimizer_chambre`), pas l'entité de configuration

### ❌ "Thermostat ne change jamais"
**Cause**: Preset non supporté par le thermostat
**Solution**:
- Vérifier attribut `preset_modes` du thermostat
- Utiliser uniquement les presets supportés
- Blueprint applique fallback température si preset indisponible

---

## 📝 Changelog Priorités

### v3.8 / v2.11 / v7.15 / v7.19 (Calendriers)
- ✅ Planning avec **calendriers HA** (au lieu de schedules)
- ✅ Calendar `on` = événement actif
- ✅ Calendar `off` = pas d'événement
- ✅ Planning **actif uniquement si alarme désarmée** (inchangé)

### Versions précédentes
- Planning avec 4 schedules (matin, midi, soir, nuit)
- Simplifié vers 1 seul schedule
- Puis migré vers calendar
