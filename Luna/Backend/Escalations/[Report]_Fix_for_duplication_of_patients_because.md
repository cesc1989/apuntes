# ✅ [Report] Fix for duplication of patients because of Treatment Completed Reason

*Last updated on: March 13th.*

----------

## About

The subquery to get a care plan’s treatment completed reason available in the CSV Download in a Clinical Dashboard caused an error. The error is results from the query would list a patient multiple times. This happened because the subquery was fetching all patient’s care plan’s reasons.

If a patient had three care plans with three different reasons, all three reasons would be listed and the patient would appear trice.

To fix this we have to use a patient’s most recent care plan’s treatment completed reason.

# Before the fix

The screenshot below shows duplicates of patient “P01T5” in Luna Care San Francisco’s dashboard in alpha environment.

![[110.dups.alpha.png]]

All three discharged reasons (last column, Goals met), correspond to different care plans.

Another example is patient “Ty opt -1”.

![[112.dups.alpha.png]]

## Total patient count before the fix (for this query only)

The duplicates made the total count of patients to be 59.

# After applying the fix.

The next screenshot shows only one instance of the previous patient.

![[111.nodups.alpha.png]]

Same for second example patient

![[total.query.count.after.fix.png]]

## Total patient count after the fix (for this query only)

The fix makes the total count of patients to be 54.

