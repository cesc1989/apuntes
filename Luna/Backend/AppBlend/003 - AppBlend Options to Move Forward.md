# AppBlend Options to Move Forward

The upgrade has turned very complicated to complete with lots of stuff rising up as we correct errors and test the system. Here I suggest two options to continue this milestone.

**Option A**: Keep testing locally to gain more confidence and find errors earlier. Once local test provides good confident repeat testing in Alpha env. In this option there's no deadline in sight because there's no sure way to tell when it's going to be ready. Instead, provide updates in a small cadence (daily).

**Option B**: Take a step back to apply upgrades to Edge in small pull requests. This is upgrading to latest version of Rails 6.x prior to Rails 7.0.4. This option considers also bringing in changes done in the current upgrade branch and apply them individually. This approach would try to minimize risk which is currently high in the in progress branch.

IMPORTANT: What are the initiatives AppBlend Milestone 2 might be blocking? Those initiative can indicate the best option to follow.

## Option A

No deadline in sight. It's very difficult to accurately say a date and too compromising to provide an estimate.

Considering CI is all green now. Do the following:

- **Provide daily or every two days status updates**
- Do QA in local env thoroughly to find out errors with less risk than in Alpha
	- GraphQL requests
	- Requests between services
	- Luxe
	- Background Jobs
- Do local env QA until there's good confidence to send the changes to Alpha
- Do QA in Alpha
	- QA in alpha should be done in off hours to not cause mismatches with other devs work branches
	- Test same things as in Dev
- With a successful QA and an approved PR do the release at midnight or very early in the morning.
	- Do not do the release on a Friday.


## Option B

Take a step back to try to minimize our losses. Considering most of the errors showed up when activating Rails 7.0.4 this option suggests applying, in a PR by PR basis, upgrades to Edge until the most recent version before the target.

I suggest this so that a) we gain more confidence on upgrading Edge to newer versions of Rails, and b) minimize the risk of the current work branch with lots of things going on.

By following this approach, I expect to make it easier to do the upgrade from the latest Rails 6.x release and Rails 7.0.4. It'll also be less conflicting to do this upgrades.

Finally, it's also possible to bring some changes from the current work branch to apply them in isolation. Most of the errors appeared when activating Rails 7.0.4 but we could bring in the changes that fixed those errors and integrate them in the codebase.

### List of Rails 6.x versions before Rails 7.0.4

1. 6.1.6.1
2. 6.1.7
3. 6.1.7.1
4. 6.1.7.2
5. 6.1.7.3
6. 6.1.7.4
7. 6.1.7.5
8. 6.1.7.6
9. 6.1.7.7
10. 6.1.7.8

### List of possible items from the current branch to integrate individually

- Gems upgrades
	- active_record-postgres-constraints
	- acts-as-taggable-on
	- audited
	- devise
	- devise_token_auth
	- paranoia
	- psych
	- stateful_enum
	- factory_bot_rails
	- lol_dba
- Activate Zeitweirk mode
	- About 15 files updated to support it
