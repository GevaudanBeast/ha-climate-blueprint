# 🔄 Simplification: Retour Pré-Planning

## 📅 Date: 2025-12-30

---

## 🎯 Décision

Après réflexion sur la complexité croissante du système de planning horaire, décision de **simplifier** en retirant le planning de 3 blueprints sur 4.

### Raison de la Simplification

1. **Complexité excessive** : Le système de planning avec calendriers ajoutait une couche de complexité
2. **Cas d'usage limité** : Le planning n'est vraiment utile que pour des cas spécifiques (ex: salle de bain)
3. **Confusion avec priorités** : L'interaction entre planning, alarme et comportement par défaut créait de la confusion
4. **Préparation SRM** : En vue de développer Smart Room Manager pour une gestion centralisée

### Cas Spécifique: Bathroom

La salle de bain **garde le planning** car elle a un besoin réel :
- ❌ Lumière souvent laissée allumée comme "lumière d'ambiance"
- ❌ Lumière oubliée allumée par les enfants
- ✅ Planning permet de bloquer le chauffage même si lumière allumée

---

## ✅ Changements Effectués

### Blueprints Restaurés (sans planning)

| Blueprint | Version Avant | Version Après | Changement |
|-----------|---------------|---------------|------------|
| **Thermostat Heat** | v3.8 (avec calendriers) | v3.6 | ❌ Planning retiré |
| **Room Thermostat** | v2.11 (avec calendriers) | v2.9 | ❌ Planning retiré |
| **X4FP Room** | v7.15 (avec calendriers) | v7.13 | ❌ Planning retiré |
| **X4FP Bathroom** | v7.19 (avec calendriers) | v7.19 | ✅ **Planning conservé** |

### Lignes de Code

- **Supprimées** : 188 lignes (code planning)
- **Restaurées** : 23 lignes (code original)
- **Net** : -165 lignes de complexité ✅

---

## 📊 Priorités Simplifiées

### Avant (avec planning) - v3.8, v2.11, v7.15

```
1. ☀️ ÉTÉ
2. 🪟 FENÊTRE
3. ⚡ SOLAR OPTIMIZER
4. 🔒 ALARME ARMÉE
5. 📅 PLANNING (calendar ON/OFF)    ← Complexité ajoutée
6. 🌡️ THERMIQUE/DÉFAUT
```

**Problème** : Confusion entre planning ON/OFF et comportement par défaut

---

### Après (sans planning) - v3.6, v2.9, v7.13

```
1. ☀️ ÉTÉ
2. 🪟 FENÊTRE
3. ⚡ SOLAR OPTIMIZER
4. 🔒 ALARME ARMÉE → preset_armed (eco)
5. 🔒 ALARME DÉSARMÉE → preset_disarmed (comfort)
6. 🌡️ THERMIQUE (selon capteurs)
```

**Avantage** : Logique claire et directe, pas de niveau intermédiaire

---

### Bathroom (conserve planning) - v7.19

```
1. ☀️ ÉTÉ
2. 🪟 FENÊTRE
3. ⚡ SOLAR OPTIMIZER
4. 🔒 ALARME ARMÉE
5. 📅 PLANNING OFF → Bloque lumière (force eco)    ← Conservé
6. 💡 LUMIÈRE ON/OFF (si planning ON ou absent)
```

**Justification** : Planning bloque chauffage même si lumière oubliée allumée

---

## 🔧 Configuration Simplifiée

### Avant (avec planning)

**Thermostat Heat** - 11 paramètres :
```yaml
- Thermostat
- Été
- Fenêtres
- Alarme
  - Preset armée
  - Preset désarmée
- 📅 Calendar                        ← 3 paramètres planning
  - Preset planning ON
  - Preset planning OFF
- Températures fallback (4)
- Tick
```

### Après (sans planning)

**Thermostat Heat** - 8 paramètres :
```yaml
- Thermostat
- Été
- Fenêtres
- Alarme
  - Preset armée
  - Preset désarmée
- Températures fallback (4)
- Tick
```

**Réduction** : -3 paramètres par blueprint ✅

---

## 🎯 Impact Utilisateur

### Ce qui change pour vous

#### ✅ Thermostats Simples (Heat, Room, X4FP Room)

**Avant** :
- Alarme désarmée + Calendar ON → Confort
- Alarme désarmée + Calendar OFF → Eco
- Alarme armée → Eco (ignore calendar)

**Après** :
- Alarme désarmée → Confort (ou preset configuré)
- Alarme armée → Eco (ou preset configuré)
- **Pas de planning horaire**

Si vous utilisiez le planning sur ces blueprints, vous devrez :
1. ❌ Supprimer les calendriers configurés (ou les laisser, ils seront ignorés)
2. ✅ Recharger les automatisations dans Home Assistant
3. ✅ Configurer uniquement les presets alarme armée/désarmée

---

#### ✅ Bathroom (X4FP Bathroom)

**Aucun changement** - le planning continue de fonctionner :
- Calendar OFF → Bloque chauffage (force eco)
- Calendar ON → Autorise chauffage si lumière
- Alarme armée → Force eco (ignore planning)

---

## 📝 Actions à Effectuer

### 1. ⚠️ OBLIGATOIRE: Recharger les Automatisations

**Après avoir mis à jour les blueprints, vous DEVEZ recharger** :

#### Option A: Via l'Interface
1. **Outils de développement** → **YAML**
2. Cliquer **"Recharger Automatisations"**

#### Option B: Redémarrer Home Assistant
- Paramètres → Système → Redémarrer

---

### 2. Vérifier les Automatisations

Pour chaque automatisation utilisant les blueprints modifiés :

#### A) Thermostat Heat (v3.6)
- ✅ Vérifier presets alarme armée/désarmée
- ❌ Supprimer calendriers si configurés (optionnel, seront ignorés)

#### B) Room Thermostat (v2.9)
- ✅ Vérifier presets alarme armée/désarmée
- ❌ Supprimer calendriers si configurés (optionnel)

#### C) X4FP Room (v7.13)
- ✅ Vérifier presets alarme armée/désarmée
- ✅ Vérifier configuration thermique (capteur + consigne)
- ❌ Supprimer calendriers si configurés (optionnel)

#### D) X4FP Bathroom (v7.19)
- ✅ **Garder le calendar configuré** (toujours fonctionnel)
- ✅ Vérifier planning ON autorise / OFF bloque

---

### 3. Tester le Comportement

#### Test 1: Alarme désarmée
1. Alarme désarmée
2. Vérifier thermostat = preset_disarmed (défaut: comfort)

#### Test 2: Alarme armée
1. Armer alarme
2. Attendre état `armed_away` ou `armed_home`
3. Vérifier thermostat = preset_armed (défaut: eco)

#### Test 3: Bathroom avec planning (si configuré)
1. Calendar OFF
2. Allumer lumière salle de bain
3. Vérifier chauffage reste en eco (bloqué) ✅

---

## 🚀 Prochaine Étape: Smart Room Manager

### Pourquoi Smart Room Manager?

Le retrait du planning des blueprints simplifie, mais pour des besoins avancés de planification horaire, **Smart Room Manager** sera la solution :

#### Avantages SRM:
- ✅ **Gestion centralisée** de toutes les pièces
- ✅ **Planning horaire flexible** sans complexifier les blueprints
- ✅ **Règles inter-pièces** (ex: si salon chauffé, réduire chambres)
- ✅ **Profils de pièces** (chambres, pièces communes, salle de bain)
- ✅ **Gestion priorités globale** (alarme, été, fenêtres, planning)

#### Architecture Cible:

```
Smart Room Manager (automation centrale)
├─ Surveille alarme, été, calendriers
├─ Définit presets pour chaque pièce selon profil
└─ Envoie commandes aux blueprints

Blueprints (automatisations par pièce)
├─ Gèrent logique locale (fenêtres, thermique, lumière)
├─ Reçoivent presets de SRM via input_select
└─ Appliquent comportements spécifiques (SO, X4FP, etc.)
```

---

## 📋 Checklist Migration

- [ ] Recharger automatisations dans Home Assistant
- [ ] Vérifier presets alarme armée/désarmée (tous blueprints)
- [ ] Supprimer calendriers configurés (Heat, Room, X4FP Room) - optionnel
- [ ] Vérifier bathroom garde son calendar (v7.19)
- [ ] Tester alarme armée → eco
- [ ] Tester alarme désarmée → comfort
- [ ] Tester bathroom planning OFF bloque lumière
- [ ] Documenter besoins pour Smart Room Manager

---

## 🐛 Problèmes Potentiels

### ❌ "Automatisation ne change plus"

**Cause** : Automatisations pas rechargées
**Solution** : Outils de développement → YAML → Recharger Automatisations

### ❌ "Calendar ne fonctionne plus (Heat/Room/X4FP Room)"

**Cause** : Normal, planning retiré de ces blueprints
**Solution** : Utiliser presets alarme armée/désarmée uniquement

### ❌ "Bathroom calendar ne fonctionne plus"

**Cause** : Automatisations pas rechargées (bathroom garde planning)
**Solution** : Recharger automatisations, vérifier calendar configuré

---

## 📊 Statistiques

### Avant Simplification
- **4 blueprints** avec planning
- **~300 lignes** de code planning
- **11 paramètres** par blueprint (avec planning)
- **Complexité** : Élevée

### Après Simplification
- **1 blueprint** avec planning (bathroom uniquement)
- **~75 lignes** de code planning (bathroom)
- **8 paramètres** par blueprint (sans planning)
- **Complexité** : Réduite ✅

---

## 🎓 Leçons Apprises

### Ce qui n'a pas fonctionné
1. **Planning universel trop complexe** : Pas adapté à tous les cas d'usage
2. **Interaction planning/alarme confuse** : Priorités difficiles à comprendre
3. **Configuration redondante** : preset_schedule_on/off vs preset_disarmed

### Ce qui a bien fonctionné
1. **Planning bathroom utile** : Cas d'usage clair (bloquer lumière oubliée)
2. **Blueprints simples efficaces** : Alarme armée/désarmée suffit dans la majorité des cas
3. **Solar Optimizer intégré** : Priorité solaire fonctionne bien

### Prochaines Améliorations
1. **Smart Room Manager** pour gestion centralisée planning
2. **Blueprints focalisés** sur logique locale (fenêtres, thermique, SO)
3. **Séparation claire** entre gestion globale (SRM) et locale (blueprints)

---

## 📚 Références

- **TABLEAU_PRIORITES.md** : Ordre de priorité détaillé (mis à jour)
- **DIAGNOSTIC_ALARM_PRIORITY.md** : Diagnostic problème alarme
- **Commit** : `9270f96` - revert versions pré-planning

---

## ✅ Statut

**Date** : 2025-12-30
**Action** : ✅ Complétée
**Blueprints modifiés** : 3 (Heat v3.6, Room v2.9, X4FP Room v7.13)
**Blueprint conservé** : 1 (Bathroom v7.19 avec planning)
**Prochaine étape** : Conception Smart Room Manager
