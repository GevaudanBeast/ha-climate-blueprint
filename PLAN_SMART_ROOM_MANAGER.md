# 🏠 Plan: Smart Room Manager (SRM)

## 🎯 Objectif

Créer un système de **gestion centralisée** pour le chauffage de toutes les pièces, qui :
- Gère les **priorités globales** (alarme, été, planning horaire)
- Définit les **presets cibles** pour chaque pièce
- Délègue l'**exécution locale** aux blueprints (fenêtres, thermique, SO)

---

## 📐 Architecture Cible

```
┌─────────────────────────────────────────────────────────┐
│          SMART ROOM MANAGER (Automation)                │
│                                                           │
│  Surveille:                                              │
│  - Alarme (armée/désarmée)                              │
│  - Été (indicateur saison)                              │
│  - Calendriers planning (par pièce ou global)           │
│  - Profils pièces (chambre, salon, salle de bain)      │
│                                                           │
│  Calcule:                                                │
│  - Preset cible pour chaque pièce                       │
│  - Selon priorités: Alarme > Planning > Défaut         │
│                                                           │
│  Applique:                                               │
│  - Définit input_select.preset_xxx pour chaque pièce   │
│  - Les blueprints écoutent ces input_select            │
└─────────────────────────────────────────────────────────┘
                              │
                              │ Commandes (input_select)
                              ▼
┌─────────────────────────────────────────────────────────┐
│              BLUEPRINTS (Par pièce)                      │
│                                                           │
│  Écoutent: input_select.preset_[piece]                  │
│                                                           │
│  Gèrent priorités LOCALES:                              │
│  1. Été (OFF si politique été)                          │
│  2. Fenêtre ouverte (OFF/Away)                          │
│  3. Solar Optimizer (prioritaire si actif)              │
│  4. Preset SRM (reçu via input_select)                  │
│  5. Thermique (X4FP) / Lumière (Bathroom)               │
│                                                           │
│  Appliquent:                                             │
│  - Changent preset thermostat                           │
│  - Gèrent logique spécifique (hystérésis, lumière)     │
└─────────────────────────────────────────────────────────┘
```

---

## 🎨 Concepts Clés

### 1. Séparation Responsabilités

#### Smart Room Manager (Global)
- ✅ Alarme armée/désarmée
- ✅ Planning horaire (calendriers)
- ✅ Profils de pièces
- ✅ Règles inter-pièces

#### Blueprints (Local)
- ✅ Été (OFF si nécessaire)
- ✅ Fenêtres ouvertes
- ✅ Solar Optimizer
- ✅ Thermique / Lumière
- ✅ Logique spécifique pièce

---

### 2. Communication via Input Select

Chaque pièce a un `input_select` pour recevoir son preset:

```yaml
input_select.preset_salon:
  options:
    - eco
    - comfort
    - away
    - boost
  current: comfort

input_select.preset_chambre:
  options:
    - eco
    - comfort
    - away
  current: eco
```

**SRM** met à jour ces input_select selon le contexte.
**Blueprints** écoutent ces input_select et appliquent les presets.

---

### 3. Profils de Pièces

Chaque pièce a un profil définissant son comportement:

```yaml
profils:
  chambre:
    alarme_armée: away
    alarme_désarmée: eco
    planning_actif: comfort
    planning_inactif: eco

  salon:
    alarme_armée: away
    alarme_désarmée: comfort
    planning_actif: comfort
    planning_inactif: comfort

  salle_de_bain:
    alarme_armée: away
    alarme_désarmée: eco
    planning_actif: lumière_autorisée
    planning_inactif: lumière_bloquée
```

---

## 📋 Fonctionnalités

### Phase 1: Gestion Basique

#### ✅ Fonctionnalités MVP
1. **Alarme globale** : armée → all away, désarmée → selon profil
2. **Profils pièces** : chambre, salon, salle de bain
3. **Input select** : 1 par pièce pour communiquer preset
4. **Blueprints compatibles** : écoutent input_select au lieu de calculer preset

#### 🔧 Configuration Minimale
```yaml
smart_room_manager:
  alarme: alarm_control_panel.maison
  pièces:
    - entity_id: input_select.preset_salon
      nom: "Salon"
      profil: salon
      preset_armée: away
      preset_désarmée: comfort

    - entity_id: input_select.preset_chambre
      nom: "Chambre"
      profil: chambre
      preset_armée: away
      preset_désarmée: eco
```

---

### Phase 2: Planning Horaire

#### ✅ Fonctionnalités Avancées
1. **Calendar global** : 1 calendrier pour toutes les pièces
2. **Calendars par pièce** : 1 calendrier par pièce (optionnel)
3. **Événements typés** : "confort", "eco", "boost" dans nom événement
4. **Preset par événement** : détermine preset selon type événement

#### 🔧 Configuration Avancée
```yaml
smart_room_manager:
  alarme: alarm_control_panel.maison

  planning:
    global: calendar.planning_chauffage_global  # Optionnel
    par_pièce:
      salon: calendar.planning_salon
      chambre: calendar.planning_chambre

  pièces:
    - entity_id: input_select.preset_salon
      nom: "Salon"
      profil: salon
      preset_armée: away
      preset_désarmée: comfort
      preset_planning_on: comfort
      preset_planning_off: eco
```

---

### Phase 3: Règles Inter-Pièces

#### ✅ Fonctionnalités Avancées
1. **Cascades** : "si salon confort, chambres max eco"
2. **Limites globales** : "max 3 pièces en confort simultanément"
3. **Priorisation** : "salon prioritaire sur chambres"
4. **Zones** : groupes de pièces (étage 1, étage 2)

#### 🔧 Configuration Avancée
```yaml
smart_room_manager:
  règles_inter_pièces:
    - nom: "Limite confort simultané"
      condition: count(preset=comfort) > 3
      action: "downgrade chambres à eco"

    - nom: "Cascade salon → chambres"
      condition: preset_salon == comfort
      action: "chambres max eco"
```

---

## 🏗️ Implémentation

### Étape 1: Créer Input Selects

Créer 1 input_select par pièce dans `configuration.yaml`:

```yaml
input_select:
  preset_salon:
    name: "Preset Salon (SRM)"
    options:
      - eco
      - comfort
      - away
      - boost
    initial: comfort

  preset_chambre:
    name: "Preset Chambre (SRM)"
    options:
      - eco
      - comfort
      - away
    initial: eco

  preset_salle_de_bain:
    name: "Preset SDB (SRM)"
    options:
      - eco
      - comfort
      - away
    initial: eco
```

---

### Étape 2: Créer Smart Room Manager

Blueprint ou automation centrale qui:

```yaml
alias: "Smart Room Manager"
trigger:
  - platform: state
    entity_id: alarm_control_panel.maison
  - platform: state
    entity_id: calendar.planning_chauffage
  # ... autres triggers

action:
  - variables:
      is_away: "{{ is_state('alarm_control_panel.maison', 'armed_away') }}"
      planning_on: "{{ is_state('calendar.planning_chauffage', 'on') }}"

  # Calculer preset pour chaque pièce
  - service: input_select.select_option
    target:
      entity_id: input_select.preset_salon
    data:
      option: >-
        {% if is_away %}
          away
        {% elif planning_on %}
          comfort
        {% else %}
          comfort
        {% endif %}

  - service: input_select.select_option
    target:
      entity_id: input_select.preset_chambre
    data:
      option: >-
        {% if is_away %}
          away
        {% elif planning_on %}
          comfort
        {% else %}
          eco
        {% endif %}
```

---

### Étape 3: Adapter Blueprints

Modifier blueprints pour écouter input_select au lieu de calculer preset:

#### Avant (Blueprint calcule preset)
```yaml
variables:
  is_away: "{{ states('alarm_control_panel.maison') | ... }}"
  target_preset: >-
    {% if is_away %}
      {{ preset_armed }}
    {% else %}
      {{ preset_disarmed }}
    {% endif %}
```

#### Après (Blueprint écoute SRM)
```yaml
input:
  srm_preset:
    name: "Input Select SRM (preset cible)"
    selector:
      entity:
        domain: input_select

trigger:
  - platform: state
    entity_id: !input srm_preset

variables:
  target_preset: "{{ states(srm_preset) }}"
```

**Priorités locales** (fenêtres, SO) restent dans le blueprint et **override** SRM si nécessaire.

---

## 🎯 Avantages Smart Room Manager

### ✅ Simplicité
- **1 seul endroit** pour gérer alarme/planning
- **Configuration centralisée** des profils pièces
- **Blueprints simplifiés** (pas de gestion alarme)

### ✅ Flexibilité
- **Planning global ou par pièce**
- **Profils personnalisables**
- **Règles inter-pièces** (si nécessaire)

### ✅ Maintenabilité
- **1 automation** à déboguer pour alarme/planning
- **Blueprints focalisés** sur logique locale
- **Évolutions centralisées** (nouveau profil = 1 ligne)

### ✅ Performance
- **Calculs centralisés** (1 fois au lieu de N fois)
- **Moins de triggers** (SRM déclenche, blueprints réagissent)
- **Optimisation possible** (batch updates)

---

## 📊 Comparaison Approches

### Approche Actuelle (Blueprints Autonomes)

```
Blueprint Salon
├─ Calcule is_away (alarme)
├─ Calcule schedule_preset (planning)
└─ Applique preset

Blueprint Chambre
├─ Calcule is_away (alarme)          ← Duplication
├─ Calcule schedule_preset (planning) ← Duplication
└─ Applique preset

Blueprint SDB
├─ Calcule is_away (alarme)          ← Duplication
├─ Calcule schedule_preset (planning) ← Duplication
└─ Applique preset
```

**Problème** : Logique dupliquée N fois, difficile à maintenir

---

### Approche SRM (Centralisée)

```
Smart Room Manager
├─ Calcule is_away (alarme)          ← 1 seule fois
├─ Calcule planning (1 ou N calendars)
├─ Définit preset_salon → input_select
├─ Définit preset_chambre → input_select
└─ Définit preset_sdb → input_select

Blueprint Salon
├─ Écoute input_select.preset_salon
├─ Gère fenêtres locales
├─ Gère Solar Optimizer
└─ Applique preset

Blueprint Chambre
├─ Écoute input_select.preset_chambre
├─ Gère fenêtres locales
└─ Applique preset

Blueprint SDB
├─ Écoute input_select.preset_sdb
├─ Gère fenêtres + lumière
└─ Applique preset
```

**Avantage** : Logique centralisée, blueprints simples et focalisés

---

## 🚀 Roadmap

### Milestone 1: MVP (1-2h)
- [ ] Créer input_selects pour toutes les pièces
- [ ] Créer SRM automation basique (alarme seulement)
- [ ] Adapter 1 blueprint pour tester (thermostat_heat)
- [ ] Tester avec alarme armée/désarmée

### Milestone 2: Planning (2-3h)
- [ ] Ajouter support calendar global dans SRM
- [ ] Ajouter profils pièces (chambre, salon, sdb)
- [ ] Adapter tous les blueprints pour écouter input_select
- [ ] Tester avec planning actif/inactif

### Milestone 3: Avancé (optionnel)
- [ ] Support calendars par pièce
- [ ] Règles inter-pièces (cascades, limites)
- [ ] Dashboard de monitoring SRM
- [ ] Logs et diagnostics

---

## 📝 Décisions à Prendre

### 1. Scope Initial
- **MVP** : Alarme seulement ?
- **Planning** : Inclure dès le début ?
- **Inter-pièces** : Plus tard ou jamais ?

### 2. Architecture
- **Blueprint SRM** ou **Automation YAML** ?
- **Input select** ou **input_text JSON** ?
- **Profils statiques** ou **configurables** ?

### 3. Migration
- **Progressive** : 1 pièce à la fois ?
- **Big bang** : Toutes les pièces d'un coup ?
- **Cohabitation** : SRM + blueprints autonomes en parallèle ?

---

## 💬 Questions Ouvertes

1. **Planning** : 1 calendar global ou 1 par pièce ?
2. **Profils** : Statiques (3-4 types) ou totalement configurables ?
3. **Bathroom** : Garde planning local ou migre vers SRM ?
4. **Solar Optimizer** : Reste local (blueprints) ou monte dans SRM ?
5. **Fenêtres** : Restent locales ou remontent dans SRM ?

---

## 🎓 Inspiration / Références

### Systèmes similaires existants
- **Schedy** (AppDaemon) : Scheduling avancé, mais complexe
- **Heating Control** (HACS) : Blueprint scheduling, mais limité
- **Room Assistant** : Présence par pièce, pas de chauffage

### Architecture inspirée de:
- **Microservices** : séparation préoccupations (SRM = orchestrateur, blueprints = workers)
- **Event-driven** : SRM publie états, blueprints réagissent
- **CQRS** : SRM commande (write), blueprints exécutent (write), dashboards lisent (read)

---

## ✅ Prochaines Actions

1. **Valider scope** avec utilisateur (MVP ou complet ?)
2. **Créer input_selects** dans configuration.yaml
3. **Prototyper SRM MVP** (alarme seulement)
4. **Adapter 1 blueprint** pour tester
5. **Itérer** selon résultats

---

**Statut** : 📝 Planification
**Prochaine étape** : Décider du scope et démarrer implémentation MVP
