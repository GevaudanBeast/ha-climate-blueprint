# Home Assistant Climate Blueprints

Collection de blueprints Home Assistant pour la gestion intelligente du chauffage et de la climatisation.

## Description

Ce repository contient des blueprints d'automatisation Home Assistant pour gérer vos thermostats, climatiseurs et systèmes de chauffage fil pilote (X4FP). Ces blueprints offrent des fonctionnalités avancées tout en restant simples à configurer.

## Fonctionnalités principales

- Gestion automatique des modes Été/Hiver
- Détection d'ouverture de fenêtres avec temporisation
- Intégration avec les systèmes d'alarme (mode Away)
- Support du Solar Optimizer pour optimiser l'autoconsommation solaire
- Contrôle thermique avec hystérésis
- Gestion par preset ou température
- Ré-application périodique paramétrable (tick)
- Logs détaillés dans le logbook

## Blueprints disponibles

### 1. Thermostat Chauffage Simple (v3.6)
**Fichier:** `blueprint_hvac_thermostat_heat.yaml`

Blueprint pour thermostat de chauffage avec gestion alarme Eco/Confort.

**Cas d'usage:**
- Chauffage électrique simple
- Radiateurs avec thermostat intégré
- Chaudière avec thermostat

**Fonctionnalités:**
- Alarme armée → mode ECO
- Alarme désarmée → mode CONFORT
- Gestion fenêtres (pause chauffage)
- Mode été (arrêt automatique)
- Fallback température pour tous les presets
- Tick périodique

**[Documentation détaillée](docs/thermostat_heat.md)**

---

### 2. Thermostat/Climatisation Pièce (v2.9)
**Fichier:** `blueprint_hvac_room_thermostat.yaml`

Blueprint universel pour pièce avec thermostat ou climatisation réversible.

**Cas d'usage:**
- Climatisation réversible
- Pompe à chaleur air/air
- Radiateurs avec thermostat programmable

**Fonctionnalités:**
- Bascule automatique Été (cool) / Hiver (heat)
- Gestion Away avec presets (away/home/none)
- Support Solar Optimizer (prioritaire sur Away)
- Consignes de température paramétrables
- Gestion fenêtres en mode heat ET cool
- Tick périodique

**[Documentation détaillée](docs/room_thermostat.md)**

---

### 3. X4FP Salle de Bain avec Lumière (v7.16)
**Fichier:** `blueprint_hvac_X4FP_bathroom.yaml`

Blueprint pour sèche-serviettes fil pilote avec détection de présence via lumière.

**Cas d'usage:**
- Sèche-serviettes fil pilote
- Radiateur salle de bain avec détection présence
- Intégration Solar Optimizer

**Fonctionnalités:**
- Lumière ON → mode CONFORT
- Lumière OFF → mode ECO
- Mode toggle (bascule eco ↔ comfort)
- Solar Optimizer prioritaire
- Ordre de priorité: Été > Fenêtre > SO actif > Away > Lumière
- Autorisation SO en mode Away (optionnel)
- Tick périodique

**[Documentation détaillée](docs/x4fp_bathroom.md)**

---

### 4. X4FP Pièce avec Contrôle Thermique (v7.13)
**Fichier:** `blueprint_hvac_X4FP_room.yaml`

Blueprint pour radiateur fil pilote avec contrôle thermique par hystérésis.

**Cas d'usage:**
- Radiateurs électriques fil pilote
- Chauffage avec capteur de température externe
- Optimisation autoconsommation solaire

**Fonctionnalités:**
- Contrôle thermique avec hystérésis réglable
- Consigne via input_number
- Garde-fous température min/max
- Solar Optimizer prioritaire
- Ordre de priorité: Été > Fenêtre > SO actif > Away > Thermique
- Autorisation SO en mode Away (optionnel)
- Tick périodique

**[Documentation détaillée](docs/x4fp_room.md)**

---

## Installation

### Méthode 1 : Import direct (recommandé)

1. Ouvrez Home Assistant
2. Allez dans **Paramètres** → **Automatisations & Scènes** → **Blueprints**
3. Cliquez sur **Importer un Blueprint**
4. Collez l'URL du blueprint :

```
https://github.com/GevaudanBeast/ha-climate-blueprint/blob/main/blueprints/blueprint_hvac_thermostat_heat.yaml
https://github.com/GevaudanBeast/ha-climate-blueprint/blob/main/blueprints/blueprint_hvac_room_thermostat.yaml
https://github.com/GevaudanBeast/ha-climate-blueprint/blob/main/blueprints/blueprint_hvac_X4FP_bathroom.yaml
https://github.com/GevaudanBeast/ha-climate-blueprint/blob/main/blueprints/blueprint_hvac_X4FP_room.yaml
```

### Méthode 2 : Installation manuelle

Consultez le guide détaillé : **[INSTALLATION.md](INSTALLATION.md)**

---

## Comparaison des blueprints

| Caractéristique | Thermostat Heat | Room Thermostat | X4FP Bathroom | X4FP Room |
|-----------------|-----------------|-----------------|---------------|-----------|
| Type appareil | Thermostat simple | Thermostat/Clim | Fil pilote X4FP | Fil pilote X4FP |
| Été/Hiver auto | ❌ (Été = OFF) | ✅ (cool/heat) | ❌ (Été = OFF/ECO) | ❌ (Été = OFF/ECO) |
| Fenêtres | ✅ | ✅ | ✅ | ✅ |
| Alarme/Away | ✅ (preset) | ✅ (preset) | ✅ (preset) | ✅ (preset) |
| Solar Optimizer | ❌ | ✅ | ✅ (prioritaire) | ✅ (prioritaire) |
| Lumière | ❌ | ❌ | ✅ | ❌ |
| Contrôle thermique | ❌ | ❌ | ❌ | ✅ (hystérésis) |
| Preset/Température | Preset + fallback | Température | Preset | Preset |

---

## Prérequis

- Home Assistant 2023.8 ou supérieur
- Entités `climate.*` configurées pour vos thermostats/climatiseurs
- Entités `binary_sensor.*` pour fenêtres/portes (optionnel)
- Entité `alarm_control_panel.*` pour la gestion Away (optionnel)
- Solar Optimizer configuré (optionnel)
- Capteurs de température `sensor.*` (pour X4FP Room)
- Entités `input_number.*` pour consignes (pour X4FP Room)
- Entités `light.*` pour détection présence (pour X4FP Bathroom)

---

## Configuration rapide

1. Importez le blueprint souhaité
2. Créez une nouvelle automatisation basée sur ce blueprint
3. Configurez les entités requises
4. Ajustez les paramètres selon vos besoins
5. Sauvegardez et testez

Pour une configuration détaillée, consultez la documentation de chaque blueprint.

---

## Support et Contribution

### Signaler un bug

Ouvrez une [issue](https://github.com/GevaudanBeast/ha-climate-blueprint/issues) en décrivant :
- Le blueprint concerné
- Le comportement attendu vs observé
- Vos logs Home Assistant
- Votre configuration (anonymisée)

### Contribuer

Les Pull Requests sont les bienvenues ! Voir [CONTRIBUTING.md](CONTRIBUTING.md) pour les guidelines.

---

## Licence

MIT License - Voir [LICENSE](LICENSE)

---

## Auteur

**LaCaseHome**

---

## Changelog

### v7.16 / v7.13 / v2.9 / v3.6 (2025-12-18) - CORRECTION CRITIQUE COMPLÈTE
- **Fix détection complète** : Ajout `.lower()` pour toutes les détections d'état
- **Thermostat Heat v3.6** : Détection alarme robuste
- **Room Thermostat v2.9** : Détection Solar Optimizer robuste
- **X4FP Bathroom v7.16** : Détection été, alarme, Solar Optimizer ET lumière robustes
- **X4FP Room v7.13** : Détection été, alarme ET Solar Optimizer robustes
- **Résultat** : Tous les blueprints utilisent `.lower()` de manière cohérente
- Corrige bascules mode non déclenchées selon casse état (alarme, été, SO, lumière)

### v3.5 / v2.8 / v7.14 / v7.11 (2025-12-05) - CORRECTION CRITIQUE
- **Fix majeur** : Triggers alarme/été fiables (retour string vs booléen)
- Corrige bascules ECO/CONFORT non systématiques
- Élimine erreurs UndefinedError dans les logs
- Détection changements d'état garantie à 100%
- **Tous les blueprints corrigés** (8 triggers)

### v7.2 (X4FP Bathroom & Room)
- Fix trigger Solar Optimizer conditionnel
- Amélioration logs
- Support preset "none" pour SO

### v3.0 (Thermostat Heat)
- Triggers alarme STATE explicites et fiables
- Fallback température COMPLET pour tous les presets
- Tick optimisé + logs conditionnels

### v2.0 (Room Thermostat)
- Support Solar Optimizer
- Bascule auto été/hiver
- Preset away/home/none

---

## FAQ

**Q: Quel blueprint choisir pour mon radiateur électrique ?**

R:
- Radiateur avec thermostat intégré → `Thermostat Heat`
- Radiateur fil pilote sans capteur → `X4FP Bathroom` (si salle de bain) ou adapter `X4FP Room`
- Radiateur fil pilote avec capteur température → `X4FP Room`

**Q: Puis-je utiliser plusieurs blueprints ensemble ?**

R: Oui, mais évitez de créer plusieurs automatisations pour le même thermostat. Choisissez le blueprint le plus adapté à votre usage.

**Q: Le Solar Optimizer est-il obligatoire ?**

R: Non, c'est optionnel. Si non configuré, les blueprints fonctionnent normalement sans cette fonctionnalité.

**Q: Comment tester mes automatisations ?**

R: Utilisez le mode trace dans Home Assistant et consultez le logbook pour voir les actions effectuées.

---

## Documentation complémentaire

- [Guide d'installation détaillé](INSTALLATION.md)
- [Documentation Thermostat Heat](docs/thermostat_heat.md)
- [Documentation Room Thermostat](docs/room_thermostat.md)
- [Documentation X4FP Bathroom](docs/x4fp_bathroom.md)
- [Documentation X4FP Room](docs/x4fp_room.md)
- [Solar Optimizer Integration](docs/solar_optimizer.md)
- [Troubleshooting](docs/troubleshooting.md)

---

## Remerciements

Merci à la communauté Home Assistant pour les retours et suggestions d'amélioration.
