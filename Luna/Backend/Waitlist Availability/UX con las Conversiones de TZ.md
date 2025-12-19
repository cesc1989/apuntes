# Problema de UX con las zonas horarias en el calendario de Waitlist Availability

Resumen de Claudio:
> This is the fundamental UX problem with timezone conversion for availability calendars.

## The Issue

Original availability (Houston/Central):
- 6:30 AM - 9:30 PM CST

After proper timezone conversion to NY (EST):
- 7:30 AM - 10:30 PM EST ✓ (acceptable)

After proper conversion to LA (PST):
- 5:30 AM - 8:30 PM PST ✓ (acceptable)

After proper conversion to Bogota (COT):
- 6:30 AM - 12:30 AM next day ✗ (looks broken)

### Why the Original Code Used "Wall Clock Preservation"

The old `coerced_to_time_zone` method tried to keep "9:30 PM → 9:30 PM" exactly to avoid this confusing UX. But that created invalid timezone data and caused the midnight-crossing filtering issue.

### The Real Solution ❌

Don't convert timezones at all. Show times in the patient's region timezone with a clear timezone indicator.

Why this makes sense:
- Patient availability is about when the patient is available in their local time
- A care coordinator in NY doesn't need to see "12:30 AM" - they need to know the patient is available until "9:30 PM CST"
- The coordinator can mentally convert or the UI can show "9:30 PM CST (11:30 PM EST)"

This is likely why the calendar was designed this way originally - healthcare availability scheduling needs to stay in the patient's local context.

## Commit original sobre el método `coerced_to_time_zone`

Del commit original:
> tldr; the scheduler component we use doesn't allow us to specify what time zone we're actually doing scheduling in, so we need to hack around and coerce between the user's browser timezone (which is what the scheduler uses), and the patient's time zone (which is what we want to schedule availability blocks in).
>
> Commit: [`#c379396`](https://github.com/lunacare/backend/commit/c3793964d8dec8378259aaf6cfbe70bdb2576750)


 ### Options Moving Forward

1. Keep only the GraphQL rrule fix - Solves original reported bug, document timezone limitation
2. Switch calendar library - Find one that supports timezone specification
3. Restrict usage - Document that calendar works best when user timezone matches patient region
4. Expand calendar hours to 0-24 - Allow midnight crossing to display