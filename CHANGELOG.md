# Changelog

Tous les changements notables de ce projet seront documentés dans ce fichier.

Le format est basé sur [Keep a Changelog](https://keepachangelog.com/fr/1.0.0/),
et ce projet adhère au [Semantic Versioning](https://semver.org/lang/fr/).

---

## [2025-01-16] - Corrections bugs et améliorations

### 🐛 Corrections Critiques
- **Room Thermostat v2.2** : Correction bug variable `current_temp` manquante
  - Variable utilisée ligne 313 mais non définie
  - Impact : Blueprint NON FONCTIONNEL pour application température
  - Fix : Ajout `current_temp` dans variables runtime (ligne 194)
  - **Origine** : Suppression accidentelle lors simplification du 15 janvier

### ✨ Améliorations
- **Thermostat Heat v3.2** : Triggers alarme conditionnels
  - Remplacé 2 triggers `state` par 1 trigger `template` conditionnel
  - Automation maintenant créable SANS alarme configurée
  - Cohérence avec Room Thermostat et X4FP blueprints
  - Réduction -13 lignes de code

### 📝 Documentation
- **Room Thermostat** : Description `alarm_entity` clarifiée "Requis" → "Optionnel"
- **Thermostat Heat** : Description `alarm_entity` clarifiée
- Ajout `RAPPORT_CONTROLE_EXECUTION.md` (contrôle minutieux 4 blueprints)
- Ajout `VERIFICATION_CORRECTIONS.md` (validation corrections)

### 🔧 Versions
- Room Thermostat : v2.1 → **v2.2**
- Thermostat Heat : v3.1 → **v3.2**

### ✅ Statut Final
- Room Thermostat v2.2 : 🟢 FONCTIONNEL (était NON FONCTIONNEL)
- Thermostat Heat v3.2 : 🟢 FONCTIONNEL (avec ou sans alarme)
- X4FP Bathroom v7.3 : 🟢 FONCTIONNEL
- X4FP Room v7.3 : 🟢 FONCTIONNEL

---

## [2025-01-16] - Uniformisation et optimisations finales

### 🔧 Modifié
- **Tous les blueprints** : Uniformisation de la variable `tick`
  - `tick_m` → `tick_minutes` dans Room Thermostat et Thermostat Heat
  - Cohérence nomenclature avec X4FP Bathroom et X4FP Room
- **Versions bumpées** :
  - Thermostat Heat : v3.0 → v3.1
  - Room Thermostat : v2.0 → v2.1
  - X4FP Bathroom : v7.2 → v7.3
  - X4FP Room : v7.2 → v7.3

### 📝 Documentation
- Ajout CHANGELOG.md pour suivi des modifications

---

## [2025-01-15] - Simplifications et corrections critiques

### 🐛 Corrections Critiques
- **X4FP Bathroom** : Correction bug variable `tick_m` non définie dans trigger (ligne 172)
  - Variable était référencée avant définition
  - Impact : tick périodique pouvait ne pas fonctionner

### ✨ Optimisations (45 lignes supprimées, -3.2%)

#### Thermostat Heat (v3.0)
- Suppression variable inutilisée `trig` (ligne 199)
- Simplification template `is_summer` : 6 lignes → 2 lignes
- Simplification template `is_away` : 5 lignes → 2 lignes
- Simplification trigger `optional_entities_change` : 14 lignes → 6 lignes
- **Total économisé** : 12 lignes

#### Room Thermostat (v2.0)
- Suppression variable inutilisée `current_temp` (ligne 202)
- Simplification trigger `optional_entities_change` : 14 lignes → 6 lignes
- **Total économisé** : 11 lignes

#### X4FP Bathroom (v7.2)
- Correction bug critique `tick_m` (voir ci-dessus)
- Simplification trigger `optional_entities_change` : 14 lignes → 6 lignes
- Suppression `default([])` redondant dans `win_any_open`
- **Total économisé** : 8 lignes

#### X4FP Room (v7.2)
- Simplification clamp consigne : 6 lignes → 1 ligne (utilisation min/max)
  ```yaml
  # Avant
  {% set s = sp_raw %}
  {% if s < sp_min %} {{ sp_min }}
  {% elif s > sp_max %} {{ sp_max }}
  {% else %} {{ s }}
  {% endif %}

  # Après
  {{ [[sp_raw, sp_max] | min, sp_min] | max }}
  ```
- Simplification trigger `optional_entities_change` : 14 lignes → 6 lignes
- Suppression `default([])` redondant dans `win_any_open`
- **Total économisé** : 14 lignes

### 📊 Impact
- **Code économisé** : 45 lignes (-3.2% du total)
- **Lisibilité** : Améliorée (templates plus concis)
- **Maintenance** : Simplifiée (code plus uniforme)
- **Fonctionnalité** : 100% préservée

### 📝 Documentation
- Ajout `ANALYSE_SIMPLIFICATION.md` (rapport complet 443 lignes)
  - Analyse détaillée des 4 blueprints
  - 12 optimisations identifiées
  - Comparaisons avant/après
  - Plan d'action et priorités

---

## [2025-01-14] - Organisation initiale du repository

### ✨ Nouveau
- **Structure de dossiers** :
  - `blueprints/` : Tous les blueprints YAML
  - `docs/` : Documentation complète
  - `images/` : Assets visuels

- **Blueprints déplacés** :
  - `blueprint_hvac_thermostat_heat.yaml`
  - `blueprint_hvac_room_thermostat.yaml`
  - `blueprint_hvac_X4FP_bathroom.yaml`
  - `blueprint_hvac_X4FP_room.yaml`

### 📝 Documentation (118 pages au total)
- `README.md` : Vue d'ensemble, tableau comparatif, FAQ
- `INSTALLATION.md` : Guide d'installation détaillé (3 méthodes)
- `CONTRIBUTING.md` : Guide de contribution
- `LICENSE` : Licence MIT
- `docs/thermostat_heat.md` : Documentation Thermostat Heat (18 pages)
- `docs/room_thermostat.md` : Documentation Room Thermostat (15 pages)
- `docs/x4fp_bathroom.md` : Documentation X4FP Bathroom (22 pages)
- `docs/x4fp_room.md` : Documentation X4FP Room (25 pages)
- `docs/solar_optimizer.md` : Guide Solar Optimizer (18 pages)
- `docs/troubleshooting.md` : Guide dépannage (20 pages, 30+ problèmes)

### 🔍 Vérifications
- ✅ Cohérence des 4 blueprints validée
- ✅ Fonctionnalité testée
- ✅ Documentation complète

---

## Versions des Blueprints

### HVAC – Thermostat Chauffage (Alarme = Eco/Confort)
- **v3.2** (2025-01-16) : Triggers alarme conditionnels, automation créable sans alarme
- **v3.1** (2025-01-16) : Uniformisation tick variable
- **v3.0** (2025-01-15) : Simplifications templates, suppression variable inutilisée
- **v2.x** : Versions antérieures (non documentées)

### HVAC – Pièce (Thermostat/Clim) – Été/Hiver + Away + SO
- **v2.2** (2025-01-16) : Correction bug current_temp manquante (critique)
- **v2.1** (2025-01-16) : Uniformisation tick variable
- **v2.0** (2025-01-15) : Simplifications templates, suppression variable inutilisée
- **v1.x** : Versions antérieures (non documentées)

### HVAC – X4FP Salle de bain (Lumière + SO compatible)
- **v7.3** (2025-01-16) : Nettoyage nom version
- **v7.2** (2025-01-15) : Correction critique bug tick_m, simplifications
- **v7.x** : Versions antérieures (non documentées)

### HVAC – X4FP (Thermique + SO compatible)
- **v7.3** (2025-01-16) : Nettoyage nom version
- **v7.2** (2025-01-15) : Simplification clamp, simplifications templates
- **v7.x** : Versions antérieures (non documentées)

---

## Légende

- **✨ Nouveau** : Nouvelles fonctionnalités ou fichiers
- **🔧 Modifié** : Changements dans des fonctionnalités existantes
- **🐛 Corrections** : Corrections de bugs
- **📝 Documentation** : Changements dans la documentation
- **⚡ Performance** : Améliorations de performance
- **🔒 Sécurité** : Corrections de sécurité
- **📊 Impact** : Résumé quantitatif des changements
