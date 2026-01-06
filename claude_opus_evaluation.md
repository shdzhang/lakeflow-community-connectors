# Lakeflow Community Connectors Evaluation

**Evaluation Date:** January 6, 2026

---

## Summary Scores

| Connector | Completeness (50%) | Methodology & Reusability (30%) | Code Quality (20%) | **Total** |
|-----------|-------------------|--------------------------------|-------------------|-----------|
| **Qualtrics** | 3 | 3 | 3 | **3.00** |
| **GitHub** | 3 | 3 | 3 | **3.00** |
| **Stripe** | 3 | 2 | 3 | **2.70** |
| **HubSpot** | 2 | 2 | 3 | **2.30** |
| **Zendesk** | 2 | 2 | 2 | **2.00** |
| **Mixpanel** | 2 | 2 | 2 | **2.00** |

---

## Detailed Evaluations

---

## 1. Qualtrics Connector

**Overall Score: 3.00 / 3.00**

### Scores

| Criterion | Score | Weight |
|-----------|-------|--------|
| Completeness & Functionality | 3 | 50% |
| Methodology & Reusability | 3 | 30% |
| Code Quality & Efficiency | 3 | 20% |

### Completeness & Functionality (3/3)

**Strengths:**
- 8 fully implemented tables: surveys, survey_definitions, survey_responses, distributions, mailing_lists, mailing_list_contacts, directory_contacts, directories
- Auto-consolidation feature: fetches data from all surveys without specifying surveyId - unique among connectors
- Comprehensive 3-step export workflow for responses (create job → poll → download)
- Mix of CDC, append, and snapshot ingestion types appropriately per table
- Per-survey cursor tracking for incremental sync in consolidation mode

**Opportunities for Improvement:**
- Add survey questions as separate table for easier analytics
- Consider adding survey quotas and embedded data endpoints
- Support for survey blocks/flow as separate tables vs JSON strings

### Methodology & Reusability (3/3)

**Strengths:**
- `QualtricsConfig` class for centralized configuration constants - highly reusable pattern
- `_iterate_all_surveys()` generic helper for auto-consolidation - reusable template for parent-child APIs
- `_fetch_paginated_list()` generic paginated list API helper
- `_normalize_keys()` snake_case normalization utility
- Comprehensive versioned documentation (v1.1, v1.2 release notes)
- Detailed API documentation file (`qualtrics_api_doc.md`)

**Opportunities for Improvement:**
- Extract pagination helpers to shared library
- Add unit test coverage for edge cases

### Code Quality & Efficiency (3/3)

**Strengths:**
- Configurable polling intervals (fast/slow based on progress %)
- Exponential backoff for HTTP retries
- Rate limiting with configurable delays between surveys
- Comprehensive logging throughout
- Robust error handling with graceful continuation on partial failures
- Schema validation test file exists

**Opportunities for Improvement:**
- Consider async/concurrent survey fetching for better throughput
- Add request timeout configuration

### Top 3 Recommended Next Actions

1. **Add concurrent survey processing** - Implement async fetching for auto-consolidation to reduce total execution time
2. **Extract shared utilities to library** - Move `_fetch_paginated_list`, `_normalize_keys`, `_iterate_all_surveys` to `libs/` for reuse
3. **Add survey questions flattened table** - Parse JSON questions field into separate table for easier downstream analytics

---

## 2. GitHub Connector

**Overall Score: 3.00 / 3.00**

### Scores

| Criterion | Score | Weight |
|-----------|-------|--------|
| Completeness & Functionality | 3 | 50% |
| Methodology & Reusability | 3 | 30% |
| Code Quality & Efficiency | 3 | 20% |

### Completeness & Functionality (3/3)

**Strengths:**
- 12 fully implemented tables covering major GitHub API surface
- Appropriate ingestion types: CDC (issues, PRs, comments), append (commits, reviews), snapshot (repos, users, orgs)
- Lookback window implementation for cursor handling (handles late-arriving updates)
- Rich nested schema preservation (user, labels, milestone structs)
- Reviews table supports both single PR and full repo consolidation

**Opportunities for Improvement:**
- Add milestones table
- Add releases/tags endpoints
- Support GitHub Actions workflows/runs data

### Methodology & Reusability (3/3)

**Strengths:**
- Detailed `github_api_doc.md` (800+ lines) - excellent API documentation methodology
- Static `_extract_next_link()` helper for Link header parsing - reusable
- Reusable nested struct schemas (user_struct, label_struct, etc.)
- Clear separation of table-specific read methods
- Research log documenting sources with confidence levels

**Opportunities for Improvement:**
- Generalize pagination helper for reuse
- Add schema discovery from OpenAPI spec

### Code Quality & Efficiency (3/3)

**Strengths:**
- Session-based HTTP with persistent headers
- Configurable `per_page`, `max_pages_per_batch`, `lookback_seconds` per table
- Proper timeout handling (30s per request)
- Comprehensive static schemas with type safety
- Clean method organization by table type

**Opportunities for Improvement:**
- Add rate limit header monitoring
- Implement retry with exponential backoff for 5xx errors

### Top 3 Recommended Next Actions

1. **Add rate limit monitoring** - Parse `X-RateLimit-*` headers and implement proactive throttling
2. **Add releases table** - High-value addition for release tracking analytics
3. **Implement retry mechanism** - Add exponential backoff for transient failures (currently only basic error raising)

---

## 3. Stripe Connector

**Overall Score: 2.70 / 3.00**

### Scores

| Criterion | Score | Weight |
|-----------|-------|--------|
| Completeness & Functionality | 3 | 50% |
| Methodology & Reusability | 2 | 30% |
| Code Quality & Efficiency | 3 | 20% |

### Completeness & Functionality (3/3)

**Strengths:**
- 16 tables covering comprehensive Stripe API surface
- All major payment, billing, and financial objects supported
- Deletion tracking for customers, products, plans
- Comprehensive nested schema handling (billing_details, payment_method_details)
- Support for polymorphic payment method types (card, ACH, various regional methods)

**Opportunities for Improvement:**
- Add Connect accounts and transfers
- Add Tax objects (tax_rates, tax_transactions)
- Support Stripe Radar (reviews, rules)

### Methodology & Reusability (2/3)

**Strengths:**
- Centralized `_object_config` for table metadata
- Reusable nested schema definitions (_address_schema, _shipping_schema, etc.)
- Clean separation of full vs incremental reads

**Opportunities for Improvement:**
- No separate API documentation file
- Missing novel approaches - standard pagination implementation
- README lacks methodology description

### Code Quality & Efficiency (3/3)

**Strengths:**
- Comprehensive 850+ line schema definitions with proper typing
- Centralized `_schema_config` dictionary
- Basic rate limiting (0.1s delay between requests)
- `test_connection()` method for validation
- Proper authentication using tuple auth pattern

**Opportunities for Improvement:**
- Add configurable rate limiting
- Implement exponential backoff (noted in README but not implemented)

### Top 3 Recommended Next Actions

1. **Create stripe_api_doc.md** - Document API endpoints, schemas, and field mappings similar to GitHub
2. **Add Connect objects** - High value for marketplace/platform use cases (accounts, transfers, payouts)
3. **Implement exponential backoff** - Currently mentioned in docs but not implemented in code

---

## 4. HubSpot Connector

**Overall Score: 2.30 / 3.00**

### Scores

| Criterion | Score | Weight |
|-----------|-------|--------|
| Completeness & Functionality | 2 | 50% |
| Methodology & Reusability | 2 | 30% |
| Code Quality & Efficiency | 3 | 20% |

### Completeness & Functionality (2/3)

**Strengths:**
- 10 standard CRM objects + custom objects discovery
- Dynamic schema discovery via Properties API
- Association support (contacts↔companies, deals↔tickets, etc.)
- Both full refresh and incremental modes

**Opportunities for Improvement:**
- Missing marketing objects (forms, emails, campaigns)
- No activities/engagements stream
- Missing pipelines/pipeline stages
- Line items for deals not supported

### Methodology & Reusability (2/3)

**Strengths:**
- Centralized `_object_config` dictionary pattern
- Schema and metadata caching
- `_default_object_config` for custom objects

**Opportunities for Improvement:**
- No separate API documentation file
- Missing novel approaches
- Generic pagination could be extracted

### Code Quality & Efficiency (3/3)

**Strengths:**
- Schema caching to avoid repeated API calls
- Clean type mapping function `_map_hubspot_type_to_spark`
- Unified `_read_data` method for both modes
- `test_connection()` method
- Rate limiting implemented (0.1s delay)

**Opportunities for Improvement:**
- Properties API call per schema discovery could be cached longer
- Consider batch association lookups

### Top 3 Recommended Next Actions

1. **Add marketing objects** - Forms, emails, campaigns are high-value HubSpot data
2. **Create hubspot_api_doc.md** - Document API endpoints and field mappings
3. **Add pipeline/deal stages table** - Critical for sales analytics use cases

---

## 5. Zendesk Connector

**Overall Score: 2.00 / 3.00**

### Scores

| Criterion | Score | Weight |
|-----------|-------|--------|
| Completeness & Functionality | 2 | 50% |
| Methodology & Reusability | 2 | 30% |
| Code Quality & Efficiency | 2 | 20% |

### Completeness & Functionality (2/3)

**Strengths:**
- 8 core tables covering main Zendesk objects
- Uses Zendesk incremental API for supported objects
- Help Center articles included
- Community topics support

**Opportunities for Improvement:**
- Missing ticket_fields, ticket_forms
- No satisfaction_ratings table
- Missing macros, triggers, automations
- Views/search API not supported

### Methodology & Reusability (2/3)

**Strengths:**
- Clean separation of incremental vs paginated reads
- api_config dictionary for endpoint mapping

**Opportunities for Improvement:**
- No API documentation file
- Basic README without detailed methodology
- No reusable helpers extracted

### Code Quality & Efficiency (2/3)

**Strengths:**
- Proper incremental API usage with `start_time` cursor
- Handles both incremental and paginated endpoints
- Clean schema definitions

**Opportunities for Improvement:**
- Missing `ingestion_type` in `read_table_metadata` return
- No rate limiting implementation
- No retry logic for failed requests
- Hard-coded 1000 page limit could cause issues
- ticket_comments parsing has complex nested logic without error handling

### Top 3 Recommended Next Actions

1. **Add ingestion_type to metadata** - Currently missing from `read_table_metadata` return dict
2. **Implement rate limiting** - Zendesk has strict limits (200 req/min); currently no protection
3. **Add ticket_fields table** - Essential for interpreting custom field values in tickets

---

## 6. Mixpanel Connector

**Overall Score: 2.00 / 3.00**

### Scores

| Criterion | Score | Weight |
|-----------|-------|--------|
| Completeness & Functionality | 2 | 50% |
| Methodology & Reusability | 2 | 30% |
| Code Quality & Efficiency | 2 | 20% |

### Completeness & Functionality (2/3)

**Strengths:**
- Core analytics tables: events, cohorts, cohort_members, engage
- Supports both service account and API secret auth
- Regional support (US/EU endpoints)
- 7-day batch chunking for events API

**Opportunities for Improvement:**
- Only 4 tables - missing funnels, retention, insights
- No JQL query support
- Missing user segmentation data
- No revenue/transaction analytics endpoints

### Methodology & Reusability (2/3)

**Strengths:**
- Standard/custom property separation pattern
- Multiple datetime format parsing
- Configurable historical_days parameter

**Opportunities for Improvement:**
- No API documentation file
- Reusable patterns not extracted to shared library
- Basic pagination approach

### Code Quality & Efficiency (2/3)

**Strengths:**
- Rate limiting implementation (0.34s delay)
- Schema caching
- Graceful handling of rate limit errors (returns partial data)
- JSONL parsing for export endpoint

**Opportunities for Improvement:**
- `print()` statements instead of proper logging
- No retry logic for non-rate-limit errors
- Hard-coded 7-day batch size not configurable
- cohort_members uses inefficient per-cohort API calls

### Top 3 Recommended Next Actions

1. **Replace print() with logging** - Use Python logging module for proper log levels
2. **Add funnels/retention tables** - High-value analytics tables missing
3. **Make batch size configurable** - Currently hard-coded BATCH_SIZE_DAYS = 7

---

## Cross-Connector Recommendations

### Templates/Patterns to Extract

| Pattern | Source Connector | Value |
|---------|-----------------|-------|
| `QualtricsConfig` centralized config class | Qualtrics | High |
| `_iterate_all_surveys()` parent-child consolidation | Qualtrics | High |
| `_extract_next_link()` Link header parsing | GitHub | Medium |
| `_normalize_keys()` snake_case conversion | Qualtrics | Medium |
| Reusable nested schema definitions | Stripe | Medium |

### Common Gaps Across Connectors

1. **Inconsistent logging** - Mix of print() and logger usage
2. **Rate limit handling** - Varies widely in sophistication
3. **Retry logic** - Not all connectors implement exponential backoff
4. **API documentation** - Only GitHub and Qualtrics have detailed API docs

### Priority Improvements for Template

1. Add standard logging configuration to base template
2. Create shared HTTP client with retry/rate-limit handling
3. Require API documentation file for all connectors
4. Add standard test suite requirements

---

## Appendix: API Coverage Analysis

### HubSpot API Coverage

| API Category | Covered | Missing |
|--------------|---------|---------|
| CRM Objects | ✅ contacts, companies, deals, tickets | |
| Engagements | ✅ calls, emails, meetings, tasks, notes | |
| Marketing | ❌ | forms, emails, campaigns |
| Analytics | ❌ | web analytics, reports |
| Custom Objects | ✅ | |

### GitHub API Coverage

| API Category | Covered | Missing |
|--------------|---------|---------|
| Issues | ✅ issues, comments | |
| Pull Requests | ✅ PRs, reviews | |
| Repositories | ✅ repos, branches, collaborators | milestones, releases |
| Users/Orgs | ✅ users, orgs, teams, assignees | |
| Commits | ✅ commits | |
| Actions | ❌ | workflows, runs |

### Qualtrics API Coverage

| API Category | Covered | Missing |
|--------------|---------|---------|
| Surveys | ✅ surveys, definitions | survey versions |
| Responses | ✅ responses with export workflow | |
| Distributions | ✅ distributions | |
| Contacts | ✅ directories, mailing lists, contacts | |
| XM Directory | ✅ full support | |

### Stripe API Coverage

| API Category | Covered | Missing |
|--------------|---------|---------|
| Payments | ✅ charges, payment_intents, payment_methods | |
| Billing | ✅ invoices, subscriptions, plans, prices | quotes |
| Financial | ✅ balance_transactions, payouts, refunds | transfers |
| Products | ✅ products, prices, coupons | |
| Connect | ❌ | accounts, transfers |
| Tax | ❌ | tax_rates, tax_transactions |

### Zendesk API Coverage

| API Category | Covered | Missing |
|--------------|---------|---------|
| Tickets | ✅ tickets, ticket_comments | ticket_fields, forms |
| Users | ✅ users, organizations, groups | |
| Help Center | ✅ articles | sections, categories |
| Community | ✅ topics | posts |
| Admin | ❌ | macros, triggers, automations |

### Mixpanel API Coverage

| API Category | Covered | Missing |
|--------------|---------|---------|
| Events | ✅ raw events export | |
| People | ✅ engage profiles | |
| Cohorts | ✅ cohorts, members | |
| Analytics | ❌ | funnels, retention, insights |
| Revenue | ❌ | transactions |
