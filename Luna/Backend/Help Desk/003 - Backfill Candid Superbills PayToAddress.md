# 003 - Backfill de Candid Superbills PayToAddress

Es una rake que existía y luego se borró. Restauré todo con Claude. No entendía muy bien de qué va así que le pregunté y dijo:

The `backfill_pay_to_address` rake task (lines 55-74) is designed to update Candid encounters for Luna Direct clinics that have PO Box addresses.

**Purpose**: Updates the `pay_to_address` field for existing Candid encounters where clinics have PO Box addresses.

What it does:
1. Finds appointments that meet these criteria:
	- Have associated Candid encounters
	- Scheduled between Jan 1, 2023 and Sep 15, 2025
	- Belong to clinics where `pay_to_address` starts with "PO Box"
2. Schedules background jobs: For each matching appointment, it queues a `Candid::Egress::PayToAddressWorker` job with a 2-second delay between jobs to avoid overwhelming the system.
3. Provides progress info: Shows total count and estimated completion time.