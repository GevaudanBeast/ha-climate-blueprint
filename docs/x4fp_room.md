# Blueprint : X4FP Pièce avec Contrôle Thermique (v7.2)

## Description

Blueprint pour radiateur fil pilote X4FP avec contrôle thermique par hystérésis et intégration Solar Optimizer prioritaire. Idéal pour pièces de vie avec capteur de température externe et consigne personnalisée.

**Fichier :** `blueprint_hvac_X4FP_room.yaml`
**Version :** 7.2
**Auteur :** LaCaseHome

---

## Cas d'usage

- Radiateurs fil pilote dans pièces de vie (salon, chambre, bureau)
- Chauffage avec capteur de température déporté
- Optimisation autoconsommation solaire
- Contrôle précis de la température avec consigne personnalisée
- Radiateur fil pilote 4 ou 6 ordres (X4FP)

---

## Qu'est-ce que X4FP ?

**X4FP** = Fil Pilote 4 ordres (ou 6 ordres)

Protocole de commande pour radiateurs électriques français. Voir [X4FP Bathroom](x4fp_bathroom.md#quest-ce-que-x4fp) pour les détails.

---

## Fonctionnalités principales

### 1. Contrôle thermique avec hystérésis (OPTIONNEL mais recommandé)

Le blueprint peut gérer la température de manière intelligente :

**Principe de l'hystérésis :**
```
Consigne = 20°C
Hystérésis = 0.5°C

Température ≤ 19.5°C (20 - 0.5) → HEAT (preset confort)
Température ≥ 20.5°C (20 + 0.5) → IDLE (preset éco)
Entre 19.5°C et 20.5°C → Pas de changement (évite les oscillations)
```

**Avantages :**
- Évite les cycles marche/arrêt trop fréquents
- Température stable
- Consommation optimisée
- Confort amélioré

**Prérequis :**
- Capteur de température externe (`sensor.*`)
- Consigne via `input_number.*`

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
5. Contrôle thermique (HEAT ou IDLE selon température)
```

### 4. Garde-fous température

Pour éviter les consignes aberrantes :
- **Température mini** : Limite basse de consigne (défaut 17°C)
- **Température maxi** : Limite haute de consigne (défaut 23°C)

Si `input_number` est hors limites → Valeur automatiquement limitée (clamped).

### 5. Gestion été/hiver

**Politique en Été :**
- **OFF** : Radiateur arrêté
- **ECO** : Radiateur en mode éco

### 6. Fenêtres (optionnel)

- Détection ouverture avec délai
- Preset configurable (défaut : away)

### 7. Tick périodique

- Ré-application des réglages toutes les N minutes
- Évite les désynchronisations

---

## Configuration

### Paramètres obligatoires

| Paramètre | Type | Description |
|-----------|------|-------------|
| **Nom logique de la pièce** | Texte | Nom pour les logs (ex: "Salon") |
| **Entité climate (X4FP)** | climate.* | Module fil pilote (X4FP) |

### Paramètres optionnels

#### Contrôle thermique

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| **Capteur de température (°C)** | sensor.* | "" | Température actuelle de la pièce |
| **Consigne (input_number)** | input_number.* | "" | Température souhaitée |
| **Hystérésis (°C)** | Nombre | 0.5 | Écart avant changement d'état |
| **Garde-fou: consigne mini (°C)** | Nombre | 17 | Température minimale autorisée |
| **Garde-fou: consigne maxi (°C)** | Nombre | 23 | Température maximale autorisée |

**Note :** Si capteur OU consigne non configurés → Contrôle thermique désactivé

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
| **Preset quand on chauffe (demande)** | Select | comfort | eco / comfort / comfort-1 / comfort-2 / away / boost |
| **Preset quand on n'a plus besoin** | Select | eco | eco / comfort / comfort-1 / comfort-2 / away / boost |
| **Preset fenêtre ouverte** | Select | away | eco / comfort / comfort-1 / comfort-2 / away / boost |
| **Preset Away** | Select | away | eco / comfort / comfort-1 / comfort-2 / away / boost |

#### Tick

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| **Fréquence ré-application (min)** | Nombre | 10 | 0 pour désactiver, sinon 1-120 min |

---

## Exemples de configuration

### Exemple 1 : Configuration basique sans contrôle thermique

```yaml
Nom de la pièce: Salon
Entité climate: climate.x4fp_salon
Preset quand on chauffe: comfort
Preset quand on n'a plus besoin: eco
```

**Comportement :**
- Pas de contrôle thermique automatique
- Le blueprint ne gère que été/fenêtre/away
- **Usage :** Si vous gérez la température autrement

### Exemple 2 : Avec contrôle thermique complet

```yaml
Nom de la pièce: Chambre
Entité climate: climate.x4fp_chambre
Capteur de température: sensor.temp_chambre
Consigne: input_number.consigne_chambre
Hystérésis: 0.5°C
Garde-fou mini: 17°C
Garde-fou maxi: 22°C
Preset quand on chauffe: comfort
Preset quand on n'a plus besoin: eco
Fréquence ré-application: 10 min
```

**Comportement :**
- Température ≤ 19.5°C → COMFORT (chauffe)
- Température ≥ 20.5°C → ECO (réduit)
- Si consigne modifiée à 22°C mais > maxi (22°C) → Clamped à 22°C

### Exemple 3 : Avec Solar Optimizer et alarme

```yaml
Nom de la pièce: Bureau
Entité climate: climate.x4fp_bureau
Capteur de température: sensor.temp_bureau
Consigne: input_number.consigne_bureau
Hystérésis: 0.3°C
Alarme: alarm_control_panel.alarmo
Solar Optimizer – SWITCH: switch.solar_optimizer_bureau
Autoriser SO en Away: true
Preset quand SO chauffe: comfort
Preset Away: away
Fréquence ré-application: 10 min
```

**Comportement :**
- **SO ON** → CONFORT (priorité absolue, même si Away)
- **SO OFF + Away** → AWAY (hors-gel)
- **SO OFF + Home + T° ≤ 19.7°C** → COMFORT (chauffe)
- **SO OFF + Home + T° ≥ 20.3°C** → ECO (réduit)

**Scénario réel :**
```
10h00 - Surplus solaire
→ SO ON → COMFORT (chauffe au max)

12h00 - Plus de surplus
→ SO OFF
→ Température = 21°C (≥ 20.3°C)
→ ECO (maintien)

14h00 - Température descend à 19.5°C
→ COMFORT (recharge thermique)

18h00 - Départ maison (alarme armée)
→ AWAY (hors-gel)
```

### Exemple 4 : Avec fenêtres et été

```yaml
Nom de la pièce: Salon
Entité climate: climate.x4fp_salon
Capteur de température: sensor.temp_salon
Consigne: input_number.consigne_salon
Hystérésis: 0.5°C
Capteurs fenêtre:
  - binary_sensor.fenetre_salon_1
  - binary_sensor.fenetre_salon_2
Délai avant PAUSE: 3 min
Indicateur Été: input_boolean.ete
Politique en Été: off
Preset fenêtre ouverte: away
```

**Comportement :**
- Été → OFF (radiateur arrêté)
- Fenêtre ouverte 3 min → AWAY (hors-gel)
- Hiver + Fenêtre fermée + T° basse → COMFORT

---

## Contrôle thermique détaillé

### Configuration des helpers

**Capteur de température :**
Doit retourner une température en °C (float).

**Exemples :**
- `sensor.temperature_salon` (Zigbee)
- `sensor.salon_temperature` (ESPHome)
- `sensor.temp_thermometre_salon` (DIY)

**Consigne (input_number) :**

Dans `configuration.yaml` :
```yaml
input_number:
  consigne_salon:
    name: Consigne Salon
    min: 16
    max: 24
    step: 0.5
    unit_of_measurement: "°C"
    icon: mdi:thermometer
    mode: slider
```

Ou via interface : **Paramètres** → **Appareils & Services** → **Entrées** → **Ajouter une entrée** → **Nombre**

### Principe de l'hystérésis

**Sans hystérésis (problématique) :**
```
Consigne = 20°C
T° = 19.9°C → Chauffe
T° = 20.0°C → Arrête
T° = 19.9°C → Chauffe
T° = 20.0°C → Arrête
...
→ Cycles rapides, usure, inconfort
```

**Avec hystérésis = 0.5°C (solution) :**
```
Consigne = 20°C
T° = 19.4°C → Chauffe (≤ 19.5°C)
T° = 19.9°C → Chauffe (toujours)
T° = 20.2°C → Chauffe (toujours)
T° = 20.6°C → Arrête (≥ 20.5°C)
T° = 20.3°C → Arrête (toujours)
T° = 19.8°C → Arrête (toujours)
T° = 19.4°C → Chauffe (≤ 19.5°C)
→ Cycles longs, confort, économie
```

### Réglage de l'hystérésis

| Valeur | Usage | Cycles | Précision |
|--------|-------|--------|-----------|
| 0.2°C | Très précis | Fréquents | Haute |
| 0.5°C | **Recommandé** | Normaux | Bonne |
| 1.0°C | Économique | Rares | Moyenne |
| 2.0°C | Très économique | Très rares | Faible |

**Conseil :** Commencez avec 0.5°C et ajustez selon vos besoins.

### Garde-fous

**Pourquoi ?**
- Éviter erreurs de manipulation
- Protéger contre bugs
- Limiter la consommation

**Exemple :**
```yaml
Consigne mini: 17°C
Consigne maxi: 23°C
```

Si `input_number.consigne_salon` = 26°C (erreur)
→ Blueprint utilise 23°C (clamped)

Si `input_number.consigne_salon` = 12°C (erreur)
→ Blueprint utilise 17°C (clamped)

**Logs :**
```
🌡️ T° 18.5°C ≤ 22.5°C → HEAT
(note: consigne clamped de 26°C à 23°C)
```

### Désactiver le contrôle thermique

Pour désactiver :
- Laissez "Capteur de température" vide
- OU laissez "Consigne" vide

Le blueprint n'appliquera alors que les règles été/fenêtre/away.

---

## Intégration Solar Optimizer

Identique à [X4FP Bathroom](x4fp_bathroom.md#intégration-solar-optimizer).

**Résumé :**
- SO surveille un switch d'action (ON = chauffe)
- Priorité absolue sur tout
- Peut être autorisé en mode Away

---

## Triggers (déclencheurs)

L'automatisation se déclenche sur :

1. **Fenêtre ouverte** → Après délai
2. **Fenêtre fermée** → Après délai
3. **Été/Alarme/SO switch** → Via template (gère les valeurs vides)
4. **Tick périodique** → Toutes les N minutes

**Note :** Pas de trigger sur température/consigne, c'est le tick qui gère.

**Pourquoi pas de trigger température ?**
- Évite trop de déclenchements
- Le tick toutes les N minutes suffit
- Évite les oscillations

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
| 🌡️ | T° XX.X°C ≤ YY.Y°C → HEAT | Température basse, chauffe |
| 🌡️ | T° XX.X°C ≥ YY.Y°C → IDLE | Température haute, réduit |

### Consulter les logs

**Logbook :**
```
Filtrer par : "Salon" (nom de pièce)
```

**Mode Trace :**
```
Automatisation → ⋮ → Trace
Voir l'ordre d'exécution
Vérifier les valeurs de température
```

---

## Prérequis techniques

### Entités requises

- **climate.*** : Module fil pilote X4FP
  - Doit supporter les presets : eco, comfort, away (minimum)
  - Peut supporter : comfort-1, comfort-2, boost

### Entités optionnelles

- **sensor.*** : Capteur température (retourne float en °C)
- **input_number.*** : Consigne de température
- **binary_sensor.*** : Capteurs fenêtre/porte (ON = ouvert)
- **input_boolean / calendar / sensor** : Indicateur été (ON = été)
- **alarm_control_panel.*** : Système d'alarme
- **switch.*** : Solar Optimizer action switch (ON = chauffe)

### Modules X4FP compatibles

Voir [X4FP Bathroom](x4fp_bathroom.md#modules-x4fp-compatibles).

### Capteurs de température compatibles

- Capteurs Zigbee (Aqara, Sonoff, Tuya)
- Capteurs Z-Wave
- Capteurs ESPHome
- Capteurs WiFi (Shelly, etc.)
- DHT22, BME280, etc. via DIY

**Important :** Le capteur doit retourner une température en °C (pas °F).

---

## Dépannage

### Le contrôle thermique ne fonctionne pas

**Vérifications :**
1. Capteur température ET consigne configurés (tous les deux obligatoires)
2. Le capteur retourne bien une valeur numérique (pas "unknown" ou "unavailable")
3. L'input_number est modifiable et a une valeur
4. Le tick est activé (> 0 min)

**Test :**
```yaml
# Outils de développement → États
# Vérifiez sensor.temp_xxx → doit avoir une valeur (ex: 19.5)
# Vérifiez input_number.consigne_xxx → doit avoir une valeur (ex: 20)
```

### Le preset ne change jamais

**Causes possibles :**
1. Hystérésis trop large
2. Température toujours dans la zone neutre
3. Preset configuré inexistant

**Test :**
Modifiez temporairement la consigne :
- Consigne actuelle = 20°C, T° = 20°C
- Montez consigne à 23°C → devrait passer HEAT
- Descendez consigne à 18°C → devrait passer IDLE

### La température oscille trop

**Solution :**
Augmentez l'hystérésis :
```yaml
Hystérésis: 0.5°C → 1.0°C
```

### La température ne monte/descend pas assez vite

**Solution :**
Réduisez l'hystérésis :
```yaml
Hystérésis: 0.5°C → 0.3°C
```

### Garde-fou : consigne bloquée

**Symptôme :**
Vous modifiez `input_number` à 25°C mais le radiateur ne chauffe pas autant.

**Explication :**
Le garde-fou maxi (23°C) limite la consigne.

**Solution :**
Augmentez le garde-fou maxi :
```yaml
Garde-fou maxi: 23°C → 25°C
```

### SO ne prend pas la priorité

Voir [X4FP Bathroom - Dépannage SO](x4fp_bathroom.md#so-ne-prend-pas-la-priorité).

---

## Scénarios avancés

### Consigne différente jour/nuit

**Option 1 : Deux automatisations**

Automatisation 1 (jour) :
```yaml
trigger: time 07:00
action:
  - service: input_number.set_value
    target:
      entity_id: input_number.consigne_salon
    data:
      value: 21
```

Automatisation 2 (nuit) :
```yaml
trigger: time 22:00
action:
  - service: input_number.set_value
    target:
      entity_id: input_number.consigne_salon
    data:
      value: 18
```

**Option 2 : Scheduler intégré**

Utilisez l'intégration "Scheduler" de HACS.

### Contrôle thermique seulement le soir

Créez un input_boolean "controle_thermique_actif" :

```yaml
input_boolean:
  controle_thermique_salon:
    name: Contrôle thermique Salon
```

Automatisation :
```yaml
# ON à 18h, OFF à 8h
```

Modifiez le blueprint :
- Si "controle_thermique_salon" = OFF → Laissez capteur/consigne vides dans le blueprint
- Si "controle_thermique_salon" = ON → Configurez capteur/consigne

**Limitation :** Nécessite de modifier le blueprint (avancé).

### Away la nuit automatiquement

Créez un input_boolean "nuit" :
```yaml
input_boolean:
  nuit:
    name: Nuit
```

Automatisation :
```yaml
# ON à 23h, OFF à 6h
```

Configurez :
```yaml
Indicateur Été: input_boolean.nuit
Politique en Été: eco  # ou off si vous voulez éteindre
```

**Résultat :** Entre 23h et 6h, le radiateur passe en ECO (ou OFF).

---

## Comparaison avec X4FP Bathroom

| Caractéristique | X4FP Bathroom | X4FP Room |
|-----------------|---------------|-----------|
| Lumière | **Obligatoire** (détection présence) | Non |
| Contrôle thermique | Non | **Oui** (hystérésis) |
| Capteur température | Non | Oui (optionnel) |
| Consigne | Non | input_number (optionnel) |
| Solar Optimizer | Oui (prioritaire) | Oui (prioritaire) |
| Usage | Salle de bain | Pièces de vie |

**Conseil :**
- Salle de bain → X4FP Bathroom
- Autres pièces → X4FP Room

---

## Compatibilité

### Modules X4FP testés

Voir [X4FP Bathroom](x4fp_bathroom.md#modules-x4fp-testés).

### Home Assistant

- Version minimum : **2023.8**
- Testé jusqu'à : **2024.11**

---

## Changelog

### v7.2
- **Fix trigger SO conditionnel** (gère les valeurs vides)
- Support preset "none" pour SO (mode passif)
- Amélioration logs
- Optimisation triggers optionnels
- Garde-fous température avec clamp

### v7.x
- Solar Optimizer prioritaire
- Ordre de priorité strict
- Support "Autoriser SO en Away"
- Contrôle thermique avec hystérésis

---

## Liens utiles

- [Retour au README principal](../README.md)
- [Guide d'installation](../INSTALLATION.md)
- [Solar Optimizer Integration](solar_optimizer.md)
- [X4FP Bathroom (avec lumière)](x4fp_bathroom.md)
- [Troubleshooting général](troubleshooting.md)
