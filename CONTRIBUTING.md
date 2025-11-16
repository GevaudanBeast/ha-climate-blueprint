# Guide de Contribution

Merci de votre intérêt pour contribuer aux blueprints Climate pour Home Assistant ! Ce guide vous aidera à contribuer efficacement.

---

## Table des matières

1. [Code de conduite](#code-de-conduite)
2. [Comment contribuer](#comment-contribuer)
3. [Signaler un bug](#signaler-un-bug)
4. [Suggérer une fonctionnalité](#suggérer-une-fonctionnalité)
5. [Soumettre une Pull Request](#soumettre-une-pull-request)
6. [Standards de code](#standards-de-code)
7. [Documentation](#documentation)
8. [Tests](#tests)

---

## Code de conduite

Ce projet suit le [Code de conduite Contributor Covenant](https://www.contributor-covenant.org/). En participant, vous vous engagez à respecter ce code.

**Principes :**
- Soyez respectueux et inclusif
- Acceptez les critiques constructives
- Concentrez-vous sur ce qui est meilleur pour la communauté
- Faites preuve d'empathie envers les autres membres

---

## Comment contribuer

Il existe plusieurs façons de contribuer :

### 1. Signaler des bugs
Voir [Signaler un bug](#signaler-un-bug)

### 2. Suggérer des fonctionnalités
Voir [Suggérer une fonctionnalité](#suggérer-une-fonctionnalité)

### 3. Améliorer la documentation
- Corriger des fautes de frappe
- Clarifier des explications
- Ajouter des exemples
- Traduire (si pertinent)

### 4. Soumettre du code
- Corriger des bugs
- Ajouter des fonctionnalités
- Optimiser le code existant

### 5. Aider les autres
- Répondre aux questions dans les issues
- Partager votre expérience
- Aider au dépannage

---

## Signaler un bug

### Avant de signaler

1. **Vérifiez les issues existantes**
   - Recherchez si le bug n'a pas déjà été signalé
   - Consultez les issues fermées également

2. **Consultez la documentation**
   - [Troubleshooting](docs/troubleshooting.md)
   - Documentation du blueprint concerné
   - [FAQ](README.md#faq)

3. **Reproduisez le bug**
   - Assurez-vous que c'est reproductible
   - Testez avec configuration minimale

### Créer une issue

**Template à utiliser :**

```markdown
## Description du bug
[Description claire et concise du problème]

## Blueprint concerné
- [ ] Thermostat Heat (v3.0)
- [ ] Room Thermostat (v2.0)
- [ ] X4FP Bathroom (v7.2)
- [ ] X4FP Room (v7.2)

## Environnement
- **Home Assistant** : [version, ex: 2024.11.1]
- **Installation** : [HA OS / Docker / Core / Supervised]
- **Blueprint version** : [ex: v7.2]

## Configuration
[Configuration du blueprint - ANONYMISÉE]

```yaml
# Exemple (anonymisé)
Nom de la pièce: Pièce1
Entité climate: climate.radiateur_xxx
# etc.
```

## Comportement attendu
[Ce qui devrait se passer]

## Comportement observé
[Ce qui se passe réellement]

## Étapes pour reproduire
1. [Première étape]
2. [Deuxième étape]
3. [...]

## Logs
[Logs Home Assistant pertinents]

```
[Coller les logs ici]
```

## Mode Trace
[Screenshot ou export du mode trace si pertinent]

## Informations supplémentaires
[Tout autre contexte utile]
```

**Conseils :**
- Soyez précis et concis
- Anonymisez vos données personnelles
- Fournissez les logs complets
- Incluez le mode trace si possible

---

## Suggérer une fonctionnalité

### Avant de suggérer

1. **Vérifiez les issues existantes**
   - Quelqu'un a peut-être déjà suggéré la même chose

2. **Vérifiez que ce n'est pas déjà disponible**
   - Consultez la documentation complète
   - Vérifiez les dernières versions

### Créer une issue

**Template à utiliser :**

```markdown
## Résumé de la fonctionnalité
[Résumé en une phrase]

## Motivation / Cas d'usage
[Pourquoi cette fonctionnalité est utile]
[Quel problème elle résout]
[Cas d'usage concret]

## Solution proposée
[Comment vous imaginez que ça devrait fonctionner]

## Alternatives considérées
[Autres solutions que vous avez envisagées]

## Blueprint(s) concerné(s)
- [ ] Thermostat Heat
- [ ] Room Thermostat
- [ ] X4FP Bathroom
- [ ] X4FP Room
- [ ] Nouveau blueprint

## Informations supplémentaires
[Tout autre contexte, screenshots, exemples]
```

**Conseils :**
- Expliquez le "pourquoi" avant le "comment"
- Donnez des exemples concrets
- Soyez ouvert aux alternatives

---

## Soumettre une Pull Request

### Workflow

1. **Forkez le repository**
   ```bash
   # Sur GitHub, cliquez "Fork"
   ```

2. **Clonez votre fork**
   ```bash
   git clone https://github.com/VOTRE_USERNAME/ha-climate-blueprint.git
   cd ha-climate-blueprint
   ```

3. **Créez une branche**
   ```bash
   git checkout -b feature/ma-fonctionnalite
   # ou
   git checkout -b fix/mon-bug
   ```

4. **Faites vos modifications**
   - Suivez les [standards de code](#standards-de-code)
   - Testez vos changements
   - Mettez à jour la documentation

5. **Committez**
   ```bash
   git add .
   git commit -m "feat: Description de la fonctionnalité"
   # ou
   git commit -m "fix: Description du fix"
   ```

6. **Pushez**
   ```bash
   git push origin feature/ma-fonctionnalite
   ```

7. **Créez la Pull Request**
   - Sur GitHub, cliquez "New Pull Request"
   - Remplissez le template
   - Attendez la review

### Template Pull Request

```markdown
## Description
[Description claire des changements]

## Type de changement
- [ ] Bug fix (changement non-breaking qui corrige un bug)
- [ ] New feature (changement non-breaking qui ajoute une fonctionnalité)
- [ ] Breaking change (correction ou fonctionnalité qui change le comportement existant)
- [ ] Documentation (amélioration de la documentation)

## Blueprint(s) concerné(s)
- [ ] Thermostat Heat
- [ ] Room Thermostat
- [ ] X4FP Bathroom
- [ ] X4FP Room
- [ ] Documentation
- [ ] Autre

## Checklist
- [ ] Mon code suit les standards du projet
- [ ] J'ai testé mes changements
- [ ] J'ai mis à jour la documentation
- [ ] J'ai mis à jour le CHANGELOG (si pertinent)
- [ ] Mes commits suivent les conventions
- [ ] J'ai ajouté des tests (si applicable)

## Tests effectués
[Description des tests]

## Screenshots / Logs
[Si pertinent]

## Issues liées
Closes #[numéro issue]
```

### Convention de commits

Utilisez [Conventional Commits](https://www.conventionalcommits.org/) :

```bash
feat: Ajout support preset boost
fix: Correction trigger alarme
docs: Mise à jour README Solar Optimizer
refactor: Optimisation logique hystérésis
test: Ajout tests contrôle thermique
chore: Mise à jour dépendances
```

**Types :**
- `feat`: Nouvelle fonctionnalité
- `fix`: Correction de bug
- `docs`: Documentation seulement
- `style`: Formatage, virgules, etc. (pas de changement de code)
- `refactor`: Refactoring (ni feat ni fix)
- `perf`: Amélioration de performance
- `test`: Ajout ou correction de tests
- `chore`: Maintenance (dépendances, CI, etc.)

---

## Standards de code

### Blueprints (YAML)

**Indentation :**
```yaml
# Utilisez 2 espaces (PAS de tabulations)
blueprint:
  name: Nom du Blueprint
  input:
    param1:
      name: Paramètre 1
      default: valeur
```

**Nommage :**
```yaml
# Snake_case pour IDs
param_name:
  name: Param Name  # Titre pour utilisateur

# Variables claires
is_summer: "{{ ... }}"  # Préfixe is_ pour booléens
preset_heat: !input preset_heat  # Nom explicite
```

**Commentaires :**
```yaml
# Commentaires en français (cohérent avec le projet)
# Expliquez les sections complexes

# 1) ÉTÉ (priorité absolue)
- choose:
    - conditions: "{{ is_summer }}"
      sequence:
        # ...
```

**Templates :**
```yaml
# Lisible et indenté
value_template: >-
  {% set temp = states('sensor.temp') | float(0) %}
  {% set consigne = states('input_number.consigne') | float(20) %}
  {{ temp <= (consigne - hys) }}
```

### Documentation (Markdown)

**Structure :**
```markdown
# Titre niveau 1

## Titre niveau 2

### Titre niveau 3

**Gras** pour mise en évidence
*Italique* pour nuance

`code inline` pour noms techniques

```yaml
# Bloc de code avec langage
```

**Tableaux :**
```markdown
| Colonne 1 | Colonne 2 |
|-----------|-----------|
| Valeur A  | Valeur B  |
```

**Liens :**
```markdown
[Texte du lien](chemin/vers/fichier.md)
[Lien externe](https://example.com)
```

### Cohérence

- **Langue** : Français (blueprints et docs principales)
- **Émojis** : Utilisés avec parcimonie (logs, titres)
- **Termes** :
  - "Preset" (pas "Mode preset")
  - "Blueprint" (pas "Modèle")
  - "Away" (mode absent)
  - "Solar Optimizer" ou "SO"

---

## Documentation

### Quoi documenter

Toute modification doit inclure mise à jour documentation :

1. **Nouveau paramètre** → Documentation blueprint concerné
2. **Nouveau comportement** → README + Doc blueprint
3. **Breaking change** → CHANGELOG + Migration guide
4. **Bug fix** → CHANGELOG

### Structure documentation

```
README.md                   # Vue d'ensemble
INSTALLATION.md             # Guide installation
CONTRIBUTING.md             # Ce fichier
docs/
  ├── thermostat_heat.md    # Doc détaillée blueprint 1
  ├── room_thermostat.md    # Doc détaillée blueprint 2
  ├── x4fp_bathroom.md      # Doc détaillée blueprint 3
  ├── x4fp_room.md          # Doc détaillée blueprint 4
  ├── solar_optimizer.md    # Guide Solar Optimizer
  └── troubleshooting.md    # Guide dépannage
```

### Exemples

Incluez toujours des exemples concrets :

```yaml
# ✅ BON
## Exemple : Salle de bain avec SO

```yaml
Nom: Salle de Bain
Climate: climate.x4fp_sdb
Lumière: light.sdb
SO Switch: switch.solar_optimizer_sdb
```

# ❌ MAUVAIS
Configurez les paramètres selon vos besoins.
```

---

## Tests

### Tests manuels requis

Avant de soumettre une PR :

1. **Test basique**
   - Importez le blueprint modifié
   - Créez automatisation avec config minimale
   - Testez déclenchement manuel

2. **Test des triggers**
   - Testez chaque type de trigger
   - Vérifiez délais
   - Vérifiez logs

3. **Test mode trace**
   - Exécutez avec trace activé
   - Vérifiez toutes les branches (choose)
   - Vérifiez variables calculées

4. **Test edge cases**
   - Entités manquantes (optionnelles)
   - Valeurs extrêmes
   - États "unknown"/"unavailable"

### Documentation des tests

Dans la PR, décrivez vos tests :

```markdown
## Tests effectués

### Configuration testée
- HA version: 2024.11.1
- Blueprint: X4FP Room
- Thermostat: Qubino Fil Pilote (Zigbee)

### Scénarios testés
- [x] Contrôle thermique basique (T° monte/descend)
- [x] Hystérésis (pas de changement dans zone neutre)
- [x] Garde-fous (consigne > max → clamped)
- [x] SO prioritaire (override contrôle thermique)
- [x] Away (bloque SO si non autorisé)
- [x] Fenêtre ouverte (pause)

### Résultats
Tous les scénarios fonctionnent comme attendu.
Logs corrects, pas d'erreurs dans HA.
```

---

## Processus de review

### Pour les contributeurs

1. **Patience**
   - Les reviews prennent du temps
   - Les mainteneurs sont bénévoles

2. **Réactivité**
   - Répondez aux commentaires rapidement
   - Faites les modifications demandées

3. **Discussion**
   - Si désaccord, expliquez votre point de vue
   - Soyez ouvert aux suggestions

### Pour les reviewers

1. **Bienveillance**
   - Soyez constructif
   - Expliquez le "pourquoi"

2. **Clarté**
   - Commentaires précis
   - Suggestions concrètes

3. **Tests**
   - Testez le code si possible
   - Validez la documentation

---

## Licence

En contribuant, vous acceptez que vos contributions soient sous licence MIT (même licence que le projet).

---

## Questions ?

- **Issues GitHub** : Pour questions publiques
- **Discussions** : Pour questions générales (si activé)

---

## Remerciements

Merci à tous les contributeurs qui aident à améliorer ces blueprints ! 🙏

- Chaque contribution, même petite, est appréciée
- Les retours d'expérience sont précieux
- Le partage de connaissances aide toute la communauté

---

**Liens utiles :**
- [README principal](README.md)
- [Guide d'installation](INSTALLATION.md)
- [Troubleshooting](docs/troubleshooting.md)
- [Code de conduite](https://www.contributor-covenant.org/)
