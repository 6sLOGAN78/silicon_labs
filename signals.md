# VitalDB Signal Channel Reference

This document describes the physiological signals used in the dataset and groups them by source device.

---

## BIS Signals (Brain Activity / Depth of Anesthesia)

| Signal       | Description                                              |
| ------------ | -------------------------------------------------------- |
| BIS/BIS      | Bispectral Index score used to estimate anesthesia depth |
| BIS/EEG1_WAV | EEG waveform channel 1                                   |
| BIS/EEG2_WAV | EEG waveform channel 2                                   |
| BIS/EMG      | Muscle activity (EMG)                                    |
| BIS/SEF      | Spectral edge frequency                                  |
| BIS/SQI      | Signal quality index                                     |
| BIS/SR       | Suppression ratio                                        |
| BIS/TOTPOW   | Total EEG power                                          |

---

## Primus Signals (Anesthesia Machine / Ventilator)

### Gas Measurements

| Signal       | Description                     |
| ------------ | ------------------------------- |
| Primus/CO2   | Carbon dioxide concentration    |
| Primus/ETCO2 | End tidal CO₂                   |
| Primus/FIO2  | Inspired oxygen fraction        |
| Primus/FEO2  | Expired oxygen fraction         |
| Primus/FIN2O | Inspired nitrous oxide fraction |
| Primus/FEN2O | Expired nitrous oxide fraction  |
| Primus/INCO2 | Inspired CO₂                    |

### Anesthetic Agents

| Signal           | Description                        |
| ---------------- | ---------------------------------- |
| Primus/INSP_DES  | Inspired desflurane concentration  |
| Primus/EXP_DES   | Expired desflurane concentration   |
| Primus/INSP_SEVO | Inspired sevoflurane concentration |
| Primus/EXP_SEVO  | Expired sevoflurane concentration  |
| Primus/MAC       | Minimum alveolar concentration     |

### Ventilator Variables

| Signal            | Description                      |
| ----------------- | -------------------------------- |
| Primus/AWP        | Airway pressure                  |
| Primus/COMPLIANCE | Lung compliance                  |
| Primus/MV         | Minute ventilation               |
| Primus/TV         | Tidal volume                     |
| Primus/PIP_MBAR   | Peak inspiratory pressure        |
| Primus/PEEP_MBAR  | Positive end expiratory pressure |
| Primus/PPLAT_MBAR | Plateau pressure                 |
| Primus/MAWP_MBAR  | Mean airway pressure             |
| Primus/VENT_LEAK  | Ventilator leak percentage       |
| Primus/RR_CO2     | Respiratory rate from CO₂        |

### Ventilator Settings

| Signal                | Description                  |
| --------------------- | ---------------------------- |
| Primus/SET_FIO2       | Set oxygen fraction          |
| Primus/SET_FRESH_FLOW | Fresh gas flow setting       |
| Primus/SET_INSP_TM    | Inspiratory time             |
| Primus/SET_PIP        | Inspiratory pressure setting |
| Primus/SET_RR_IPPV    | Respiratory rate setting     |
| Primus/SET_TV_L       | Target tidal volume          |
| Primus/SET_INTER_PEEP | PEEP setting                 |
| Primus/SET_INSP_PAUSE | Inspiratory pause setting    |
| Primus/SET_AGE        | Configured patient age       |

---

## SNUADC Signals (Raw Waveforms)

| Signal        | Description                      |
| ------------- | -------------------------------- |
| SNUADC/ART    | Arterial blood pressure waveform |
| SNUADC/ECG_II | ECG Lead II waveform             |
| SNUADC/ECG_V5 | ECG Lead V5 waveform             |
| SNUADC/PLETH  | Plethysmography waveform         |

---

## Solar8000 Patient Monitor Signals

### Blood Pressure

| Signal             | Description                       |
| ------------------ | --------------------------------- |
| Solar8000/ART_SBP  | Arterial systolic blood pressure  |
| Solar8000/ART_DBP  | Arterial diastolic blood pressure |
| Solar8000/ART_MBP  | Mean arterial pressure (MAP)      |
| Solar8000/NIBP_SBP | Non-invasive systolic BP          |
| Solar8000/NIBP_DBP | Non-invasive diastolic BP         |
| Solar8000/NIBP_MBP | Non-invasive mean BP              |

### Heart / Oxygen / Respiration

| Signal               | Description              |
| -------------------- | ------------------------ |
| Solar8000/HR         | Heart rate               |
| Solar8000/PLETH_HR   | Pulse-derived heart rate |
| Solar8000/PLETH_SPO2 | Oxygen saturation (SpO₂) |
| Solar8000/ETCO2      | End tidal CO₂            |
| Solar8000/RR_CO2     | Respiratory rate         |
| Solar8000/BT         | Body temperature         |

### ECG Features

| Signal           | Description         |
| ---------------- | ------------------- |
| Solar8000/ST_I   | ST segment lead I   |
| Solar8000/ST_II  | ST segment lead II  |
| Solar8000/ST_III | ST segment lead III |
| Solar8000/ST_AVF | ST segment lead aVF |
| Solar8000/ST_AVL | ST segment lead aVL |
| Solar8000/ST_AVR | ST segment lead aVR |

### Ventilator Variables

| Signal                 | Description                 |
| ---------------------- | --------------------------- |
| Solar8000/VENT_MV      | Minute ventilation          |
| Solar8000/VENT_PIP     | Peak inspiratory pressure   |
| Solar8000/VENT_PPLAT   | Plateau pressure            |
| Solar8000/VENT_MAWP    | Mean airway pressure        |
| Solar8000/VENT_RR      | Ventilator respiratory rate |
| Solar8000/VENT_TV      | Measured tidal volume       |
| Solar8000/VENT_SET_TV  | Set tidal volume            |
| Solar8000/VENT_SET_PCP | Pressure control setting    |
| Solar8000/VENT_INSP_TM | Inspiratory time            |

---

## Recommended Signals for Hypotension Prediction

**Waveforms:**

* SNUADC/ART
* SNUADC/ECG_II
* SNUADC/PLETH

**Vitals:**

* Solar8000/HR
* Solar8000/PLETH_SPO2
* Solar8000/ETCO2
* Solar8000/RR_CO2

**Prediction Target:**

* Solar8000/ART_MBP (MAP)
