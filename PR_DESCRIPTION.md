# Pull Request : Planning Horaire avec Schedules Home Assistant

## 📅 Nouvelle Fonctionnalité Majeure

Ajout du **planning horaire** avec support d'une entité `schedule` de Home Assistant dans **tous les 4 blueprints**.

---

## ✨ Résumé des Changements

### Blueprints Mis à Jour

| Blueprint | Ancienne Version | Nouvelle Version | Changements Principaux |
|-----------|------------------|------------------|------------------------|
| **Thermostat Heat** | v3.6 | **v3.7** | Planning horaire (1 schedule, 2 presets) |
| **Room Thermostat** | v2.9 | **v2.10** | Planning horaire + été/hiver |
| **X4FP Bathroom** | v7.16 | **v7.17** | Planning prioritaire sur lumière |
| **X4FP Room** | v7.13 | **v7.14** | Planning prioritaire sur thermique |

### Fonctionnalités Ajoutées

✅ **1 seul schedule** contenant toutes vos périodes de confort
✅ **2 presets configurables** : preset quand schedule ON, preset quand schedule OFF
✅ **Utilise une entité `schedule`** de Home Assistant (Helper)
✅ **Un schedule peut contenir plusieurs périodes** (ex: matin 06h-08h + soir 17h-22h)
✅ **Actif uniquement si alarme désarmée** (présence à la maison)
✅ **Rétrocompatible** : configurations existantes fonctionnent sans modification

---

## 🎯 Ordre de Priorité

```
1. Été → OFF (ou ECO selon configuration)
2. Fenêtre ouverte → OFF (ou preset fenêtre)
3. Solar Optimizer → COMFORT (si actif)
4. Alarme ARMÉE → preset alarme (ignore le planning)
5. ⭐ Alarme DÉSARMÉE + Schedule ON → preset_schedule_on
6. ⭐ Alarme DÉSARMÉE + Schedule OFF → preset_schedule_off
7. Alarme DÉSARMÉE + Pas de planning → comportement par défaut
```

**Résultat** :
- 🏠 **Présent** (alarme off) + **Schedule ON** → Suit preset_schedule_on (ex: comfort)
- 🏠 **Présent** (alarme off) + **Schedule OFF** → Suit preset_schedule_off (ex: eco)
- 🚪 **Absent** (alarme on) → Force eco/away (ignore complètement le planning)

---

## 📝 Modifications Techniques

### Principe de Fonctionnement

L'utilisateur crée **UN SEUL** schedule dans Home Assistant qui contient **toutes ses périodes de confort**. Par exemple :

```yaml
schedule.chauffage_confort:
  Lundi-Vendredi:
    - 06:00-08:00  # Matin
    - 17:00-22:00  # Soir
  Samedi-Dimanche:
    - 08:00-22:00  # Toute la journée
```

Le blueprint vérifie simplement l'état du schedule :
- **Schedule ON** → Applique `preset_schedule_on` (ex: comfort)
- **Schedule OFF** → Applique `preset_schedule_off` (ex: eco)

### 1. Thermostat Heat v3.7

**Fichier** : `blueprints/blueprint_hvac_thermostat_heat.yaml`

**Ajouts** :
- 3 nouveaux inputs (1 schedule + 2 presets)
- Variable `schedule_id`, `preset_sched_on`, `preset_sched_off`
- Trigger `schedule_change` qui surveille l'état du schedule
- Logique `schedule_preset` dans variables runtime
- Logique `target_preset` intègre le schedule

**Lignes modifiées** : +75 lignes

**Inputs ajoutés** :
```yaml
schedule_entity:
  name: "📅 Planning Horaire (optionnel)"
  description: "Schedule Home Assistant. Définissez vos périodes (ex: ON 06h-08h et 17h-22h, OFF le reste). Actif uniquement si alarme désarmée."
  default: ""
  selector:
    entity:
      domain: schedule

preset_schedule_on:
  name: "Preset quand Planning ACTIF (ON)"
  description: "Preset à appliquer quand le schedule est ON"
  default: comfort
  selector:
    select:
      options: [eco, comfort, comfort-1, comfort-2, frost_protection, boost, none]

preset_schedule_off:
  name: "Preset quand Planning INACTIF (OFF)"
  description: "Preset à appliquer quand le schedule est OFF"
  default: eco
  selector:
    select:
      options: [eco, comfort, comfort-1, comfort-2, frost_protection, boost, none]
```

**Logique** :
```yaml
schedule_preset: >-
  {% if not is_away and schedule_id and schedule_id != '' %}
    {% set sched_state = states(schedule_id) | lower %}
    {% if sched_state == 'on' %}
      {{ preset_sched_on }}
    {% else %}
      {{ preset_sched_off }}
    {% endif %}
  {% else %}
    none
  {% endif %}

target_preset: >-
  {% if is_away %}
    {{ preset_armed }}
  {% elif schedule_preset != 'none' %}
    {{ schedule_preset }}
  {% else %}
    {{ preset_disarmed }}
  {% endif %}
```

### 2. Room Thermostat v2.10

**Fichier** : `blueprints/blueprint_hvac_room_thermostat.yaml`

**Ajouts** :
- 3 nouveaux inputs avec presets adaptés (none, eco, comfort, home, away, boost)
- Variables schedule
- Trigger `schedule_change`
- Logique schedule intégrée dans section "Alarme désarmée"

**Lignes modifiées** : +70 lignes

**Presets disponibles** : `[none, eco, comfort, home, away, boost]`

### 3. X4FP Bathroom v7.17

**Fichier** : `blueprints/blueprint_hvac_X4FP_bathroom.yaml`

**Ajouts** :
- 3 nouveaux inputs
- Section "# 5) PLANNING HORAIRE" avant "# 6) LUMIÈRE"
- Planning prioritaire sur gestion lumière
- Stop execution si planning actif (ignore lumière)

**Lignes modifiées** : +80 lignes

**Presets disponibles** : `[eco, comfort, comfort-1, comfort-2, away, boost]`

**Spécificité** : Le planning **remplace** la gestion par lumière si actif. Cela permet de garantir une salle de bain chaude selon un planning (ex: 06h-08h matin, 18h-20h soir) sans dépendre de la lumière.

### 4. X4FP Room v7.14

**Fichier** : `blueprints/blueprint_hvac_X4FP_room.yaml`

**Ajouts** :
- 3 nouveaux inputs
- Section "# 5) PLANNING HORAIRE" avant "# 6) THERMIQUE"
- Planning prioritaire sur contrôle thermique
- Stop execution si planning actif (ignore thermique)

**Lignes modifiées** : +77 lignes

**Presets disponibles** : `[eco, comfort, comfort-1, comfort-2, away, boost]`

**Spécificité** : Le planning **remplace** le contrôle thermique si actif. Cela permet de forcer des périodes de confort (ex: matin, soirée) sans dépendre de la température mesurée.

---

## 📚 Documentation

### Nouveaux Fichiers

1. **GUIDE_PLANNING.md** (Guide complet)
   - Principe de fonctionnement détaillé (1 schedule avec périodes multiples)
   - Configuration étape par étape
   - 4 exemples concrets d'utilisation
   - Configuration par blueprint
   - Checklist de vérification
   - Section dépannage complète

### Fichiers Mis à Jour

1. **CHANGELOG.md**
   - Section complète "Planning Horaire"
   - Exemples de configuration 1-schedule
   - Documentation des nouveautés

2. **README.md**
   - Versions mises à jour (v3.7, v2.10, v7.17, v7.14)
   - Ajout "Planning horaire" dans fonctionnalités principales
   - Mise à jour descriptions de chaque blueprint
   - Lien vers GUIDE_PLANNING.md

3. **CHECKLIST_TESTS.md**
   - Section "Tests Planning Horaire" pour chaque blueprint
   - Tests de priorité avec alarme
   - Vérification traces et logbook
   - Tests schedule ON/OFF

---

## 🔄 Rétrocompatibilité

✅ **100% compatible** avec les configurations existantes

**Pourquoi ?**
- Tous les nouveaux paramètres sont **optionnels**
- Valeurs par défaut : schedule vide (`""`)
- Logique : `schedule_preset = "none"` si aucun schedule configuré
- Comportement identique si planning non configuré

**Actions requises** :
- ✅ **Aucune** : Les automatisations existantes continuent de fonctionner
- ⚠️ **Recommandé** : Recharger les automatisations pour bénéficier des dernières corrections

---

## 🧪 Tests Effectués

### Scénarios Testés

- [x] Configuration sans schedule → Comportement identique à avant
- [x] Configuration avec schedule → Planning actif selon état ON/OFF
- [x] Alarme armée + schedule ON → Planning ignoré, alarme prioritaire
- [x] Alarme désarmée + schedule ON → Planning appliqué (preset_schedule_on)
- [x] Alarme désarmée + schedule OFF → Planning appliqué (preset_schedule_off)
- [x] Été actif + schedule → Été prioritaire
- [x] Fenêtre ouverte + schedule → Fenêtre prioritaire
- [x] Solar Optimizer + schedule → Solar prioritaire
- [x] Traces : trigger `schedule_change` détecté
- [x] Logbook : messages `📅 Planning → COMFORT` ou `📅 Planning → ECO` corrects

### Blueprints Validés

- [x] Thermostat Heat v3.7
- [x] Room Thermostat v2.10
- [x] X4FP Bathroom v7.17
- [x] X4FP Room v7.14

---

## 📊 Statistiques

### Lignes de Code

| Fichier | Lignes Ajoutées | Lignes Supprimées | Total |
|---------|-----------------|-------------------|-------|
| blueprint_hvac_thermostat_heat.yaml | +77 | -2 | +75 |
| blueprint_hvac_room_thermostat.yaml | +72 | -2 | +70 |
| blueprint_hvac_X4FP_bathroom.yaml | +82 | -2 | +80 |
| blueprint_hvac_X4FP_room.yaml | +79 | -2 | +77 |
| GUIDE_PLANNING.md | +647 | 0 | +647 |
| CHANGELOG.md | +97 | -87 | +10 |
| README.md | +10 | -7 | +3 |
| CHECKLIST_TESTS.md | +47 | -37 | +10 |
| **TOTAL** | **+1111** | **-139** | **+972** |

### Commits

```
89eb6cf feat: ajout planning horaire avec schedules HA dans tous les blueprints
57dd567 docs: ajout guide complet planning horaire + mise à jour tests
eee4292 docs: ajout diagnostic complet problème alarme
91bb2f2 docs: ajout guide de migration v3.6/v2.9/v7.16/v7.13
```

---

## 💡 Exemples d'Utilisation

### Exemple 1 : Planning Semaine de Travail

**Contexte** : Maison vide en journée du lundi au vendredi.

**Schedule à créer** :
```yaml
Nom: Chauffage Confort
ID: schedule.chauffage_confort

Configuration:
  Lundi-Vendredi:
    - 06:00-08:00  # Matin
    - 17:00-22:00  # Soir
  Samedi-Dimanche:
    - 08:00-22:00  # Toute la journée
```

**Configuration Blueprint** :
```yaml
schedule_entity: schedule.chauffage_confort
preset_schedule_on: comfort   # Pendant les périodes définies
preset_schedule_off: eco      # En dehors des périodes
```

**Résultat** :
- 06:00 → Schedule ON → COMFORT (réveil)
- 08:00 → Schedule OFF → ECO (départ travail)
- 17:00 → Schedule ON → COMFORT (retour maison)
- 22:00 → Schedule OFF → ECO (sommeil)

### Exemple 2 : Télétravail

**Contexte** : Télétravail certains jours (mercredi et vendredi).

**Schedule à créer** :
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

### Exemple 3 : Économies Nocturnes Importantes

**Contexte** : Réduire fortement le chauffage la nuit.

**Schedule à créer** :
```yaml
schedule.chauffage_jour:
  Tous les jours:
    - 06:00-23:00  # Journée complète
```

**Configuration Blueprint** :
```yaml
schedule_entity: schedule.chauffage_jour
preset_schedule_on: comfort
preset_schedule_off: frost_protection  # Hors-gel la nuit
```

---

## 🐛 Issues Résolues

Cette PR résout également les issues liées aux détections sensibles à la casse (déjà corrigées dans versions précédentes) :

- ✅ Alarme ne déclenchant pas le changement de mode (casse)
- ✅ Mode été non détecté si état avec majuscules
- ✅ Solar Optimizer non détecté correctement

**Correctifs inclus** (de la PR précédente) :
- Détection alarme avec `.lower()`
- Détection été avec `.lower()`
- Détection Solar Optimizer avec `.lower()`
- Détection lumière avec `.lower()` (X4FP Bathroom)

---

## ⚠️ Breaking Changes

**Aucun breaking change !**

Tous les changements sont rétrocompatibles. Les utilisateurs peuvent :
- Continuer d'utiliser leurs configurations actuelles
- Migrer progressivement vers le planning horaire
- Tester le planning sur une automatisation avant de l'appliquer partout

---

## 📖 Documentation pour les Utilisateurs

Après merge, les utilisateurs devront :

1. **Importer les nouveaux blueprints** (ou mettre à jour)
2. **Optionnel** : Créer un schedule dans Home Assistant avec leurs périodes de confort
3. **Optionnel** : Configurer le planning dans leurs automatisations (3 paramètres)
4. **Recommandé** : Consulter [GUIDE_PLANNING.md](GUIDE_PLANNING.md)

**Guides disponibles** :
- [GUIDE_PLANNING.md](GUIDE_PLANNING.md) : Configuration complète du planning (1 schedule)
- [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md) : Migration depuis anciennes versions
- [CHECKLIST_TESTS.md](CHECKLIST_TESTS.md) : Tests de validation

---

## 🚀 Prochaines Étapes

Après merge de cette PR :

- [ ] Annoncer la nouvelle fonctionnalité sur le forum Home Assistant
- [ ] Créer des exemples de dashboards avec schedules
- [ ] Potentiellement ajouter support pour calendrier (au lieu de schedules)
- [ ] Recueillir retours utilisateurs sur le planning horaire

---

## 🙏 Remerciements

Merci à la communauté Home Assistant pour les suggestions d'amélioration et les retours sur les blueprints.

---

**Cette PR est prête pour review et merge ! 🎉**
