# A Pragmatic Approach to Threat Modeling

We’ve seen it over and over: whiteboards covered in STRIDE grids, threat trees, and post-its full of potential issues. Threat modeling sessions often generate **plenty of awareness**, but leave one obvious question hanging in the air:  
> **“Which ones do we fix first?”**

Frameworks like **STRIDE**, **PASTA**, and **OCTAVE** do a solid job of **identifying** threats—they give categories, language, and structure. But they stop short of solving the real problem:

> **Priority**

They tell us what could go wrong, but not which of those things **actually matters right now**.

What I needed was a model that:
- Helps me **focus attention** where it counts,
- Respects the **technical and business context** of a threat,
- And provides a **simple, visual, and scalable way to score risk**—without needing a PhD in risk theory.

So I started using the **L × I × S model**—a pragmatic approach to threat modeling that not only surfaces threats, but lets me **rank them** based on:
- How likely they are to occur (**Likelihood**),
- What the technical consequence would be (**Impact**),
- And how damaging it would be based on where it happens (**Sensitivity**).

---

## Why I Added Sensitivity (S)

Most risk models stop at **Likelihood × Impact**—which sounds reasonable on paper. But in practice, I found it lacking.

The same vulnerability can have **wildly different consequences**, depending on *what* it's exposing or affecting.

Consider this:
- An internal misconfig exposing **marketing data** is inconvenient.
- The same flaw exposing **medical records** is a breach, a lawsuit, and a regulatory nightmare.

> The **impact hasn’t changed**—but the **sensitivity** of the data has.

This is why I introduced **S (Sensitivity)**: to provide a way to **contextualize risk** based on **what’s at stake**. Not in abstract business-speak—but in a way that aligns with how I actually reason about risk in different domains.

By isolating **Sensitivity** as its own dimension, I avoid polluting "Impact" with context-specific baggage. Impact stays technical. Sensitivity captures **the value of the asset** or **the regulatory weight of the data**.

It’s the difference between saying “this is bad” and “this is bad *here*.”

---

## Scoring Sensitivity (S)

To make Sensitivity (S) meaningful—but not overly complex—I use a simple **Fibonacci-based scale**:  
**0, 1, 2, 3, 5, 8, 13**

This gives a **non-linear weighting** that reflects how dramatically consequences can escalate when more sensitive data is involved. The idea is simple:
> **The more sensitive the asset, the more it amplifies the impact.**

| S   | Data Sensitivity       | Examples                                           |
|-----|------------------------|----------------------------------------------------|
| 0   | Public                 | Open datasets, public web content                  |
| 1   | Paid-for / Licensed    | Market research, subscription content              |
| 2   | Internal               | Business plans, internal emails                    |
| 3   | Customer PII           | Names, emails, addresses                           |
| 5   | Regulated PII          | Passport numbers, bank details                     |
| 8   | Highly Sensitive PII   | Medical records, biometrics                        |
| 13  | State Secrets          | Classified intelligence, critical infrastructure   |

I intentionally assigned **public data a value of 0**—because if it's truly public, exposing it carries **no risk**. From there, the scale accelerates, recognizing that **risk isn’t linear** when sensitive data is involved.

---

## Scoring Impact (I)

**Impact (I)** measures the **technical consequence** of a threat being realized. It answers the question:
> *What can an attacker actually do if this threat is exploited?*

It’s assessed independently of who’s affected or how sensitive the data is—that’s what **Sensitivity (S)** is for. Impact stays focused on **system-level outcomes**.

I use a **1–10 scale**:

| I   | Technical Impact                             | Examples                                                             |
|-----|----------------------------------------------|----------------------------------------------------------------------|
| 1   | Negligible                                   | Cosmetic bug, no data accessed or modified                          |
| 2   | Minor                                        | Metadata leak, internal IP exposure                                 |
| 3   | Low                                          | Read access to harmless or test data                                |
| 4   | Moderate                                     | Unintended read access to internal or non-sensitive user data       |
| 5   | Elevated                                     | Access to internal tooling or business functions                    |
| 6   | Serious                                      | Unauthorized data tampering or limited privilege abuse              |
| 7   | High                                         | Horizontal privilege escalation (e.g., other users' records)        |
| 8   | Severe                                       | Vertical privilege escalation (e.g., higher roles or permissions)   |
| 9   | Critical                                     | Admin/root access, broad system control                             |
| 10  | Catastrophic                                 | Full compromise: RCE, infrastructure control, irreversible data loss|

---

## Scoring Likelihood (L)

**Likelihood (L)** represents how probable it is that a given threat will be realized. It's the **width of the hole**, so to speak—the broader it is, the easier it is to fall into.

I use a **1–10 scale**:

| L   | Likelihood               | Examples                                                         |
|-----|--------------------------|------------------------------------------------------------------|
| 1   | Extremely unlikely       | Obscure flaw, multiple preconditions                            |
| 2   | Rare                     | Requires uncommon access pattern or user behavior               |
| 3   | Low                      | Internal-only flaw, requires credentials                        |
| 4   | Possible                 | Low-profile exposure, needs some recon                          |
| 5   | Moderate                 | Externally reachable, but constrained                           |
| 6   | Likely                   | Known flaw in public system, exploitable with effort            |
| 7   | Very likely              | Public-facing, minimal protections, known vuln category         |
| 8   | Active targeting likely  | Attacks observed in wild, e.g. OWASP Top 10                     |
| 9   | Highly likely            | Easily discovered and exploited with automated tools            |
| 10  | Practically guaranteed   | Exploit is public, trivial, widespread (e.g., copy-paste PoC)   |

Likelihood is **situational and fluid**—it may change as exposure changes or exploits become public.

---

## Bringing It All Together: L × I × S

Once I’ve scored **Likelihood**, **Impact**, and **Sensitivity**, I just multiply them:
> Risk score = L × I × S

This gives a **single, comparative score** that answers the question:
> *How much should I care about this?*

Each factor plays a clear role:
- **L (Likelihood):** How likely it is to be exploited
- **I (Impact):** What the technical consequence is
- **S (Sensitivity):** How badly it matters in *this* context

---

## Same Flaw, Different Context

Here’s a practical example of how L × I × S plays out across domains:

> A broken access control flaw in an internal dashboard allows users to export all customer profiles.

### Healthcare Context

- **Likelihood (L)** = 3
- **Impact (I)** = 4
- **Sensitivity (S)** = 5 (regulated personal health data)
> Risk Score = 3 × 4 × 5 = 60


That’s a high-priority issue with real compliance implications (e.g. HIPAA).

---

### Financial Context

- **Likelihood (L)** = 3
- **Impact (I)** = 4
- **Sensitivity (S)** = 3 (PII, but not medical or payment data)
> Risk Score = 3 × 4 × 3 = 36


Same technical flaw.  
Same likelihood of being hit.  
But different **data sensitivity** means different urgency.

That’s what S brings to the table.

---

## A Visual Analogy

In practice, I often sketch threats as **holes** during a threat modeling session:

- **Width = Likelihood**
- **Depth = Impact**

So:

- A **wide, shallow hole**? Common, but low-priority.
- A **narrow, deep hole**? Rare but severe—track it closely.
- A **wide, deep hole**? Drop everything and fix it!

This simple metaphor helps **visually prioritise risk**—not by gut feel, but by structured, context-aware reasoning.

---

## Wrapping Up

This isn’t a new framework. It’s not a standard. It’s not an academic paper.  
It’s just a **tool**—something that helps me do the thing that matters most in security:

> **Prioritise**

L × I × S doesn’t try to replace STRIDE or CVSS or any existing framework.  
It sits beside them. It plugs the gap between *identifying threats* and *prioritising them in the real world*.

Use it in whiteboard sessions. Use it when grooming security backlog. Use it when staring at twenty vulnerabilities and wondering which one to fix first.

No rituals. No overhead. Just a simple mental model that helps cut through the noise.

Because not all threats are equal.  
And sometimes the simplest model is the one that finally gets people to pay attention to the right things.
