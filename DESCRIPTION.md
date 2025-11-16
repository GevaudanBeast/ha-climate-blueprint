# Blueprints HVAC pour Home Assistant

Collection de blueprints avancés pour l'automatisation intelligente du chauffage et de la climatisation dans Home Assistant.

## 🎯 Vue d'ensemble

4 blueprints optimisés et testés pour gérer automatiquement vos systèmes de chauffage selon vos besoins : présence, saisons, fenêtres ouvertes, et énergie solaire.

### ✨ Caractéristiques principales

- 🏠 **Gestion présence** : Basculement automatique eco/confort selon alarme
- 🌡️ **Saisonnier** : Détection été/hiver avec modes heat/cool automatiques
- 🪟 **Détection fenêtres** : Pause automatique si ouverture (avec délais configurables)
- ☀️ **Solar Optimizer** : Priorité au chauffage solaire gratuit
- ⚙️ **Fil Pilote X4FP** : Support natif des radiateurs électriques français
- 🔄 **Tick optimisé** : Ré-application périodique pour fiabilité maximale
- 📝 **Logs détaillés** : Traçabilité complète dans le journal Home Assistant

## 📦 Les 4 blueprints

### 1. Thermostat Chauffage (Alarme = Eco/Confort)
Blueprint simple pour chauffage classique piloté par alarme. Idéal pour chauffage central ou thermostats intelligents.

**Cas d'usage :** Maison avec thermostat unique, réduction automatique quand absent.

### 2. Pièce (Thermostat/Clim) – Été/Hiver + Away + SO
Blueprint universel avec bascule automatique chauffage/climatisation selon la saison.

**Cas d'usage :** Climatisations réversibles, chambres avec clim, bureaux.

### 3. X4FP Salle de bain (Lumière + SO)
Contrôle sèche-serviettes par détection de présence (lumière). Compatible Solar Optimizer.

**Cas d'usage :** Salle de bain avec sèche-serviettes électrique fil pilote.

### 4. X4FP Pièce (Thermique + SO)
Régulation avancée par hystérésis avec capteur température. Le plus sophistiqué.

**Cas d'usage :** Radiateurs électriques fil pilote avec régulation fine de température.

## 🚀 Installation rapide

```bash
# Importer dans Home Assistant
Paramètres > Automations et scènes > Blueprints > Importer un Blueprint
```

Puis coller l'URL du blueprint souhaité :
```
https://github.com/GevaudanBeast/ha-climate-blueprint/blob/main/blueprints/blueprint_hvac_*.yaml
```

## 📊 Comparaison rapide

| Fonction | Thermostat Heat | Room Thermo | X4FP Bathroom | X4FP Room |
|----------|:---------------:|:-----------:|:-------------:|:---------:|
| Alarme eco/away | ✅ | ✅ | ✅ | ✅ |
| Été/Hiver auto | ❌ | ✅ | ❌ | ❌ |
| Fenêtres | ✅ | ✅ | ✅ | ✅ |
| Solar Optimizer | ❌ | ✅ | ✅ | ✅ |
| X4FP (Fil Pilote) | ❌ | ❌ | ✅ | ✅ |
| Lumière = présence | ❌ | ❌ | ✅ | ❌ |
| Régulation T° | ❌ | ❌ | ❌ | ✅ |

## 💡 Points forts

- ✅ **Production-ready** : Testés et validés en conditions réelles
- ✅ **Documentation complète** : 118 pages de guides détaillés
- ✅ **Code optimisé** : Simplifications, -4.1% lignes, 0 bug
- ✅ **Solar Optimizer** : Maximise utilisation énergie solaire gratuite
- ✅ **Logs intelligents** : Uniquement lors de changements réels
- ✅ **Français** : Documentation et messages en français

## 📚 Documentation

- **[README.md](README.md)** - Vue d'ensemble et tableau comparatif complet
- **[INSTALLATION.md](INSTALLATION.md)** - Guide d'installation pas à pas
- **[docs/](docs/)** - Guides détaillés par blueprint (18-25 pages chacun)
- **[CHANGELOG.md](CHANGELOG.md)** - Historique des versions
- **[troubleshooting.md](docs/troubleshooting.md)** - Résolution 30+ problèmes courants

## 🏆 Versions actuelles

| Blueprint | Version | Statut |
|-----------|---------|--------|
| Thermostat Heat | v3.2 | 🟢 Stable |
| Room Thermostat | v2.2 | 🟢 Stable |
| X4FP Bathroom | v7.3 | 🟢 Stable |
| X4FP Room | v7.3 | 🟢 Stable |

**Dernière mise à jour :** 16 janvier 2025

## 🇬🇧 English Version

A complete English version is available in [DESCRIPTION_EN.md](DESCRIPTION_EN.md).

## 🤝 Contribution

Les contributions sont les bienvenues ! Voir [CONTRIBUTING.md](CONTRIBUTING.md) pour les guidelines.

## 📄 Licence

MIT License - Voir [LICENSE](LICENSE) pour détails.

---

**Auteur :** LaCaseHome
**Support :** [Issues GitHub](https://github.com/GevaudanBeast/ha-climate-blueprint/issues)
