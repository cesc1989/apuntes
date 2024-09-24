# 🐞 Duplicated patients because of multiple escalations

In the patients treated table, for Dr. Saly, we can see **multiple occurrences** of same patient but with different escalation reasons.

![[01.duplicated.escalations.png]]

However, when inspecting the reasons we can evidence they are duplicates:

    Home is unsafe for post-op recovery
    Home is unsafe for post-op recovery
    Needs DME (shower chair, walker, etc.)
    Needs DME (shower chair, walker, etc.)
    Safety risk post-op due to stairs
    Safety risk post-op due to stairs
    Sudden flare-up due to traumatic incident
    Sudden flare-up due to traumatic incident

# Testing in Alpha

In alpha, for Dr. Saly as well, we can observe same behavior for patient Demarco O’Hara.

![[02.duplicated.example.png]]

This time, though, it’s worst. There’s 76 occurrences of the patient but we can see in Luxe that it should only be 14 times (still not ideal but better)

![[03.check.in.luxe.png]]

# Underlying data in Alpha queries

This query produces the ideal 14 records for Demarco O’Hara:
```sql
select count(pe.id), pe.id as escalation_id,
	ep.id as episode_id
from protocol_escalations pe
join protocol_survey_submissions pss on pe.submission_id = pss.id
join appointments app on pss.submitted_for_appointment_id = app.id
join episodes ep on app.episode_id = ep.id
where ep.id = '4ff6be2b-9ad0-4607-b53a-8ae696180b67'
group by pe.id, ep.id
order by pe.id asc;
```

Results:
escalation_id | episode_id

    0923436e-badb-4262-959a-b61fd3c4a7bc        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    1140f9c8-00df-4c3e-9f5e-8b4e91b70408        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    29b617a6-a887-4095-9fdf-94f96641340e        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    3e4d9c63-accd-49cc-8998-32f2fa305103        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    41d50594-32c7-4e0d-b505-701947ec9ed6        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    478a34f9-ee82-4375-8ba4-bf171b982ff0        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    4f41be0e-76a7-4c50-a205-05aabcdcf499        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    7370dafa-0118-43ec-8c14-470d3a9f48e7        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    9a219898-2fa5-4602-be64-72690b9fbab4        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    a28ccbac-6905-4019-bbc0-de907dbcf326        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    b938511a-69de-48b4-bfbe-3f257d76b5f6        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    bad13d88-c7b4-4239-a198-55672edf6d07        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    d25a9a7d-5166-4eb5-9237-60465d2285d8        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    f730ec7c-a11b-4634-a513-471e3cf680e7        4ff6be2b-9ad0-4607-b53a-8ae696180b67

## Where does the duplication start?

In this point when joining `protocol_escalations` to `protocol_surveys`:
```sql
  select pe.id as escalation_id,
      ep.id as episode_id
    from protocol_escalations pe
    join protocol_survey_submissions pss on pe.submission_id = pss.id
    join appointments app on pss.submitted_for_appointment_id = app.id
    join episodes ep on app.episode_id = ep.id
    join protocol_surveys ps on ps.episode_id = ep.id
    where ep.id = '4ff6be2b-9ad0-4607-b53a-8ae696180b67'
    order by pe.id asc;
```

Example.
escalation_id | episode_id

    0923436e-badb-4262-959a-b61fd3c4a7bc        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    0923436e-badb-4262-959a-b61fd3c4a7bc        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    0923436e-badb-4262-959a-b61fd3c4a7bc        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    1140f9c8-00df-4c3e-9f5e-8b4e91b70408        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    1140f9c8-00df-4c3e-9f5e-8b4e91b70408        4ff6be2b-9ad0-4607-b53a-8ae696180b67
    1140f9c8-00df-4c3e-9f5e-8b4e91b70408        4ff6be2b-9ad0-4607-b53a-8ae696180b67

## What’s the deal with protocol_surveys table?

When a therapist escalates a patient, they answer a survey with 5 yes/no questions. If they answer “yes” for any of them, a new record is created for the same escalation.

This can be evidenced in this query results in alpha. Notice how “text” column varies and causes the duplication.

![[04.query.results.png]]

To help us fix this, the chosen approach was to get escalation information only for the most recent and complete appointment of the patient.

Let’s see how to make this work.

# First Attempt to fix the query ❌

This query produces the ideal 14 results:
```sql
select
	distinct pe.id,
	ep.id as episode_id
from protocol_escalations pe
join protocol_survey_submissions pss on pe.submission_id = pss.id
join appointments app on pss.submitted_for_appointment_id = app.id
join episodes ep on app.episode_id = ep.id
left join protocol_surveys ps on ps.episode_id = ep.id
join protocol_phases pp on ps.phase_id = pp.id
join protocol_escalation_questions peq on peq.phase_id = pp.id
join clinical_escalation_questions ceq on peq.question_id = ceq.id
where ep.id = '4ff6be2b-9ad0-4607-b53a-8ae696180b67'
order by pe.id asc;
```

Once I add `ceq.text` the duplication happens again.

**Finally, this way I could get everything with way much less duplicates:**
```sql
select
	distinct pe.id,
	ep.id as episode_id,
	max(ceq.text)
from protocol_escalations pe
join protocol_survey_submissions pss on pe.submission_id = pss.id
join appointments app on pss.submitted_for_appointment_id = app.id
join episodes ep on app.episode_id = ep.id
left join protocol_surveys ps on ps.episode_id = ep.id
join protocol_phases pp on ps.phase_id = pp.id
join protocol_escalation_questions peq on peq.phase_id = pp.id
join clinical_escalation_questions ceq on peq.question_id = ceq.id
where ep.id = '4ff6be2b-9ad0-4607-b53a-8ae696180b67'
group by pe.id, ep.id
;
```

Why distinct and left join?
```sql
distinct pe.id
(...)
left join protocol_surveys ps on ps.episode_id = ep.id
```

> Explanation by ChatGPT
> 
> By using a `LEFT JOIN` with "protocol_surveys", you'll include all records from the left table (protocol_escalations) regardless of whether there is a match in the right table (protocol_surveys). Then, by applying `DISTINCT`, you'll ensure that only unique combinations of "escalation_id" and "episode_id" are returned.

Why this returns duplicates?
```sql
max(ceq.text)
```

Adding `ceq.text` to the query will produce different results per row. To prevent that the `max()` returns one of the values. Same could happen with `min()`.

## Works but not enough yet

However this was not enough. We would still get some duplicates and only one the escalation reasons.

![[05.first.fix.png]]

We want this to show only one patient and an aggregate of escalation reasons when hovering over the red indicator.

# Second Attempt to fix the query ❌

This query produces the expected and valid 15 results (15 different escalation_ids), the one for the most recent and completed appointments first, and the escalation reasons aggregated.

```sql
select
	distinct(pe.id) as "escalation_id",
	ep.id as "episode_id",
	pe.created_at,
	string_agg(ceq.text, ', ') as "text",
	app.scheduled_date
from protocol_escalations pe
join protocol_survey_submissions pss on pe.submission_id = pss.id
join appointments app ON pss.submitted_for_appointment_id = app.id
join episodes ep on app.episode_id = ep.id
left join protocol_surveys ps on ps.episode_id = ep.id
join protocol_phases pp on ps.phase_id = pp.id
join protocol_escalation_questions peq on peq.phase_id = pp.id
join clinical_escalation_questions ceq on peq.question_id = ceq.id
where ep.id = '4ff6be2b-9ad0-4607-b53a-8ae696180b67'
group by pe.id, ep.id, pe.created_at, app.scheduled_date
order by app.scheduled_date DESC
;
```

We can see the results and how the reasons are aggregated in the “text” column.

![[06.second.fix.query.results.png]]

But this query cannot be translated as is to the summary writer worker. The final result, using the important parts of the query looks like this:
```sql
real_protocol_escalation
AS (
	SELECT DISTINCT pe.id AS "escalation_id"
		,ep.id AS "episode_id"
		,pe.created_at
		,string_agg(ceq.text, ', ') as "text"
		,app.scheduled_date
	FROM real_episode ep
	JOIN real_appointments app ON app.episode_id = ep.id
	JOIN protocol_survey_submissions pss ON pss.submitted_for_appointment_id = app.id
	JOIN protocol_escalations pe ON pe.submission_id = pss.id
	LEFT JOIN protocol_surveys ps ON pss.survey_id = ps.id
	JOIN protocol_phases pp ON ps.phase_id = pp.id
	JOIN protocol_escalation_questions peq ON peq.phase_id = pp.id
	JOIN clinical_escalation_questions ceq ON peq.question_id = ceq.id
	WHERE app.state = 2
	GROUP BY pe.id, ep.id, app.scheduled_date
	ORDER BY app.scheduled_date DESC
	)
,care_pathway_escalation
AS (
	SELECT rep.patient_id
	,(
		CASE
			WHEN EXISTS (SELECT 1 FROM real_protocol_escalation rpe WHERE rpe.episode_id = rep.id)
				THEN true
			ELSE false
			END
		) AS escalated
	,rpe.created_at as "escalated_at"
	,rpe.text as "escalated_reason"
	,rpe.escalation_id
	,rpe.scheduled_date
	FROM real_episode rep
	JOIN real_protocol_escalation rpe ON rpe.episode_id = rep.id
	ORDER BY rpe.scheduled_date DESC
	LIMIT 1
)
```

We still select distinct escalation_id, the reasons are aggregated into a single column, left join to protocol_surveys table, only querying completed appointments (state column = 2), ordering by latest completed appointment.

The `real_protocol_escalation` table would return the 15 results for “Demarco O’Hara”. In order to only return one in the final result, we indicate that in the next table: `care_pathway_escalation`.

In the end of the query, we order the records again by most recent scheduled appointment and limit the results to just one record.

And this works in alpha for Dr. Saly’s dashboard

![[07.second.fix.alpha.png]]

The aggregates look like this

![[08.second.fix.tooltip.png]]

# Update March 15th ✅ 

Turns out, the query in the 2nd attempt did not work. I was seeing Demarco escalation because it was the most recent one in general.

The “LIMIT 1” in the “care_pathway_escalation” CTE didn’t work. It was only returning one record instead of returning “one record for each found patient”.

I fixed it in the 3rd attempt.

# Third Attempt to fix the query ✅

I couldn’t use the LIMIT clause because it would only return one record in total. I needed to make the second CTE return only one record per patient escalated. Let’s see some query results to understand this.

## real_protocol_escalation CTE

The query:
```sql
with real_protocol_escalation
AS (
	SELECT DISTINCT pe.id AS "escalation_id"
		,ep.id AS "episode_id"
		,pe.created_at
		,string_agg(ceq.text, ', ') as "text"
		,app.scheduled_date
		,app.state
		,ROW_NUMBER() OVER (PARTITION BY ep.patient_id ORDER BY app.scheduled_date DESC) AS row_num
	FROM episodes ep
	JOIN appointments app ON app.episode_id = ep.id
	JOIN protocol_survey_submissions pss ON pss.submitted_for_appointment_id = app.id
	JOIN protocol_escalations pe ON pe.submission_id = pss.id
	left JOIN protocol_surveys ps ON pss.survey_id = ps.id
	JOIN protocol_phases pp ON ps.phase_id = pp.id
	JOIN protocol_escalation_questions peq ON peq.phase_id = pp.id
	JOIN clinical_escalation_questions ceq ON peq.question_id = ceq.id
	WHERE app.state = 2
	GROUP BY pe.id, ep.id, app.scheduled_date, app.state
	ORDER BY app.scheduled_date desc
 )
```

returns this:

![[09.third.attempt.query.results.png]]

What to look for? Look at the circled in red groups:

1. Notice how the episode_id is the same for the first 5 rows.
2. Each row is number 1 to 5 in the column “row_num”.
3. In the second circle something similar happens but a different episode_id is mixed
4. The same kind of episode_id multiple times, number 1 to X happens multiple times

The key for the fix is the ROW_NUMBER function.

```sql
,ROW_NUMBER() OVER (PARTITION BY ep.patient_id ORDER BY app.scheduled_date DESC) AS row_num
```

ChatGPT said:

> to assign a unique row number to each row partitioned by `**patient_id**` and ordered by `**scheduled_date**` in descending order.

This attribute is going to be important in the next CTE query.

## care_pathway_escalation CTE

This is the query:
```sql
SELECT rep.patient_id
	,(
		CASE
			WHEN EXISTS (SELECT 1 FROM real_protocol_escalation rpe WHERE rpe.episode_id = rep.id)
				THEN true
			ELSE false
			END
		) AS escalated
	,rpe.created_at as "escalated_at"
	,rpe.text as "escalated_reason"
	,rpe.escalation_id
	,rpe.scheduled_date
	,rpe.state
	,rpe.row_num
	FROM episodes rep
	JOIN real_protocol_escalation rpe ON rpe.episode_id = rep.id
	where rpe.row_num = 1
```

Just as in 2nd attempt, we are use the “real_protocol_escalation” CTE as a base to fetch the attributes we are interested in. The big difference is that we only select row_num 1.

**What does this mean?**
In the screenshot above, we saw the same episode_id with row_num from 1 to 5.

When choosing only:
```sql
where rpe.row_num = 1
```

We’re telling the query to only pick one (or the first) element of those “same episode_id” groups. And this let us achieve what we really want:

> Read patient escalation info from the most recent completed appointment.

This is the way to bring patient escalation without duplicating the final output and it reflects like so in Athena

![[10.third.attempt.fix.inspect.png]]

What to look for?

- Patient rows with a light-orange background color in previous queries would appear multiple times.
- We can see the were escalated, there’s an escalation date, reasons, and an escalation_id
- Other patients do not appear duplicated

And in Clinical Dashboard in Alpha for Dr. Saly

![[11.third.attempt.in.alpha.png]]

# Update March 13th ❌ 

What follows next is cause by a different subquery in the PatientSummaryWriter.

See [[[Report]_Fix_for_duplication_of_patients_because]]

## It worked~~Still not enough yet again~~

In Omega, it produces the contrary effect. We now get duplicates but without escalation.

I was able to detect this new bad behavior when selecting Luna Care San Francisco clinic

![[12.alpha.san.francisco.png]]
