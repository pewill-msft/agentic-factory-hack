# Contoso Tires — Machine Maintenance Manual

**Document ID**: CT-MNT-2025-001
**Revision**: 3.2
**Effective Date**: January 15, 2025
**Department**: Manufacturing Engineering

---

## 1. Preventive Maintenance Schedules

### 1.1 Tire Building Machine (TB-200 / TB-300)

| Interval | Task | Estimated Time | Parts Required |
|----------|------|---------------|----------------|
| Daily | Inspect drum alignment and tension rollers | 15 min | None |
| Weekly | Lubricate drum bearings and guide rails | 30 min | Bearing grease (CT-LUB-200) |
| Monthly | Calibrate ply tension load cells | 1 hour | Calibration weights |
| Quarterly | Replace drum shaft bearings | 4 hours | Drum bearing set (TBM-BRG-5200) |
| Annually | Full servo motor inspection and PID retune | 8 hours | Servo coupling (TBM-COUP-300) |

### 1.2 Tire Curing Press (TC-100 / TC-200)

| Interval | Task | Estimated Time | Parts Required |
|----------|------|---------------|----------------|
| Daily | Verify temperature sensor readings against reference | 10 min | None |
| Weekly | Inspect hydraulic seals and pressure lines | 45 min | None |
| Monthly | Clean and calibrate thermocouple array | 2 hours | Thermocouple (TCP-TC-K400) |
| Quarterly | Replace hydraulic pump seals | 3 hours | Seal kit (TCP-HYD-SEAL) |
| Semi-annually | Full heating element inspection | 6 hours | Heating element (TCP-HTR-4KW) |

### 1.3 Banbury Mixer (BM-500)

| Interval | Task | Estimated Time | Parts Required |
|----------|------|---------------|----------------|
| Daily | Check rotor clearance and temperature readings | 15 min | None |
| Weekly | Inspect mixing chamber door seals | 30 min | None |
| Monthly | Lubricate rotor bearings and gearbox | 1.5 hours | Gearbox oil (BM-GBX-OIL) |
| Quarterly | Inspect rotor blades for wear | 4 hours | Rotor blade set (BNM-ROT-BLADE) |
| Annually | Full gearbox overhaul | 12 hours | Gearbox rebuild kit (BNM-GBX-KIT) |

---

## 2. Safety Checklists

### 2.1 Pre-Service Checklist (All Machines)

Before performing any maintenance task, complete all items:

- [ ] Machine is powered off and locked out (LOTO procedure CT-SAFE-001)
- [ ] Hydraulic pressure is fully released and verified at 0 bar
- [ ] Temperature has dropped below 40°C (use infrared thermometer)
- [ ] All moving parts are stationary and secured
- [ ] Personal protective equipment (PPE) is worn: safety glasses, heat-resistant gloves, steel-toe boots
- [ ] Maintenance work order has been created and approved
- [ ] Notify shift supervisor and adjacent workstation operators
- [ ] Verify fire extinguisher is accessible within 5 meters

### 2.2 Post-Service Checklist (All Machines)

After completing maintenance, verify:

- [ ] All tools and parts accounted for (no foreign objects left in machine)
- [ ] Guards and safety interlocks reinstalled and tested
- [ ] Lockout/tagout devices removed by authorized personnel only
- [ ] Machine powered on and run through a 5-minute idle cycle
- [ ] Sensor readings confirmed within normal operating thresholds
- [ ] Maintenance log updated with work performed, parts used, and technician name
- [ ] Work order closed in the maintenance management system

---

## 3. Part Replacement Procedures

### 3.1 Replacing Drum Shaft Bearings (TB-200 / TB-300)

**Required parts**: Drum bearing set (TBM-BRG-5200)
**Required tools**: Bearing puller, torque wrench (45–60 Nm), dial indicator
**Estimated time**: 4 hours
**Skill required**: Mechanical maintenance Level 2 or higher

**Procedure**:

1. Complete the Pre-Service Checklist (Section 2.1).
2. Remove the drum access panel (4x M10 bolts).
3. Disconnect the servo motor coupling from the drum shaft.
4. Use the bearing puller to remove the old bearings from both ends of the drum shaft.
5. Clean the shaft journals with solvent and inspect for scoring or wear.
6. Press new bearings onto the shaft — ensure they seat fully against the shoulder.
7. Torque the bearing retaining nuts to 55 Nm (±5 Nm).
8. Reconnect the servo motor coupling and check alignment with a dial indicator (max 0.05 mm runout).
9. Reinstall the drum access panel.
10. Run the drum at low speed for 10 minutes and verify vibration is below 3.0 mm/s.
11. Complete the Post-Service Checklist (Section 2.2).

### 3.2 Replacing Heating Elements (TC-100 / TC-200)

**Required parts**: Heating element (TCP-HTR-4KW)
**Required tools**: Multimeter, thermal paste, torque wrench (25–35 Nm)
**Estimated time**: 3 hours
**Skill required**: Electrical maintenance Level 2 + thermal systems certification

**Procedure**:

1. Complete the Pre-Service Checklist (Section 2.1).
2. Verify mold temperature is below 40°C using an infrared thermometer.
3. Disconnect power leads from the heating element terminal block.
4. Remove the heating element retaining bolts (6x M8).
5. Slide the old heating element out of the platen cavity.
6. Clean the cavity and apply a thin layer of thermal paste to the new element.
7. Insert the new heating element and torque retaining bolts to 30 Nm (±5 Nm).
8. Reconnect power leads — verify polarity and tighten terminal screws.
9. Use a multimeter to confirm element resistance is within specification (18–22 Ω).
10. Power on the press and verify the heating zone reaches target temperature (170°C) within 25 minutes.
11. Complete the Post-Service Checklist (Section 2.2).

### 3.3 Replacing Rotor Blades (BM-500)

**Required parts**: Rotor blade set (BNM-ROT-BLADE)
**Required tools**: Hydraulic torque wrench (200–250 Nm), feeler gauge set
**Estimated time**: 6 hours
**Skill required**: Mechanical maintenance Level 3 + mixer safety certification

**Procedure**:

1. Complete the Pre-Service Checklist (Section 2.1).
2. Drain residual compound from the mixing chamber.
3. Open the mixing chamber door and secure it in the fully open position.
4. Remove the rotor blade retaining bolts using the hydraulic torque wrench.
5. Extract the worn rotor blades using the blade removal jig (tool CT-JIG-BM01).
6. Inspect the rotor hub for wear or damage. Measure clearance with feeler gauge — minimum 1.5 mm.
7. Install new rotor blades and torque retaining bolts to 230 Nm (±20 Nm).
8. Verify blade-to-chamber clearance is within specification (1.5–2.5 mm) at all points.
9. Close the mixing chamber door and verify door seal integrity.
10. Run a test batch at reduced speed and verify mixing temperature stays within specification.
11. Complete the Post-Service Checklist (Section 2.2).

---

## 4. Emergency Contacts

| Role | Name | Extension |
|------|------|-----------|
| Shift Supervisor | On-duty roster | Ext. 2100 |
| Safety Officer | Maria Chen | Ext. 2200 |
| Electrical Lead | James Park | Ext. 2301 |
| Mechanical Lead | Sarah Williams | Ext. 2302 |
| Parts Warehouse | Main desk | Ext. 2400 |

---

*This document is controlled. Do not distribute printed copies. Always refer to the current version in the maintenance management system.*
