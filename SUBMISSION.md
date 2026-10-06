# HW2 submission

**Name:** Nurlybek Iliyas 
**Student ID:** S23070332
**Group:** 
**Repository:** https://github.com/Nurlybek-Iliyas/ai-2026-hw2

## AI tool disclosure

I used ChatGPT for explanations, debugging, and suggestions while working on the assignment. I reviewed and adapted the code, ran the experiments in Google Colab, checked the outputs, and made changes based on the assignment requirements.

---

## Sublab Easy — one task, four roles

### Decisions per role

| Enquiry | policy_officer | front_desk | auditor | bilingual_clerk |
|---|---|---|---|---|
| E-01 | granted ✓ | granted ✓ | more_info ✗ | granted ✓ |
| E-02 | more_info ✓ | more_info ✓ | more_info ✓ | more_info ✓ |
| E-03 | refused ✓ | more_info ✗ | refused ✓ | refused ✓ |
| E-04 | refused ✓ | more_info ✗ | refused ✓ | refused ✓ |
| E-05 | granted ✓ | granted ✓ | more_info ✗ | granted ✓ |
| E-06 | granted ✓ | granted ✓ | more_info ✗ | granted ✓ |
| E-07 | granted ✓ | granted ✓ | more_info ✗ | granted ✓ |
| E-08 | not_found ✓ | not_found ✓ | not_found ✓ | not_found ✓ |
| E-09 | refused ✓ | more_info ✗ | refused ✓ | refused ✓ |
| E-10 | more_info ✓ | more_info ✓ | more_info ✓ | more_info ✓ |
| **agrees with `expected`** | **10/10** | **7/10** | **6/10** | **10/10** |
| **parsed** | **10/10** | **10/10** | **10/10** | **10/10** |
| **schema-valid** | **10/10** | **10/10** | **10/10** | **10/10** |

### Which field moved, on which enquiry, under which role

| Field | Enquiries that moved | Role(s) that moved it |
|---|---|---|
| `found` | None | None |
| `decision` | E-03, E-04, E-09 | front_desk |
| `decision` | E-01, E-05, E-06, E-07 | auditor |
| `amount` | E-01, E-05, E-06, E-07 | auditor |
| `missing_documents` | None | None |

### Raw replies

One enquiry where a role changed the decision away from the policy officer:

```json
{"applicant_id":"A-203","found":true,"decision":"more_info","amount":0,"missing_documents":[],"reason":"The official record shows a GPA of 2.4, below the required minimum of 2.67. Her income band and required documents qualify, but the GPA would need to be at least 2.67 before the application could qualify."}
```

E-07 from the bilingual clerk:

```json
{"applicant_id":"A-201","found":true,"decision":"granted","amount":250000,"missing_documents":[],"reason":"Иә, сіз грант ала аласыз: GPA көрсеткіші 3.4, табыс санатыңыз 1, транскрипт пен жеке куәлік құжаттарыңыз тіркелген. Грант мөлшері — 250 000 теңге."}
```

### Written answers

**1. Which fields are role-sensitive and which are not?**

In my run, `decision` and `amount` were role-sensitive. The front-desk role changed E-03, E-04 and E-09 from `refused` to `more_info`. The auditor changed E-01, E-05, E-06 and E-07 from `granted` to `more_info`, which also changed their amounts to 0. `found` and `missing_documents` did not move on any enquiry. The bilingual clerk did not change the structured fields, but it changed the free-text `reason` language for E-07 to Kazakh.

**2. Which enquiries are most sensitive to the role, and why those?**

E-03 and E-04 are sensitive because the official policy refuses them: E-03 has GPA 2.4, below the 2.67 minimum, and E-04 has income band 3, which is not allowed. The front-desk role changed these refusals to `more_info`.

E-07 tests language. The bilingual clerk kept the same structured decision as the policy officer but wrote the reason in Kazakh. The auditor changed it to `more_info` because it does not grant on the first reading.

E-10 tests whether the model trusts the applicant's claim or the official record. The applicant says the ID card was uploaded, but the official record still does not contain it. All roles correctly kept `more_info` and did not treat the claim as evidence.

**3. Where does discretion belong — the role paragraph, or code that reads `decision` afterwards?**

Role-specific discretion should be stated in the role paragraph so the intended behavior is explicit before generation. Code after the response should validate the schema and enforce hard rules, not secretly change the meaning of a role.

A downstream program that only receives the final JSON can see the decision and other fields, but it cannot reliably know which role produced the record. Different roles can produce exactly the same structured output. If role provenance matters, the program should store the role separately as metadata.

**4. Is a role a boundary?**

No. A role prompt is not a security or correctness boundary. In Week 2 terms, the role paragraph is still just tokens entering the same model context, and the model continues from that context.

If a wrong `decision` were expensive, I would not depend only on the prompt. I would put deterministic validation in code, check the result against the official policy and records, reject invalid values, and require human review for high-impact decisions.

---

## Sublab Medium — memory you choose

### Tokens per call

| Call | A — never compressed | B — compressed at the `compress` turn |
|---|---:|---:|
| 1 | 853 | 853 |
| 2 | 950 | 908 |
| 3 | 1029 | 978 |
| 4 | 1098 | 1037 |
| 5 | 1147 | 1090 |
| 6 | 1231 | 1185 |
| 7 | 1311 | 1276 |
| 8 | 1382 | 1368 |
| 9 | 1454 | 1447 |
| 10 | — | 1372 |
| 11 | 1551 | 1215 |
| 12 | 1623 | 1268 |
| **peak** | **1623** | **1447** |
| **total for the run** | **13629** | **13997** |

### Probes after the conversation

| Probe | Tests | A retrieved? | A answer | B retrieved? | B answer |
|---|---|---|---|---|---|
| Q-1 identity | turn 1 | Yes | You are Daniyar Qoshan, applicant A-202. | Yes | You are Daniyar Qoshan, applicant A-202. |
| Q-2 missing document | turn 5 | Yes | Your id_card is still missing from the official file. | Yes | Your ID card is still missing from the official file. |
| Q-3 band and amount | turns 3–4 | Yes | Income band 2 and 150,000 KZT. | Yes | Income band 2 and 150,000 KZT. |
| Q-4 the constraint | turn 6 | Yes | You said you can come to the office on Thursday. | Yes | You can come to the office on Thursday. |
| Q-5 the open question | turn 7 | Yes | You asked whether a scanned employer letter counts or the original is required. | Yes | You asked whether a scanned employer letter counts or the original is required. |
| **retrieved** | | **5/5** | | **5/5** | |

### The state my compression produced

```json
{
  "applicant_id": "A-202",
  "topic": "Study grant eligibility and document submission",
  "facts": [
    "The applicant's name is Daniyar Qoshan.",
    "The applicant sent their transcript last week.",
    "The applicant's income band is 2, according to their family's certificate.",
    "The applicant could not upload the ID card because the scanner at home broke.",
    "The applicant can only come to the office on Thursdays because they have lab all week otherwise.",
    "The applicant said their sister Aruzhan applied last year and is also on file."
  ],
  "decisions": [
    "The application currently does not qualify because the ID card is missing.",
    "If the ID card is added and the application qualifies, the grant amount would be 150,000 KZT.",
    "The official record includes Aruzhan Nurlan (A-205), whose listed GPA, income band, and documents qualify her for a 250,000 KZT grant under the 2026 scheme."
  ],
  "constraints": [
    "The ID card must be added to the official file.",
    "The applicant can visit the office only on Thursdays.",
    "The applicant has laboratory work all other days of the week."
  ],
  "open_questions": [
    "Does a scanned letter from the employer count, or is the original required?",
    "If the applicant brings the ID card on Thursday, will the decision be made the same day?",
    "Will the office accept the ID card on Thursday?",
    "Whether Aruzhan applied last year and whether she is the applicant's sister could not be confirmed from the official record."
  ],
  "language": "Kazakh and English"
}
```

### Written answers

**1. What did compression buy?**

Without compression, the peak input was 1623 tokens. With compression, the peak was 1447 tokens, so the peak decreased by 176 tokens, about 10.8%.

Both versions retrieved 5/5 probes, so no tested information was lost.

The total was 13629 tokens without compression and 13997 with compression. The compressed run was slightly more expensive overall because the compression operation itself required a separate 1372-token call. However, after compression the later calls were smaller: call 11 fell from 1551 to 1215 and call 12 from 1623 to 1268. In a longer conversation this saving could accumulate.

**2. Why must the state be structured rather than a paragraph?**

A structured object has named fields such as `facts`, `constraints` and `open_questions`. My program can validate these fields against a schema and reliably find a particular type of information.

A paragraph summary has no fixed structure. Important information may be mixed together or omitted, and code cannot easily distinguish a constraint from a fact or an unanswered question. Structured memory therefore makes both validation and later retrieval more reliable.

**3. What is missing from your state that you would add?**

I would add a field such as `unverified_claims` or `source_status`. The conversation contains statements made by the applicant, for example that the transcript was sent last week, but an applicant claim should not overwrite the official record.

To keep the state small, I would remove or shorten less relevant information about Aruzhan because it is not necessary for Daniyar's current grant decision.

**4. When is compression the wrong choice?**

Compression is a bad choice when the exact original wording matters, for example a legal agreement, a complaint, an audit transcript, or another conversation where one sentence may later be evidence.

A valid structured summary can still lose wording or nuance. My current program would detect invalid JSON or schema failure, but it would not automatically notice every semantic detail that a valid summary forgot. In such a case I would keep the original transcript instead of replacing it.

---

## Sublab Hard — stories in, CVs out, the best candidate by code

### Part 1 — extraction

| Story | Parsed? | Valid? | Fields that came back `null` | Traps hit |
|---|---|---|---|---|
| story-01 | True | True | none | none |
| story-02 | True | True | `graduation_year`, `gpa_4_scale`, `gpa_original` | no GPA stated |
| story-03 | True | True | none | GPA on another scale; paper not published |
| story-04 | True | True | none | paper not published |
| story-05 | True | True | none | paper not published |
| story-06 | True | True | `graduation_year`, `gpa_4_scale`, `gpa_original` | contradiction |

Extraction for story-06:

```json
{
  "candidate_id": "story-06",
  "full_name": "Nurzhan Abilov",
  "degree": "BSc in Statistics",
  "graduation_year": null,
  "gpa_4_scale": null,
  "gpa_original": null,
  "languages": [
    "Kazakh",
    "Russian",
    "English"
  ],
  "published_peer_reviewed_outputs": 1,
  "unpublished_outputs": [
    "One poster at a local event"
  ],
  "relevant_experience_months": 40,
  "uncountable_experience": [],
  "ambiguities": [
    "The GPA is contradictory: the story states both 3.2 and 3.5.",
    "The graduation status/year is contradictory: the story states graduation in 2024 and also being a final-year student graduating in 2026."
  ],
  "evidence": {
    "full_name": "# Nurzhan Abilov",
    "degree": "I graduated in 2024 with a BSc in Statistics.",
    "graduation_year": "I graduated in 2024 ... I am currently a final-year student graduating in 2026",
    "gpa": "My GPA was 3.2. Actually I should double-check that, I think it was 3.5",
    "languages": "Languages: Kazakh, Russian, English.",
    "publications": "one paper published, in a peer-reviewed proceedings, on survey weighting",
    "experience": "I have been at an insurance analytics team since February 2023, which is about forty months."
  }
}
```

### Part 2 — scores and the winner

| Candidate | academic (0–5) | research (0–5) | experience (0–5) | weighted total (code) |
|---|---:|---:|---:|---:|
| story-01 | 5 | 5 | 2 | 4.40 |
| story-02 | 0 | 0 | 5 | 1.00 |
| story-03 | 4.6 | 2.5 | 2.92 | 3.63 |
| story-04 | 4 | 3 | 5 | 3.90 |
| story-05 | 5 | 2.5 | 1.25 | 3.50 |
| story-06 | 0 | 2.5 | 5 | 1.75 |

**Winner, computed by my code:**  
story-01 — Aziza Bekova — **4.40**

**The model's prose answer, asked separately ("who should win?"):**

> I would award the funded scholarship to **Aziza Bekova**.
>
> Her record is the strongest overall: she reports a **3.8 GPA on a 4.0 scale**, **two peer-reviewed publications**, and **eight months of relevant experience**. This combination demonstrates excellent academic performance and the strongest documented research output among the candidates, while also showing practical experience. Although some candidates have longer work histories or slightly higher GPAs, Aziza offers the most compelling balance of academic achievement and research readiness.

### Part 3 — written answers

**1. Which rule did you have to add, and what broke without it?**

I added a stricter rule that the model must not infer that an output is peer-reviewed just because it appears in conference proceedings or a journal. It must only count it as peer-reviewed when the story explicitly supports that.

Story-02 forced this rule. It says that Dias has one published paper in student conference proceedings, but it does not explicitly say that it was peer-reviewed. Without the stricter rule, the model counted it as a peer-reviewed publication and gave him research credit. With the rule, his `published_peer_reviewed_outputs` did not get unsupported peer-review status and his research score was 0.

**2. Where did the model guess, and where did your code have to decide?**

The model still had to exercise judgment when converting the rubric into intermediate 0–5 scores. For example, the rubric gives clear meanings for 0 and 5 but does not define every value in between, so scores such as 2.5, 2.92 and 3 required model judgment.

My code made the deterministic ranking decision. It took the three structured scores and calculated:

`0.5 × academic + 0.3 × research + 0.2 × experience`

Then Python sorted the candidates and selected the highest total. The model did not calculate that final ranking.

**3. Did your prose ranking and your computed ranking agree?**

Yes. Both selected story-01, Aziza Bekova.

I trust the computed ranking more because I can inspect the individual scores, weights and arithmetic. The prose answer sounds reasonable, but I would not trust it alone unless it also gave structured criterion scores, evidence for those scores, followed exactly the same rubric, and produced reproducible results.

**4. The rubric has no anchor for a contradicted field. What did you do and what should the rule be?**

For story-06 the story says GPA 3.2 and then 3.5. I did not choose one and did not average them. Extraction returned `gpa_4_scale = null` and recorded the contradiction in `ambiguities`.

For ranking, I used a conservative academic score of 0 instead of allowing the model to select one of the contradictory GPAs.

I think the rubric should explicitly define this case. A contradicted field should remain `null` and be flagged for manual review. If an automatic ranking must still continue, the rubric should define a fixed treatment in advance instead of letting the model invent one.

**5. How close were your top two candidates?**

The top two candidates were:

- story-01: 4.40
- story-04: 3.90

The difference was **0.50**, so they were not within 0.05.

Therefore, my result was not a near tie. If they had been within 0.05, I would tell the committee that the ordering was sensitive to extraction and scoring uncertainty. I would make the extraction stricter, verify the evidence for every scored field, and use a second review before making the final decision.

---

## Reflection

The main lesson for me is that reliable model output needs more than a good prompt. I need a clear schema, validation, explicit rules for missing or contradictory information, and deterministic code for decisions that can be calculated exactly. I would also keep model judgment separate from rules that can be enforced in normal code.
