# Taqwa ERP — COA upgrade notes

## What changed in this review package

- Added `ChartOfAccountsUpgradeSeeder`, an additive/idempotent upgrade seeder for member profit payable, customer advances, retained earnings, and project-expense leaf accounts.
- The seeder preserves existing `is_system` flags and account balances; it refuses code/name/type conflicts, soft-deleted code conflicts, unexpected parent relationships, and attempts to convert a group account while ledger lines or bank links exist.
- Reconfigured `1200 Bank Accounts` and `5100 Project Expense` as groups only when their existing records match expected names/types and have no direct posting lines (and `1200` has no direct bank-account link).
- Updated investment and project-expense category mapping to use leaf accounts `5110`–`5190`.
- Updated sales posting account lookup to reject group accounts.
- Changed retained earnings configuration/default from `3100` to `3200`.
- Added `PROFIT_AUTHORIZED_ADMIN_IDS` support. It is intentionally fail-closed when blank.

## Before running

1. Keep a fresh database backup outside this ZIP.
2. Confirm the `admins` table and identify the intended profit-authorized administrator IDs using your own admin records.
3. Add those verified numeric IDs as a comma-separated list in your local `.env`, e.g. `PROFIT_AUTHORIZED_ADMIN_IDS=2,5` **only if IDs 2 and 5 are confirmed to be the authorized administrators**. Do not copy this example literally without checking.
4. Run syntax checks first.
5. Run the seeder only after reviewing the current COA and backup:
   `php artisan db:seed --class=Database\Seeders\ChartOfAccountsUpgradeSeeder`
6. Verify account codes, parent IDs, group flags, and bank-account links afterward.
7. Clear cached configuration after updating `.env`: `php artisan config:clear`.

## Important

- This package does not include `.env` because it may contain secrets. Keep your existing local `.env`; do not overwrite it.
- Do not run `migrate:fresh`, database reset commands, or financial cleanup again.
- The profit formula `Sales - Investments - Project Expenses` is preserved. Confirm operationally that amounts recorded as Investments are not duplicated in Project Expenses; the source code alone cannot establish that business fact.
- This package is a code-level correction and has not been executed against your live database. Test on a restored copy first.
