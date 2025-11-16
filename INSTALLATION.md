# Guide d'Installation

Ce guide détaille les différentes méthodes d'installation des blueprints Climate pour Home Assistant.

## Méthode 1 : Import direct via l'interface (Recommandé)

Cette méthode est la plus simple et la plus rapide.

### Étapes

1. **Ouvrez Home Assistant** dans votre navigateur

2. **Accédez aux Blueprints**
   - Cliquez sur **Paramètres** (⚙️) dans la barre latérale
   - Sélectionnez **Automatisations & Scènes**
   - Cliquez sur l'onglet **Blueprints**
   - Cliquez sur le bouton **Importer un Blueprint** (en bas à droite)

3. **Importez le blueprint souhaité**

   Copiez et collez l'une des URLs suivantes selon vos besoins :

   **Thermostat Chauffage Simple (v3.0)**
   ```
   https://github.com/GevaudanBeast/ha-climate-blueprint/blob/main/blueprints/blueprint_hvac_thermostat_heat.yaml
   ```

   **Thermostat/Climatisation Pièce (v2.0)**
   ```
   https://github.com/GevaudanBeast/ha-climate-blueprint/blob/main/blueprints/blueprint_hvac_room_thermostat.yaml
   ```

   **X4FP Salle de Bain avec Lumière (v7.2)**
   ```
   https://github.com/GevaudanBeast/ha-climate-blueprint/blob/main/blueprints/blueprint_hvac_X4FP_bathroom.yaml
   ```

   **X4FP Pièce avec Contrôle Thermique (v7.2)**
   ```
   https://github.com/GevaudanBeast/ha-climate-blueprint/blob/main/blueprints/blueprint_hvac_X4FP_room.yaml
   ```

4. **Aperçu et validation**
   - Home Assistant affichera un aperçu du blueprint
   - Vérifiez le nom et la description
   - Cliquez sur **Importer le Blueprint**

5. **Création de l'automatisation**
   - Cliquez sur **Créer une automatisation**
   - Ou allez dans **Automatisations** → **Créer une automatisation** → **Démarrer avec un blueprint** → Sélectionnez votre blueprint

---

## Méthode 2 : Installation manuelle via fichier

Cette méthode est utile si vous n'avez pas accès direct à GitHub depuis Home Assistant ou si vous souhaitez modifier les blueprints.

### Prérequis

- Accès aux fichiers de configuration Home Assistant (via SSH, File Editor, Samba, etc.)

### Étapes

1. **Accédez au dossier de configuration**

   Connectez-vous à votre instance Home Assistant via :
   - File Editor (add-on)
   - Visual Studio Code Server (add-on)
   - SSH / Terminal
   - Samba Share

2. **Créez le dossier blueprints (s'il n'existe pas)**

   Dans votre dossier `config/`, créez la structure suivante :
   ```
   config/
   └── blueprints/
       └── automation/
           └── lacasehome/
   ```

   Commandes via Terminal/SSH :
   ```bash
   cd /config
   mkdir -p blueprints/automation/lacasehome
   ```

3. **Téléchargez les fichiers**

   **Option A : Via wget/curl (si disponible)**
   ```bash
   cd /config/blueprints/automation/lacasehome

   # Thermostat Heat
   wget https://raw.githubusercontent.com/GevaudanBeast/ha-climate-blueprint/main/blueprints/blueprint_hvac_thermostat_heat.yaml

   # Room Thermostat
   wget https://raw.githubusercontent.com/GevaudanBeast/ha-climate-blueprint/main/blueprints/blueprint_hvac_room_thermostat.yaml

   # X4FP Bathroom
   wget https://raw.githubusercontent.com/GevaudanBeast/ha-climate-blueprint/main/blueprints/blueprint_hvac_X4FP_bathroom.yaml

   # X4FP Room
   wget https://raw.githubusercontent.com/GevaudanBeast/ha-climate-blueprint/main/blueprints/blueprint_hvac_X4FP_room.yaml
   ```

   **Option B : Copie manuelle**
   - Ouvrez les fichiers sur GitHub
   - Cliquez sur "Raw"
   - Copiez le contenu
   - Créez un nouveau fichier dans `/config/blueprints/automation/lacasehome/`
   - Collez le contenu
   - Sauvegardez avec le bon nom

4. **Rechargez les blueprints**

   Dans Home Assistant :
   - Allez dans **Outils de développement** (🔨)
   - Onglet **YAML**
   - Cliquez sur **Recharger les blueprints d'automatisation**

5. **Vérification**

   - Allez dans **Paramètres** → **Automatisations & Scènes** → **Blueprints**
   - Vérifiez que vos nouveaux blueprints apparaissent

---

## Méthode 3 : Installation via HACS (si supporté dans le futur)

*Cette méthode n'est pas encore disponible. Le repository n'est pas actuellement intégré à HACS.*

---

## Configuration post-installation

### 1. Vérifier les entités requises

Avant de créer une automatisation, assurez-vous d'avoir :

**Pour tous les blueprints :**
- Entité `climate.*` de votre thermostat/climatiseur

**Optionnel selon le blueprint :**
- `binary_sensor.*` pour fenêtres/portes
- `alarm_control_panel.*` pour la gestion Away
- `input_boolean.*` ou `calendar.*` pour l'indicateur Été
- `switch.solar_optimizer_*` pour Solar Optimizer
- `sensor.*` pour capteurs de température (X4FP Room)
- `input_number.*` pour consignes de température (X4FP Room)
- `light.*` pour détection de présence (X4FP Bathroom)

### 2. Créer les entités manquantes

**Indicateur Été (input_boolean)**

Dans `configuration.yaml` ou via l'interface :
```yaml
input_boolean:
  ete:
    name: Mode Été
    icon: mdi:weather-sunny
```

**Consigne température (input_number)**

```yaml
input_number:
  consigne_salon:
    name: Consigne Salon
    min: 16
    max: 23
    step: 0.5
    unit_of_measurement: "°C"
    icon: mdi:thermometer
```

**Capteurs de fenêtre**

Si vous avez des contacts de porte/fenêtre, vérifiez qu'ils sont bien configurés comme `binary_sensor` avec :
- `device_class: window` ou `device_class: door`
- État `on` quand ouvert
- État `off` quand fermé

### 3. Test de fonctionnement

1. **Créez une automatisation de test**
   - Utilisez le blueprint choisi
   - Configurez avec vos entités
   - Activez le mode trace

2. **Testez les déclencheurs**
   - Ouvrez/fermez une fenêtre
   - Armez/désarmez l'alarme
   - Allumez/éteignez la lumière (X4FP Bathroom)
   - Modifiez la consigne (X4FP Room)

3. **Consultez les logs**
   - Onglet **Logbook** (📖)
   - Filtrez par nom de pièce
   - Vérifiez les actions effectuées

4. **Mode trace**
   - Dans l'automatisation, cliquez sur **⋮** → **Trace**
   - Consultez le déroulement détaillé

---

## Mise à jour des blueprints

### Import direct

Si vous avez utilisé l'import direct :
1. Allez dans **Blueprints**
2. Trouvez le blueprint à mettre à jour
3. Cliquez sur **⋮** → **Réimporter**
4. Confirmez

### Installation manuelle

Si vous avez installé manuellement :
1. Remplacez le fichier YAML dans `/config/blueprints/automation/lacasehome/`
2. Rechargez les blueprints (Outils de développement → YAML → Recharger blueprints)

**Note :** Les automatisations existantes ne seront pas affectées tant que vous ne les modifiez pas.

---

## Désinstallation

### Supprimer une automatisation

1. Allez dans **Automatisations**
2. Trouvez l'automatisation basée sur le blueprint
3. Cliquez sur **⋮** → **Supprimer**
4. Confirmez

### Supprimer un blueprint

**Méthode Interface (Import direct)**
1. Allez dans **Blueprints**
2. Trouvez le blueprint
3. Cliquez sur **⋮** → **Supprimer**
4. Confirmez

**Méthode Fichiers (Installation manuelle)**
1. Accédez à `/config/blueprints/automation/lacasehome/`
2. Supprimez le fichier YAML du blueprint
3. Rechargez les blueprints

---

## Résolution de problèmes

### Le blueprint n'apparaît pas après l'import

**Solutions :**
1. Rechargez les blueprints : Outils de développement → YAML → Recharger blueprints
2. Videz le cache du navigateur (Ctrl+F5)
3. Vérifiez les logs : Paramètres → Système → Logs

### Erreur "Invalid blueprint"

**Causes possibles :**
- Fichier YAML mal formaté
- Version Home Assistant trop ancienne
- Copie incomplète du fichier

**Solutions :**
1. Vérifiez que vous avez copié tout le contenu
2. Vérifiez la syntaxe YAML (pas de tabulations, indentation correcte)
3. Mettez à jour Home Assistant (minimum 2023.8)

### L'automatisation ne se déclenche pas

**Vérifications :**
1. L'automatisation est bien activée (toggle ON)
2. Les entités existent et sont correctement nommées
3. Mode trace activé pour débugger
4. Consultez le logbook pour voir les actions

### Erreur "Entity not found"

**Solutions :**
1. Vérifiez que toutes les entités configurées existent
2. Respectez la casse (majuscules/minuscules)
3. Utilisez l'outil de sélection d'entités dans l'interface
4. Pour les entités optionnelles, laissez le champ vide

---

## Support

Si vous rencontrez des problèmes :

1. Consultez le [Troubleshooting](docs/troubleshooting.md)
2. Vérifiez les [Issues GitHub](https://github.com/GevaudanBeast/ha-climate-blueprint/issues)
3. Créez une nouvelle issue avec :
   - Version Home Assistant
   - Blueprint concerné
   - Logs d'erreur
   - Configuration (anonymisée)

---

## Prochaines étapes

Après l'installation, consultez :
- [Documentation des blueprints](README.md#blueprints-disponibles)
- [Guide de configuration détaillé](docs/)
- [FAQ](README.md#faq)
