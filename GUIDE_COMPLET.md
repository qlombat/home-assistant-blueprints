# 🌱 Smart Plant Watering v6 - Configuration Guide

## ⚠️ Nouveautés v6 (Breaking Changes)

Si vous migrez depuis v5, **vous devez reconfigurer l'automatisation** :

### Noms d'inputs changés (français → anglais)
- `switch_arrosage` → `watering_switch`
- `heure_arrosage` → `watering_time`
- `capteur_pluie_jour` → `rain_sensor_today`
- `seuil_pluie_annulation` → `rain_cancel_threshold`
- `seuil_pluie_reduction` → `rain_reduction_threshold`
- `duree_base_minutes` → `base_duration_minutes`
- `duree_max_minutes` → `max_duration_minutes`
- `duree_min_minutes` → `min_duration_minutes`
- `compteur_jours_sans_pluie` → `dry_days_counter`
- `capteur_volume_eau` → `water_volume_sensor`
- `compteur_litres_cumules` → `cumulative_liters_counter`
- `statut_actuel` → `current_status`
- `historique_arrosages` → `watering_history`
- `activer_notifications` → `enable_notifications`
- `service_notification` → `notification_service`

### Helpers à recréer (noms anglais)
| Ancien ID | Nouveau ID |
|-----------|------------|
| `input_number.jours_sans_pluie` | `input_number.dry_days_counter` |
| `input_number.arrosage_litres_cumules` | `input_number.watering_cumulative_liters` |
| `input_text.arrosage_statut_actuel` | `input_text.watering_current_status` |
| `input_text.arrosage_historique` | `input_text.watering_history` |

---

## Fonctionnement Global

Ce blueprint gère l'arrosage automatique adaptatif avec :
- Calcul intelligent basé sur température, humidité, vent, pluie
- Suivi des jours consécutifs sans pluie
- Mesure réelle de la consommation via SONOFF SWV
- Affichage temps réel du statut sur dashboard

## 📋 Prérequis

### 1. Créer les Helpers (Paramètres > Appareils & services > Assistants)

#### **input_number** (Nombre)

| Nom | ID | Min | Max | Pas | Usage |
|-----|-----|-----|-----|-----|-------|
| Watering - Dry Days Counter | `input_number.dry_days_counter` | 0 | 30 | 1 | Compteur de jours secs |
| Watering - Cumulative Liters | `input_number.watering_cumulative_liters` | 0 | 99999 | 0.1 | Total litres utilisés |

#### **input_text** (Texte)

| Nom | ID | Longueur max | Usage |
|-----|-----|--------------|-------|
| Watering - Current Status | `input_text.watering_current_status` | 255 | Statut en temps réel |
| Watering - History | `input_text.watering_history` | 255 | 5 dernières sessions |

### 2. Matériel

- **SONOFF SWV** (Smart Water Valve) connecté via Zigbee2MQTT ou ZHA
- Capteurs exposés :
  - `sensor.<nom>_water_consumed` (volume cumulé en litres)
  - `switch.<nom>` (contrôle vanne)

### 3. Intégration Météo

- **IRM KMI** (Belgique) : `weather.irm_kmi_home`
- Ou tout autre service météo exposant : température, humidité, vent, condition

## 🔧 Configuration du Blueprint

Lors de la création de l'automatisation :

### Entités obligatoires
- **Switch arrosage** : `switch.water_valve` (votre SONOFF SWV)
- **Entité météo** : `weather.irm_kmi_home`
- **Capteur volume** : `sensor.water_valve_water_consumed`
- **Compteur jours secs** : `input_number.dry_days_counter`

### Entités optionnelles
- **Capteur pluie jour** : capteur pluie cumulée (si disponible)
- **Compteur litres** : `input_number.watering_cumulative_liters`
- **Statut actuel** : `input_text.watering_current_status`
- **Historique** : `input_text.watering_history`

### Paramètres recommandés (Belgique)
- **Heure déclenchement** : `06:00:00`
- **Température normale** : `22°C`
- **Seuil canicule** : `30°C`
- **Pluie annulation** : `5mm`
- **Pluie réduction** : `2mm`
- **Durée base** : `60 min`
- **Durée max** : `120 min`
- **Durée min** : `15 min`

## 📊 Dashboard

Copiez le contenu de `watering_dashboard_example.yaml` dans votre dashboard Lovelace.

### Exemple d'affichage

```
┌─────────────────────────────────────┐
│ 🌱 Smart Watering Status            │
├─────────────────────────────────────┤
│ Current Status: ✅ Completed:       │
│ 42.5L in 60min (22/05 06:58)       │
│                                     │
│ Consecutive Dry Days: 3             │
│ Last Session: 42.5L (60min)         │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│ 📊 Water Usage per Session          │
├─────────────────────────────────────┤
│ - 22/05: 42.5L - 60min, 24°C       │
│ - 21/05: 38.2L - 55min, 22°C       │
│ - 20/05: 45.1L - 65min, 26°C       │
│ - 19/05: 31.4L - 45min, 19°C       │
│ - 18/05: 48.9L - 70min, 28°C       │
│                                     │
│ --- Total Cumulative: 205.9 L       │
└─────────────────────────────────────┘
```

## 🧮 Logique de Calcul

```
Durée finale = Durée base × coef_temp × coef_humidité × coef_vent × coef_jours_secs × coef_pluie × coef_prévision
```

### Coefficients

| Facteur | Logique |
|---------|---------|
| 🌡️ **Température** | +2% par °C au-dessus de 22°C, -2% en dessous |
| 💧 **Humidité** | Référence 50%. +1% par point sous 50% |
| 🌬️ **Vent** | +5% par tranche de 10 km/h |
| ☀️ **Jours secs** | +8% par jour sans pluie (≥2mm) |
| 🌧️ **Pluie jour** | Annulation si ≥5mm, réduction proportionnelle si ≥2mm |
| 🔮 **Prévision** | -50% si pluie probable >70%, -25% si >40% |

### Mise à jour du compteur jours secs
- Si pluie ≥ 2mm → reset à 0
- Sinon → incrémente de 1

## 🔔 Notifications

Activables ou désactivables. Format :

**Début :**
```
🌱 Smart watering started
Duration: 60 min (base 60 min × coef 1.0)
📊 Details:
🌡️ Temp: 24°C (coef 1.04)
💧 Humidity: 60% (coef 0.90)
🌬️ Wind: 15 km/h (coef 1.08)
☀️ Dry days: 3 (coef 1.24)
🌧️ Rain: 0mm (coef 1.00)
🔮 Forecast: 0% (coef 1.00)
```

**Fin :**
```
✅ Watering complete
Watering finished after 60 minutes.
💦 Water used: 42.5 L (measured by SWV)
```

**Annulation :**
```
🌧️ Watering cancelled
Watering cancelled: Sufficient rain today (6mm).
Temperature: 18°C | Humidity: 85%
```

## 🛠️ Dépannage

### Le statut ne s'affiche pas
- Vérifiez que les `input_text` sont créés
- Vérifiez les IDs dans la config du blueprint

### Les litres sont à 0
- Attendez 10 sec après fermeture vanne (délai intégré)
- Vérifiez que le capteur `water_consumed` fonctionne dans les états

### L'historique est tronqué
- Limite 255 caractères pour `input_text`
- Format compact : `22/05: 60min, 42.5L, 24°C`
- Seulement 5 dernières sessions

## 📈 Évolutions futures

- [ ] Support multi-zones (plusieurs vannes)
- [ ] Prévisions météo avancées (radar pluie)
- [ ] Graphiques consommation par mois
- [ ] Export CSV historique détaillé
- [ ] Intégration capteur humidité sol

## 🆘 Support

Problème ? Ouvrez une issue sur GitHub avec :
- Version Home Assistant
- Logs de l'automatisation
- Capture d'écran de la config






