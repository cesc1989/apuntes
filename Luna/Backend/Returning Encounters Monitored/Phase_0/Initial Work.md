# Initial Work

## Authentication & User Management

- **Replace ClinicalDashboard::User with AdminUser** - Eliminate duplicate authentication system and use existing Google OAuth
- **Remove clinical_dashboard_users table** - Drop unnecessary table after user migration
- **Update controllers to use current_admin_user** - Replace current_clinical_dashboard_user references
- **Add clinical dashboard permissions to AdminAbility** - Integrate with existing authorization system
- **Remove devise_for :users from clinical dashboard routes** - Clean up duplicate Devise configuration

## String & Utility Improvements

- **Replace String#to_bool monkey patch with utility class** - Remove global String modification and create dedicated converter or explicitly check for booleans (just 20 occurrences)
- **Remove config/initializers/string.rb** - Clean up global monkey patch initializer

## Database & Infrastructure

- **Replace Athena queries with direct PostgreSQL** - Eliminate AWS dependency and use existing Rails models
- **Remove 68+ Athena-related files** - Clean up complex query execution system
- **Create app/queries/clinical_dashboard/ structure** - Build PostgreSQL query objects for complex queries if necessary
- **Remove RequestQueryExecutionWorker and execution tracking** - Simplify background job system
- **Fix manual ID generation race conditions** - Replace ClinicalDashboardPrimaryKey with proper defaults on postgres db
- **Remove ExecutionId model and table** - Clean up Athena tracking infrastructure

## API & Service Consolidation

- **Replace Edge API calls with direct service calls** - Remove internal HTTP overhead
- **Update PlanOfCareSigner to call services directly** - Remove HTTP client for plan of care actions

## Kong Gateway Removal

- **Remove Kong token authentication**
- **Remove EDGE_API_TOKEN and EDGE_API_DOMAIN usage** - Clean up internal API authentication
- **Remove KONG_* environment variables** - Clean up Kong-related configuration

## Code Deduplication & Service Reuse

- **Consolidate email services with existing mailers** - Merge clinical dashboard email with existing email infrastructure

## HubSpot Integration Improvements

- **Replace HTTParty with HubSpot gem connection** - Use existing HubSpot gem instead of raw HTTP calls
- **Consolidate HubSpot API calls in UnseenPatients services** - Leverage existing HubSpot service patterns
- **Remove manual HTTP.auth Bearer token usage** - Use HubSpot gem authentication methods
- **Standardize HubSpot error handling** - Use existing HubSpot exception patterns
- **Reuse existing HubSpot contact search functionality** - Leverage existing contact management services

## JWT & Token Cleanup

- **Simplify JWT authentication system** - Reduce complex token management (reuse existing JWT gem)
- **Remove unnecessary access token controllers** - Consolidate with admin session management

## RuboCop & Code Quality

- **Fix RuboCop offenses in clinical dashboard files** - Address code style violations
- **Standardize code formatting across clinical dashboard** - Apply consistent Ruby style guide

## Testing & Documentation

- **Replace HTTP mocks with direct service call tests** - Improve test reliability and speed
- **Add integration tests for consolidated services** - Ensure merged functionality works correctly
- **Update API documentation** - Reflect simplified authentication and direct service calls
- **Document classes and project structure** - Update in code documentation

## Environment & Configuration Cleanup

- **Remove Athena retry contexts from retryable.rb** - Clean up retry configuration