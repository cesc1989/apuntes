# Comanditos Edge

Etiquetas: #comanditos

## BD en Omega

Edge:

```bash
luna db get-pass -e omega -d backend
```

Therapist

```bash
luna db get-pass -e omega -d therapist-signup
```

Marketplace

```bash
luna db get-pass -e omega -d marketplace
```

## Queries

### Pacientes con Care Plan con Payer Authorizations

```sql
select
  p.id patient_id,
  e.id care_plan_id,
  acc.hubspot_id,
  count(pa.id) payer_auths_qty
from patients p
join accounts acc on acc.id = p.account_id
join episodes e on e.patient_id = p.id
join payer_authorizations pa on pa.episode_id = e.id
where e.status in (0, 1, 2)
and acc.hubspot_id is not null
group by p.id, e.id, acc.hubspot_id
having count(pa.id) > 1 
order by payer_auths_qty desc
;
```

### Pacientes que no tienen Care Plans

Para poder probar la creación de uno desde cero.

```ruby
Patient.where.not(
  Episode
  .unscope(where: :status)
  .where("episodes.patient_id = patients.id")
  .select(1).arel.exists
)
```

O estas alternativas.
```ruby
Patient
.left_joins(:episodes)
.where(episodes: { id: nil })
```

```ruby
Patient.where.not(id: Episode.select(:patient_id).distinct)
```