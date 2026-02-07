# Nuclei Template PR Research Report
## Date: 2026-02-07

## Phase 1: Quality Bar from Accepted PRs

Key observations from studying ~10 recently merged CVE template PRs in projectdiscovery/nuclei-templates:

### Required Fields (present in ALL accepted templates)
- `id` matching the CVE ID
- `info.name` format: `Product Name Version - Vulnerability Type`
- `info.author`
- `info.severity` (must match CVSS score)
- `info.description` (multi-line, detailed)
- `info.impact` (what an attacker can do)
- `info.remediation` (upgrade/fix advice)
- `info.reference` (NVD link + vendor advisory/PoC)
- `info.classification` with `cvss-metrics`, `cvss-score`, `cve-id`, `cwe-id`
- `info.metadata.verified: true`
- `info.metadata.max-request` (number of HTTP requests)
- `info.tags` (comma-separated, includes cve, cveYYYY, product, vuln-type)

### Common Optional Fields
- `epss-score` and `epss-percentile` in classification
- `vendor` and `product` in metadata
- `shodan-query`, `fofa-query`, `publicwww-query` in metadata
- `cpe` in classification

### Matcher Quality
- Exploitation matchers preferred over version detection
- Time-based SQLi uses `duration>=N` with `@timeout` annotation
- RCE uses `md5()` or unique random strings
- XSS reflects unique payloads and checks DOM context
- Multi-step exploits use `flow:` with `internal: true` matchers
- Always combine with status code or content-type checks

### PR Description
- Standard template with checkboxes for TP/FP validation
- Best PRs include lab setup instructions
- Some include `nuclei -debug` output (redacted)
- Reference links to advisories

### Algora Bounty Info
- $200 bounties for specific CVE template requests (labeled vKEV)
- 21 open bounties totaling $2,202
- Key: templates must be for CVEs in the Known Exploited Vulnerabilities catalog

---

## Phase 2: Existing Template Audit

### CVE-2026-25241 (PEAR pearweb SQLi) - READY TO PR
**Status:** Ready with minor fixes applied
**Changes made:**
- Added `max-request: 1`
- Added `@timeout: 15s` for time-based detection
- Added GitHub advisory reference (GHSA-63fv-vpq5-gv8p)
- Added `shodan-query`
- Improved secondary matcher (content-type check instead of broad status codes)

**PR Title:** `Add CVE-2026-25241 - PEAR pearweb Unauthenticated SQL Injection`
**PR Description:** Standard template with TP validation checkbox. Reference the GitHub advisory.

### CVE-2026-1499 (WP Duplicate File Upload) - READY TO PR (with caveat)
**Status:** Ready but template only demonstrates step 1 of a 2-step exploit chain
**Changes made:**
- Fixed reference URL (was pointing to wrong plugin)
- Added Wordfence and trac source code references
- Added `max-request: 1`
- Added `publicwww-query`
- Fixed CVSS to 9.9 (PR:L with scope change per Wordfence)

**Caveat:** The template proves the missing authorization on `process_add_site()` but does not chain the full file upload RCE. Reviewers may request the full chain.

**PR Title:** `Add CVE-2026-1499 - WordPress WP Duplicate Missing Authorization`
**PR Description:** Include lab setup with WP + local-sync plugin install. Note the two-step exploit chain.

### CVE-2025-15268 (Infility Global SQLi) - READY TO PR
**Status:** Ready with fixes applied
**Changes made:**
- Added `max-request: 1`
- Added `@timeout: 15s`
- Added Wordfence and trac references
- Added `publicwww-query`
- Added status code matcher
- Fixed vendor/product metadata

**PR Title:** `Add CVE-2025-15268 - WordPress Infility Global Unauthenticated SQL Injection`
**PR Description:** Standard template. Include WP lab setup with infility-global plugin.

### CVE-2020-37123 (Pinger RCE) - READY TO PR
**Status:** Ready, strongest template of the set
**Changes made:**
- Added `max-request: 1`
- Added exploit-db and vulncheck advisory references
- Added GitHub repo reference

**PR Title:** `Add CVE-2020-37123 - Pinger 1.0 Remote Code Execution`
**PR Description:** Uses command injection via ping parameter with md5 hash verification. Clean exploitation proof.

### CVE-2026-25234 (PEAR pearweb Category SQLi) - NEEDS WORK / LOW PRIORITY
**Status:** Deprioritized
**Issues:**
- CVSS 4.0 from GitHub CNA shows PR:L and VI:L (low severity, approximately CVSS 3.1 4.3)
- Our original template claimed 9.8 critical, which was incorrect
- Requires authenticated access to category manager
- Time-based only detection with no secondary validation
- Low severity = low interest from reviewers and bounty program

**Changes made:** Corrected severity to medium, fixed CVSS to 4.3, updated title to note "Authenticated"
**Recommendation:** Submit only after the other 4 are merged. Low bounty potential.

---

## Phase 3: New CVE Opportunities Found

### CVE-2025-15030 (WordPress User Profile Builder < 3.15.2 - Unauthenticated Password Reset)
- **Severity:** Critical (CVSS 9.8)
- **Status:** NOT in nuclei-templates repo
- **Why promising:** 100k+ active installs, unauthenticated, allows admin account takeover
- **Blocker:** Need to reverse-engineer the exact password reset flow (multi-step). WPScan has details behind paywall.

### CVE-2025-12166 (Simply Schedule Appointments <= 1.6.9.9 - Unauth SQLi)
- **Severity:** High (CVSS 7.5)
- **Status:** NOT in nuclei-templates repo
- **Why promising:** Unauthenticated blind SQLi via `order` and `append_where_sql` parameters
- **Blocker:** Need to identify the exact vulnerable REST API endpoint path. Plugin source code review needed.

### CVE-2025-14533 (ACF Extended <= 0.9.2.1 - Privilege Escalation)
- **Severity:** Critical (CVSS 9.8), 100k+ sites affected
- **Status:** NOT in nuclei-templates repo
- **Why promising:** Actively targeted in the wild, high profile (BleepingComputer coverage)
- **Blocker:** Requires a specific form configuration (Create User form with role field). Detection would be conditional.

### Recommendation
CVE-2025-12166 is the strongest candidate for a new template once the exact endpoint is identified. It follows the same pattern as many accepted WordPress SQLi templates (admin-ajax or REST API with time-based detection).

---

## Phase 5: Summary and PR Priority

### PR Submission Order (recommended)
1. **CVE-2020-37123** (Pinger RCE) - Cleanest template, clear exploitation proof
2. **CVE-2026-25241** (PEAR pearweb SQLi) - Strong unauthenticated SQLi with advisory
3. **CVE-2025-15268** (Infility Global SQLi) - Standard WordPress plugin SQLi pattern
4. **CVE-2026-1499** (WP Duplicate) - Good but may need reviewer feedback on exploit chain
5. **CVE-2026-25234** (PEAR category SQLi) - Low priority, low severity

### Templates NOT Ready
- CVE-2025-15030, CVE-2025-12166, CVE-2025-14533: Need more exploit research before writing templates
