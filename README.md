# What we found scanning 30 public payment integrations

We ran the Smartriarch PCI rule set against 30 public repositories that touch card
payments — processor SDKs, e-commerce platforms, billing applications, and payment
plugins. **20 of the 30 (67%) had at least one PCI violation.**

This report is aggregate only. No repository is named, and no per-repository row is
published, because most of these findings sit in code paths that never run in
production and a per-repo breakdown would be a disclosure decision rather than a
data-sharing one.

---

## Headline numbers

| Metric | Value |
|---|---|
| Repositories scanned | 30 |
| Repositories with ≥1 finding | 20 (67%) |
| Repositories with ≥1 error-severity finding | 16 (53%) |
| Repositories fully clean | 10 (33%) |
| Total findings | 548 |
| Error severity | 188 (34%) |
| Warning severity | 360 (66%) |
| Average findings per repository | 18.3 |
| Most common violation | Payment route served with no Content-Security-Policy header — **7 of 30 repositories** |

Scan date: 2026-08-05. A finding is a Semgrep match against the Smartriarch rule
set — 95 rule files covering Python, JavaScript, TypeScript, Ruby, PHP, Java and Go.
Severity is the rule's own `ERROR` / `WARNING` classification.

The average is not a typical repository. The distribution is heavily skewed: the
median repository has 3 findings, and a small number of large codebases account for
most of the 548. Read the 67% and 53% rates as the representative figures, not the
average.

---

## Most common violations

Counted by **number of repositories affected**, not by number of occurrences — a
rule that fires 40 times inside one repository counts once. Rules that affected
fewer than three repositories are omitted.

| Violation | Severity | Repos affected |
|---|---|---:|
| Payment route served with no Content-Security-Policy header | Warning | 7 |
| Raw card number in an Adyen API call (JavaScript) | Error | 6 |
| Raw card number in an Adyen payment request (Python) | Error | 4 |
| Payment API called over HTTP instead of HTTPS | Error | 4 |
| Hardcoded API secret in PHP source | Error | 4 |
| TLS certificate verification disabled | Error | 3 |
| Raw card number in a Worldpay order payload | Error | 3 |
| PAN + CVV assembled server-side | Error | 3 |
| Weak or deprecated cryptographic algorithm | Error | 3 |

Two things stand out.

**The most common finding is not a card-data leak.** It is a missing
Content-Security-Policy header on a payment page. PCI DSS 4.0 Requirement 6.4.3 made
script control on payment pages an explicit requirement, and a missing CSP header is
exactly the kind of gap that never surfaces in code review, because nothing breaks
when it is absent.

**Raw PAN handling is widespread.** Counting every rule that flags a raw card number
being assembled or forwarded, **12 of the 30 repositories** do it somewhere. That is
the single largest driver of SAQ scope: the difference between SAQ A and SAQ D is
mostly the question of whether a card number ever touches your own infrastructure.

---

## Violation rate by repository type

These breakdowns cover **27 of the 30 repositories**. Three are excluded — see
[Method and limitations](#method-and-limitations) for why.

Rates only. Finding volumes are deliberately not broken down by group.

| Category | Repos | With ≥1 finding | Rate |
|---|---:|---:|---:|
| Processor SDKs and client libraries | 12 | 6 | 50% |
| Payment-accepting applications and platforms | 15 | 11 | 73% |
| **Total** | **27** | **17** | **63%** |

Applications that accept payments fail at a meaningfully higher rate than the SDKs
they build on, which is the expected direction: an SDK demonstrates the correct
pattern, and the integration is where it gets bent.

## Violation rate by language

| Detected stack | Repos | With ≥1 finding | Rate |
|---|---:|---:|---:|
| JavaScript / TypeScript, no Python present | 19 | 12 | 63% |
| Python, with or without JavaScript | 8 | 5 | 63% |
| **Total** | **27** | **17** | **63%** |

Language is not predictive. Both groups land within a point of each other and within
a point of the overall rate. Payment-integration mistakes are a function of the
integration, not the runtime.

---

## Method and limitations

**How the scan ran.** Each repository was cloned at its default branch and scanned in
a single pass with Semgrep against the Smartriarch rule set, using `--jobs 1`.
Single-worker mode is not optional here: with default parallelism, Semgrep silently
dropped findings between runs over the same tree, while reporting zero errors, zero
skipped paths and zero timeouts. Every figure above comes from a re-scan on
2026-08-05 after that fix, and is reproducible run to run.

**Why three repositories are excluded from the breakdowns.** The two grouped tables
cover 27 of the 30. Three repositories are held out because their finding volume or
their category is distinctive enough that the resulting group cells would function as
single-repository disclosures — one of them alone accounted for over 90% of its
category's findings. Excluding them keeps every published cell at eight repositories
or more. The headline numbers and the violation-frequency table cover all 30; the two
grouped tables cover 27, so they are not expected to reconcile with each other. No
group in this report contains fewer than eight repositories, and no finding counts
are published per group.

**What a finding is and is not.** A finding is a static pattern match. It is not a
confirmed breach, not an exploited vulnerability, and not proof that a project is
non-compliant. A substantial share of these matches are in test fixtures, example
code, and integration-test helpers that never run in production — expected in SDK
repositories in particular, where demonstrating a raw-card request is part of the
documentation. Treat these numbers as a measure of how often risky payment patterns
appear in code, not as a compliance verdict on any project.

**Known under-reporting.** 25 of the rule set's 34 identifier-matching filters are
start-anchored, so `card_number` matches but `billing_card_number` does not. These
are pure false negatives: the real counts are higher than what is reported here,
never lower. The 67% and 53% rates are floors.

**Category and language labels.** Category is assigned by hand; the scanner does not
infer repository type. The language label is the scanner's dominant-language
heuristic, which counts only `.py`, `.js`, `.ts`, `.mjs` and `.cjs` files, so a
PHP-first or Ruby-first repository that ships JavaScript assets is labelled
JavaScript. The rules themselves cover all seven supported languages regardless of
that label, and PHP rules did fire inside repositories the heuristic called
JavaScript.

**Sample.** 30 repositories is a small, non-random sample, selected for being public,
payment-adjacent, and reasonably well known. It is not a representative survey of
production payment code, and the rates here should not be extrapolated to the wider
ecosystem.

---

*Smartriarch is not a Qualified Security Assessor and this is not a compliance
certification. Automated scanning finds patterns in code; it does not assess your
cardholder data environment, your policies, or your controls. Use these results as
engineering input, and confirm your PCI DSS status with a QSA.*
