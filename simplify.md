# Simplification.
The setpoint **is** the control signal — you don't need explicit on/off. Here's why:

- Setpoint 72°C → buffer at 40°C → HP runs hard
- Setpoint 45°C → buffer at 40°C → HP runs gently
- Setpoint 20°C → buffer already above setpoint → HP does nothing

The HP's internal controller handles everything else: compressor speed, defrost, pump, antifreeze protection. You just tell it the target temperature.

---

## Simplified architecture

```
EXTERNAL SYSTEM
  PV inverter API  ──┐
  Buffer sensor    ──┤→ Logic → Write reg 2 (setpoint) → HP
  Weather API      ──┘

HP INTERNAL
  Runs/stops itself based on water temp vs setpoint
  Handles defrost, antifreeze, pump, compressor modulation
```

---

## The only Modbus command you need

| Action | Register | Value | When |
|---|---|---|---|
| Strong solar surplus | 2 | 72 | PV > 3.5 kW, buffer < 68°C |
| Moderate solar | 2 | 65 | PV > 2.0 kW, buffer < 68°C |
| Night top-up | 2 | 45 | Buffer < 38°C |
| Buffer full / off | 2 | 20 | Buffer top > 68°C |

---

## What this means for the flow

Dramatic simplification. The entire control section becomes:

```
PV reading + buffer temp
        ↓
  Calculate setpoint
        ↓
  Write reg 2 only
```

Optionally read reg 124 (error code) once per minute for fault monitoring. Everything else — sensors, logic, decisions — lives outside the HP entirely in your own system.

Shall I rewrite the flow to this minimal version?
