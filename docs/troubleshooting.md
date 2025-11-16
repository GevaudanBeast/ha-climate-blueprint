# Guide de Dépannage (Troubleshooting)

Ce guide regroupe les problèmes courants rencontrés avec les blueprints Climate et leurs solutions.

---

## Table des matières

1. [Problèmes d'installation](#problèmes-dinstallation)
2. [Problèmes de configuration](#problèmes-de-configuration)
3. [Problèmes de déclenchement](#problèmes-de-déclenchement)
4. [Problèmes de preset](#problèmes-de-preset)
5. [Problèmes de température](#problèmes-de-température)
6. [Problèmes Solar Optimizer](#problèmes-solar-optimizer)
7. [Problèmes de logs](#problèmes-de-logs)
8. [Outils de diagnostic](#outils-de-diagnostic)

---

## Problèmes d'installation

### Le blueprint n'apparaît pas après l'import

**Symptômes :**
- Import semble réussi mais blueprint invisible
- Pas d'erreur affichée

**Solutions :**

1. **Rechargez les blueprints**
   ```
   Outils de développement → YAML → Recharger blueprints d'automatisation
   ```

2. **Videz le cache du navigateur**
   ```
   Ctrl + F5 (Windows/Linux)
   Cmd + Shift + R (Mac)
   ```

3. **Vérifiez les logs Home Assistant**
   ```
   Paramètres → Système → Logs
   Recherchez "blueprint" ou "automation"
   ```

4. **Vérifiez le chemin du fichier**
   ```
   /config/blueprints/automation/lacasehome/blueprint_xxx.yaml
   ```

### Erreur "Invalid blueprint"

**Symptômes :**
- Message d'erreur à l'import
- Blueprint rejeté

**Causes possibles :**
- Fichier YAML mal formaté
- Copie incomplète
- Caractères spéciaux corrompus

**Solutions :**

1. **Vérifiez la copie complète**
   - Fichier doit commencer par `blueprint:`
   - Fichier doit finir par la dernière ligne d'action
   - Pas de lignes manquantes

2. **Vérifiez l'indentation**
   - YAML utilise des espaces (PAS de tabulations)
   - 2 espaces par niveau d'indentation
   - Utilisez un éditeur avec validation YAML

3. **Réimportez depuis GitHub**
   - Utilisez le bouton "Raw" sur GitHub
   - Copier/Coller depuis Raw

4. **Vérifiez la version Home Assistant**
   ```
   Minimum requis : 2023.8
   Paramètres → Système → Informations
   ```

### Erreur "Failed to import"

**Symptômes :**
- Import échoue avec message d'erreur

**Solutions :**

1. **Vérifiez l'URL**
   - Doit pointer vers fichier .yaml
   - Doit être accessible (public)

2. **Vérifiez la connexion GitHub**
   ```
   Outils de développement → Console navigateur
   Recherchez erreurs réseau
   ```

3. **Utilisez l'installation manuelle**
   Voir [INSTALLATION.md](../INSTALLATION.md#méthode-2--installation-manuelle-via-fichier)

---

## Problèmes de configuration

### Entité introuvable ("Entity not found")

**Symptômes :**
- Erreur lors de la sauvegarde de l'automatisation
- "Entity XYZ not found"

**Solutions :**

1. **Vérifiez que l'entité existe**
   ```
   Outils de développement → États
   Recherchez l'entité
   ```

2. **Respectez la casse**
   ```
   climate.Salon != climate.salon
   ```

3. **Utilisez le sélecteur d'entités**
   - Cliquez sur l'icône 🔍
   - Sélectionnez dans la liste
   - Ne tapez pas manuellement

4. **Pour entités optionnelles, laissez vide**
   - Ne mettez PAS "none" ou "null"
   - Laissez le champ vide

### L'automatisation ne se sauvegarde pas

**Symptômes :**
- Clic sur "Sauvegarder" sans effet
- Retour au formulaire

**Solutions :**

1. **Vérifiez les champs obligatoires**
   - Tous les champs sans "(optionnel)" doivent être remplis
   - Nom de l'automatisation
   - Entités requises

2. **Consultez les logs**
   ```
   Outils de développement → Console navigateur (F12)
   Recherchez erreurs JavaScript
   ```

3. **Simplifiez la configuration**
   - Commencez avec le minimum
   - Ajoutez options une par une

### Valeurs par défaut étranges

**Symptômes :**
- Valeurs pré-remplies incorrectes
- Nombre à virgule affiché bizarrement

**Solutions :**

1. **C'est normal**
   - Les défauts sont définis dans le blueprint
   - Modifiez selon vos besoins

2. **Pour nombres décimaux**
   - Utilisez le point (.) pas la virgule (,)
   - Exemple : 20.5 pas 20,5

---

## Problèmes de déclenchement

### L'automatisation ne se déclenche jamais

**Symptômes :**
- Aucune action, même après changements
- Logbook vide

**Vérifications :**

1. **L'automatisation est activée**
   ```
   Automatisations → Trouvez votre automatisation
   Toggle doit être ON (bleu)
   ```

2. **Les entités existent**
   ```
   Outils de développement → États
   Vérifiez toutes les entités configurées
   ```

3. **Les triggers sont valides**
   - Fenêtre : binary_sensor doit passer à "on"
   - Alarme : alarm_control_panel doit changer d'état
   - Tick : Doit être > 0 minutes

**Test manuel :**

1. **Activez le mode trace**
   ```
   Automatisation → ⋮ → Trace
   ```

2. **Déclenchez manuellement**
   ```
   Automatisation → ⋮ → Exécuter
   ```

3. **Consultez le trace**
   - Identifiez où ça bloque
   - Vérifiez les conditions
   - Vérifiez les valeurs des variables

### L'automatisation se déclenche trop souvent

**Symptômes :**
- Multiples déclenchements rapides
- Logs saturés

**Solutions :**

1. **Vérifiez le tick**
   ```
   Si tick = 1 minute → Passe à 10 minutes
   ```

2. **Vérifiez les triggers**
   - Entités qui changent souvent ?
   - Capteurs instables ?

3. **Mode restart vs queued**
   - Blueprints utilisent `mode: restart`
   - C'est normal, nouvelle exécution annule l'ancienne

### Délais fenêtre non respectés

**Symptômes :**
- Fenêtre ouverte, action immédiate au lieu d'attendre

**Vérifications :**

1. **Délai configuré**
   ```
   Délai avant PAUSE : Doit être > 0
   ```

2. **État du binary_sensor**
   ```
   Outils de développement → États
   binary_sensor.fenetre_xxx
   État doit être "on" quand ouvert
   ```

3. **Trigger "for"**
   - Le blueprint utilise `for: { minutes: X }`
   - C'est correct

**Test :**
```yaml
# Simulez ouverture fenêtre
# Attendez le délai configuré
# Consultez logbook pour confirmation
```

---

## Problèmes de preset

### Le preset ne s'applique pas

**Symptômes :**
- Logs indiquent preset appliqué
- Mais thermostat reste sur autre preset

**Vérifications :**

1. **Le preset existe**
   ```
   Outils de développement → États
   climate.votre_thermostat
   Attribut "preset_modes" : ["eco", "comfort", ...]
   ```

2. **Le nom est exact**
   - Case sensitive : `comfort` ≠ `Comfort`
   - Pas d'espaces
   - Vérifiez l'orthographe

3. **Le thermostat supporte preset_mode**
   ```
   climate.set_preset_mode doit être supporté
   ```

**Solution si preset n'existe pas :**

**Thermostat Heat :**
- Utilise automatiquement fallback température
- Vérifiez logs : "preset XXX indisponible → fallback YY°C"

**X4FP :**
- Ajustez les presets configurés selon votre module
- Exemples communs :
  - Qubino : eco, comfort, away, boost
  - Nodon : eco, comfort, comfort-1, comfort-2, away
  - Heatzy : eco, comfort, frost_protection

### Preset change puis revient

**Symptômes :**
- Preset appliqué
- Puis revient à l'ancien immédiatement

**Causes :**

1. **Autre automatisation concurrente**
   - Plusieurs automatisations pour même thermostat
   - Conflit de priorité

2. **Thermostat en mode manuel**
   - Certains thermostats ont mode manuel/auto
   - En manuel, ils ignorent les commandes

3. **Tick trop court**
   - Tick = 1 min peut créer conflits
   - Passez à 10 min

**Solutions :**

1. **Désactivez les autres automatisations**
   ```
   Gardez seulement le blueprint actif
   ```

2. **Vérifiez mode du thermostat**
   ```
   Mode manuel → Auto
   ```

3. **Augmentez le tick**
   ```
   Tick : 1 min → 10 min
   ```

---

## Problèmes de température

### La température ne change pas

**Symptômes :**
- Blueprint semble fonctionner
- Mais température reste identique

**Vérifications :**

1. **Le thermostat supporte set_temperature**
   ```
   Outils de développement → Services
   Testez climate.set_temperature
   ```

2. **La température est différente de l'actuelle**
   ```
   Attribut "temperature" du climate
   Compare avec la consigne appliquée
   ```

3. **Le thermostat n'est pas en mode OFF**
   ```
   État climate doit être "heat" ou "cool"
   Pas "off"
   ```

**Pour blueprints avec fallback température :**

- Vérifiez logs : température fallback appliquée ?
- Vérifiez que preset n'existe PAS (sinon fallback ignoré)

### Température appliquée incorrecte

**Symptômes :**
- Blueprint applique 21°C
- Thermostat affiche 18°C

**Causes :**

1. **Preset override température**
   - Si preset appliqué après température
   - Preset a priorité

2. **Thermostat avec offset**
   - Certains thermostats ont calibration/offset
   - Température affichée ≠ température cible

3. **Garde-fous activés** (X4FP Room)
   ```
   Consigne demandée : 25°C
   Garde-fou maxi : 23°C
   → Température appliquée : 23°C (clamped)
   ```

**Solutions :**

1. **Vérifiez l'ordre des actions**
   - Preset après température → Preset gagne
   - C'est le comportement normal des blueprints

2. **Vérifiez attributs thermostat**
   ```
   Attribut "temperature" : Cible
   Attribut "current_temperature" : Actuelle
   ```

### Contrôle thermique ne fonctionne pas (X4FP Room)

**Symptômes :**
- Température monte mais pas de passage ECO
- Température descend mais pas de passage COMFORT

**Vérifications :**

1. **Capteur ET consigne configurés**
   ```
   Les DEUX sont obligatoires
   Si l'un manque, contrôle désactivé
   ```

2. **Capteur retourne valeur valide**
   ```
   Outils de développement → États
   sensor.temp_xxx doit avoir nombre (ex: 19.5)
   PAS "unknown", "unavailable", "null"
   ```

3. **Input_number modifiable**
   ```
   input_number.consigne_xxx doit avoir valeur
   Testez en le modifiant manuellement
   ```

4. **Tick activé**
   ```
   Tick doit être > 0 minutes
   Contrôle thermique vérifié à chaque tick
   ```

5. **Hystérésis compris**
   ```
   Consigne = 20°C, Hystérésis = 0.5°C
   HEAT si T° ≤ 19.5°C
   IDLE si T° ≥ 20.5°C
   Entre 19.5 et 20.5 : PAS DE CHANGEMENT
   ```

**Test :**

```yaml
# Modifiez consigne à 25°C (bien au-dessus température actuelle)
# Attendez 1 tick
# Vérifiez logbook : "T° XX°C ≤ YY°C → HEAT"
```

---

## Problèmes Solar Optimizer

### SO ne prend jamais la priorité

**Symptômes :**
- SO actif mais blueprint n'applique pas preset SO
- Autre règle appliquée à la place

**Vérifications :**

1. **Switch SO configuré**
   ```
   Solar Optimizer – SWITCH : doit être rempli
   ```

2. **Switch SO est ON**
   ```
   Outils de développement → États
   switch.solar_optimizer_xxx : "on"
   ```

3. **Ordre de priorité**
   ```
   Été et Fenêtre passent AVANT SO
   Si fenêtre ouverte, SO bloqué (normal)
   ```

4. **Version blueprint**
   ```
   X4FP Bathroom et Room v7.2+ requis
   Versions antérieures ont bugs SO
   ```

5. **Preset SO != "none"** (sauf si voulu)
   ```
   Si preset = "none", blueprint se met en retrait
   Vérifiez logs : "Blueprint en retrait"
   ```

**Test mode trace :**

```
Activez SO manuellement
→ Déclenchez automatisation
→ Consultez trace :
   - Véri Été : Non
   - Véri Fenêtre : Non
   - Véri SO : OUI → STOP ici
```

### SO et Away : conflit

**Symptômes :**
- SO devrait chauffer
- Mais Away appliqué à la place

**Cause :**
```yaml
Autoriser SO en Away : false (défaut)
```

**Solution :**
```yaml
Autoriser SO en Away : true
```

**Vérification :**
```
Mode trace → Variable "solar_can_override_away"
Si false : Away bloque SO
Si true : SO override Away
```

### SO chauffe mais blueprint override

**Symptômes :**
- SO ON
- Blueprint applique autre preset

**Causes :**

1. **Été ou Fenêtre prioritaires**
   ```
   Ordre : Été > Fenêtre > SO
   Si été ou fenêtre, SO ignoré
   ```

2. **Switch SO pas le bon**
   ```
   Doit être le switch D'ACTION
   Pas le switch enable/disable global SO
   ```

3. **Trigger SO pas mis à jour**
   ```
   Version < v7.2 a bug trigger SO
   Mettez à jour blueprint
   ```

---

## Problèmes de logs

### Pas de logs dans Logbook

**Symptômes :**
- Automatisation fonctionne (trace le montre)
- Mais pas de logs dans Logbook

**Causes :**

1. **C'est normal si aucune action**
   - Blueprint ne log que si changement effectué
   - Si déjà dans le bon état, pas de log

2. **Filtre Logbook**
   ```
   Logbook → Filtrer par entité/nom
   Cherchez le nom de votre pièce
   ```

3. **Service logbook.log désactivé**
   ```
   Vérifiez configuration HA
   Logbook doit être activé
   ```

**Vérification :**

```yaml
# Mode trace
# Section "service: logbook.log"
# Doit être exécutée
```

### Logs incomplets

**Symptômes :**
- Certains logs manquants
- Actions effectuées mais pas loggées

**Causes :**

1. **Conditions if pas remplies**
   ```
   Blueprint ne log que si changement nécessaire
   Si déjà au bon preset, pas de log
   ```

2. **Stop précoce**
   ```
   Si Été → stop
   Les actions suivantes pas exécutées
   Donc pas de logs pour les actions suivantes
   ```

**Vérification :**

Mode trace montre tout, même sans logs.

---

## Outils de diagnostic

### Mode Trace

**Accès :**
```
Automatisation → ⋮ → Trace
```

**Utilité :**
- Voir chaque étape d'exécution
- Voir valeurs des variables
- Identifier où ça bloque
- Voir conditions true/false

**Comment lire :**
1. **Trigger** : Ce qui a déclenché
2. **Variables** : Valeurs calculées
3. **Conditions** : Choix effectués
4. **Actions** : Services appelés
5. **Stop** : Pourquoi l'automatisation s'arrête

### Logbook

**Accès :**
```
Logbook (📖) dans menu
```

**Utilité :**
- Historique des actions
- Messages du blueprint
- Chronologie

**Filtrage :**
```
Recherchez le nom de votre pièce
Exemple : "Salon", "Chambre", etc.
```

### Outils de développement

**États :**
```
Outils de développement → États
Recherchez vos entités
Vérifiez attributs
```

**Services :**
```
Outils de développement → Services
Testez manuellement les services
climate.set_preset_mode
climate.set_temperature
climate.set_hvac_mode
```

**Template :**
```
Outils de développement → Template
Testez les templates du blueprint
Exemple : {{ states('sensor.temp_salon') | float }}
```

### Logs système

**Accès :**
```
Paramètres → Système → Logs
```

**Recherche :**
- "automation"
- "blueprint"
- "climate"
- Votre nom d'entité

**Niveaux :**
- **Error** : Erreurs critiques
- **Warning** : Avertissements
- **Info** : Informations
- **Debug** : Détails (activer debug logging)

---

## Activer le debug logging

Pour logs détaillés :

**configuration.yaml :**
```yaml
logger:
  default: info
  logs:
    homeassistant.components.automation: debug
    homeassistant.components.climate: debug
```

**Redémarrez Home Assistant**

**Consultez les logs**
```
Paramètres → Système → Logs
Filtrez par "automation" ou "climate"
```

---

## Signaler un bug

Si problème non résolu :

1. **Vérifiez les issues GitHub**
   ```
   https://github.com/GevaudanBeast/ha-climate-blueprint/issues
   Peut-être déjà signalé
   ```

2. **Créez une nouvelle issue**
   ```
   Cliquez "New Issue"
   Remplissez le template
   ```

3. **Informations à fournir**
   - Version Home Assistant
   - Version blueprint
   - Blueprint concerné
   - Configuration (anonymisée)
   - Logs d'erreur
   - Mode trace (screenshot ou export)
   - Comportement attendu vs observé

4. **Anonymisez les données**
   ```
   Remplacez noms de pièces
   Remplacez noms d'entités
   Supprimez infos personnelles
   ```

---

## Liens utiles

- [Retour au README principal](../README.md)
- [Guide d'installation](../INSTALLATION.md)
- [Documentation des blueprints](../README.md#blueprints-disponibles)
- [Solar Optimizer](solar_optimizer.md)
- [Issues GitHub](https://github.com/GevaudanBeast/ha-climate-blueprint/issues)
