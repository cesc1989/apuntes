# Logical Replication Alpha Discrepancy

Korey shared this:
```bash
busyapp@busyapp-74c957c5c8-lvkbc:~$ bash lr-compare-physician-portal-to-backend.sh
Exporting variables for connections
Done getting vars

Fetching table names from publication: physician_portal_to_backend

DOES NOT MATCH ... TABLE public.clinical_dashboard_activities -- num source: 2139, num target: 2199

DOES NOT MATCH ... TABLE public.clinical_dashboard_execution_ids -- num source: 248405, num target: 247638

MATCHES ... TABLE public.clinical_dashboard_users -- num source: 15, num target: 15

DOES NOT MATCH ... TABLE public.clinical_dashboards -- num source: 51, num target: 52

DOES NOT MATCH ... TABLE public.clinical_dashboard_links -- num source: 1592, num target: 1600
```

There is a difference in tables:

- clinical_dashboard_activities
- clinical_dashboards
- clinical_dashboard_links

in the target database (`backend`). Why? Because frontend changed the Base URL to point to `backend` service instead of `physician-portal`. Meaning every action taken in Clinical Dashboard during QA was stored in the `backend` tables.

The exception to the prior is table clinical_dashboard_execution_ids. This one has more records on `physician-portal` because the workers were still running in that service instead of in `backend`. That was changed past week.