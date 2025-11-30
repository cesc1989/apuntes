# Anexo de Caso 020: duplicados en tc_therapists

Estos son los términos para el desempate para hallar los candidatos a ser deduplicados:

## Candidate Selection Logic

### Primary Selection: Completeness Score

Records are scored based on the presence of these fields:

- `af_completed` (attestation form completed date)
- `ca_completed` (credentialing application completed date)
- `credentialing_active_attested_id`
- `cred_info_updated` (credentialing information association)
- `immu_updated` (immunization association)
- `prof_updated` (professional history association)
- `pref_updated` (preferences association)
- `npi_updated` (NPI/CAQH application association)
- `pay_updated` (payout information association)

**The record with the LOWEST completeness score is selected as the candidate for deletion.**

### Tiebreaker: Most Outdated Record

When two or more records have the **same completeness score**, the tiebreaker is the `updated_at` timestamp.

**The record with the OLDEST `updated_at` value (least recently updated) is selected as the candidate.**