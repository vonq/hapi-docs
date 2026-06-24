# Changelog

## 2026-06-11
### Changed
- `GET /v3/ats/atsuser/me/` now documents typed `settings.campaigns` fields instead of a free-form object: `duplicate_hash_prevention_minutes`, `enable_weekly_working_minutes`, `job_marketing_editing`, `validate_description_html_tags`, and `loose_validation` (the `job_post` / `marketplace` fields that may skip strict validation).

## 2026-06-08
### Added
- `features` object on products (`GET /products/search/`, `/products/single/{product_id}/`, `/products/multiple/{products_ids_or_portfolio_id}/`) exposing product feature flags. Currently includes `direct_apply` (`n/a` | `optional` | `mandatory`).
- New schemas `ProductFeatures` and `DirectApplyEnum`.

## 2026-05-25
### Changed
- Schema examples now use deterministic placeholder UUIDs, `development@vonq.com`, and example/acme URLs instead of real-looking partner, campaign, wallet, product, and job values.
- The schema now marks `ATSPublicSettings.ai_suggestions` as deprecated, clarifies wallet and billing-portal operation text, uses `Resource - Action` operation summaries, declares authentication for campaign takedown, and removes stale Stoplight v1 links from campaign descriptions.
- Documentation now centralizes supported `Accept-Language` values in API Overview, link guide references to that section, and restores Wallets & Payments details from the legacy guide for direct charge ordering, billing portal expiry, payment iframe sizing, negative-balance top-ups, and refunds.

## 2026-05-13
### Removed
- `ats_user_jwt_allow_renewal` field from `ATSSSetting`; `GET /v3/ats/atsuser/me/` now exposes only `ats_user_jwt_expiration` under `settings`.
### Changed
- `GET /v3/ats/atsuser/me/` now references `PublicAuthenticatedATSUser` in the schema instead of the internal `ATSUserAuthenticated` component; the documented response fields are unchanged.
- Product endpoint descriptions were clarified for `GET /products/search/` and `GET /products/single/{product_id}/`, including the updated bulk-retrieval path `/products/multiple/{products_ids_or_portfolio_id}/`.

## 2026-05-08
### Added
- `PATCH /campaigns/{campaignId}/edit` - sandbox-only endpoint to simulate campaign and ordered-product status transitions. Useful for exercising webhook handlers and status-driven UI without waiting for real job board responses. Blocked in production. See [Status & Lifecycle](guides/08-campaigns/status.md#sandbox-status-simulation).

## 2026-05-04
### Removed
- `force_knockout_enabled` field on `ScreeningJobSettings` (knockout is now always enabled). Affects `POST /v3/screening/jobs/`.
### Changed
- `injectFavorites` and `injectTopOrdered` query parameters on `GET /products/search/` are now triggered by `offset=0` (rather than "page 1"). When enabled, the promoted products are excluded from the regular paginated result stream.
- The schema for `GET /products/multiple/{products_ids_or_portfolio_id}/` now documents the existing paginated response envelope (`count`, `next`, `previous`, `results`, optional `facets` and `debug`) instead of a single product object.

## 2026-04-30
### Added
- `notes` field (max 2000 characters) on `ScreeningJobInfo` for `POST /v3/screening/jobs/` and `GET /v3/screening/jobs/{id}/`.
- `force_knockout_enabled` boolean (default `false`) on `ScreeningJobSettings` for `POST /v3/screening/jobs/`. _(Removed again on 2026-05-04 - see above.)_
### Changed
- `notes` on `ScreeningJobCreateData` is now bounded to 2000 characters.

## 2026-04-28
### Added
- `sortBy` query parameter on `GET /products/job-titles/`: accepts `relevance` (default), `frequency.asc`, `frequency.desc`, `name.asc`, `name.desc`.
### Changed
- `channel_id` and `contract_id` in `SmartfillPostingRequirementsDetail` are now nullable, supporting HAPI Job Marketing flows without a contract.

## 2026-04-22
### Added
- `injectFavorites` (boolean) query parameter on `GET /products/search/`: prepends the authenticated ATS user's favorite products to page 1 of the results.
- `injectTopOrdered` query parameter on `GET /products/search/`: prepends the user's top-ordered products for a given window (`1w`, `1m`, `6m`, `12m`) to page 1.
### Changed
- `channel_id` and `contract_id` in `SmartfillPostingRequirementsDetail` are now read-only strings (previously required UUID-format input), allowing Smartfill use on Job Marketing without a contract.
- `GET /products/multiple/{products_ids_or_portfolio_id}/` now rejects requests that also pass `products_ids_or_portfolio_id` as a query parameter; pass it only in the URL path.

## 2026-04-15
### Added
- Product Favorites endpoints (ATS-user authenticated):
  - `GET /v3/products/favorites/` - list favorite product IDs for the authenticated ATS user
  - `POST /v3/products/favorites/` - add a product to favorites
  - `DELETE /v3/products/favorites/{product_id}/` - remove a product from favorites
- New schemas: `ProductFavorite`, `ProductFavoriteCreate`, `ProductFavoriteCreateRequest`.
- `GET /products/multiple/{products_ids_or_portfolio_id}/` now also accepts:
  - `favorites` - returns the authenticated user's favorite products
  - `orders-top-{1w|1m|6m|12m}` - returns the user's most-ordered products for a time window
  - a Portfolio Selector UUID - returns products belonging to a pre-configured selector
### Changed
- Path-parameter names were clarified in the schema with no behavioral change. The documented paths now use:
  - `/products/multiple/{products_ids_or_portfolio_id}/`
  - `/products/delivery-time/{products_ids}/`
  - `/contracts/multiple/{contracts_ids}/`
  - `/contracts/posting-requirements/{channel_id_or_contract_id}/{posting-requirement-name}/`
  - `/igb/contracts/groups/{group_idx}/`
  - `/v3/smartfill/posting-requirements/{task_id}/`
  - `/v3/smartfill/product-search-filters/{task_id}/`
  - `/v3/smartfill/vacancy-fields/{task_id}/`
- Improved descriptions for the multiple-products endpoint (documents the new `favorites`, `orders-top-*`, and portfolio-selector inputs) and Product Favorites endpoints.

## 2026-04-10
### Added
- Initial publication of the **v2 API schema** (HTP-3096) at `schema/build/public.json` (OpenAPI 3.0.3, ~71 paths, ~228 component schemas). Endpoint coverage now includes:
  - **Campaigns** - listing, retrieval, ordering, status, editing, cancellation, product-cancellation, and validation (`/campaigns`, `/campaigns/{campaignId}`, `/campaigns/order`, `/campaigns/{campaignId}/edit`, `/campaigns/{campaignId}/cancellation`, `/campaigns/{campaignId}/product_cancellation`, `/campaigns/{campaignId}/status`, `/campaigns/validate-*`)
  - **Contracts** - CRUD, listing, multiple, posting requirements, contract groups (`/contracts/...`, `/igb/contracts/groups/...`)
  - **Products** - search, single, multiple, specs, facet options, delivery time, channel MOCs, taxonomy filters (`/products/...`, `/products/channels/mocs/...`)
  - **Taxonomy** - education levels, seniority (`/taxonomy/education-levels`, `/taxonomy/seniority`)
  - **ATS Users** - CRUD, JWT issuance, profile, settings (`/v3/ats/...`)
  - **Screening** - jobs and applications, with attachments and dossiers (`/v3/screening/jobs/...`)
  - **CPA+** - campaign applications and attachment downloads (`/v3/cpacampaigns/{hapi_campaign_id}/applications/...`)
  - **Smartfill** - posting requirements, product-search filters, vacancy fields (`/v3/smartfill/...`)
  - **Wallets & Payments** - wallet creation, billing portal, top-up (`/wallet/...`)
  - **Apply / Direct Apply** - application feedback (`/v3/apply-applications/application-feedback/`)

## 2026-04-02
### Changed
- `postingDetails.workingLocation.allowsRemoteWork` now also accepts the string `hybrid`

## 2025-12-10
### Added
- Italian and Spanish as partially supported languages

## 2025-08-21
### Changed
- Smartfill for posting requirements now returns standardized data types

## 2025-06-04
### Added
- Metadata for campaigns and campaigns ordered products
- CPA+ endpoints for applications, CPA+ info in product and ordered product

## 2025-05-08
### Added
- Taxonomy details in the response for Smartfill Vacancy Fields
### Changed
- Providing unsupported image types for `postingDetails.organization.companyLogo` in "Order a new Campaign" endpoint and the campaign validation endpoints now triggers a validation error

## 2025-04-23
### Added
- Added Smartfill endpoints for product search filters

## 2025-04-05
### Changed
- `salaryIndication.from` and `salaryIndication.to` in the Campaign's payload now support decimal values.

## 2025-03-03
### Changed
-  Smartfill character limit has been increased from 10,000 to 20,000 characters per context type.

## 2025-02-25
### Added
- On demand, if the campaign is not in a cancellable state, the campaign cancellation endpoint will return a bad request response.

## 2024-11-18
### Added
- Added `isBundle` filter to `/products/search/` endpoint

## 2024-10-21
### Added
- Added `weeklyWorkingMinutes` as alternative to `weeklyWorkingHours` in the Campaign's `PostingDetails` object

## 2024-10-11
### Added
- Added Smartfill endpoints.

## 2024-08-14
### Added
- Added French as a partially supported language.

## 2024-05-29
### Changed
- `postingDetails.salaryIndication` is no longer required.

## 2024-05-07
### Added
- `labels` property for contract listing endpoint

## 2024-03-14
### Added
- `hourly` as option to `postingDetails.salaryIndication.period` for `/campaigns/order` endpoint

## 2024-01-17
### Changed
- `postingDetails.contactInfo.phoneNumber` now has a maximum length, 40 characters

## 2023-10-16
### Added
- new channel type: `product_bundle`
- new property for the product: `bundle_products_ids`

## 2023-09-21
### Added
- new endpoint that allows taking individual products offline within a campaign

## 2023-08-15
### Added
- add labels for campaign ordering and filtering
- campaign editing
- add posting duration days
- expose message in facets payload
- add autocomplete support (lazy load) for HIER and SELECT facets

## 2023-06-05
### Added
- `direct_charge` as possible value for campaign `paymentMethod`
- the Payment Widget can now receive a `campaignId` as parameter

## 2023-06-01
### Added
- Allow custom posting expiry during campaign ordering for mc products

## 2023-05-12
### Changed
- When creating a wallet, the `customerName` field is optional.

## 2023-05-09
### Added
- `missing_billing_keys` wallet parameter

## 2023-05-02
### Added
- product prices can be shown in AUD.
- `currency` as optional parameter when creating a wallet.
- `min_topup`, `max_purchase_order`, `max_outstanding_balance` wallet parameters

### Removed
The `amount` parameter of the Top-Up Iframe `/wallet/topup.html`

## 2023-04-27
### Changed
- `statusDescription` may now contain a standardized error message.
### Added
- `statusSolution` and `statusRawError` fields for the `/campaigns/{campaignId}/status`, `/campaigns` and `/campaigns/{campaignId}` endpoints.

## 2023-02-03
### Changed
- Deprecated `amount` parameter of the Top-Up Iframe `/wallet/topup.html`

## 2022-10-11
### Changed
 - List autocomplete values for posting requirement can now receive multiple terms in payload for certain types of posting requirements

## 2022-09-29
### Added
 - pagination parameters for Retrieve Multiple Products endpoint

## 2022-09-01
### Added
 - campaign common fields validation
 - campaign full payload validation
 - campaign product posting validation

### Changed
 - error response object shape changed for contracts creation
 - added requires property on facet options

## 2022-08-18
### Added
- Channel and Contract channels logos
- Autocomplete endpoint for facets and contracts

### Changed
- mc_only products can be fetched via `/products/single/{{product_id}}/` too
- improved the response for errors during contract creation
- removed contract_credentials_is_oauth from the channels endpoints
- remove credentials from the posting_requirements for auto-complete facets


## 2022-08-10
### Added
- Users must accept terms of service before being able to top-up a wallet

## 2022-08-08
### Changed
- Added contracts group support


## 2022-07-18
### Added
- Minimum top-up amount for wallets

## 2022-06-06
### Added
- Allow setting an alias during contracts creation
- Added display rules for channel facets
- Payments


### Changed
- Single contract per channel limitation was removed

## 2022-03-07
### Added
- Adding new endpoint to retrieve MC channels

## 2022-02-21
### Added
- Adding information about products statuses to the Check campaign status response

## 2021-12-01
### Added
- Adding support for multi-posting


## 2021-09-09
### Added
- Adding flag allowsRemoteWork in campaign posting details working location

## 2021-08-10
### Added
- Adding new productId field in campaign postings objects

## 2021-06-29
### Added
- Adding new time to setup response field for products.

## 2021-03-24
### Added
- Products can now be sorted by recency.
- Salary ranges can now specify a currency.

## 2021-03-15
### Added
- Specify salary expects integers

## 2021-03-12
### Added
- Adding audience_group to products response

## 2021-02-22
### Added
- Adding square and rectangle logo urls to products response

## 2021-02-18
### Added
- Adding new multiple products detail endpoint

## 2021-02-03
### Added
- Adding durationPeriod for each order's product

## 2020-12-02
### Added
- Adding duration and jobBoardLink for each order's product

## 2020-12-01
### Added
- Adding new product order endpoint

## 2020-11-26
### Added
- Adding status and deliveredOn for each order's product

## 2020-11-20
### Remove
- Data requirements section removed from documentation

## 2020-10-26
### Changed
- Changed the Contact Information in Posting details from mandatory to optional

## 2020-09-22
### Added
- Adding UTM parameters optionally for each order's product

## 2020-09-21
### Changed
- Rate limits documented

## 2020-07-08
### Changed
- Fixed documentation of `CampaignOrder`.`orderedProducts` from array[integer] to be array[string]
- Changed single product [/portfolio/{productId}] endpoint's parameter `productId` to accept string instead of integer

## 2020-04-06
### Changed
- Updated dependencies
- Fix that SalaryIndication 'from' could be a negative number
- Get 'cta-clicks' from Clickdata instead of unique visitors

## 2020-03-30
### Added
- Retrieve Clickdata from Data Web Services
- Clickdata cronjob for all campaigns

### Changed
- Simplify updating of product names from product portfolio

## 2020-03-23
### Added
- Integration test for Campaign service
- Domain model for Click Data
- Add campaign performance clicks to response data

### Changed
- Convert invalid UUID to 400 errors
- Use prefix for Redis keys
- Add throttling back to sandbox

## 2020-03-16
### Added
- Create API token from dashboard

### Changed
- Disable rate limiting in sandbox
- Don't serve robots.txt
- Default datetime for migrations so required datetime fields don't break
- use lowercase postcode for API

## 2020-03-09
### Added
- Add Partner name to Slack notification
- Add API Team Partner Id to sandbox to filter internal testing

### Changed
- Coding standards updated
- Base Docker image updated

### Fixed
- Salary Range should allow to and from to be equal

## 2020-03-02
### Added
- Dashboard for viewing campaign statistics in the API
- Track imported campaigns
- Add US and Canada regions to Region taxonomy
- Generated API tokens for US partners

### Changed
- Updated dependencies
- Show campaign details
- Ensure SalaryRange 'from' is less than 'to'
- Production and Sandbox settings
- Support sandbox campaign service for APITEAM

## 2020-02-24
### Added
- Added request ID to the headers when dispatching to the Campaign Service

### Changed
- Updated dependencies

## 2020-02-17
### Added
- Added more info when getting a 400 from CS
- Added error responses to API Documentation

### Changed
- Set default timezone
- Map the Recruiter Id in Marketplace to Recruiter ExternalID in Campaign Service
- Map Marketplace CompanyId to CampaignService ExternalCompanyId
- More consistent error responses

### Fixes
- Added microseconds to datadog link
- Better handling of Campaign-Service error response
- Fixed logic for Time To Process

## 2020-02-10
### Added
- Added documentation about code style linter

### Changed
- Disable basic auth on /health endpoint
- Remove Traefik configuration
- Clean up old debug code

### Fixes
- Fix incorrect link for Datadog logs
- Use UTC datetime everywhere

## 2020-02-03
### Added
- Announce new orders to Marketplace API Slack channel
- Additional information in API documentation
- Added Cancel Campaign endpoint

### Changed
- Upgrade to PHP 7.4.2
- The CampaignService now accepts the EmploymentType
- Move additional variables to AWS secrets

## 2020-01-27
### Added
- Added Campaign Details endpoint
- Added ORM integration tests
- Added code style checks for configuration files

### Changed
- Always return JSON for all responses
- Updated Symfony to 5.0.3
- Improved deployment environment setup and logging
- Use AWS for application secrets instead of hard-coded variables

### Fixes
- Fixed wrongly mapped Taxonomy
- Fixed incorrectly mapped serializer

## 2020-01-20
### Added
- Multiple Employment Types
- Retry mechanism for internal sending of orders
- Production keys for production users
- Campaign Status
- Specific Campaign Status endpoint for Partners to check status of a campaign

### Changed
- Moved beta keys to sandbox environment
- Store price per product in an order

## 2020-01-13
### Added
- Automatically process orders from Marketplace

### Fixes
- Fixed a wrongly mapped price field

## 2020-01-06
### Added
- Better logging of orders

### Changed
- Upgraded to latest framework version

## 2019-12-30
### Added
- Added sandbox environment
- Added automatic provisioning of sandbox
- Separate data flow for production and sandbox

## 2019-12-23
### Fixes
- An empty list of campaigns now returns the correct offset for pagination

## 2019-12-16
### Changed
- WorkingHours no longer requires `from` field to be required
- Recruiter Info now only has the `name` field required

## 2019-12-09
### Added
- Use production settings on production server

## 2019-12-02
### Changed
- Taxonomy is now correctly mapped

## 2019-11-25
### Changed
- CampaignID is now UUID5

### Fixes
- Wrongly mapped Education Level
- Updated framework to latest version

## 2019-11-18
### Added
- Added totalPrice of a campaign to an ordered campaign
- Added automatic import of the Taxonomy

### Changed
- Use actual real life data for Portfolio
- Use actual real life data for Taxonomy

## 2019-11-11
### Added
- Added locking mechanism for migration
- Added automatic import of the Portfolio

### Changed
- Updated base images for application

### Fixes
- Fixes security issues from dependancy
- Fixes migration script

## 2019-11-04
### Added
- Added API_VERSION_ID to response to see which version of the API is used
- Added TimeToProcess field in product output to show how long a product takes to get published
- Made DB ready for production
- Added automatic publishing of Documentation

### Fixes
- Fixes an issue with generating documentation

## 2019-10-28
### Added
- Added the Changelog to the API Documentation

## 2019-10-21
### Added
- Import functionality to use the Portfolio Service to fulfill the local Product Storage
- Ability to import the full portfolio from another source or a single product if needed
- Adds a DB implementation for the local Portfolio Storage

### Changed
- Updated dependencies to the latest versions
- Improved the way the dev environment sets the db initially
- Upgraded the Base image we use for deploy our webservices

## 2019-10-21 - Portfolio Service
### Added
- A Build definition for the export/import script as a container
- Product Logos now contain a generated ID
- The initial dumps to start development on the Taxonomy for Portfolio Service

### Changed
- Updated dependencies to the latest versions
- Updated the Taxonomy results documentation

### Fixes
- A Production error when certain files were not available after deploying
- Export/Import script DSN parsing behaviour
- Upgraded the Base image we use for deploy our webservices

## 2019-10-14
### Added
- Specific tags for the different services for ease of consumption in our Logging Management Platform
- **Added another potential Alpha Partner**
- CI/CD pipeline reports status in Slack

### Changed
- Improved the Service deployment definition for ease of creation and modification of the service
- Updated dependencies to the latest versions

## 2019-10-14 - Portfolio Service
### Added
- Endpoint to export Portfolio Information to the Marketplace API
- Specific tags for the different services for ease of consumption in our Logging Management Platform
- Filters to avoid blacklisted products to be shown in the Portfolio Service

### Changed
- Improved the Service deployment definition for ease of creation and modification of the service
- Portfolio entries will convert prices in GBP to EUR automatically
- Database export/import script deployed as a container to be executed in production

## 2019-10-07
### Added
- Support for ElasticSearch data stores
- Monitor to check Marketplace API errors in real time
- **Added another possible Alpha Partner to the enabled ones**

### Changed
- Upgraded the Base image we use for deploy our webservices
- Updated dependencies to the latest versions

## 2019-10-07 - Portfolio Service
### Added
- Deployment script for the Portfolio Service

### Changed
- Updated the MySQL we're using in dev and prod to version 8.0
- Updated dependencies to the latest versions
- Improved the Database export/import script for Portfolio Service
- Improved error handling of Portfolio Service for when the application isn't started yet
