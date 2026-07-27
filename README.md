Smart Circle International
The Business & The Data
An onboarding guide for new analysts and reporting team members
Draft v1 · July 2026 · Maintained by the Reporting & Insights team
 
How to use this document
This guide has two halves. Part I explains the business — what Smart Circle actually does, who the players are, and how money moves. Part II explains the data — the three core tables you will live in, what every important field means, how the tables join, and the traps that catch every new analyst. Read Part I before Part II: none of the data makes sense until the business does.
Worked examples run throughout, following the same fictional characters (Maria the ICD owner, Jake the field rep, Margaret the customer, Krystal the call center agent). All dollar figures in examples are illustrative, not real contract terms. Items marked ⚠ are believed correct but unverified — confirm them before relying on them in published numbers.
How to use this document	2
Part I — The Business	3
What Smart Circle does, in one paragraph	3
The five players	3
Follow the money: Maria and Jake	3
The two call-center workflows	4
One customer, end to end: Margaret	4
Part II — The Data	5
The systems map	5
Table 1 — The field lead table ("REBORN_CABINET")	5
Table 2 — The call log ("Prod_Call_Center_New")	5
Table 3 — The agent state log ("Call_Center_hours")	6
The status glossary — how ~150 statuses become metrics	6
How the tables join (the lead spine)	7
Metric definitions (and their denominators)	7
Known data quirks — the analyst trap list	7
Open questions a new joiner should ask early	8
Quick-reference scale card	8
Glossary of terms	8

 
Part I — The Business
What Smart Circle does, in one paragraph
Smart Circle International is a broker of face-to-face sales. Large consumer brands (telecom, water delivery, home improvement, pest control) want customers signed up in person — at Costco tables, inside Lowe's, door-to-door. Smart Circle holds the relationships with those brands and with the retailers, but it does not employ the salespeople. Instead, it contracts a nationwide network of small, independently owned sales companies — called ICDs — whose reps staff the retail tables. Smart Circle's own call center function then verifies or confirms what those reps sign up. Smart Circle earns the spread between what clients pay for a verified result and what it remits to the ICD network.
The five players
Player	Who they are	Employed / paid by
Clients	National brands: a telecom carrier, Primo (water), LeafGuard, Leaf Filter, Jacuzzi, Expo Home, PJ Fitzpatrick, Thompson Creek, pest-control brands, and others	Pay Smart Circle per verified result
Retailers	Venues where selling happens: Costco, Lowe's, Sam's Club, BJ's, Canadian Tire, Home Hardware, Giant Tiger (yes — Canada too)	Partner relationship, not payment chain
ICDs	Independent Corporate Directors — owner-operators of small sales companies (~64 active offices). Think franchise-like, though legally independent	Own their company; earn per verified lead/sale
Field reps	Salespeople at the tables (~680+ active). Recruited, trained, scheduled and paid by their ICD — not by Smart Circle	The ICD's company; heavily commission-based
Smart Circle corporate	Analysts (you), client account teams, ICD relations, finance, and the call-center function (outsourced vendor + oversight)	Smart Circle payroll

The single most common new-joiner confusion: neither the ICD owners nor the field reps are on Smart Circle's payroll. When a rep has a pay question, they call their ICD, not Smart Circle HR. Smart Circle's levers over the field force are data, standards, and payment — not direct management.
Follow the money: Maria and Jake
Maria owns "Maria Gonzalez Marketing LLC", an ICD office with eight reps. Her rep Jake works a table at Costco #482. Jake signs up a customer for a fiber internet plan. Smart Circle's call center verifies the sale. Then (illustrative numbers):
Step	Payment	Amount (example)
1	Client pays Smart Circle for the verified sale	$50
2	Smart Circle keeps its brokerage margin	~$10
3	Smart Circle remits to Maria's LLC	$40
4	Maria pays Jake his commission	~$20
5	Maria keeps the rest for rent, recruiting, and profit	~$20

Notice what gates the entire chain: verification. If the call center rejects Jake's lead, nobody below Smart Circle earns anything. This is why verification data is politically hot — every rejected lead is a rep's lost commission and an ICD's lost margin — and why your reporting must be statistically defensible, not just directionally interesting.
Industry context: this brokered model is the standard architecture of the outsourced face-to-face sales niche. Direct competitors run the same structure under different names — Cydcor's offices are "ICLs", Credico's are "ISOs". The vocabulary maps one-to-one.
The two call-center workflows
The call center does two structurally different jobs, and almost every metric must respect the difference:
Workflow A — TPV (Third-Party Verification): inbound, live, at the table
Used for telecom, water, and pest campaigns. The moment the customer agrees, the rep phones the call center while the customer is still standing there (a "warm transfer"; the call log even shows 3-way calls). The agent confirms identity and consent, and the sale is approved on the spot. The entire economics of TPV hinge on answer speed: a customer on hold at a Costco table walks away in minutes, and an abandoned TPV call is usually a lost sale.
Workflow B — Appointment confirmation: mixed inbound/outbound
Used for home-improvement clients (LeafGuard, Jacuzzi, etc.). The "product" is a confirmed in-home appointment. Agents verify the appointment, check expectations ("the estimate visit is free; both homeowners should be present"), chase no-answers with outbound dials and voicemails, and handle reschedules. Success here is a confirmed appointment that later "sits" — and the customer's story continues into the field funnel: Issued → Demoed → Sold, with real dollars attached (median sale ≈ $17k for these clients).
One customer, end to end: Margaret
Saturday 11:00 — Margaret stops at the LeafGuard booth in Costco. Jake signs her up for a free gutter estimate and enters her details in the field app. A lead record is born (Part II, Table 1). 11:05 — Jake calls the confirmation line; agent Krystal answers after 16 seconds, confirms Tuesday 2 PM, checks expectations. A call record is born (Table 2), and Krystal's time was already being tracked all morning (Table 3). Over the following weeks, Margaret's lead record fills in: Verified ✓ → appointment Issued to LeafGuard ✓ → consultant visits (Demoed ✓) → she buys ($16,900, Sold ✓). Or she cancels, and a cancel reason fills in instead. Three tables, one story — the customer ledger, the phone log, and the timeclock.
 
Part II — The Data
The systems map
Four systems matter. Convoso is the dialer/call-center platform — it generates the call log and the agent state log, both synced into Salesforce, which is where we query them. Field leads originate in client- or platform-specific capture systems (e.g., Centah in the Lowe's ecosystem) and land in the Salesforce lead object. Client outcome feeds (demoed/sold/sale amounts) flow back into the same lead object. ⚠ The exact feed mechanics and lags for client outcomes are unconfirmed — ask before publishing sold-rate trends for recent weeks.
Table 1 — The field lead table ("REBORN_CABINET")
Grain: one row per field lead (a customer signed up at a venue). ~850k rows spanning 2021→present. Despite the historical name, it covers the whole home-improvement portfolio: Leaf Filter (largest), LeafGuard, Jacuzzi, Expo Home, PJ Fitzpatrick, Thompson Creek, across Costco, Lowe's, Sam's Club and Canadian retailers.
The fields that matter most:
•	Lead_ID__c — the lead's identifier. Three formats coexist (numeric, "A-1218873", "2026-7-12_9461_xxx" = date + rep number + suffix) because different client systems mint IDs differently.
•	ICD_Owner / ICD_Rep (+ IDs) — who generated the lead. ~96% populated. The backbone of all rep- and office-level reporting.
•	Retailer, Retail_Location, store number — where. 100% populated.
•	Funnel flags with dates — Verified, Issued, Demoed, Sold, plus cancel/dead reasons. The lead's life story as checkboxes.
•	Gross_Sales__c — the sale amount when Sold. (Amount_of_Sale and Net_Sale_Amount also exist; ⚠ which is authoritative is unconfirmed.
•	Centah_ID__c — the cross-system key that links to the call log for some clients (see "How the tables join").
Reference funnel (trailing 12 months, all clients): ~278k leads → 64% verified → 44% issued → 33% demoed → 12.5% sold ≈ $148M gross. Memorize the shape, not the digits — they move.
Table 2 — The call log ("Prod_Call_Center_New")
Grain: one row per phone call (not per customer — a lead called three times appears three times). ~500k calls per six months. Source: Convoso via Salesforce.
•	Campaign_Name / Queue_Name — which client program and queue the call belonged to (e.g., "ATT Costco SC TPV", "Primo TPV Costco Spanish", "CONFIRMATION COSTCO LEAFGUARD").
•	Status_Name__c — the disposition the agent selects when the call ends. ~150 distinct values; each campaign speaks its own dialect. This field powers every outcome metric — see "The status glossary".
•	Call_Type — INBOUND (warm transfers; the majority), MANUAL (agent outbound dials, mostly confirmation chasing), 3WAY, OUTBOUND.
•	Queue_Wait_Time (sec), Talk_Time (sec), Wrap_up_Time (sec) — the timing trio. ~78% of calls show zero queue wait (answered instantly or never queued).
•	Agent_ID — who handled it. Agent 666667 is not a person: it is the system placeholder for calls no human answered (abandons, queue timeouts, after-hours). Exclude it from all per-agent metrics; include it when counting abandonment.
•	Order_Number__c — for appointment campaigns, carries the field system's identifier — this is the bridge to Table 1.
•	Lead_ID__c — caution: this is the dialer's own lead number. It does NOT match Table 1's Lead_ID. Zero overlap. See the join section.
•	Cost__c — per-call telephony cost in dollars (⚠ interpretation unconfirmed), enabling cost-per-approved-call analysis.
Table 3 — The agent state log ("Call_Center_hours")
Grain: one row per time segment of an agent's day — Login 8:59, Available 8:59–9:41, Bathroom 9:41–9:45, Available…, Lunch, Logout. This is the timeclock: it knows where every agent-minute went, and nothing about customers.
•	Availability_Code__c — a 28-code taxonomy: Available, Lunch, Break, Bathroom, Outbound Manual Dialing, Special Project, Approved Admin Work, trainings, system-issue codes, forced logouts.
•	State__c — the rollup: Ready / Not Ready / Login / Logout.
•	Campaign__c — agents log into one campaign at a time; most agent-days never switch, though all agents are cross-trained.
•	Duration — beware formats: Excel exports render it as a time value, CSV exports as "H:MM:SS" text. Parse deliberately.
Reference shape: agents work ~10-hour days; roughly 65–76% of logged time is "Available" depending on the month; total shrinkage (everything not call-ready) runs ~25–35%, in line with industry norms. The vague codes — Special Project and Approved Admin Work — deserve permanent scrutiny: they are where capacity quietly disappears.
The status glossary — how ~150 statuses become metrics
Success is defined per campaign: ATT counts statuses containing "Approved"; Primo's success word is literally "Sale"; pest control uses "welcome call completed"; verification campaigns use "Working - Appointment Set"; confirmation campaigns use "Appointment Confirmed" and three cousins. Consequence: an approval rate blended across campaigns is directional only — always compare within a campaign.
The extended glossary additionally categorizes every non-success status into loss families, each pointing at a different owner of the fix:
Category	Example statuses	Whose problem
Queue Abandon	Call Abandoned In Queue, Queue Drop	Staffing / routing
System / Telephony	DEAD AIR, Convoso Error, Bad Connection	Platform vendor
Contact – Failed Verification	UV: Not Homeowner, UV: Flat Roof, UWC: Card Declined	Field quality / rep coaching
No Contact	Voicemail, No Answer, disconnected number	List quality / attempt strategy
Hang Up / Incomplete	HU: No Interaction, Caller Hung Up	Expectations set at the table
Rework – Unlock/Mod	UNLOCK: …	First-time-wrong work
Non-Lead / Service	FRC: General Question, Wrong Dept	Routing / IVR

Prefix decoder: UV: unable to verify (with reason — this family is the rejection-reason taxonomy), UWC: unable to complete welcome call, HU: hang up, CS: customer service. ⚠ "FRC:" meaning unconfirmed.
How the tables join (the lead spine)
From	To	Key	Notes
Call log	Field leads	Order_Number__c → Centah_ID__c (sometimes → Lead_ID__c)	~96% match for appointment campaigns; varies by client
Call log	Agent hours	Agent_ID__c	Enables occupancy = talk time ÷ available time
Field leads	ICDs/reps/venues	ICD/rep IDs, store fields	Native columns — no join needed

The trap that catches everyone: the call log's Lead_ID and the lead table's Lead_ID are different ID systems with zero overlap. The dialer mints its own. The bridge is Order_Number ↔ Centah_ID. Also: one lead ↔ many calls, so any join of the two tables must aggregate calls to the lead grain before touching revenue, or sales will double-count.
Metric definitions (and their denominators)
Metric	Definition	Denominator
Calls Offered	All call rows, incl. system rows	—
Answered	Reached a human (agent ≠ 666667, talk > 0)	—
Approved	Campaign-specific success per official glossary	Answered
Abandon Rate	Queue Abandon category	Offered (they never reached an agent)
Avg Queue Wait	Mean wait of answered calls (report p90 too)	Answered
Unlock Rate	UNLOCK statuses	Approved
Verification Rate (field)	Verified flag on leads	Leads created

Denominators are where meetings go to argue. State them every time. Two people computing "approval rate" with different denominators will both be right and the meeting will still be a mess.
Known data quirks — the analyst trap list
•	Agent 666667 = system rows (abandons live here). Exclude from agent metrics; never from abandonment.
•	~15.7k leads carry 1999–2000 placeholder created dates. Filter them out.
•	~21k duplicate Lead_IDs in the lead table — dedupe before counting leads.
•	Timestamps come in UTC/PST pairs ~7–8h apart; pick one canonical zone (⚠ official zone unconfirmed) — intraday analyses are wrong otherwise.
•	The WFM availability taxonomy changed in June 2026 ("Training" retired; Special Project / Admin / Client Training grew) — historical trending across that boundary needs a mapping.
•	Status text has traps: trailing spaces ("Reschedule  "), inconsistent case — always TRIM + UPPER before matching.
•	Legacy campaigns can keep receiving calls after migrations (see: "ATT TPV" post-April) — watch small campaigns with extreme abandon rates.
•	Recent-month funnel rates are censored: young leads haven't had time to sell yet. Exclude the last ~4–6 weeks from sold-rate trends or label them immature.
•	Ad-hoc exports cap silently (a suspiciously round row count like exactly 15,000 is a truncation, not a coincidence).
Open questions a new joiner should ask early
•	What exactly triggers client payment on each campaign (approved call? issued appointment? sat? sale)?
•	Which revenue field is authoritative: Gross_Sales, Amount_of_Sale, or Net_Sale_Amount?
•	Who owns Convoso status creation and routing, and what is the change process?
•	Is there a schedule/roster export (planned shifts)? Without it, adherence cannot be measured — only actuals.
•	What do FRC: statuses stand for; what are Special Project and Approved Admin Work concretely?
•	What is the canonical reporting timezone?
Quick-reference scale card
Fact	Value (mid-2026)
Calls per month	~77k
Field leads per month	~20–25k (home-improvement portfolio)
Active ICD offices / field reps	~64 / ~680+
Call center agents	~120–190 depending on month
Typical field funnel	64% verify · 44% issue · 33% demo · 12.5% sell
Median home-improvement sale	≈ $17,000
Trailing-12-month gross sales traced	≈ $148M
Glossary of terms
Term	Meaning
ICD	Independent Corporate Director — owner of an independent field sales office
TPV	Third-Party Verification — live verification of a sale by the call center
Warm transfer	Rep calls the center with the customer present; speed-to-answer is everything
Convoso	The dialer / call-center platform generating Tables 2 and 3
Centah	Lead-management platform in the Lowe's ecosystem; source of Centah_ID
Issued / Demoed / Sold	Appointment sent to client / consultant sat the appointment / customer bought
Chargeback	Client claws back payment for a lead/sale later disputed
Unlock / Mod	Reopening a submitted record to change it — a rework/defect signal
Shrinkage	Paid time not available for calls (breaks, training, admin…)
Occupancy	Share of available time actually spent handling calls
Lead spine	The governed one-row-per-lead join of field, call, and outcome data

