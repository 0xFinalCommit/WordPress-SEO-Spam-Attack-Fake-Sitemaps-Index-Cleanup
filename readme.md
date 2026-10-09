# Case Study: Cleaning a WordPress SEO Spam Malware Attack

**Site:** protechsecure.ca (Canadian security services website, WordPress)
**Service:** Malware investigation, cleanup and hardening
**Done by:** Mr.Morningstar


---

## TL;DR

A WordPress business website was silently hijacked by an **SEO spam (black-hat indexing) attack**. Attackers planted hidden code and **six giant fake sitemap files, ~293,000 lines each (roughly 290,000 spam URLs in total)**, inside `/wp-includes/`, pushing adult-content pages into Google's index under the client's domain. The site's layout also broke. We traced the infection to the core files, removed the malicious code and fake sitemaps, and secured the site.

| | |
|---|---|
| **Problem** | Thousands of adult-content URLs indexed on a security company's domain; layout broken |
| **Root cause** | Malicious indexing code + fake sitemaps injected into WordPress core folder |
| **Attack type** | SEO spam / doorway page injection |
| **Impact** | Brand/reputation risk, Google penalty risk, wasted server resources, broken design |

---

## 1. The Client & The Situation

The client runs a professional security-services business in Canada. A site like this depends entirely on **trust**. The owner noticed that the website layout had broken and something looked wrong, but there was no obvious defacement on the homepage. That is typical of SEO spam attacks: the hacker's content is shown mostly to **Googlebot**, not to normal visitors.

---

## 2. Investigation

### Step 1: Server traffic analysis (first red flag)

In the hosting panel's analytics, the last 7 days showed **8,063 requests, with 6,875 from the United States**, even though this is a Canadian local business. That traffic mix did not match the real audience.



### Step 2: Access log review

Opening the access logs showed a clear pattern: requests to strange URLs ending in `-Porn` (and similar), under paths like `/knowledge/`, `/general/`, `/hype/`, `/discover/`, `/newsfeed/`, `/insights/`. None of these pages belong on a security company's website.

Key observations:
- The user-agent was **Googlebot** (IPs in the `66.249.x.x` Google range). Google was actively **crawling the spam pages**.
- Response sizes were almost identical (~6,910 to 6,912 bytes), which shows pages generated from **one template**, a classic doorway-page pattern.
- Crawls came every few minutes, around the clock.


### Step 3: Searching the core files

Since no normal plugin or theme explains URLs like these, we inspected the WordPress core directory. Inside `/wp-includes/` we found **files that do not belong there**:

- `sitemap-discover.xml` (9.46 MiB)
- `sitemap-general.xml` (9.41 MiB)
- `sitemap-hype.xml` (9.27 MiB)
- `sitemap-insights.xml` (9.44 MiB)
- `sitemap-knowledge.xml` (9.52 MiB)
- `sitemap-newsfeed.xml` (9.45 MiB)
- `sitemap-index.xml` (982 B), the master file tying them together

`/wp-includes/` is for WordPress core PHP files. It should never hold sitemaps, and certainly not ~9 MB ones. Also note the `shortcodes.php` file modified recently, which was reviewed as part of the code inspection.



### Step 4: Analysing the fake sitemaps

The fake `sitemap-index.xml` pointed to six sub-sitemaps, all created on the same date (2026-01-18), and all hosted in `/wp-includes/` instead of a normal sitemap location.


Each sub-sitemap was enormous. `sitemap-discover.xml` runs to **over 293,000 lines**, with URLs set to `priority 0.8` and `changefreq weekly` to make Google crawl them aggressively.



`sitemap-hype.xml` showed the same structure and the same scale (~293,000 lines).



**Estimated scale:** each URL entry takes 6 lines of XML, so ~293,000 lines is roughly 49,000 URLs per sitemap. Across six sitemaps that is about **290,000 spam URLs** submitted for indexing on the client's domain.

### Step 5: WordPress user review

We also reviewed WordPress users to check for attacker persistence (a rogue admin is a common way attackers regain access after cleanup). Three accounts existed: two Administrators and one Editor. Email addresses are blurred in this report for privacy.


---

## 3. How the Attack Worked

1. **Entry point:** attacker gained write access (typical causes: outdated/vulnerable plugin or theme, weak or leaked credentials, compromised admin account).
2. **Injected indexing code** into core files so the site would respond to the fake spam URLs and serve templated adult-content pages.
3. **Generated massive fake sitemaps** in `/wp-includes/` and registered them so Google would discover and crawl them.
4. **Googlebot crawled the spam pages**, leading to indexing under the client's trusted domain, which is the real goal: borrowing the domain's authority for the attacker's own rankings.
5. **Side effects:** layout breakage, heavy server load, and risk of a Google "hacked site" flag or de-ranking.

---

## 4. Why This Is Dangerous for a Business

- **Reputation damage:** adult results showing under a security company's name.
- **SEO penalty:** Google can flag the site as hacked, drop rankings, or show a warning in search results.
- **Lost trust and leads** from potential clients who see these results.
- **Server strain:** thousands of bot hits and multi-MB files.
- **Persistence:** if only the visible symptoms are fixed, the infection returns.

---

## 5. Remediation

1. **Backup first:** full backup of files and database before changes (kept for forensics).
2. **Removed malicious code** from the core file(s) responsible for generating the spam pages.
3. **Deleted all six rogue sitemaps** and the fake sitemap index from `/wp-includes/`.
4. **Replaced WordPress core** with a fresh, clean copy of the same version (6.9.4) so no modified core file remains.
5. **Audited plugins, themes and uploads** for backdoors and unknown files.
6. **Reviewed users:** removed unknown accounts, reset all admin passwords.
7. **Rotated credentials:** hosting, FTP/SFTP, database and WordPress salts.
8. **Fixed the layout** broken by the infection.
9. **Search cleanup:** requested removal of spam URLs in Google Search Console, submitted a clean sitemap, and monitored index status.
10. **Updated** all plugins and themes to current versions.

---

## 6. Hardening Recommendations Given to the Client

- Keep WordPress core, themes and plugins updated; remove unused ones.
- Enable two-factor authentication for all administrators.
- Use strong, unique passwords and limit admin accounts.
- Disable file editing in the dashboard (`DISALLOW_FILE_EDIT`).
- Block PHP execution in uploads, and restrict writes to core directories.
- Use a firewall / malware scanner with file-integrity monitoring.
- Schedule regular off-site backups.
- Monitor Google Search Console for sudden jumps in indexed pages.

---

## 7. Results

- Spam URLs removed from the site and a clean sitemap submitted
- Googlebot crawling of spam paths stopped
- Layout restored and site stable
- Server load back to normal
- Search Console: indexed spam pages dropping / manual action avoided

---

## 8. Key Takeaways

- Symptoms like broken layout, odd traffic or odd search results can hide a **core-level** infection.
- **Access logs are gold:** the infection pattern was visible in minutes.
- Attackers hide malware in places site owners never check, like `/wp-includes/`.
- Real cleanup means finding the **root cause**, removing persistence, and hardening, not just deleting visible junk.

---


