# Explorando calculo del total expected visits range

Esta función:
```python
# app/marketplace/services/visit_plan.py

def calculate_total_expected_visits_range(
		entries: Iterable[entities.VisitPlanEntry], appointment_completion_dates: Iterable[pendulum.Date]
) -> Optional[entities.OptionalRange]:
```

# Apuntes generales

```python
(Pdb) current_entry
VisitPlanEntry(effective_from=Date(2021, 12, 29), frequency=OptionalRange(lower=3, upper=3), total=OptionalRange(lower=12, upper=12), duration=OptionalRange(lower=4, upper=4))

(Pdb) measure_from_day
Date(2021, 12, 29)

(Pdb) current_and_future_entries
[VisitPlanEntry(effective_from=Date(2021, 12, 29), frequency=OptionalRange(lower=3, upper=3), total=OptionalRange(lower=12, upper=12), duration=OptionalRange(lower=4, upper=4))]

(Pdb) entry
VisitPlanEntry(effective_from=Date(2022, 1, 3), frequency=OptionalRange(lower=4, upper=4), total=OptionalRange(lower=16, upper=16), duration=OptionalRange(lower=4, upper=4))

(Pdb) entry_to_minimized_entry[entry]
VisitPlanEntry(effective_from=Date(2022, 1, 3), frequency=OptionalRange(lower=4, upper=4), total=OptionalRange(lower=16, upper=16), duration=OptionalRange(lower=4, upper=4))

(Pdb) minimized_entry.effective_from
Date(2022, 1, 3)

(Pdb) minimized_entry.effective_from.add(days=int(7 * minimized_entry.expected_duration_lower()) - 1)
Date(2022, 1, 30)

(Pdb) pendulum.Period(minimized_entry.effective_from, effective_from_until)
<Period [2022-01-03 -> 2022-01-30]>
```