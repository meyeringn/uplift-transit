# Equity Framework

This document states the equity principles that
shape every technical decision in this project.
These are not aspirational values added at the end.
They are design constraints applied from the start.

---

## Principle 1: The Primary User Is the Person
Most Likely to Be Missed

Conventional systems — including most civic tech —
are designed for the median user. The median user
has reliable internet access, speaks English, can
navigate complex interfaces, and has enough slack
in their schedule to seek help before a crisis hits.

This project is not designed for the median user.

The primary user is a Disabled resident who cannot
regulate their body temperature normally. A low-income
renter whose landlord won't fix the AC. An elderly
person living alone who hasn't told anyone they're
struggling. A transit rider who showed up to a
broken elevator with nowhere else to go.

If this tool doesn't work for those people first,
it doesn't work.

---

## Principle 2: Asymmetric Error Costs Are a
Justice Question, Not a Technical Preference

Every prediction model makes two kinds of mistakes:
false positives (flagging something that isn't a
problem) and false negatives (missing something
that is).

In most ML applications, these errors are treated
as symmetric — a tradeoff to be optimized based
on business cost.

In this project, they are not symmetric.

A false negative means a Disabled person is stranded.
A chronically ill neighbor doesn't get a wellness
check. A high-risk ZIP code doesn't get a cooling
bus. People are hospitalized. People die.

The asymmetry is not a technical preference.
It is a justice position: the cost of missing a
vulnerable person is categorically different from
the cost of an unnecessary service call.

Every threshold, every metric, every design decision
in this project reflects that asymmetry explicitly.

---

## Principle 3: Explainability Is a Community
Accountability Tool, Not a Technical Feature

SHAP (SHapley Additive exPlanations) is built into
this project's design from the start. It allows
anyone to see which features drove a particular
prediction — why this ZIP code was flagged, or
why this elevator was ranked high-risk.

Most ML projects include explainability because
reviewers ask for it or regulations require it.

This project includes it because the communities
affected by these predictions have a right to
understand and challenge them.

A score that cannot be explained is a score that
cannot be contested. A score that cannot be
contested by the people it affects is not a
civic tool. It is a black box with a mission
statement on top.

---

## Principle 4: Identity-First Language Is the
Default, Not an Option

This project uses identity-first language throughout:
Disabled people, not "people with disabilities."

This is not a style preference. It reflects the
position of many Disabled people and disability
rights organizations that disability is not
something separate from a person — it is part
of who they are, shaped by both their body and
the systems that were not built to include them.

Language that treats disability as something
a person "has" — like a condition to be managed —
embeds a medical model that this project explicitly
rejects. Disability is a political and social
category as much as a medical one. The communities
this project serves deserve language that reflects
that.

---

## Principle 5: Community Co-Design Is a
Requirement, Not a Consultation

This project includes a formal community engagement
framework (see `COMMUNITY_ENGAGEMENT.md`) that
involves affected communities at every phase —
not just as end users who receive a product,
but as co-designers who shape what it does.

This includes a community challenge mechanism:
a formal process for residents and CBOs to
contest scores they believe are wrong.

The reason is simple: the communities most likely
to be missed by historical data are also the
communities most likely to be underflagged by
a model trained on that data. No amount of
technical sophistication fixes a data gap
that community knowledge can close.

---

## Principle 6: Open Source Is an Equity Position

This project is published under an MIT license.
The code, data sources, methodology, and
documentation are publicly available.

This is not a default choice. It is a deliberate
one. Civic tools built on public data, for public
benefit, funded by public resources — or seeking
public partnership — should be open to public
scrutiny, replication, and improvement.

A civic tech tool that is proprietary is a
civic tech tool that has already decided who
gets to maintain it.

---

## Who Wrote This and Why It's Personal

I am a Disabled person. I have Congenital Central
Hypoventilation Syndrome — a condition affecting
the autonomic nervous system, including temperature
regulation. My wife has acquired disabilities
including chronic illness with autonomic dysfunction.

I am also Chairman of the Philadelphia Mayor's
Commission on People with Disabilities.

I did not arrive at these equity principles through
a framework workshop. I arrived at them because
the systems this project is trying to fix have
failed people I love — including me.

These principles are not window dressing.
They are the reason this project exists.

---

**Nico Meyering, MPA** · Philadelphia, PA
meyeringn@gmail.com ·
[LinkedIn](https://www.linkedin.com/in/nicolasmeyering)

*Equitech Futures Civic Tech Institute, CTI 2026*
