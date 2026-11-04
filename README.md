# Battery Monitor

Integrazione HACS per Home Assistant: rileva da sola le batterie, le elenca con lo stato di carica, notifica quando scendono sotto soglia (% e/o mV) e genera eventi utilizzabili nelle automazioni.

Richiede Home Assistant 2024.12 o successivo.

## Installazione
1. HACS → menu ⋮ → **Repository personalizzati**.
2. Repository: `https://github.com/gerrypc72/Gbattery-monitor`, categoria **Integrazione** → Aggiungi.
3. Cerca **Battery Monitor** in HACS → Scarica.
4. Riavvia Home Assistant.
5. Impostazioni → Dispositivi e servizi → Aggiungi integrazione → **Battery Monitor**.

## Cosa rileva
- `sensor` con `device_class: battery` (%)
- `binary_sensor` con `device_class: battery` (`on` = scarica)
- `sensor` con `device_class: voltage` il cui entity_id contiene `batter` o `vbat` (V o mV, convertiti in mV)

Le entità dello stesso dispositivo vengono unite in una sola batteria. Le nuove entità sono rilevate automaticamente.

## Opzioni (Configura)
| Opzione | Default | Note |
|---|---|---|
| Soglia % | 20 | scatta se il livello è **sotto** la soglia |
| Soglia mV | 0 | 0 = disattivata |
| Ripeti notifica ogni (ore) | 24 | 0 = mai |
| Notifica persistente | sì | |
| Servizi di notifica | nessuno | es. `notify.mobile_app_telefono` |
| Entità escluse | nessuna | |

## Sensori
- `sensor.battery_monitor_batteries`: numero di batterie, attributo `batteries` con l'elenco ordinato per carica (`name`, `entity_id`, `level`, `voltage_mv`, `low`, `reasons`)
- `sensor.battery_monitor_low_batteries`: come sopra, solo le scariche

## Eventi
- `battery_monitor_low`: al passaggio sotto soglia e a ogni ripetizione (`repeat: true`)
- `battery_monitor_recovered`: quando la batteria torna sopra soglia

Dati: `name`, `entity_id`, `device_id`, `level`, `voltage_mv`, `reasons` (`percent` / `voltage` / `binary`), `threshold_pct`, `threshold_mv`, `repeat`.

```yaml
automation:
  - alias: Batteria scarica
    trigger:
      - platform: event
        event_type: battery_monitor_low
    action:
      - service: notify.mobile_app_telefono
        data:
          message: "{{ trigger.event.data.name }}: {{ trigger.event.data.level }}%"
```

## Servizio
`battery_monitor.rescan`: rileva di nuovo le entità batteria.

## Limiti
- Nessuna isteresi: una batteria che oscilla intorno alla soglia può generare più eventi.
- Le tensioni sono riconosciute solo dal nome dell'entity_id.
