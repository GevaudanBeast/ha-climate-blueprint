# HVAC Blueprints for Home Assistant

Advanced blueprints collection for intelligent heating and air conditioning automation in Home Assistant.

## 🎯 Overview

4 optimized and tested blueprints to automatically manage your heating systems based on your needs: presence, seasons, open windows, and solar energy.

### ✨ Key Features

- 🏠 **Presence management**: Automatic eco/comfort switching based on alarm status
- 🌡️ **Seasonal**: Summer/winter detection with automatic heat/cool modes
- 🪟 **Window detection**: Automatic pause on opening (with configurable delays)
- ☀️ **Solar Optimizer**: Priority to free solar heating
- ⚙️ **X4FP Wire Pilot**: Native support for French electric heaters
- 🔄 **Optimized tick**: Periodic reapplication for maximum reliability
- 📝 **Detailed logs**: Full traceability in Home Assistant logbook

## 📦 The 4 Blueprints

### 1. Thermostat Heat (Alarm = Eco/Comfort)
Simple blueprint for classic heating controlled by alarm. Ideal for central heating or smart thermostats.

**Use case:** Home with single thermostat, automatic reduction when away.

### 2. Room (Thermostat/AC) – Summer/Winter + Away + SO
Universal blueprint with automatic heating/air conditioning switching based on season.

**Use case:** Reversible air conditioners, rooms with AC, offices.

### 3. X4FP Bathroom (Light + SO)
Towel warmer control via presence detection (light). Solar Optimizer compatible.

**Use case:** Bathroom with electric towel warmer (wire pilot).

### 4. X4FP Room (Thermal + SO)
Advanced hysteresis regulation with temperature sensor. The most sophisticated.

**Use case:** Electric wire pilot radiators with fine temperature regulation.

## 🚀 Quick Installation

```bash
# Import into Home Assistant
Settings > Automations & Scenes > Blueprints > Import Blueprint
```

Then paste the URL of the desired blueprint:
```
https://github.com/GevaudanBeast/ha-climate-blueprint/blob/main/blueprints/blueprint_hvac_*.yaml
```

## 📊 Quick Comparison

| Feature | Thermostat Heat | Room Thermo | X4FP Bathroom | X4FP Room |
|---------|:---------------:|:-----------:|:-------------:|:---------:|
| Alarm eco/away | ✅ | ✅ | ✅ | ✅ |
| Summer/Winter auto | ❌ | ✅ | ❌ | ❌ |
| Windows | ✅ | ✅ | ✅ | ✅ |
| Solar Optimizer | ❌ | ✅ | ✅ | ✅ |
| X4FP (Wire Pilot) | ❌ | ❌ | ✅ | ✅ |
| Light = presence | ❌ | ❌ | ✅ | ❌ |
| Temp regulation | ❌ | ❌ | ❌ | ✅ |

## 💡 Strengths

- ✅ **Production-ready**: Tested and validated in real conditions
- ✅ **Complete documentation**: 118 pages of detailed guides
- ✅ **Optimized code**: Simplifications, -4.1% lines, 0 bugs
- ✅ **Solar Optimizer**: Maximizes free solar energy usage
- ✅ **Smart logs**: Only on real changes
- ✅ **French & English**: Bilingual documentation

## 📚 Documentation

- **[README.md](README.md)** - Overview and complete comparison table
- **[INSTALLATION.md](INSTALLATION.md)** - Step-by-step installation guide
- **[docs/](docs/)** - Detailed guides per blueprint (18-25 pages each)
- **[CHANGELOG.md](CHANGELOG.md)** - Version history
- **[troubleshooting.md](docs/troubleshooting.md)** - Solving 30+ common issues

## 🏆 Current Versions

| Blueprint | Version | Status |
|-----------|---------|--------|
| Thermostat Heat | v3.2 | 🟢 Stable |
| Room Thermostat | v2.2 | 🟢 Stable |
| X4FP Bathroom | v7.3 | 🟢 Stable |
| X4FP Room | v7.3 | 🟢 Stable |

**Last update:** January 16, 2025

## 🇫🇷 Version Française

Une version française complète est disponible dans [DESCRIPTION.md](DESCRIPTION.md).

## 🤝 Contributing

Contributions are welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## 📄 License

MIT License - See [LICENSE](LICENSE) for details.

---

**Author:** LaCaseHome
**Support:** [GitHub Issues](https://github.com/GevaudanBeast/ha-climate-blueprint/issues)
