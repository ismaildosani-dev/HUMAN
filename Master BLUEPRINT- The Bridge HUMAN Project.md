# The Bridge — Master Blueprint

*A last-mile welfare entitlement and outcome system for India.*
*Draft v1. Not legal advice. Every "MUST" below is a decision to be ratified, not a fact.*

---

# 0. The one-sentence definition

> **We tell a household what they are owed, help them get it, and measure whether it actually arrived — and we publish the gap.**

Everything that does not serve that sentence is scope creep. Read it before every roadmap meeting.

**What we are NOT** — write these on the wall:

- Not a beneficiary registry for the government
- Not a verification/deduplication service for NGOs
- Not a socio-economic scoring bureau
- Not a payments or fund-handling entity
- Not a research panel

Each of those is a business someone will offer to pay you to become. Each is the death of the first sentence.

---

# 1. WHY — theory of change, stated falsifiably

## 1.1 The problem, precisely

India's welfare failure is not primarily a *design* failure. The schemes exist and are, on paper, generous. The failure is **at three specific joints**:

| Joint | Failure | Who currently measures it |
|---|---|---|
| **Awareness** | The household doesn't know the scheme exists, or that they qualify | Nobody |
| **Access** | They know, but are blocked — missing document, wrong office, tout demanding money, 4 trips they can't afford | Nobody |
| **Arrival** | The department "sanctioned" it; the money never landed (unseeded account, name mismatch, dormant account, skim) | **Nobody** |

The third is the big one and it is invisible by construction: government dashboards measure *sanction*, not *receipt*. Nobody phones the beneficiary.

## 1.2 The theory of change

> IF a household is told precisely what they are entitled to, in their language, by someone they trust,
> AND is helped past the specific document or office that blocks them,
> AND someone follows up until the money is in hand,
> THEN receipt rates rise materially against the counterfactual.
>
> AND IF the resulting gap data (identified → applied → sanctioned → received) is published,
> THEN administrative and political pressure improves delivery for people we never touched — which is the only path to scale that isn't a fantasy.

## 1.3 Falsifiable hypotheses — test these, in this order

| # | Hypothesis | Test | Kill condition |
|---|---|---|---|
| H1 | Households don't know what they're entitled to | Baseline survey, 200 HH | >70% already know → the problem isn't awareness; pivot |
| H2 | Knowing isn't enough; the block is documents/offices | Track informed-but-didn't-apply | If most apply unaided, you're not needed |
| H3 | Assisted households receive more than unassisted | **Randomised**: assist 150, waitlist 150, compare receipt at 6 months | No significant difference → **stop. The product does not work.** |
| H4 | Sanctioned≠Received gap is large and measurable | Phone every sanctioned case | Gap <5% → your best asset doesn't exist |
| H5 | Published gaps change administrative behaviour | Publish district data; watch a comparable district | No change → you're a service, not infrastructure. Fine, but say so. |

**H3 is the whole company.** Run it as a real randomised waitlist — it's ethical (you can't serve everyone anyway; waitlist by lottery), it's cheap, and it's the only way you or anyone else will ever know if this works. Almost nobody in this sector does it. That alone will differentiate you.

---

# 2. THE THREAT MODEL — how this becomes a weapon

Do this before architecture, because it constrains architecture.

## 2.1 The asset you are creating

A ranked, address-level, searchable database of India's poorest households, indexed by caste, religion, disability, pregnancy, income, and benefits received.

## 2.2 Adversaries — ranked by likelihood, not drama

| # | Adversary | What they want | Likelihood | Severity |
|---|---|---|---|---|
| 1 | **You, in 3 years, needing revenue** | Score for lenders; sell aggregate segments; take the govt bulk-data contract | **Near certain** | Fatal |
| 2 | **Your own field moderator** | ₹200 per form; the power to rate | **Near certain** | Severe, corrosive |
| 3 | **A political actor** | Ward-level list of who got what, before an election | High | Fatal to legitimacy |
| 4 | **A department under pressure** | Bulk list to "purge duplicates" — i.e. to remove people | High | Direct harm: exclusion |
| 5 | **An acquirer / creditor** | The database is the only asset | Moderate, rises with time | Fatal |
| 6 | **An external attacker** | Sell it, or target a community | Moderate | Catastrophic for individuals |
| 7 | **A hostile state actor** | Targeting a minority by religion + address | Low today. **Not low over 20 years.** | Catastrophic |

Note that the top two adversaries are *you and your staff*. Most security thinking gets this backwards.

## 2.3 The design rules that fall out of it

Each of these is a *structural* answer, not a policy answer:

1. **No national identified database exists.** Data stays in the district node. The centre holds rules, aggregates, and coverage maps — no persons. *Answer to "give us the national list": there isn't one.*
2. **No Aadhaar numbers, no biometrics, no dwelling GPS.** (Block/village LGD code only.)
3. **No consolidated "benefits received" ledger.** Store the *gap* (what they're owed and haven't got), not the *holdings*.
4. **No composite score on a person.** Ever. A needs *vector*, never a needs *rating*.
5. **Religion is not collected** unless a specific scheme's eligibility turns on it — and then only for that check, with the reason stated aloud. **Never stored as a searchable profile attribute.**
6. **Moderators collect facts; they do not assign ratings.** Fact collection is auditable; judgement is not.
7. **The database is owned by a trust, not the company.** Auto-destructs on change of control.
8. **Every refusal is published.** The refusal log is the only externally-verifiable proof that any of the above is real.

---

# 3. WHAT — scope, in and out

## 3.1 The four things you build

| Module | Purpose | Public good? |
|---|---|---|
| **Rules Engine** | What is this household entitled to? | **Yes — open source it** |
| **Org Registry** | Who can actually help them, in this block, this month? | Yes — open the schema; the liveness data is yours to steward |
| **Case & Referral System** | Move the household from "identified" to "money in hand" | Core product |
| **Outcome Ledger** | Did it actually arrive? Publish the gap. | **Yes — this is the public good that justifies you** |

## 3.2 Explicitly out of scope, v1

- Money movement of any kind
- Identity verification as a service to others
- Deduplication across districts
- Any predictive model on individuals
- Multi-state operation
- A consumer app that works without a human (build it, but it is not the product)

---

# 4. SYSTEM — architecture

## 4.1 Federated, not central

```
                    ┌──────────────────────────────────┐
                    │  CENTRE  (holds no person)       │
                    │  • Rules engine + scheme catalog │
                    │  • Org registry schema           │
                    │  • Aggregates, coverage maps     │
                    │  • Outcome statistics (k-anon)   │
                    └───────────▲──────────────────────┘
                        rules ↓ │ ↑ aggregates only
              ┌─────────────────┼─────────────────┐
              │                 │                 │
      ┌───────▼──────┐  ┌───────▼──────┐  ┌───────▼──────┐
      │ DISTRICT NODE│  │ DISTRICT NODE│  │ DISTRICT NODE│
      │ • households │  │              │  │              │
      │ • cases      │  │  identified  │  │              │
      │ • consent    │  │  data STAYS  │  │              │
      │ • audit log  │  │  here        │  │              │
      └──────────────┘  └──────────────┘  └──────────────┘
```

- **District node** holds identified data, encrypted, with its own key. A breach of one node exposes one district — not a nation.
- **Centre** pulls **only counts**, k-anonymised (k≥25), never records.
- **Rules push down; statistics push up. Persons never move.**
- Node keys are held by the **Data Trust**, not the company.

This makes the architecture match the "distributed methodology" instinct you already had, and it converts the single most dangerous property of the system (national aggregation) into an impossibility rather than a promise.

## 4.2 Layers

| Layer | Contents |
|---|---|
| **L1 Facts** | Canonical fact dictionary. Rules may reference nothing else. Every fact carries `{value, source, confidence, as_of}`. |
| **L2 Rules** | Versioned scheme rulesets. Three-valued logic (TRUE/FALSE/**UNKNOWN**). UNKNOWN never collapses to FALSE. |
| **L3 Needs** | The output: an unmet-entitlement vector + blocking documents + next actions. **Not a score.** |
| **L4 Orgs** | Live, capacity-aware, block-level referral registry. Liveness decays; below floor → hard block. |
| **L5 Cases** | State machine from IDENTIFIED to RECEIVED. |
| **L6 Outcomes** | Funnel, sanctioned→received gap, rejection taxonomy, divergence detection. |
| **L7 Policy** | Purpose binding, RBAC, consent, retention — enforced at the data-access boundary, in code. |
| **L8 Publish** | k-anonymised open data + refusal log + transparency report. |

## 4.3 Identity — resolve the contradiction

**Decision: no national identity resolution.**

- Within a district: fuzzy household matching on name + village + composition. Good enough for routing. Errors are tolerable because the consequence of a false match is a duplicate outreach call, not a denial.
- **Deduplication is explicitly NOT a product.** Dedup serves the funder and the department, not the household. It is exclusion technology. If a department wants dedup, they have Aadhaar and the legal authority to use it. That is their job, not yours.
- Verification of a *fact* (income, disability %) is done by **sighting the document**, or by a **consented, one-record-at-a-time pull** from a government API. **Never a bulk mirror.**

---

# 5. STRUCTURE — entity and people

## 5.1 Entity

- **Section 8 company** (non-profit) as the operating entity.
- **Data Trust** (separate) owns the beneficiary data. Company is a *processor* under licence. Licence auto-terminates on change of control, insolvency, or constitutional breach → data transferred to an equally-bound successor or **destroyed within 30 days**.
- **Golden share** with veto over constitutional amendment and change of control, held by an external civil-society body. **Not you. Not your lead funder.**

## 5.2 Roles

| Role | Does | Explicitly cannot |
|---|---|---|
| **Moderator** (field) | Collect facts, explain entitlements, accompany to office, follow up | Assign ratings. Handle money. Decide who is "genuine". |
| **Caseworker** | Complex cases, crisis referrals, T2/T3 data | Bulk export |
| **Curator** (policy) | Encode scheme rules from the GR; maintain golden fixtures | Ship a rule without review + fixtures |
| **Verifier** (audit) | Phone 10% of households independently | Report to the field manager (reports to Ethics Board) |
| **Analyst** | Aggregate warehouse only | Reach an identified person. **No path exists.** |
| **Ethics Board** | Veto over enumerated list | Be advisory |

## 5.3 The moderator problem — the single highest risk in the plan

You are recruiting a locally-embedded person with discretionary influence over whether money reaches a family. **That is the patwari.** Structure accordingly:

- **Facts, not ratings.** They record "no disability certificate"; they never record "seems needy."
- **Never posted to their own village.** Near enough to travel; far enough to have no stake.
- **Rotate blocks every 6–12 months.** Rent extraction requires a stable relationship.
- **Two-person rule** for consequential assessments (destitution, disability).
- **Fixed salary. Never per-case, never per-conversion.** The moment income depends on a case closing, you have built a rent-extraction incentive into the last mile.
- **Independent verifier** phones 10% of households: *"Did anyone ask you for money — including our person?"*
- **One substantiated bribe = termination.** In the contract, before hiring. No discretion.
- **Any citizen can use the system without a moderator, and can bypass one they distrust.** The moderator is a facilitator, never a gatekeeper.
- **Publish the moderator-bribe rate quarterly, including when it isn't zero.**

If you cannot fund the verifier team, you cannot fund the moderator network. They are one system.

---

# 6. FRAMEWORK — needs, not ratings

## 6.1 The needs vector

**Output of the rules engine for a household:**

```
UNMET ENTITLEMENTS   : [scheme, verdict, confidence, why]
BLOCKING DOCUMENTS   : [income certificate, DBT seeding]
NEXT ACTIONS         : [get UDID — unlocks 3 more]
RISK FLAGS           : [child out of school; pregnancy unregistered]
CONFIDENCE           : derived from FACT SOURCE, never from a model
```

**Not:** `need_score: 72`.

## 6.2 Why no score — the argument, since you will be pushed on this

1. A score is a **ranking**, and a ranking under scarcity becomes a **queue**, and a queue becomes a **denial** ("your score is only 62").
2. A score assigned by a **local human with discretion** has a **price**.
3. A score is **portable and legible** — which is exactly what makes it attractive to a lender, an insurer, or a party.
4. A score **destroys the evidence**. "Unmet: PMMVY, blocked on bank account" is actionable. "72" is not.
5. Scores are how welfare systems learn to say no efficiently. You exist to make them say yes.

**Where ranking is legitimate:** your own outreach queue. Rank *your* work, internally, explainably. Never expose a number attached to a person, and never let any external party gate on it.

## 6.3 Confidence, not certainty

- `ELIGIBLE` only when every consulted fact is document- or API-verified.
- Otherwise `LIKELY_ELIGIBLE`.
- Missing fact → `NEEDS_INFO`, **never** `INELIGIBLE`.

A false promise costs more trust than a cautious "likely", and trust is the only asset the last mile has.

---

# 7. WORKFLOW & LIFECYCLE

## 7.1 Household journey

```
DISCOVER → CONSENT → PROFILE → MATCH → PLAN → ASSIST → APPLY
   → TRACK → RECEIVE → RE-REVIEW (annually / on life event)
```

**Consent** (before any fact is collected): six lines, aloud, ~60 seconds, in her language, audio-recorded with permission. Including the line everyone omits:

> *"You can say no. Saying no will not reduce any government entitlement you already have."*

Without that line, consent given to a stranger who appears to control access to benefits is not freely given, and so is not consent.

**Profile:** question planner asks the highest-information-gain question first (`schemes_blocked / cost_to_collect`). Sensitive facts (caste, pregnancy, disability) are asked **last**, and only when a candidate scheme genuinely turns on them — with the reason stated: *"I'm asking about caste only because the pre-matric scholarship requires it."*

**Plan:** conflict-resolved best legal **bundle**, not a list. Gateway items first (UDID pays ₹0 but unlocks three things).

**Assist:** the moderator's actual job — the document, the office, the trip, the follow-up.

**Track:** phone calls at day 7, 30, 60, 90, 180. Four questions, every time:
1. Did you go? (If not — why not? *This is where abandonment hides.*)
2. What happened at the counter?
3. Did the money come? How much, when? (**Never trust a portal. Ask her.**)
4. **Did anyone ask you for money — at the office, or from us?**

## 7.2 Case lifecycle

`IDENTIFIED → INFORMED → [DOCS_PENDING] → APPLIED → UNDER_PROCESS → SANCTIONED → RECEIVED`
Terminal-bad: `REJECTED`, `ABANDONED`, `LOST_CONTACT`.

`DOCS_PENDING` is a **branch, not a stage** — people stuck there are *serviceable*, not lost. Route to a documentation-assistance org and reopen.

**`SANCTIONED → RECEIVED` is the edge nobody else measures.** It is your single most valuable output.

## 7.3 The learning loop

`applied → rejected → why` — captured from **both sides**: the department's stated reason, and her account of what happened at the counter. The gap between them is where the corruption lives.

Rejections cluster into five buckets, each routed differently:

- `OUR_FAULT` → fix the rule, add a golden fixture, **publish the rate**
- `DOCUMENT_GAP` → we should have warned her before sending her; fix the flow
- `ADMINISTRATIVE` → aggregate; 200 cases is a policy finding
- `DISCRETIONARY` → **escalate; DO NOT ENCODE**
- `GENUINELY_INELIGIBLE` → close honestly; tell her what she *is* owed

**The `DISCRETIONARY` rule is the one that defines you.** If an office demands a document the Government Resolution doesn't require, the tempting fix is to add it to your eligibility rules — rejection rate falls, dashboard improves. **Don't.** Doing so destroys your ability to tell a citizen *"you were wrongly denied"*, and quietly launders an illegal demand into your product. Keep `rule_as_written` and `rule_as_practised` as permanently separate fields.

---

# 8. FUNCTIONALITY — what actually gets built

**v1 (district pilot):**
- Moderator Android app: offline-first, consent capture (audio), fact entry, case list, follow-up prompts
- Rules engine: ~50 hand-curated schemes, three-valued, explainable, versioned
- Org registry: ~35 verified orgs, block-level, capacity-aware, liveness-decaying
- Referral: hard filters (capacity, block, need, female-staff-for-GBV, red flags), then ranking
- Case tracker + call scheduler
- Outcome ledger + coverage map
- Curator console (policy people edit rules; engineers don't)
- **Beneficiary read-back screen** — the moderator can read her full record aloud, in her language, on demand. **Build this in week one, not year two.** It is her DPDP access right, and a web form is a wall, not access.

**Deliberately NOT in v1:** consumer app, chatbot, dashboards for departments, any ML.

---

# 9. SOURCING

## 9.1 Schemes

Ground truth is the **Government Resolution / notification**, not the department website (stale) and not a scrape (wrong at scale).

- ~50 schemes, hand-curated, each with: GR number, source URL, `last_verified`, owner, review cadence, and **5–15 golden test fixtures**.
- Overdue review → confidence auto-degrades to `stale` → UI warns before anyone applies.
- **Do not scrape 2,000.** A large wrong database means telling a widow she qualifies for something she doesn't.

## 9.2 Organisations

Harvest (free): NGO Darpan · **MCA National CSR Portal** (best free liveness signal — money moved = probably alive) · FCRA returns · 12A/80G lists · state Charity Commissioner · **District Collector's NGO panel** (small list, very high hit rate — get this first) · statutory bodies (DLSA, DCPU, One-Stop Centre, DDRC — these appear in no NGO database and are often the only live option in a block).

Then **call 150, visit 40.** The four questions that matter:
1. *"If a 34-year-old man with a locomotor disability walked in tomorrow, what would you actually do for him?"* (Never "do you work on disability?" — everyone says yes.)
2. *"Which talukas did your field staff physically visit last month?"*
3. **"How many new cases could you take this month? … How many did you take last month?"** The gap between those two numbers tells you whether the org is real. **Nobody asks this, which is why every existing NGO directory is fiction.**
4. *"Who else in this taluka actually does this work well?"* — snowball referrals beat Darpan by a wide margin.

## 9.3 Moderators

Recruit through existing trusted networks (SHGs, SEWA-type federations, ASHA/Anganwadi alumni) rather than cold. Prefer **women** — access to households, and materially lower bribe rates in comparable programmes. Never post to home village. Fixed salary.

## 9.4 Funding

| Source | Pulls you toward | Guardrail |
|---|---|---|
| Foundations | Rigour, slow money, mission | Best fit. Start here. |
| CSR | Their districts, their NGOs, their metrics | **Funder identity must not exist as a field the matcher can read.** External routing audit annually. |
| Government SaaS | Becoming the state's targeting tool | **Sell them the outcome layer, never the beneficiary database.** The sanctioned→received gap for their own department is genuinely valuable and requires zero names. |
| Beneficiary fees | — | **Never.** They cannot pay, and charging recreates the middleman. |

Founder-funded is not a model; it is a countdown. Budget the Ethics Board and the independent verifier team as **line items from day one** — you will never feel you can afford them later.

---

# 10. GUARDRAILS

## 10.1 Technical (enforced in code, at the data-access boundary)

- **Purpose binding.** Every read declares a purpose; the purpose gates the tiers. *If a developer can `SELECT *` and get a caste column back, you have a compliant document, not a compliant system.*
- **Write-without-read on sensitive data.** The moderator **enters** caste, disability, pregnancy; he can never read them back, list them, or export them. This kills the realistic insider threat.
- **Named-recipient consent.** Consent to *be helped* is never consent to send data to Org X. The org is named, every time.
- **Fail closed.** An unclassified field defaults to *sensitive*, not operational.
- **Denied fields are absent, not null** — otherwise "she has no caste certificate" is distinguishable from "you may not see her caste", and that distinction is itself a leak.
- **Audit log stores fact KEYS and verdicts, never VALUES.** An audit log of who-looked-at-whose-caste is itself a caste database.
- **Retention actually runs.** Cron; alert if it hasn't executed in 48h. *A retention rule that has never deleted anything is a claim, not a control.*
- **k-anonymity (k≥25) on everything published.** Two GBV cases in a block is not a statistic; it is two identifiable women.

## 10.2 Human

Fixed salaries · no home village · rotation · two-person rule · independent 10% beneficiary audit · bribe = termination · whistleblower channel routed to the **Ethics Board chair, not management**.

## 10.3 Structural

Section 8 · Data Trust · golden share · constitutional clause amendable only by 90% + Ethics Board consent + **90 days' public notice** (the notice period is the actual lock — anyone can write a prohibition; the question is how cheaply a future you can delete it).

## 10.4 Tripwires — pre-committed, automatic

| Trigger | Automatic response |
|---|---|
| Any bribe report naming our own staff | Suspend within 24h; Ethics Board notified; **published that quarter** |
| Bulk data request from any govt/political/commercial actor | Auto-escalate to Ethics Board; **logged for publication regardless of outcome** |
| `our_fault_rate` > 15% for a quarter | Freeze new scheme onboarding; full ruleset audit |
| Received rate < 15% for 3 quarters | **Wind-down review.** You are consuming trust and returning nothing. |
| Retention job not run in 48h | Page the engineer; block new data collection until fixed |
| A moderator's cases show anomalous rating patterns | *(N/A — moderators don't rate. That's the point.)* |

---

# 11. FAIR DELIVERY

## 11.1 Exclusion error is the sin; measure it

Every welfare system optimises against *inclusion* error (giving to the undeserving) because that's what auditors see. Nobody measures *exclusion* error (missing the deserving) because it is invisible by construction — the excluded don't complain, they just don't get anything.

**You must measure your own false-negative rate.** Method: take a random sample of households you screened out or never reached; have a caseworker assess them independently. Publish the number. Nobody in this sector does this. It is the single most credible thing you could publish.

## 11.2 Rationing, when you can't serve everyone

You will always have more need than moderator-hours. How you ration is an ethical decision, not an ops decision.

- **Default: lottery within a block**, not "most needy first". A "most needy first" queue requires a ranking, which requires a score, which we have banned — and which a local human would price.
- If prioritising, do it on **objective, non-discretionary facts** (household has a pregnant woman; child out of school), transparently published, never on a composite judgement.
- **Publish the rationing rule.** If you can't defend it publicly, don't use it.

## 11.3 Non-discrimination

- Coverage must be measured **by social group** — if SC/ST/Muslim households have lower receipt rates in your programme, that is your failure, not theirs.
- Publish receipt rate disaggregated by group. Deal with the discomfort.
- **A moderator's caseload composition is audited** against block demographics. Systematic avoidance of a hamlet is a red flag.

---

# 12. DATA INTEGRITY

- **Provenance on every fact**: `{value, source, confidence, as_of}`. Self-declared income and an income certificate are *different facts*, not the same fact at different quality.
- **Confidence is derived from source, never from a model.**
- **Rules validate at load time** against the canonical fact dictionary. A rule referencing an undefined fact raises on startup. This is the only thing that keeps the dictionary canonical as the catalogue grows.
- **Golden fixtures** per scheme, written by the *curator* (policy), not the engineer. CI runs the full suite on every rule edit. This catches *"someone changed the income cap and silently excluded 40,000 people."*
- **The empty-profile test**: an empty profile must return `NEEDS_INFO` for every scheme and `INELIGIBLE` for none. That single assertion is the guardrail against the failure mode that has excluded millions from PDS and pensions — a blank field read as "no."
- **Adversarial integrity**: assume some facts are false (a household overstating disability, a moderator inflating need to hit a target — which is why there are no targets). Never act on self-declared data as if verified; the verdict language must always carry the uncertainty.
- **Immutable audit trail** of every recommendation: what we said, under which ruleset version, on what facts. This is both your learning loop and your legal defence.

---

# 13. GOVERNANCE & STAKEHOLDER ACCOUNTABILITY

## 13.1 Decision rights

| Decision | Founders | Board | Ethics Board | Golden Share |
|---|---|---|---|---|
| Product, hiring, budget | **Decide** | Approve | — | — |
| New data field collected | Propose | Inform | **Veto** | — |
| Govt/commercial data MOU | Propose | Approve | **Veto** | — |
| Change to the refusal list | Propose | Approve | **Veto** | **Veto** |
| Constitutional amendment | Propose | 90% | **Veto** | **Veto** |
| Change of control | Propose | Approve | **Veto** | **Veto** |

Founders keep everything operational. The structure doesn't stop you running the company; it stops you doing the five things that would destroy the reason it exists.

## 13.2 Who is accountable for what

| Stakeholder | Accountable for | To whom | Consequence of failure |
|---|---|---|---|
| **Us** | Rule accuracy, honest metrics, moderator conduct | Beneficiaries, Ethics Board, public | Published `our_fault_rate`; wind-down triggers |
| **Moderator** | Facts, not judgements; no money | Verifier team, Ethics Board | Termination |
| **Partner NGO** | Accepting referrals they said they'd accept; reporting outcomes | Us + the beneficiary | Delisting; published acceptance/resolution rates |
| **Government dept** | Sanctioning and *delivering* | Public — via **our published sanctioned→received gap** | The gap is the accountability instrument |
| **Funder** | Not touching routing | Ethics Board, external audit | Published if they try |
| **Ethics Board** | Using its veto | Public — **it may publish dissent** | A board that cannot dissent publicly is captured |

## 13.3 The beneficiary's rights, made real

DPDP gives rights; a web form is a wall. For this population:

- **Access** = a moderator reads her full record back to her, aloud, in her language, on request.
- **Erasure** = works from a phone call to a helpline. No email, no form, no login.
- **Grievance** = a named human and a phone number, printed on the paper slip left at every household. Not a `support@` address.
- **Withdrawal** stops further processing — it does **not** erase her record of benefits already received, which would harm her.

---

# 14. REPORTING, AUDIT & PUBLISHING

## 14.1 The quarterly transparency report — fixed date, committed in advance

*(so a bad quarter cannot become a skipped quarter)*

| Published | Why it's there |
|---|---|
| **Received rate** (in hand / identified) | The only metric that matters |
| **Sanctioned→received gap**, by department | The number nobody else has |
| **Our-fault rejection rate** | A team reporting 0% is not accurate — it is not looking |
| **Bribe reports, incl. our own staff** | If you won't publish it, you already know the answer |
| **Estimated exclusion error** | The sin nobody measures |
| **Receipt rate disaggregated by social group** | Fairness, evidenced |
| **Data requests refused, and from whom (category)** | The only externally-verifiable proof your locks are real |
| **Ethics Board decisions, incl. dissents** | Proof the board isn't decorative |
| **Coverage gaps by block** | The embarrassing map. Publish it anyway. |
| **Erasure requests received/fulfilled, median days** | Proof deletion actually runs |

**Banned metrics:** "lives touched", "profiles created", "schemes surfaced", "people made aware", "app downloads". Every one can be grown to infinity while helping nobody. If a funder insists on them, that tells you what kind of funder they are.

## 14.2 Open data — publish the map, protect the person

**Publish:**
- Scheme rulesets + fixtures (**open source — nobody's competitive advantage is knowing the income cap is ₹21,000**)
- Org registry schema + verification protocol
- Coverage maps (block × need, k-anonymised)
- Outcome statistics (funnel, gaps, rejection taxonomy, k-anonymised)
- The refusal log
- Method, code, and your own failures

**Never publish, at any aggregation:** any cell < k=25 · anything disaggregated finely enough to identify (age × caste × block × disability) · crisis-category data at any granularity · named individuals or officers.

**Suppression must be automatic, not editorial.** If a human decides what's too small to publish, a human will eventually decide wrong under pressure.

## 14.3 Audit

| Audit | By whom | Cadence |
|---|---|---|
| Beneficiary conduct audit (bribes) | Internal verifier team, reporting to Ethics Board | Continuous, 10% sample |
| Rules accuracy | Curator peer review + CI fixtures | Every edit |
| Referral routing (funder capture) | **External** party | Annual |
| Data access log review | External DPO / auditor | Quarterly |
| Financial | Statutory auditor | Annual |
| Impact (H3, the randomised waitlist) | **External** research partner (IDinsight / J-PAL type) | Once, properly, early |

---

# 15. METRICS & KILL CRITERIA

**North star:** `received_rate = benefits in hand / households identified as eligible`

**Kill criteria — decided now, while it's cheap:**

- H3 (randomised waitlist) shows **no significant effect** → **stop.** The product does not work. This is the honourable outcome and almost nobody in this sector is willing to face it.
- Received rate < 15% for three consecutive quarters → wind-down review.
- Own-staff bribe rate rises across two consecutive quarters → suspend the moderator network until fixed.
- You cannot fund the verifier team → you cannot run the moderator network. Both stop.

---

# 16. ROADMAP

| Phase | Duration | Scope | Gate to pass |
|---|---|---|---|
| **0. Talk** | 6 wks | Haqdarshak, myScheme, SEWA/PRADAN, IT for Change, District Collector | Someone tells you this already exists and works → **join them instead** |
| **1. Encode** | 8 wks | 50 schemes + fixtures; harvest + call 150 orgs; visit 40 | Coverage map for 1 district |
| **2. Pilot** | 12 wks | 300 HH: **150 assisted, 150 randomised waitlist** | H3 result, honestly reported |
| **3. Publish** | 4 wks | First transparency report incl. sanctioned→received gap | The Collector takes a meeting |
| **4. Deepen** | 6 mo | Same district. Fix what broke. | Received rate stable; own-staff bribe rate measured |
| **5. Replicate** | 12 mo | 3 districts, **run by people who are not you** | If it only works when a founder is in the room, it does not scale |
| **6. State** | — | Only after 5 | — |

---

# 17. RISK REGISTER (top 10)

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| 1 | You become a scoring/targeting company | High | Fatal | Constitutional clause; Ethics Board veto; golden share |
| 2 | Moderator becomes the new middleman | High | Severe | Fixed pay, rotation, no home village, 10% audit, publish the rate |
| 3 | The intervention simply doesn't work | Moderate | Fatal | **Randomised waitlist, early.** Be willing to stop. |
| 4 | Govt bulk-data request | High | Fatal to legitimacy | Federated architecture — *there is no national list to give* |
| 5 | Funder captures referral routing | Moderate | Corrosive, invisible | Funder identity not a readable field; external annual audit |
| 6 | Rules go stale → wrong advice | High | Trust-destroying | Review cadence, staleness degradation, golden fixtures |
| 7 | Duplicating myScheme/Haqdarshak | High | Wasted years | **Phase 0. Talk to them before you build.** |
| 8 | Breach of a district node | Moderate | Severe (bounded) | Per-district keys, no Aadhaar, no GPS, minimal T2 |
| 9 | Exclusion error, invisible | **Certain** | Severe & silent | Measure it. Publish it. |
| 10 | Founder burnout / self-funding runs out | High | Terminal | Fund the Ethics Board and verifier from day one, or don't start |

---

# 18. OPEN QUESTIONS YOU MUST ANSWER

1. **Inclusion or targeting?** Pick one, in writing. Every hard tradeoff resolves differently depending on the answer, and refusing to pick means drifting toward whichever one pays.
2. **What is the "Nyab framework"?** Whatever the offline assessment methodology is, it is the most consequential design in the system — a person in a village will apply it to decide whether a family looks needy enough. It must be specified, tested for bias, and stripped of discretion.
3. **Are you willing to run the randomised waitlist**, and to stop if it fails?
4. **Are you willing to publish your own-staff bribe rate and your exclusion error?** If not, the anti-corruption claim is marketing.
5. **Who holds the golden share?** If the answer is comfortable, it's the wrong answer.

---

## The line to keep

> **You are not building a database of poor people. You are building a record of what the state owes them, and a receipt for whether it arrived.**

The first is an asset someone will eventually misuse. The second is a public good that makes misuse harder. They can look identical in a product spec. They are not the same thing, and the difference is decided by the constraints you accept before you have anything worth constraining.
