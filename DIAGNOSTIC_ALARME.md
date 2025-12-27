# Diagnostic : Problème de Changement de Mode avec l'Alarme

## 🔍 Analyse de la Trace

Vous m'avez fourni une trace d'exécution qui montre :

```json
{
  "trigger": {
    "id": "alarm_change",
    "platform": null
  },
  "variables": {
    "is_away": true,
    "target_preset": "eco",
    "current_preset": "comfort"
  },
  "action": "climate.set_preset_mode",
  "data": {
    "preset_mode": "eco"
  },
  "logbook": "🔒 Away → ECO"
}
```

### ✅ Ce qui fonctionne CORRECTEMENT

1. **Trigger détecté** : `"id": "alarm_change"` ✅
2. **État alarme détecté** : `is_away: true` ✅
3. **Preset calculé** : `target_preset: "eco"` ✅
4. **Action exécutée** : `climate.set_preset_mode` avec `eco` ✅
5. **Message logbook** : "🔒 Away → ECO" ✅

### ❓ Le `"platform": null`

**C'est NORMAL** pour les triggers de type `template`. Les blueprints utilisent :

```yaml
- id: alarm_change
  platform: template
  value_template: >-
    {{ states(alarm_id) if alarm_id and alarm_id != '' else 'none' }}
```

Les triggers `template` n'ont pas de "platform" traditionnel comme les triggers `state`, donc Home Assistant enregistre `null` dans les traces. **Ce n'est pas une erreur**.

---

## 🎯 Le Vrai Problème

Vos automatisations contiennent des **paramètres obsolètes** des anciennes versions :

### Anciennes configurations détectées :

```yaml
# ❌ Ces paramètres n'existent plus dans v3.6/v2.9/v7.16/v7.13
eco_flag_entity: input_boolean.eco_flag
solar_enable: true
solar_behavior: comfort
solar_setpoint: 22
```

### Nouvelles versions (v3.6, v2.9, v7.16, v7.13) :

```yaml
# ✅ Configuration simplifiée
alarm_entity: alarm_control_panel.votre_alarme
preset_when_armed: eco
preset_when_disarmed: comfort

# ✅ Solar Optimizer automatique (plus de solar_enable)
solar_entity: switch.solar_optimizer  # Détection auto des attributs
```

---

## 🔧 Solution : Recréer les Automatisations

### Pourquoi recréer ?

Les automatisations créées avec les anciennes versions des blueprints conservent les anciens paramètres dans leur configuration interne, même si vous avez importé les nouveaux blueprints.

**La seule solution** : supprimer et recréer les automatisations.

### Procédure complète

📖 **Suivez le guide** : [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md)

**Résumé rapide** :

1. **Sauvegardez** vos configurations actuelles (copier YAML)
2. **Supprimez** les anciennes automatisations
3. **Vérifiez** que vous avez les bons blueprints (v3.6, v2.9, v7.16, v7.13)
4. **Recréez** les automatisations avec les nouveaux paramètres
5. **Testez** avec [CHECKLIST_TESTS.md](CHECKLIST_TESTS.md)

---

## 🧪 Comment Tester Correctement

### ⚠️ Erreurs communes de test

**❌ NE PAS** :
- Tester en exécutant manuellement l'automatisation (bouton "Exécuter")
- Modifier l'état de l'alarme puis attendre 30 secondes

**✅ FAIRE** :
1. Activer le **mode Trace** sur l'automatisation
2. Armer/désarmer l'alarme via l'interface HA
3. Attendre **5 secondes maximum**
4. Vérifier qu'une **nouvelle trace** apparaît
5. Vérifier le **preset du thermostat**

### Exemple de Test Complet

```bash
# Test : Alarme Armed → Preset ECO

1. Ouvrir Traces de l'automatisation
2. Noter l'heure actuelle
3. Armer l'alarme (armed_away)
4. Attendre 5 secondes
5. Rafraîchir les Traces
6. Vérifier :
   ✓ Nouvelle trace apparue à l'heure de l'armement
   ✓ Trigger ID = "alarm_change"
   ✓ Variable is_away = true
   ✓ Action = set_preset_mode eco
   ✓ Thermostat affiche preset = eco
7. Consulter le Logbook :
   ✓ Message "🔒 Away → ECO" à l'heure correcte
```

---

## 📊 Checklist de Validation

Après avoir recréé les automatisations :

### Pour Thermostat Heat v3.6

- [ ] Armer alarme → Preset ECO (trace + vérif thermostat)
- [ ] Désarmer alarme → Preset CONFORT (trace + vérif thermostat)
- [ ] Activer été → Mode OFF (trace + vérif thermostat)
- [ ] Ouvrir fenêtre → Mode OFF après délai
- [ ] Fermer fenêtre → Mode HEAT après délai

### Pour Room Thermostat v2.9

- [ ] Armer alarme → Preset AWAY
- [ ] Désarmer alarme → Preset HOME (ou NONE)
- [ ] Activer été → Mode COOL + temp été
- [ ] Désactiver été → Mode HEAT + temp hiver
- [ ] Solar ON (jour) → Solar Optimizer prend le contrôle
- [ ] Solar OFF → Reprendre contrôle normal

### Pour X4FP Bathroom v7.16

- [ ] Allumer lumière (alarme off) → Preset CONFORT
- [ ] Éteindre lumière (alarme off) → Preset ECO
- [ ] Armer alarme → Preset AWAY
- [ ] Désarmer alarme + lumière ON → Preset CONFORT
- [ ] Été ON + politique OFF → Mode OFF
- [ ] Solar ON → Preset CONFORT (Solar prioritaire)

### Pour X4FP Room v7.13

- [ ] Armer alarme → Preset AWAY
- [ ] Désarmer alarme → Contrôle thermique
- [ ] Température < consigne → Mode HEAT (comfort)
- [ ] Température > consigne → Mode IDLE (eco)
- [ ] Solar ON → Mode CONFORT
- [ ] Été ON + politique OFF → Mode OFF

---

## 🐛 Si Ça Ne Fonctionne Toujours Pas

### 1. Vérifier l'entité alarme

**Outils de développement** → **États** → Rechercher votre alarme

Vérifiez l'état exact :
```
alarm_control_panel.votre_alarme
  state: armed_away    ✅ (minuscules)
  state: Armed_away    ✅ (fix v3.6 gère la casse)
  state: ARMED_AWAY    ✅ (fix v3.6 gère la casse)
```

### 2. Vérifier les presets supportés

**Outils de développement** → **États** → Votre thermostat

```yaml
climate.votre_thermostat
  attributes:
    preset_modes: [eco, comfort, boost, ...]
```

⚠️ Si le preset `eco` ou `comfort` n'est pas dans la liste, le blueprint utilisera les **températures fallback** à la place.

### 3. Vérifier les logs système

**Paramètres** → **Système** → **Logs**

Recherchez :
- Erreurs contenant le nom de votre automatisation
- Erreurs `climate.set_preset_mode`
- Erreurs `UndefinedError`

### 4. Recharger ou Redémarrer

1. **Rechargement automatisations** :
   - **Outils de développement** → **YAML** → **Rechargement des automatisations**

2. Si ça persiste, **redémarrer Home Assistant** :
   - **Paramètres** → **Système** → **Redémarrer**

---

## 💡 Points Clés à Retenir

1. ✅ **Les blueprints v3.6/v2.9/v7.16/v7.13 sont CORRECTS**
   - Détection insensible à la casse (`.lower()`)
   - Triggers template fonctionnels

2. ⚠️ **Les anciennes configurations sont INCOMPATIBLES**
   - Paramètres obsolètes : `eco_flag_entity`, `solar_enable`, etc.
   - **Solution** : recréer les automatisations

3. 🧪 **Tester correctement**
   - Utiliser mode Trace
   - Provoquer un changement d'état réel (pas exécution manuelle)
   - Vérifier les traces + logbook + état du thermostat

4. 📖 **Documentation complète disponible**
   - [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md) : procédure complète
   - [CHECKLIST_TESTS.md](CHECKLIST_TESTS.md) : liste de tests
   - [test_entities.yaml](test_entities.yaml) : entités de test
   - [test_scripts.yaml](test_scripts.yaml) : scripts de test

---

## 📞 Besoin d'Aide ?

Si après avoir recréé les automatisations le problème persiste :

1. Exportez la **trace complète** (JSON) d'une exécution qui ne fonctionne pas
2. Exportez la **configuration YAML** de l'automatisation
3. Vérifiez les **logs système**
4. Ouvrez une [issue GitHub](https://github.com/GevaudanBeast/ha-climate-blueprint/issues)

---

**Bon courage pour la migration ! 🚀**
