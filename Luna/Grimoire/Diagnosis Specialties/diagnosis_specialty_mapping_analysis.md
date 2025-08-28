# Diagnosis Specialty Mapping Analysis

## Diagnosis Codes Matching Specified Specialties

### Orthopedics and Sport Medicine ("Ortho")

- Z96.64
- Z96.65 
- Z96.659
- M62.08 (Primary: Pelvic Floor, Secondary: Ortho)
- Z91.81 (Primary: Ortho, Secondary: Neuro)
- M62.81 (Primary: Ortho, Secondary: Neuro)

### Pelvic Floor

- M62.08 (Primary)
- O26.9
- R10.2

### Vestibular

- R42 (Primary: Neuro, Secondary: Vestibular)
- H81.10 (Primary: Vestibular, Secondary: Neuro)
- H81.90 (Primary: Vestibular, Secondary: Neuro)

### CRPS

- G90.5

### Neurologic ("Neuro")

- G20
- G35
- G80.9
- R42 (Primary)
- I64
- S14
- G12.2
- R26.0
- G81.00

### Geriatric

- No primary assignments (only appears as secondary/fallback specialty for patients 65+)

## Diagnosis Codes NOT Matching Specified Specialties

### M-codes (Musculoskeletal disorders not mapped to specialties)

- M25.50, M25.511, M25.512, M25.521, M25.522, M25.531, M25.532
- M25.551, M25.552, M25.561, M25.562, M25.571, M25.572, M25.579
- M54.12, M54.2, M54.212, M54.4, M54.5, M54.50, M54.6
- M62.50
- M70.6, M70.61
- M75.10, M75.101, M75.4, M75.5, M75.52
- M76.5
- M79.601, M79.602, M79.604, M79.605, M79.609
- M79.621, M79.622, M79.631, M79.632, M79.641, M79.642
- M79.644, M79.645, M79.651, M79.652, M79.661, M79.662
- M79.671, M79.672, M79.674, M79.675

### R-codes (Symptoms/signs not mapped)

- R26 (different from R26.0 which IS mapped)
- R26.2, R26.89

### Generic/System codes

- PENDING
- Unspecified

## Summary

- **Total diagnosis codes in system:** 82
- **Codes mapped to specified specialties:** 20
- **Codes NOT mapped to specified specialties:** 62

## Notes

- Priority levels indicated by ">" symbol (e.g., "Pelvic Floor > Ortho" means Pelvic Floor primary, Orthopedics secondary)
- Geriatric appears only as secondary specialty for patients 65+ years old
- Some codes have gender requirements (e.g., O26.9, R10.2 require same gender, M62.08 strongly prefers same gender)