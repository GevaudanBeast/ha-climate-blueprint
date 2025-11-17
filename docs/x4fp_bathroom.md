# Blueprint : X4FP Salle de Bain avec Lumière (v7.5)

## Description

Blueprint pour sèche-serviettes ou radiateur fil pilote X4FP avec détection de présence via lumière et intégration Solar Optimizer prioritaire. Spécialement conçu pour salle de bain.

**Fichier :** `blueprint_hvac_X4FP_bathroom.yaml`
**Version :** 7.5
**Auteur :** LaCaseHome

---

## Cas d'usage

- Sèche-serviettes fil pilote
- Radiateur salle de bain avec détection présence
- Chauffage avec optimisation autoconsommation solaire
- Radiateur fil pilote 4 ordres (Confort, Eco, Hors-gel, Arrêt)
- Radiateur fil pilote 6 ordres (+ Confort-1, Confort-2)

---

## Qu'est-ce que X4FP ?

**X4FP** = Fil Pilote 4 ordres (ou 6 ordres)

C'est un protocole de commande pour radiateurs électriques français permettant d'envoyer des ordres via 2 fils (phase + fil pilote) :

**Ordres standards (4 ordres) :**
- **Confort** : Température de confort
- **Eco** : Confort -3°C ou -4°C
- **Hors-gel** : Température minimale (~7°C)
- **Arrêt** : Radiateur éteint

**Ordres étendus (6 ordres) :**
- **Confort -1** : Confort -1°C
- **Confort -2** : Confort -2°C

En Home Assistant, ces ordres sont gérés via des **presets** sur l'entité climate.

---

## Fonctionnalités principales

### 1. Détection présence via lumière (OBLIGATOIRE)

Le blueprint utilise la lumière de la salle de bain pour détecter la présence :

**Comportements configurables :**

| Mode | Lumière ON | Lumière OFF |
|------|------------|-------------|
| **Force comfort** (défaut) | CONFORT | ECO |
| **Force eco** | ECO | ECO |
| **Toggle** | Bascule ECO ↔ CONFORT | Pas d'action |
| **Aucun** | Pas d'action | ECO |

**Exemple d'usage :**
- Douche le matin (lumière ON) → Confort (chaud)
- Nuit (lumière OFF) → Eco (tiède)

### 2. Solar Optimizer (PRIORITAIRE)

Si configuré, le Solar Optimizer a la **priorité absolue** :

- SO actif (ON) → Preset configuré (défaut : comfort)
- SO inactif (OFF) → Blueprint reprend le contrôle

**Autorisation SO en mode Away :**
- Si activé : SO peut chauffer même quand alarme armée
- Si désactivé : Away bloque SO

### 3. Ordre de priorité strict

```
1. Été (OFF ou ECO)
2. Fenêtre ouverte (AWAY ou preset configuré)
3. Solar Optimizer ACTIF (COMFORT ou preset configuré)
4. Alarme armée / Away (AWAY ou preset configuré)
5. Lumière (CONFORT ou ECO selon configuration)
```

### 4. Gestion été/hiver

**Politique en Été :**
- **OFF** : Radiateur arrêté
- **ECO** : Radiateur en mode éco (température minimale)

### 5. Fenêtres (optionnel)

- Détection ouverture avec délai
- Preset configurable (défaut : away = hors-gel)

### 6. Tick périodique

- Ré-application des réglages toutes les N minutes
- Évite les désynchronisations

---

## Configuration

### Paramètres obligatoires

| Paramètre | Type | Description |
|-----------|------|-------------|
| **Nom logique de la pièce** | Texte | Nom pour les logs (ex: "SdB Étage") |
| **Entité climate (X4FP)** | climate.* | Module fil pilore (X4FP) |
| **Lumière de la pièce** | light.* | **OBLIGATOIRE** - Lumière pour détection présence |

### Paramètres optionnels

#### Comportement Lumière

| Paramètre | Type | Défaut | Options |
|-----------|------|--------|---------|
| **Quand lumière ON** | Select | force_comfort | force_comfort / force_eco / toggle_on / none |
| **Quand lumière OFF** | Select | force_eco | force_eco / force_comfort / none |

#### Fenêtres

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| **Capteurs fenêtre(s)/porte(s)** | binary_sensor.* | [] | Liste des capteurs (ON = ouvert) |
| **Délai avant PAUSE (min)** | Nombre | 2 | Minutes avant pause |
| **Délai avant REPRISE (min)** | Nombre | 2 | Minutes avant reprise |

#### Été / Away

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| **Indicateur Été** | entity | "" | ON = Été (calendar/input_boolean/sensor) |
| **Politique en Été** | Select | off | "OFF (arrêté)" / "ECO (laisser en éco)" |
| **Alarme (Alarmo)** | alarm_control_panel.* | "" | Away si armed_* |

#### Solar Optimizer

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| **Solar Optimizer – SWITCH** | switch.* | "" | Switch d'action SO (ON = SO chauffe) |
| **Autoriser SO en Away** | Boolean | false | SO peut chauffer en mode Away ? |
| **Preset quand SO chauffe** | Select | comfort | Preset à appliquer quand SO actif |

#### Presets

| Paramètre | Type | Défaut | Options |
|-----------|------|--------|---------|
| **Preset "confort"** | Select | comfort | eco / comfort / comfort-1 / comfort-2 / away / boost |
| **Preset "éco"** | Select | eco | eco / comfort / comfort-1 / comfort-2 / away / boost |
| **Preset fenêtre ouverte** | Select | away | eco / comfort / comfort-1 / comfort-2 / away / boost |
| **Preset Away** | Select | away | eco / comfort / comfort-1 / comfort-2 / away / boost |

#### Tick

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| **Fréquence ré-application (min)** | Nombre | 10 | 0 pour désactiver, sinon 1-120 min |

---

## Exemples de configuration

### Exemple 1 : Salle de bain basique (sans SO)

```yaml
Nom de la pièce: Salle de Bain
Entité climate: climate.x4fp_sdb
Lumière: light.sdb
Quand lumière ON: force_comfort
Quand lumière OFF: force_eco
Preset "confort": comfort
Preset "éco": eco
```

**Comportement :**
- Lumière ON → CONFORT
- Lumière OFF → ECO
- Pas de gestion été/fenêtre/alarme/SO

### Exemple 2 : Avec fenêtre et été

```yaml
Nom de la pièce: Salle de Bain RDC
Entité climate: climate.x4fp_sdb_rdc
Lumière: light.sdb_rdc
Capteurs fenêtre: binary_sensor.fenetre_sdb_rdc
Indicateur Été: input_boolean.ete
Politique en Été: off
Preset fenêtre ouverte: away
Fréquence ré-application: 10 min
```

**Comportement :**
- Été → OFF (radiateur arrêté)
- Fenêtre ouverte → AWAY (hors-gel)
- Lumière ON → CONFORT
- Lumière OFF → ECO

### Exemple 3 : Avec Solar Optimizer complet

```yaml
Nom de la pièce: Salle de Bain Étage
Entité climate: climate.x4fp_seche_serviette
Lumière: light.sdb_etage
Alarme: alarm_control_panel.alarmo
Solar Optimizer – SWITCH: switch.solar_optimizer_seche_serviette
Autoriser SO en Away: true
Preset quand SO chauffe: comfort
Quand lumière ON: force_comfort
Quand lumière OFF: force_eco
Preset Away: away
Fréquence ré-application: 10 min
```

**Comportement :**
- **SO ON** → CONFORT (priorité absolue, même si Away)
- **SO OFF + Away** → AWAY
- **SO OFF + Home + Lumière ON** → CONFORT
- **SO OFF + Home + Lumière OFF** → ECO

**Scénario :**
```
8h00 - Surplus solaire disponible
→ SO active le switch → CONFORT (même si personne)

9h00 - Plus de surplus
→ SO désactive le switch → Lumière OFF → ECO

18h00 - Douche (lumière ON)
→ CONFORT

18h30 - Fin douche (lumière OFF)
→ ECO
```

### Exemple 4 : Mode Toggle (avancé)

```yaml
Nom de la pièce: Salle de Bain
Entité climate: climate.x4fp_sdb
Lumière: light.sdb
Quand lumière ON: toggle_on
Quand lumière OFF: force_eco
```

**Comportement :**
- **1er allumage lumière** : ECO → CONFORT
- **2ème allumage lumière** : CONFORT → ECO
- **3ème allumage lumière** : ECO → CONFORT
- **Lumière OFF** : Toujours ECO

**Usage :** Permet d'alterner entre chauffé/pas chauffé à chaque allumage.

---

## Intégration Solar Optimizer

### Qu'est-ce que le Solar Optimizer ?

Système qui utilise le surplus de production solaire pour chauffer l'eau ou les radiateurs, optimisant ainsi l'autoconsommation.

### Configuration avec ce blueprint

Le blueprint surveille un **switch d'action** :
- **ON** = Solar Optimizer chauffe activement maintenant
- **OFF** = Solar Optimizer ne chauffe pas

**Exemple de switch :**
```yaml
Solar Optimizer – SWITCH: switch.solar_optimizer_seche_serviette
```

### Ordre de priorité

```
SO ACTIF (ON)
   ↓
Preset COMFORT (ou configuré)
   ↓
Logs: "⚡ SO ACTIF → COMFORT"
   ↓
Blueprint en retrait (stop)
```

Si **"Autoriser SO en Away" = true** :
- SO peut chauffer même si alarme armée
- Utile pour profiter du surplus même en absence

Si **"Autoriser SO en Away" = false** :
- Away bloque SO
- SO ne peut pas chauffer si alarme armée

### Preset "none" pour SO

Si vous configurez **"Preset quand SO chauffe" = none** :
- Le blueprint se met complètement en retrait
- SO pilote le radiateur comme il veut
- Logs : "⚡ SO ACTIF → Blueprint en retrait (preset none)"

**Usage :** Si votre Solar Optimizer gère lui-même les presets.

---

## Triggers (déclencheurs)

L'automatisation se déclenche sur :

1. **Lumière ON** → Immédiat (OBLIGATOIRE)
2. **Lumière OFF** → Immédiat (OBLIGATOIRE)
3. **Fenêtre ouverte** → Après délai
4. **Fenêtre fermée** → Après délai
5. **Été/Alarme/SO switch** → Via template (gère les valeurs vides)
6. **Tick périodique** → Toutes les N minutes

**Note :** Les triggers optionnels utilisent un template qui gère les entités non configurées.

---

## Logs et monitoring

### Types de logs

| Emoji | Message | Signification |
|-------|---------|---------------|
| ☀️ | Été → OFF | Mode été, radiateur arrêté |
| ☀️ | Été → ECO | Mode été, radiateur en éco |
| 🪟 | Fenêtre ouverte → PAUSE (away) | Fenêtre détectée, pause |
| ⚡ | SO ACTIF → COMFORT | Solar Optimizer chauffe, preset appliqué |
| ⚡ | SO ACTIF → Blueprint en retrait | SO chauffe, blueprint passif |
| 🔒 | Away → AWAY | Alarme armée, preset away |
| 💡 | ON → COMFORT (comfort) | Lumière allumée, confort |
| 💡 | OFF → ECO (eco) | Lumière éteinte, éco |
| 💡 | ON (toggle) → COMFORT | Toggle activé, bascule |

### Consulter les logs

**Logbook :**
```
Filtrer par : "Salle de Bain" (nom de pièce)
```

**Mode Trace :**
```
Automatisation → ⋮ → Trace
Voir l'ordre d'exécution
```

---

## Prérequis techniques

### Entités requises

- **climate.*** : Module fil pilote X4FP
  - Doit supporter les presets : eco, comfort, away (minimum)
  - Peut supporter : comfort-1, comfort-2, boost
- **light.*** : Lumière de la salle de bain (OBLIGATOIRE)

### Entités optionnelles

- **binary_sensor.*** : Capteurs fenêtre/porte (ON = ouvert)
- **input_boolean / calendar / sensor** : Indicateur été (ON = été)
- **alarm_control_panel.*** : Système d'alarme
- **switch.*** : Solar Optimizer action switch (ON = chauffe)

### Modules X4FP compatibles

- **Qubino Fil Pilote** (Zigbee/Z-Wave)
- **Nodon Fil Pilote** (Zigbee/EnOcean)
- **Heatzy** (WiFi)
- **FGR-223** (Fibaro Z-Wave)
- **Modules DIY ESPHome**

### Presets supportés

Minimum requis : `eco`, `comfort`, `away`

Optionnel : `comfort-1`, `comfort-2`, `boost`

---

## Dépannage

### La lumière ne déclenche pas le chauffage

**Vérifications :**
1. La lumière est bien configurée
2. L'état change bien ON/OFF
3. Le comportement lumière n'est pas "none"
4. L'automatisation est activée

**Test :**
```yaml
# Outils de développement → États
# Allumez/éteignez manuellement la lumière
# Consultez le logbook
```

### SO ne prend pas la priorité

**Vérifications :**
1. Le switch SO est bien configuré
2. L'état est bien ON quand SO chauffe
3. Le preset configuré existe sur le radiateur
4. Consultez le mode trace

**Ordre d'exécution attendu :**
```
1. Vérif Été → Non
2. Vérif Fenêtre → Non
3. Vérif SO → OUI (ON) → STOP ici
```

### Le preset ne s'applique pas

**Vérification :**
```yaml
# Outils de développement → États
# climate.votre_x4fp
# Attribut "preset_modes" doit contenir le preset
```

**Presets courants :**
- Qubino : eco, comfort, away, boost
- Nodon : eco, comfort, comfort-1, comfort-2, away
- Heatzy : eco, comfort, frost_protection (away)

**Solution :**
- Ajustez les presets configurés selon votre module
- Vérifiez les noms exacts (case sensitive)

### Fenêtre : le radiateur ne se coupe pas

**Vérifications :**
1. binary_sensor bien ON quand ouvert
2. Délai respecté
3. Preset fenêtre existe

**Note :** Le blueprint applique un preset (ex: away), il ne coupe pas forcément le radiateur. Pour couper complètement, configurez "Politique en Été" = OFF et créez un input_boolean "fenetre_ouverte" comme indicateur été.

---

## Scénarios avancés

### Salle de bain sans fenêtre

Laissez "Capteurs fenêtre" vide :
```yaml
Capteurs fenêtre(s)/porte(s): []
```

### Chauffage permanent en hiver

Configurez :
```yaml
Quand lumière ON: force_comfort
Quand lumière OFF: force_comfort
```

Le radiateur restera toujours en confort (sauf été/fenêtre/away).

### Radiateur éteint la nuit (sans lumière)

Créez un input_boolean "nuit" et utilisez-le comme indicateur été :
```yaml
input_boolean:
  nuit_sdb:
    name: Nuit SdB
```

Automatisation pour le gérer :
```yaml
# ON à 23h, OFF à 6h
```

Configurez :
```yaml
Indicateur Été: input_boolean.nuit_sdb
Politique en Été: off
```

### SO seulement le matin

Si vous voulez que SO chauffe seulement le matin :
- Configurez un helper qui active SO seulement entre 8h-12h
- Le blueprint suivra automatiquement

---

## Compatibilité

### Modules X4FP testés

- Qubino Fil Pilote (ZMNHJD1)
- Nodon Pilot Wire (SIN-4-FP-21)
- Heatzy (WiFi)
- Fibaro FGR-223
- ESPHome custom

### Home Assistant

- Version minimum : **2023.8**
- Testé jusqu'à : **2024.11**

---

## Changelog

### v7.5
- **Fix logique lumière en continu** : La lumière maintient maintenant l'état confort/eco de manière continue
- Correction : la logique lumière s'applique maintenant aussi lors du tick périodique et après SO/Away
- Amélioration : retour automatique à l'état lumière après désactivation de SO ou Away
- Toggle mode conserve le comportement basé sur changement d'état

### v7.4
- Fix déclenchement inopinée

### v7.2
- **Fix trigger SO conditionnel** (gère les valeurs vides)
- Support preset "none" pour SO (mode passif)
- Amélioration logs
- Optimisation triggers optionnels

### v7.x
- Solar Optimizer prioritaire
- Ordre de priorité strict
- Support "Autoriser SO en Away"

### v6.x
- Amélioration gestion lumière
- Mode toggle

---

## Liens utiles

- [Retour au README principal](../README.md)
- [Guide d'installation](../INSTALLATION.md)
- [Solar Optimizer Integration](solar_optimizer.md)
- [X4FP Room (avec thermique)](x4fp_room.md)
- [Troubleshooting général](troubleshooting.md)
