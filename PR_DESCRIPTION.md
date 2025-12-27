# Pull Request : Planning Horaire avec Schedules Home Assistant

## 📅 Nouvelle Fonctionnalité Majeure

Ajout du **planning horaire** avec support des entités `schedule` de Home Assistant dans **tous les 4 blueprints**.

---

## ✨ Résumé des Changements

### Blueprints Mis à Jour

| Blueprint | Ancienne Version | Nouvelle Version | Changements Principaux |
|-----------|------------------|------------------|------------------------|
| **Thermostat Heat** | v3.6 | **v3.7** | Planning horaire + compatibilité rétro |
| **Room Thermostat** | v2.9 | **v2.10** | Planning horaire + été/hiver |
| **X4FP Bathroom** | v7.16 | **v7.17** | Planning prioritaire sur lumière |
| **X4FP Room** | v7.13 | **v7.14** | Planning prioritaire sur thermique |

### Fonctionnalités Ajoutées

✅ **4 périodes configurables** : Matin, Journée, Soirée, Nuit
✅ **Utilise les entités `schedule`** de Home Assistant (Helpers)
✅ **Chaque période** a son preset associé (eco, comfort, boost, etc.)
✅ **Actif uniquement si alarme désarmée** (présence à la maison)
✅ **Rétrocompatible** : configurations existantes fonctionnent sans modification

---

## 🎯 Ordre de Priorité

```
1. Été → OFF (ou ECO selon configuration)
2. Fenêtre ouverte → OFF (ou preset fenêtre)
3. Solar Optimizer → COMFORT (si actif)
4. Alarme ARMÉE → preset alarme (ignore le planning)
5. ⭐ Alarme DÉSARMÉE + Planning actif → preset du planning
6. Alarme DÉSARMÉE + Pas de planning → comportement par défaut
```

**Résultat** :
- 🏠 **Présent** (alarme off) : Suit le planning horaire
- 🚪 **Absent** (alarme on) : Force eco/away (ignore le planning)

---

## 📝 Modifications Techniques

### 1. Thermostat Heat v3.7

**Fichier** : `blueprints/blueprint_hvac_thermostat_heat.yaml`

**Ajouts** :
- 8 nouveaux inputs (4 schedules + 4 presets)
- Variables `sched_morning`, `preset_morning`, etc.
- Trigger `schedule_change` qui surveille les 4 schedules
- Logique `schedule_preset` dans variables runtime
- Logique `target_preset` intègre le schedule

**Lignes modifiées** : +80 lignes

### 2. Room Thermostat v2.10

**Fichier** : `blueprints/blueprint_hvac_room_thermostat.yaml`

**Ajouts** :
- 8 nouveaux inputs avec presets adaptés (none, eco, comfort, home, away, boost)
- Variables schedule
- Trigger `schedule_change`
- Logique schedule intégrée dans section "Alarme désarmée"

**Lignes modifiées** : +75 lignes

### 3. X4FP Bathroom v7.17

**Fichier** : `blueprints/blueprint_hvac_X4FP_bathroom.yaml`

**Ajouts** :
- 8 nouveaux inputs
- Section "# 5) PLANNING HORAIRE" avant "# 6) LUMIÈRE"
- Planning prioritaire sur gestion lumière
- Stop execution si planning actif (ignore lumière)

**Lignes modifiées** : +85 lignes

### 4. X4FP Room v7.14

**Fichier** : `blueprints/blueprint_hvac_X4FP_room.yaml`

**Ajouts** :
- 8 nouveaux inputs
- Section "# 5) PLANNING HORAIRE" avant "# 6) THERMIQUE"
- Planning prioritaire sur contrôle thermique
- Stop execution si planning actif (ignore thermique)

**Lignes modifiées** : +82 lignes

---

## 📚 Documentation

### Nouveaux Fichiers

1. **GUIDE_PLANNING.md** (18 pages)
   - Principe de fonctionnement détaillé
   - Configuration étape par étape
   - 4 exemples concrets d'utilisation
   - Configuration par blueprint
   - Checklist de vérification
   - Section dépannage complète

### Fichiers Mis à Jour

1. **CHANGELOG.md**
   - Section complète "Planning Horaire"
   - Exemples de configuration
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

---

## 🔄 Rétrocompatibilité

✅ **100% compatible** avec les configurations existantes

**Pourquoi ?**
- Tous les nouveaux paramètres sont **optionnels**
- Valeurs par défaut : schedules vides (`""`)
- Logique : `schedule_preset = "none"` si aucun schedule configuré
- Comportement identique si planning non configuré

**Actions requises** :
- ✅ **Aucune** : Les automatisations existantes continuent de fonctionner
- ⚠️ **Recommandé** : Recharger les automatisations pour bénéficier des dernières corrections

---

## 🧪 Tests Effectués

### Scénarios Testés

- [x] Configuration sans schedule → Comportement identique à avant
- [x] Configuration avec 1 schedule → Planning actif pour cette période
- [x] Configuration avec 4 schedules → Transitions correctes entre périodes
- [x] Alarme armée + schedule actif → Planning ignoré, alarme prioritaire
- [x] Alarme désarmée + schedule actif → Planning appliqué
- [x] Été actif + schedule → Été prioritaire
- [x] Fenêtre ouverte + schedule → Fenêtre prioritaire
- [x] Solar Optimizer + schedule → Solar prioritaire
- [x] Traces : trigger `schedule_change` détecté
- [x] Logbook : messages `📅 Planning → PRESET` corrects

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
| blueprint_hvac_thermostat_heat.yaml | +82 | -2 | +80 |
| blueprint_hvac_room_thermostat.yaml | +77 | -2 | +75 |
| blueprint_hvac_X4FP_bathroom.yaml | +87 | -2 | +85 |
| blueprint_hvac_X4FP_room.yaml | +84 | -2 | +82 |
| GUIDE_PLANNING.md | +603 | 0 | +603 |
| CHANGELOG.md | +87 | 0 | +87 |
| README.md | +10 | -7 | +3 |
| CHECKLIST_TESTS.md | +37 | -17 | +20 |
| **TOTAL** | **+1067** | **-32** | **+1035** |

### Commits

```
57dd567 docs: ajout guide complet planning horaire + mise à jour tests
89eb6cf feat: ajout planning horaire avec schedules HA dans tous les blueprints
eee4292 docs: ajout diagnostic complet problème alarme
91bb2f2 docs: ajout guide de migration v3.6/v2.9/v7.16/v7.13
```

---

## 💡 Exemples d'Utilisation

### Exemple 1 : Planning Semaine de Travail

**Contexte** : Maison vide en journée du lundi au vendredi.

**Configuration** :
```yaml
schedule_morning: schedule.chauffage_matin  # Lun-Ven 06:00-08:00
morning_preset: comfort

schedule_day: schedule.chauffage_journee    # Lun-Ven 08:00-17:00
day_preset: eco

schedule_evening: schedule.chauffage_soiree # Tous les jours 17:00-22:00
evening_preset: comfort

schedule_night: schedule.chauffage_nuit     # Tous les jours 22:00-06:00
night_preset: eco
```

**Résultat** :
- 06:00 → COMFORT (réveil)
- 08:00 → ECO (départ travail)
- 17:00 → COMFORT (retour maison)
- 22:00 → ECO (sommeil)

### Exemple 2 : Télétravail

**Contexte** : Télétravail certains jours (mercredi et vendredi).

**Configuration** :
```yaml
schedule_day: schedule.teletravail  # Mer-Ven 08:00-17:00
day_preset: comfort  # Confort en télétravail
```

---

## 🐛 Issues Résolues

Cette PR résout également les issues liées aux détections sensibles à la casse :

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
2. **Optionnel** : Créer des schedules dans Home Assistant
3. **Optionnel** : Configurer le planning dans leurs automatisations
4. **Recommandé** : Consulter [GUIDE_PLANNING.md](GUIDE_PLANNING.md)

**Guides disponibles** :
- [GUIDE_PLANNING.md](GUIDE_PLANNING.md) : Configuration complète du planning
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
