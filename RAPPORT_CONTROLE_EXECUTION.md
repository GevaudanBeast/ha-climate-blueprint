# Rapport de Contrôle d'Exécution – Blueprints HVAC
**Date:** 16 janvier 2025
**Versions analysées:**
- Thermostat Heat v3.1
- Room Thermostat v2.1
- X4FP Bathroom v7.3
- X4FP Room v7.3

---

## 🔴 PROBLÈMES CRITIQUES DÉTECTÉS

### 1. **Room Thermostat v2.1** – Variable `current_temp` non définie

**Fichier:** `blueprints/blueprint_hvac_room_thermostat.yaml`

**Problème:**
- **Ligne 312** : Utilisation de la variable `current_temp` dans une condition
- **Lignes 169-193** : La variable `current_temp` n'est PAS définie dans les variables runtime

**Code problématique:**
```yaml
# Ligne 312 – ERREUR: current_temp non définie
- if:
    - condition: template
      value_template: "{{ current_temp != (target_temp | float) }}"
```

**Variables runtime définies (lignes 169-193):**
```yaml
- variables:
    solar_active: "{{ solar_id != '' and is_state(solar_id, 'on') }}"
    win_any_open: ...
    is_summer: ...
    is_away: ...
    current_mode: "{{ states(climate) }}"
    current_preset: "{{ state_attr(climate, 'preset_mode') or '' }}"
    presets: "{{ state_attr(climate, 'preset_modes') or [] }}"
    # ❌ current_temp MANQUANT
```

**Impact:**
- ❌ **Erreur d'exécution garantie** quand l'automation tente d'appliquer la température
- ❌ Le blueprint Room Thermostat v2.1 est **NON FONCTIONNEL** dans la section température

**Sévérité:** 🔴 **CRITIQUE** – Empêche le fonctionnement normal

**Correction requise:**
Ajouter la définition de `current_temp` dans les variables runtime (après ligne 193):
```yaml
- variables:
    solar_active: "{{ solar_id != '' and is_state(solar_id, 'on') }}"
    win_any_open: >-
      {{ expand(window_sensors_list) | selectattr('state','eq','on') | list | count > 0 }}
    is_summer: >-
      {% if summer_entity_id and (summer_entity_id | string) != '' %}
        {% set st = states(summer_entity_id) | lower %}
        {{ st in ['on','true','open','summer','été','ete'] }}
      {% else %}
        false
      {% endif %}
    is_away: >-
      {% if alarm_id and (alarm_id | string) != '' %}
        {% set st = states(alarm_id) | lower %}
        {{ st.startswith('armed') }}
      {% else %}
        false
      {% endif %}
    current_mode: "{{ states(climate) }}"
    current_preset: "{{ state_attr(climate, 'preset_mode') or '' }}"
    presets: "{{ state_attr(climate, 'preset_modes') or [] }}"
    current_temp: "{{ state_attr(climate, 'temperature') | float(0) }}"  # ✅ AJOUTER CETTE LIGNE
```

**Origine du bug:**
Ce bug provient de la simplification effectuée le 15 janvier où la variable `current_temp` a été supprimée de Room Thermostat car elle était marquée comme "inutilisée". Cependant, elle **était** utilisée à la ligne 312, mais cette utilisation a été manquée lors de l'analyse.

---

## 🟡 PROBLÈMES POTENTIELS

### 2. **Thermostat Heat v3.1** – Triggers avec entités optionnelles

**Fichier:** `blueprints/blueprint_hvac_thermostat_heat.yaml`

**Problème:**
Les triggers d'alarme (lignes 150-161) utilisent `!input alarm_entity` qui peut être vide (default: ""):

```yaml
# Ligne 55-59 – alarm_entity PEUT être vide
alarm_entity:
  name: Alarme / Présence (Away si armée)
  description: "Armed_* = armée (Away). Disarmed = présent."
  default: ""  # ⚠️ Peut être vide
  selector: { entity: { domain: alarm_control_panel } }

# Lignes 150-161 – Triggers utilisent cette entité
- id: alarm_armed
  platform: state
  entity_id: !input alarm_entity  # ⚠️ Peut causer erreur si vide
  to:
    - armed_away
    - armed_home
    - armed_night

- id: alarm_disarmed
  platform: state
  entity_id: !input alarm_entity  # ⚠️ Peut causer erreur si vide
  to: disarmed
```

**Impact actuel:**
- En Home Assistant, un trigger `platform: state` avec `entity_id: ""` génère une **erreur de configuration**
- L'automation **ne peut pas être créée** si alarm_entity est vide

**Sévérité:** 🟡 **MOYENNE** – Empêche création automation si alarme non configurée

**Pourquoi ce n'est pas critique:**
- Le blueprint Room Thermostat (qui gère aussi l'alarme) utilise un **trigger template** pour gérer les entités optionnelles (lignes 141-148)
- Thermostat Heat pourrait adopter la même approche

**Solution recommandée:**
Adopter la même stratégie que Room Thermostat avec un trigger template conditionnel:

```yaml
# Remplacer lignes 150-161 par:
- id: alarm_change
  platform: template
  value_template: >-
    {{ states(alarm_id) if alarm_id and alarm_id != '' else 'none' }}
```

**OU** documenter que `alarm_entity` est **REQUIS** (retirer le default: "").

---

### 3. **Room Thermostat v2.1** – Gestion alarme sans trigger dédié

**Fichier:** `blueprints/blueprint_hvac_room_thermostat.yaml`

**Observation:**
Le blueprint gère l'alarme via le trigger template `optional_entities_change` (lignes 141-148), mais:
- **Ligne 58** : Description dit "Requis pour gérer preset away/home automatiquement"
- **Ligne 59** : Mais default: "" (optionnel)

**Incohérence documentation:**
```yaml
alarm_entity:
  name: Alarme / Présence (Away si armée)
  description: "Requis pour gérer preset away/home automatiquement."  # ⚠️ Dit REQUIS
  default: ""  # ⚠️ Mais est OPTIONNEL
  selector:
    entity:
      domain: alarm_control_panel
```

**Impact:**
- Confusion pour l'utilisateur
- La fonctionnalité fonctionne correctement même si vide (grâce au trigger template)

**Sévérité:** 🟢 **MINEURE** – Incohérence documentation uniquement

**Correction recommandée:**
Changer la description pour refléter la réalité:
```yaml
alarm_entity:
  name: Alarme / Présence (Away si armée)
  description: "Optionnel. Si configuré, gère preset away/home automatiquement."
  default: ""
  selector:
    entity:
      domain: alarm_control_panel
```

---

## ✅ BLUEPRINTS SANS PROBLÈMES

### X4FP Bathroom v7.3
**Statut:** ✅ **FONCTIONNEL**

**Vérifications effectuées:**
- ✅ Toutes les variables utilisées sont définies
- ✅ Triggers templates gèrent correctement les entités optionnelles
- ✅ Logique d'exécution cohérente (ordre: Été > Fenêtre > SO > Away > Lumière)
- ✅ Variables `should_act_on_light` correctement calculée avant utilisation
- ✅ Pas de variables orphelines

---

### X4FP Room v7.3
**Statut:** ✅ **FONCTIONNEL**

**Vérifications effectuées:**
- ✅ Toutes les variables utilisées sont définies
- ✅ Triggers templates gèrent correctement les entités optionnelles
- ✅ Logique thermique avec hystérésis correcte
- ✅ Clamp de consigne (sp) fonctionne correctement avec min/max
- ✅ Logique d'exécution cohérente (ordre: Été > Fenêtre > SO > Away > Thermique)
- ✅ Pas de variables orphelines

---

## 📊 RÉSUMÉ PAR BLUEPRINT

| Blueprint | Version | Statut | Problèmes Critiques | Problèmes Potentiels | Mineurs |
|-----------|---------|--------|---------------------|----------------------|---------|
| **Thermostat Heat** | v3.1 | 🟡 Attention | 0 | 1 (triggers alarme) | 0 |
| **Room Thermostat** | v2.1 | 🔴 **NON FONCTIONNEL** | **1 (current_temp)** | 0 | 1 (doc) |
| **X4FP Bathroom** | v7.3 | ✅ Fonctionnel | 0 | 0 | 0 |
| **X4FP Room** | v7.3 | ✅ Fonctionnel | 0 | 0 | 0 |

---

## 🔧 PLAN D'ACTION RECOMMANDÉ

### Priorité 1 – URGENT (Room Thermostat v2.1)
- [ ] **Ajouter variable `current_temp`** dans les variables runtime (ligne ~193)
  ```yaml
  current_temp: "{{ state_attr(climate, 'temperature') | float(0) }}"
  ```
- [ ] Tester le blueprint après correction
- [ ] Bumper version v2.1 → v2.2

### Priorité 2 – RECOMMANDÉ (Thermostat Heat v3.1)
- [ ] **Option A:** Documenter que `alarm_entity` est REQUIS (retirer default: "")
- [ ] **Option B:** Adopter trigger template conditionnel comme Room Thermostat
- [ ] Bumper version v3.1 → v3.2 si modifié

### Priorité 3 – OPTIONNEL (Room Thermostat v2.1)
- [ ] Corriger description de `alarm_entity` pour clarifier optionnel/requis
- [ ] Peut être inclus dans v2.2

---

## 📝 DÉTAILS TECHNIQUES

### Analyse des variables – Tous blueprints

#### Thermostat Heat v3.1
**Variables globales (lignes 115-128):**
```yaml
✅ room, climate, summer_entity_id, window_sensors_list, alarm_id
✅ summer_behavior_sel, comfort_t, eco_t, frost_t, boost_t
✅ tick_minutes, preset_armed, preset_disarmed
```

**Variables runtime (lignes 180-193):**
```yaml
✅ is_summer, win_any_open, is_away, target_preset
✅ presets, has_target, current_preset, current_mode, current_temp
```

**Toutes utilisées:** ✅ Aucune variable orpheline

---

#### Room Thermostat v2.1
**Variables globales (lignes 113-122):**
```yaml
✅ room, climate, summer_entity_id, alarm_id, solar_id
✅ window_sensors_list, heat_default, cool_default, tick_minutes
```

**Variables runtime (lignes 169-193):**
```yaml
✅ solar_active, win_any_open, is_summer, is_away
✅ current_mode, current_preset, presets
❌ current_temp – MANQUANTE (utilisée ligne 312)
```

**Variables supplémentaires (ligne 290-292):**
```yaml
✅ target_mode, target_temp
```

---

#### X4FP Bathroom v7.3
**Variables globales (lignes 183-221):**
```yaml
✅ room, climate, window_sensors_list, light_id
✅ light_on_behavior, light_off_behavior
✅ summer_id, summer_policy, alarm_id
✅ solar_switch_id, allow_solar_in_away, solar_preset
✅ preset_heat, preset_idle, preset_window, preset_away
✅ tick_minutes, is_summer, is_away
✅ solar_is_heating, solar_can_override_away, win_any_open
```

**Variables runtime (lignes 227-230):**
```yaml
✅ is_light_on, current_preset, current_mode
```

**Variables supplémentaires (ligne 324-328):**
```yaml
✅ should_act_on_light
```

**Toutes utilisées:** ✅ Aucune variable orpheline

---

#### X4FP Room v7.3
**Variables globales (lignes 173-234):**
```yaml
✅ room, climate, window_sensors_list
✅ temp_id, sp_id, hys, sp_min, sp_max
✅ summer_id, summer_policy, alarm_id
✅ solar_switch_id, allow_solar_in_away, solar_preset
✅ preset_heat, preset_idle, preset_window, preset_away
✅ tick_minutes, t_room, sp_raw, sp, is_summer, is_away
✅ solar_is_heating, solar_can_override_away, win_any_open, has_thermal
```

**Variables runtime (lignes 241-243):**
```yaml
✅ current_preset, current_mode
```

**Variables supplémentaires (ligne 340-342):**
```yaml
✅ need_heat, need_idle
```

**Toutes utilisées:** ✅ Aucune variable orpheline

---

## 🧪 TESTS RECOMMANDÉS APRÈS CORRECTIONS

### Test 1: Room Thermostat v2.2 (après correction current_temp)
1. Importer blueprint modifié
2. Créer automation test
3. Activer mode trace
4. Déclencher changement été/hiver
5. **Vérifier:** Température appliquée sans erreur
6. **Vérifier:** Logbook montre message température appliquée

### Test 2: Thermostat Heat (si triggers modifiés)
1. Tester avec alarm_entity vide
2. Tester avec alarm_entity configuré
3. **Vérifier:** Automation se crée dans les deux cas
4. **Vérifier:** Triggers fonctionnent correctement

---

## 💡 RECOMMANDATIONS GÉNÉRALES

### Bonnes pratiques détectées ✅
1. **Triggers template pour entités optionnelles** (Room Thermostat, X4FP)
   - Gère élégamment les inputs optionnels
   - Évite erreurs de configuration

2. **Ordre logique d'exécution avec `stop`**
   - Été > Fenêtre > Solaire > Away > Mode normal
   - Priorités claires et cohérentes

3. **Variables runtime séparées des variables globales**
   - Bonne organisation
   - Facilite maintenance

### Points d'attention ⚠️
1. **Vérifier utilisation de toutes les variables avant suppression**
   - Le bug current_temp montre l'importance de cette vérification
   - Utiliser recherche globale pour confirmer non-utilisation

2. **Triggers avec entités optionnelles**
   - Préférer triggers template pour entités avec default: ""
   - OU documenter clairement comme REQUIS

3. **Cohérence documentation vs implémentation**
   - Vérifier que descriptions reflètent le comportement réel

---

## 📅 HISTORIQUE

### 16 janvier 2025 – Contrôle complet exécution
- ✅ Analyse minutieuse des 4 blueprints
- 🔴 **Bug critique détecté:** Room Thermostat v2.1 – variable current_temp manquante
- 🟡 **Problème potentiel détecté:** Thermostat Heat v3.1 – triggers alarme avec entité optionnelle
- ✅ X4FP Bathroom v7.3 et X4FP Room v7.3 validés fonctionnels

**Action requise immédiate:** Correction Room Thermostat v2.1

---

## 🎯 CONCLUSION

### État actuel
- **2 blueprints fonctionnels** (X4FP Bathroom, X4FP Room)
- **1 blueprint non fonctionnel** (Room Thermostat) – correction simple requise
- **1 blueprint avec attention requise** (Thermostat Heat) – fonctionne mais avec contrainte

### Prochaines étapes
1. **Corriger Room Thermostat v2.1** → v2.2 (URGENT)
2. **Améliorer Thermostat Heat v3.1** → v3.2 (RECOMMANDÉ)
3. **Tester corrections** en mode trace
4. **Mettre à jour CHANGELOG**

**Temps estimé corrections:** 15-30 minutes
**Complexité:** Faible (ajout 1 ligne, modification trigger)
