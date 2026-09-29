---
ontology: http://www.example.com/ec-chamber/bundle
---

# EC Chamber Analysis

Each section is a method-owned analysis template run over the whole EC Chamber bundle. This page supplies only the inputs. Sensor failure is shown here for the temperature sensor; [Sensor Failures](./Sensor%20Failures.md) covers all three environmental sensors. Findings are summarized in `ANALYSIS.md` at the repository root.

```compose
template: http://www.example.com/method/analysis/power-budget
```

```compose
template: http://www.example.com/method/analysis/mass-rollup
```

```compose
template: http://www.example.com/method/analysis/threshold-impact
condition: http://www.example.com/ec-chamber/system-analysis/environment#ChamberTemperature
increase: 15
```

```compose
template: http://www.example.com/method/analysis/sensor-failure
sensor: http://www.example.com/ec-chamber/system-analysis/components#TemperatureSensor
```

```compose
template: http://www.example.com/method/analysis/verification-coverage
```
