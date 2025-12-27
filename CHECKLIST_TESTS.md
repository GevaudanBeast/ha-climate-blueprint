# Checklist de Tests - Blueprints HVAC

## 🎯 Blueprint: Thermostat Heat v3.7

### Tests Alarme
- [ ] Armer l'alarme (`armed_away`) → Le thermostat passe en mode ECO
- [ ] Armer l'alarme (`armed_home`) → Le thermostat passe en mode ECO
- [ ] Désarmer l'alarme (`disarmed`) → Le thermostat passe en mode CONFORT (ou planning si actif)

### Tests Planning Horaire ⭐ NEW
- [ ] Créer `schedule.test_confort` ON maintenant → Preset `preset_schedule_on` (ex: COMFORT)
- [ ] Passer le schedule à OFF → Preset `preset_schedule_off` (ex: ECO)
- [ ] Schedule ON + armer alarme → Planning ignoré, mode ECO (alarme prioritaire)
- [ ] Schedule OFF + désarmer alarme → Preset `preset_schedule_off` appliqué
- [ ] Vérifier trace : `trigger.id = schedule_change` quand schedule change
- [ ] Vérifier variable : `schedule_preset` = preset_schedule_on ou preset_schedule_off
- [ ] Vérifier logbook : message `📅 Planning → COMFORT` ou `📅 Planning → ECO`

### Tests Été
- [ ] Activer mode été → Le thermostat passe en OFF
- [ ] Désactiver mode été → Le thermostat passe en HEAT

### Tests Fenêtre
- [ ] Ouvrir une fenêtre → Le thermostat passe en OFF (après délai)
- [ ] Fermer la fenêtre → Le thermostat revient en HEAT (après délai)

### Tests Priorités
- [ ] Été actif + Planning → Été prioritaire (OFF)
- [ ] Fenêtre ouverte + Planning → Fenêtre prioritaire (OFF)
- [ ] Alarme armée + Planning → Alarme prioritaire (ignore planning)

---

## 🎯 Blueprint: Room Thermostat v2.10

### Tests Alarme
- [ ] Armer l'alarme → Preset AWAY activé (ou OFF si pas de preset away)
- [ ] Désarmer l'alarme → Preset HOME activé (ou planning si actif)

### Tests Planning Horaire ⭐ NEW
- [ ] Schedule ON (alarme désarmée) → Preset `preset_schedule_on` appliqué
- [ ] Schedule OFF (alarme désarmée) → Preset `preset_schedule_off` appliqué
- [ ] Schedule ON + armer alarme → Planning ignoré, AWAY activé
- [ ] Schedule OFF + armer alarme → Planning ignoré, AWAY activé
- [ ] Vérifier trace : `schedule_preset` = preset_schedule_on ou preset_schedule_off
- [ ] Vérifier logbook : message `📅 Planning → COMFORT` ou `📅 Planning → ECO`

### Tests Été/Hiver
- [ ] Mode été ON → Mode COOL + température été
- [ ] Mode hiver (été OFF) → Mode HEAT + température hiver

### Tests Solar Optimizer
- [ ] Solar ON (de jour) → Laisser Solar Optimizer piloter
- [ ] Solar OFF → Reprendre contrôle normal
- [ ] Solar ON (de nuit) → Ignorer Solar (nuit détectée)

### Tests Fenêtre
- [ ] Fenêtre ouverte en mode HEAT → Passe en OFF
- [ ] Fenêtre ouverte en mode COOL → Passe en OFF
- [ ] Fenêtre fermée → Reprendre le mode (HEAT ou COOL)

### Tests Priorités
- [ ] Fenêtre > Solar > Alarme > Planning > Défaut
- [ ] Vérifier chaque niveau de priorité

---

## 🎯 Blueprint: X4FP Bathroom v7.17

### Tests Lumière
- [ ] Allumer la lumière (alarme désarmée, pas de planning) → Mode CONFORT
- [ ] Éteindre la lumière (alarme désarmée, pas de planning) → Mode ECO

### Tests Planning Horaire ⭐ NEW
- [ ] Schedule ON + lumière OFF → Planning prioritaire (ignore lumière), preset `preset_schedule_on`
- [ ] Schedule ON + lumière ON → Planning prioritaire (ignore lumière), preset `preset_schedule_on`
- [ ] Schedule OFF + lumière ON → Planning appliqué (ignore lumière), preset `preset_schedule_off`
- [ ] Pas de schedule configuré + lumière ON → Gestion lumière normale (CONFORT)
- [ ] Schedule ON + armer alarme → Planning ignoré, AWAY activé
- [ ] Vérifier logbook : `📅 Planning → COMFORT` ou `📅 Planning → ECO` (pas message lumière)

### Tests Alarme
- [ ] Armer l'alarme → Mode AWAY
- [ ] Désarmer l'alarme + lumière OFF → Mode ECO (ou planning si actif)
- [ ] Désarmer l'alarme + lumière ON → Mode CONFORT (ou planning si actif)

### Tests Été
- [ ] Été + politique OFF → Mode OFF
- [ ] Été + politique ECO → Mode ECO
- [ ] Désactiver été → Reprendre contrôle normal

### Tests Solar Optimizer
- [ ] Solar ON + lumière OFF (de jour) → Mode CONFORT (Solar prioritaire)
- [ ] Solar ON + lumière ON → Solar prioritaire (CONFORT)
- [ ] Solar OFF → Reprendre contrôle (planning ou lumière)

### Tests Fenêtre
- [ ] Fenêtre ouverte → Mode AWAY (pause)
- [ ] Fenêtre fermée → Reprendre contrôle

### Tests Priorités
- [ ] Été > Fenêtre > Solar > Away > Planning > Lumière
- [ ] Vérifier chaque niveau de priorité

---

## 🎯 Blueprint: X4FP Room v7.14

### Tests Alarme
- [ ] Armer l'alarme → Mode AWAY
- [ ] Désarmer l'alarme → Contrôle thermique ou planning (si configuré)

### Tests Planning Horaire ⭐ NEW
- [ ] Schedule ON + température basse → Planning prioritaire (ignore thermique), preset `preset_schedule_on`
- [ ] Schedule ON + température haute → Planning prioritaire (ignore thermique), preset `preset_schedule_on`
- [ ] Schedule OFF + température basse → Planning appliqué (ignore thermique), preset `preset_schedule_off`
- [ ] Pas de schedule configuré → Contrôle thermique normal
- [ ] Schedule ON + armer alarme → Planning ignoré, AWAY activé
- [ ] Vérifier logbook : `📅 Planning → COMFORT` ou `📅 Planning → ECO` (pas message thermique)

### Tests Été
- [ ] Été + politique OFF → Mode OFF
- [ ] Été + politique ECO → Mode ECO
- [ ] Désactiver été → Reprendre contrôle normal

### Tests Solar Optimizer
- [ ] Solar ON (de jour) → Mode CONFORT
- [ ] Solar ON + Alarme armée + autorisation SO en Away → Solar prioritaire
- [ ] Solar ON + Alarme armée SANS autorisation → Alarme prioritaire (AWAY)

### Tests Thermique (si capteur configuré et pas de planning actif)
- [ ] Température < consigne - hystérésis → Mode HEAT (comfort)
- [ ] Température > consigne + hystérésis → Mode IDLE (eco)
- [ ] Température dans la bande → Maintenir état actuel

### Tests Fenêtre
- [ ] Fenêtre ouverte → Mode AWAY (pause)
- [ ] Fenêtre fermée → Reprendre contrôle

### Tests Priorités
- [ ] Été > Fenêtre > Solar > Away > Planning > Thermique
- [ ] Vérifier chaque niveau de priorité

---

## 📝 Comment Tester

### Méthode Simple

1. **Activer le mode Trace** sur votre automatisation
2. **Effectuer l'action** (armer alarme, allumer lumière, etc.)
3. **Attendre 5 secondes**
4. **Vérifier** :
   - L'automatisation s'est déclenchée (check dans Traces)
   - Le preset/mode est correct sur le climate
   - Le message dans le logbook correspond

### Exemple de Test

```
Test: Armer l'alarme
1. Ouvrir Traces de l'automatisation
2. Armer l'alarme (armed_away)
3. Attendre 5 secondes
4. Vérifier dans Traces:
   - Variable is_away = true ✅
   - Action: set_preset_mode = eco ✅
5. Vérifier le climate:
   - Preset = eco ✅
6. Cocher la case dans la checklist ✅
```

---

## 🐛 Si un Test Échoue

1. **Ouvrir les Traces** de l'automatisation
2. **Noter** :
   - Quelle variable est incorrecte (is_away, is_summer, etc.)
   - Quelle est sa valeur (true/false)
   - Quelle action a été exécutée
3. **Vérifier l'état** de l'entité source :
   - Developer Tools → States
   - Noter l'état exact (avec la casse)

---

## ✅ Validation Complète

### Pour considérer un blueprint comme validé :
- [ ] Tous les tests alarme passent
- [ ] Tous les tests été passent
- [ ] Tous les tests fenêtre passent
- [ ] Tous les tests de priorités passent
- [ ] Tests effectués avec états en minuscules ET majuscules

### États à tester avec différentes casses :
- `on` / `On` / `ON`
- `armed_away` / `Armed_away` / `ARMED_AWAY`
- `disarmed` / `Disarmed` / `DISARMED`
