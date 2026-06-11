# ServiceNow ITOM — Common Issues, Pitfalls & Troubleshooting

---

## Issue 1: MID Server Work Queue Backlog — Discovery Stalling

**Category:** Performance / MID Server
**Severity:** Critical — Discovery runs fall behind, CMDB data becomes stale

**The Problem:**
Discovery is scheduled to run nightly and complete in 4 hours. After several months, the same Discovery run takes 18+ hours and is still running when the next scheduled run starts. The ECC Queue shows 8,000+ unprocessed records. CMDB is updating 24-48 hours behind reality.

**Why It Happens:**
The infrastructure scope has grown (more devices added to discovery ranges) but the MID Server configuration has not been updated to match the increased load. Default worker thread count (25) is insufficient for the volume.

**Root Cause:**
MID Server worker thread count not scaled with infrastructure growth.

**Fix / Prevention:**
- Increase MID Server worker threads: edit `agent/properties/glide.properties`, set `mid.probe.ecc_workers=75` (or higher for large environments)
- Split large Discovery Schedules into multiple smaller schedules with staggered start times
- Add a second MID Server for high-density network segments
- Monitor Work Queue depth after each Discovery run — set an alert if depth exceeds 1,000 unprocessed records at run completion

---

## Issue 2: Duplicate CI Records After Discovery — IRE Not Matching Existing Records

**Category:** Data Quality / IRE
**Severity:** Critical — CMDB integrity compromised, downstream impacts on all modules

**The Problem:**
After enabling Discovery across a new network segment, the CMDB fills with duplicate CI records. A server with hostname PROD-APP-01 now has 3 CI records — one from last week's scan, one from today, and one from a manual entry.

**Why It Happens:**
IRE identification rules are using IP address or hostname as the primary identifier. Virtual machines and servers with DHCP assignments change IPs frequently; hostnames get reused after decommission. Discovery creates a new record instead of finding the existing one.

**Root Cause:**
Unstable identifiers in IRE rules cause missed matches on existing CI records.

**Fix / Prevention:**
- Update IRE identification rules to use stable attributes: Serial Number + MAC Address for physical hardware, VM UUID for virtual machines
- For servers without hardware serial numbers accessible via SSH/WMI, use hostname + domain as a compound identifier
- Run CMDB Deduplication tool to merge existing duplicate records after fixing identification rules
- Monitor the new-vs-updated CI ratio after each Discovery run — a spike in new records signals IRE matching failures

---

## Issue 3: Event Management Alert Storm — 10,000+ Events per Day

**Category:** Configuration / Event Management
**Severity:** Critical — SOC team overwhelmed, real incidents buried in noise

**The Problem:**
After connecting Splunk to Event Management, 12,000 events flood in within the first 24 hours. The alert console is unusable. The Event Management-to-Incident creation rule creates 200+ incidents before anyone can intervene.

**Why It Happens:**
The Splunk integration was configured to forward all alerts regardless of severity. Informational, debug, and low-severity Splunk alerts that were never intended to create ServiceNow incidents are being forwarded.

**Root Cause:**
No severity filtering at the Splunk source — all alerts forwarded indiscriminately.

**Fix / Prevention:**
- Apply source-side filtering in Splunk: only forward alerts at "High" or "Critical" severity to ServiceNow
- Configure Alert Rules in Event Management: map severity levels and set deduplication windows (same alert + same source within 1 hour = update existing alert, not new record)
- Disable automatic Incident creation until alert volume is tuned to an acceptable baseline
- Iteratively tune: start with only Critical alerts creating incidents, add High after validating signal quality

---

## Issue 4: CI Binding Failures — Alerts Not Linking to CMDB CIs

**Category:** Configuration / Event Management
**Severity:** High — alerts have no business context, triage is manual and slow

**The Problem:**
35% of alerts in Event Management show "Unbound" status — they are not linked to any CI in the CMDB. SOC analysts have to manually search for the affected system in every third alert, significantly slowing triage.

**Why It Happens:**
CI Binding rules use the hostname from the alert's `node` field to match against the CI `name` field in CMDB. But alerts from Splunk use FQDNs (prod-db-01.company.com) while CMDB stores short hostnames (prod-db-01). No match is found.

**Root Cause:**
Hostname format mismatch between alert payload and CMDB CI records.

**Fix / Prevention:**
- Add a secondary CI binding rule that strips the domain from the FQDN and tries the short hostname match
- Add a tertiary rule that tries IP address matching (alert `ip_address` field → CMDB `ip_address` field)
- Work with the Splunk team to standardize the `host` field format in alert payloads to match CMDB naming conventions
- After improving binding rules, measure CI binding success rate weekly — target above 90%

---

## Issue 5: Service Mapping Not Discovering All Application Tiers

**Category:** Configuration / Service Mapping
**Severity:** High — incomplete service maps, unreliable impact analysis

**The Problem:**
Service Mapping is configured for a 3-tier web application (load balancer → app server → database). The map only shows the load balancer and app servers — the database tier is not being discovered. Impact analysis treats the application as having no database dependency.

**Why It Happens:**
Service Mapping traces connections by following network connections from the entry point. If the app server connects to the database using a JDBC connection pool that was established at application startup (rather than a live connection at discovery time), Service Mapping may not see the active connection and will not trace to the database.

**Root Cause:**
Connection tracing requires active connections at discovery time, or pattern-based discovery configured to explicitly query the application's configuration for database connection strings.

**Fix / Prevention:**
- Use Patterns for the application tier: configure a pattern that reads the application's configuration file (jdbc.properties, web.xml) to extract database connection details — does not depend on active connections
- Schedule Service Mapping runs during times when the application is actively processing requests (not overnight maintenance windows)
- Manually add the database CI relationship as a baseline and configure Service Mapping to validate/update it rather than discover from scratch

---

## Issue 6: Discovery Credentials Expiring — Silent Failures in CMDB Updates

**Category:** Operations / Credential Management
**Severity:** Medium-High — CMDB data becomes stale without any obvious error

**The Problem:**
Windows server CI records in CMDB stop updating. RAM, OS version, and patch level are all months out of date. No alerts are firing about Discovery failures. The issue is only discovered during an audit.

**Why It Happens:**
The WMI service account password expired. Discovery probes are failing silently — the ECC Queue shows the probes as "Processed" but with no data (the WMI connection fails but does not generate a loud alert). CI records are not updated but no one notices because there is no monitoring on CI update frequency.

**Root Cause:**
Credential expiry with no monitoring on probe success rates or CI staleness.

**Fix / Prevention:**
- Set up a CI staleness monitor: alert when a CI in a critical category has not been updated by Discovery in more than 7 days
- Monitor probe success rate per credential: a sudden drop in WMI probe success rate signals a credential issue
- Use service accounts with non-expiring passwords for Discovery, OR set a calendar reminder 2 weeks before credential expiry
- Configure Discovery to send a notification when a scheduled run completes with more than 10% probe failures

---

## Issue 7: MID Server Upgrade Failing After Platform Upgrade

**Category:** Operations / Platform Upgrade
**Severity:** High — Discovery and integrations stop working post-upgrade

**The Problem:**
After upgrading the ServiceNow platform to a new release, the MID Server shows "Down" status and Discovery stops working. The MID Server upgrade that should have been triggered automatically did not complete.

**Why It Happens:**
MID Server auto-upgrade requires the MID Server service account to have write access to the MID Server installation directory. If this access was changed as part of a security hardening initiative, the auto-upgrade fails silently.

**Root Cause:**
Service account permission change preventing MID Server auto-upgrade.

**Fix / Prevention:**
- Add MID Server upgrade to the platform upgrade checklist: manually verify MID Server version matches platform version within 24 hours of platform upgrade
- Ensure the MID Server service account has write access to the installation directory
- In large environments, stage MID Server upgrades: upgrade one MID Server first, validate it is working, then upgrade the rest
- Subscribe to ServiceNow's MID Server compatibility matrix to know which MID Server version is required for each platform release

---

## Issue 8: ECC Queue Records Stuck in "Processing" State

**Category:** Operations / ECC Queue
**Severity:** High — Discovery and IntegrationHub requests blocked

**The Problem:**
ECC Queue records show status "Processing" for hours or days without completing. New Discovery runs cannot start because the queue is full of stuck records. IntegrationHub actions that route through the MID Server are also failing.

**Why It Happens:**
MID Server process crashed or was restarted mid-execution, leaving ECC Queue records in "Processing" state without a handler. The records are "owned" by the stopped MID Server process and cannot be picked up by the restarted process.

**Root Cause:**
Orphaned ECC Queue records from interrupted MID Server process.

**Fix / Prevention:**
- Run the "Reset ECC Queue" script (available in ServiceNow scripts): resets all "Processing" records for the affected MID Server back to "Ready" so they can be reprocessed
- Restart the MID Server and verify it reconnects successfully
- Add monitoring: alert if ECC Queue records remain in "Processing" state for more than 30 minutes

---

## Issue 9: Discovery Runs Exceeding Maintenance Window

**Category:** Performance / Scheduling
**Severity:** Medium — Discovery runs during business hours, causing network load

**The Problem:**
Discovery is scheduled for a 4-hour overnight window (10 PM to 2 AM). After network expansion, Discovery runs are consistently running until 8 AM, scanning production servers during peak business hours and generating noticeable network load complaints from application teams.

**Why It Happens:**
Discovery scope grew without adjusting MID Server capacity or Discovery scheduling strategy.

**Root Cause:**
Single Discovery Schedule covering all IP ranges sequentially, with insufficient MID Server capacity to complete within the window.

**Fix / Prevention:**
- Split the Discovery Schedule into multiple parallel schedules: separate schedules for each network segment or CI class, running concurrently on multiple MID Servers
- Prioritize: create a "Priority Tier" schedule for production servers running every night, and a "Full Scan" schedule for dev/test running weekly only
- Add MID Server capacity: a second MID Server doubles parallel probe capacity
- Set a Discovery run timeout: if Discovery has not completed within 5 hours, send an alert so the team can investigate rather than discovering the overrun at 9 AM

---

## Issue 10: Service Mapping Entry Point Not Starting — Application Not Discovered

**Category:** Configuration / Service Mapping
**Severity:** Medium — service topology map blank or incomplete

**The Problem:**
A Service Mapping run is triggered for a business service. After 30 minutes, the map shows only the entry point CI with no connections discovered. The application has a load balancer, 3 app servers, and 2 database servers that should all be visible.

**Why It Happens:**
Service Mapping entry point is configured with an incorrect IP or hostname — the entry point CI does not match any active listening service. Or the entry point host does not have an active connection to the app tier at the time of mapping.

**Root Cause:**
Entry point configuration does not match the actual running service, or the service has no active connections to trace at mapping time.

**Fix / Prevention:**
- Validate the entry point manually: SSH to the load balancer and confirm the configured port is actively listening
- Use `netstat` or equivalent to verify active connections exist between each tier before troubleshooting Service Mapping
- Check the Service Mapping task log for the specific error: "Connection refused," "No pattern matched," or "Credential failure" each point to a different root cause
- For applications with connection pooling (connections established at startup), use pattern-based discovery to read config files rather than relying on live connection tracing
