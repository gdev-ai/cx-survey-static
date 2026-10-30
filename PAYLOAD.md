# CX Survey 2026 (static v2): response payload

What `index-v2.html` sends for each response. Question set `2026-10-28` (Unified Design 1.1).

## How it is sent

- `POST` to `CONFIG.submitUrl` with `Content-Type: application/json`. The body is the JSON object below.
- **Not configured yet:** `submitUrl` is empty today, so sends are only logged to the console.
- **Several sends per customer, one `ref`:**
  - A send goes out at the end of every section, with `status: "partial"`.
  - A final send goes out when the customer finishes, with `status: "complete"`.
- **Upsert on `ref`:** the receiver should keep the latest payload per `ref`, because each send carries all answers given so far. Once a `complete` payload has arrived, the response is final.
- **Finishing early:** a customer can stop after any section. That send has `status: "complete"` and `endedEarly: true`, and the sections they skipped have no keys.
- **Missing keys:** a question that was never answered is not in the payload at all. Optional text left empty may arrive as `""`.
- **Stored values are English** whatever the survey language. Arabic is only used for display.

## Top level

| Key | Type | Values / notes |
|---|---|---|
| `survey` | string | Always `"CX-2026"` |
| `questionSet` | string | `"2026-10-28"`. Changes whenever questions change. |
| `lang` | string | `"en"` or `"ar"`: the language the customer answered in |
| `ref` | string | `CX26-XXXXXX` (6 uppercase letters or digits). Response id: one per customer per browser. |
| `status` | string | `"partial"` or `"complete"` |
| `endedEarly` | boolean | `true` when the customer chose "Finish here" before the last section |
| `sectionsDone` | string[] | Sections sent so far, in order. Possible ids: `A`, `B`, `C`, `D`, `E:Playa`, `E:Seashell`, `Closing` |
| `sectionsTotal` | number | Number of sections this customer gets (3 to 7) |
| `customerId` | string \| null | Salesforce Account Id from `?cid=` (15 or 18 characters). `null` only in demo. |
| `demo` | boolean | `true` for `?demo=1` test links. Ignore these rows. |
| `linkIssues` | string[] | Problems with the link data. Possible values: `cid:missing`, `cid:invalid`, `u:missing`, `u:skipped:<raw value>`, `u:over-limit:<raw value>` |
| `submittedAt` | string | ISO 8601 UTC time of this send |
| `customer` | object | Answers asked once per customer (Parts A and B, Closing) |
| `units` | array | One entry per unit in the link |
| `communities` | array | One entry per community where the customer has a handed-over unit |

### `units[]`

| Key | Type | Notes |
|---|---|---|
| `unit` | string | Unit name as given in the link `u=` parameter |
| `project` | string \| null | `"Playa"`, `"Seashell"` or `null`. Same value as `community`. |
| `stage` | string | `"pre"`: awaiting handover, answers Part C. `"handed"`: handed over, answers Part D. |
| `community` | string \| null | `"Playa"`, `"Seashell"` or `null` (a pre-handover unit may have no community) |
| `answers` | object | Part C keys when `stage = "pre"`, Part D keys when `stage = "handed"` |

### `communities[]`

| Key | Type | Notes |
|---|---|---|
| `community` | string | `"Playa"` or `"Seashell"` |
| `answers` | object | Part E keys. Asked once per community, however many units the customer has there. |

## Answer keys

**Value types**

- **Rating 1–5:** a number from 1 to 5.
- **Rating with N/A:** a number from 1 to 5, or the string `"N/A"`.
- **NPS:** a number from 0 to 10.
- **Multi:** an array of option strings.

### `customer`: Part A, Part B and Closing

| Key | Type | Question |
|---|---|---|
| `A1` | NPS 0–10 | Recommend G Developments to friend or family |
| `A2` | 1–5 | Overall satisfaction with service |
| `A3` | 1–5 (1 = Very difficult, 5 = Very easy) | Ease of getting requests and inquiries resolved |
| `A4` | 1–5 | Speed of response to requests and inquiries |
| `A5` | 1–5 | Clarity of information and updates |
| `A6` | 1–5 (1 = Not at all likely, 5 = Very likely) | Would buy from G Developments again |
| `B1.1` | 1–5 or N/A | Accuracy of account statements and payment schedule |
| `B1.2` | 1–5 or N/A | Ease of making payments |
| `B1.3` | 1–5 or N/A | Handling of contract and document requests (resale or transfer) |
| `B2` | multi | Preferred update channels: `App`, `WhatsApp`, `Email`, `SMS`, `Phone call`, `Site visit` |
| `B3` | 1–5 | Communication, responsiveness and support from the Customer Excellence team |
| `A7` | text (optional) | Suggestions to improve the overall experience |
| `A8` | `Yes` / `No` | May a team member contact you about your answers |

Part B is now asked once per customer. Before 28 Oct, B1 was asked for each unit.

### `units[].answers`: Part C (`stage = "pre"`)

| Key | Type | Question |
|---|---|---|
| `C1.1` | 1–5 or N/A | How regularly you receive construction progress updates |
| `C1.2` | 1–5 or N/A | Transparency about the expected delivery date |
| `C2` | 1–5 (1 = Not at all confident, 5 = Very confident) | Confidence the unit is delivered on the promised date |
| `C3` | single | Preferred update frequency: `Monthly`, `Quarterly` or `At each milestone` |

### `units[].answers`: Part D (`stage = "handed"`)

| Key | Type | Question |
|---|---|---|
| `D0` | single | When the unit was received: `Less than 6 months`, `6–12 months`, `1–2 years` or `More than 2 years` |
| `D1` | 1–5 | Communication and updates about readiness before handover |
| `D2` | 1–5 or N/A | How a change to the delivery date was communicated (`N/A` = delivered on time) |
| `D3` | 1–5 | Organisation of the handover appointment |
| `D4` | 1–5 | Customer Excellence team response and support during handover |
| `D5` | 1–5 (1 = Not at all, 5 = Fully) | Unit matched the promised specifications and finishing |
| `D6` | 1–5 | Overall condition and readiness of the unit at handover |
| `D7` | 1–5 | Support and follow-up after handover |
| `D8` | `Yes` / `No` | Outstanding handover issues to follow up on. `Yes` should open a ticket. |
| `D8_detail` | text | Short description. Required in the form when `D8 = Yes`. |
| `D9` | text (optional) | Suggestions to improve the handover experience |

D6 changed meaning on 28 Oct: it used to be a defect count and is now a 1–5 rating. D7 and D8 are now always asked.

### `communities[].answers`: Part E

All E1–E5 rows are 1–5 or N/A. **The same key means different things in the two communities**, so always read it together with `community`.

| Key | Playa | Seashell |
|---|---|---|
| `E1.1` | Beach: Cleanliness | Beach: Cleanliness |
| `E1.2` | Beach: Furniture and facilities | Beach: Furniture and facilities |
| `E1.3` | Beach: Lifeguards | Beach: Lifeguards |
| `E1.4` | Beach: Beach staff service | Beach: Beach staff service |
| `E1.5` | Beach: Beachfront commercial area | Beach: Beachfront commercial area |
| `E1.6` | Beach: Main spine commercial area | Beach: Internal commercial area (sports courts, kids' area) |
| `E1.7` | Beach: Overall experience at Playa Strip | Beach: Seashell Walk |
| `E2.1` | Lagoons cleanliness and water quality | Pool cleanliness and water quality |
| `E2.2` | Swimmable lagoon cleanliness and water quality | Pool lifeguards |
| `E2.3` | Fountains cleanliness | Swimmable lagoon cleanliness and water quality |
| `E2.4` | — (not asked) | Non-swimmable lagoon cleanliness |
| `E3.1` | Entry at the main gate | same |
| `E3.2` | Security presence inside the community | same |
| `E3.3` | How security responds to incidents | same |
| `E3.4` | Traffic and parking in residential areas | same |
| `E3.5` | Traffic and parking at the beach | same |
| `E4.1` | Maintenance of common areas | same |
| `E4.2` | Cleanliness of public areas | same |
| `E4.3` | Landscaping and green areas | same |
| `E4.4` | Pest control | same |
| `E5.1` | Request or complaint: response time | same |
| `E5.2` | Request or complaint: quality of the fix or answer | same |
| `E5.3` | Request or complaint: follow-up | same |
| `E6` | On-site operations team (1–5 or N/A) | same |
| `E7` | OneCommunity App overall (1–5 or N/A) | same |
| `E8` | Overall community experience this summer (1–5) | same |
| `E9` | Areas to improve for Summer 2027 (multi): `Beach`, `Pools & lagoons`, `Security & traffic`, `Facilities`, `Operations team`, `App`, `Other` | same |
| `E9_other` | Text when `E9` includes `Other` (optional) | same |

## Example (handed-over Playa unit and pre-handover unit, complete)

```json
{
  "survey": "CX-2026",
  "questionSet": "2026-10-28",
  "lang": "ar",
  "ref": "CX26-OG7VBR",
  "status": "complete",
  "endedEarly": false,
  "sectionsDone": ["A", "B", "C", "D", "E:Playa", "Closing"],
  "sectionsTotal": 6,
  "customerId": "0015g00000AbCdEAAB",
  "demo": false,
  "linkIssues": [],
  "submittedAt": "2026-10-30T09:15:02.118Z",
  "customer": {
    "A1": 9, "A2": 4, "A3": 4, "A4": 3, "A5": 4, "A6": 5,
    "B1.1": 4, "B1.2": 5, "B1.3": "N/A", "B2": ["WhatsApp", "Email"], "B3": 4,
    "A7": "Faster replies on WhatsApp", "A8": "Yes"
  },
  "units": [
    { "unit": "Villa B12-03", "project": "Playa", "stage": "handed", "community": "Playa",
      "answers": { "D0": "6–12 months", "D1": 4, "D2": "N/A", "D3": 5, "D4": 4, "D5": 4, "D6": 3, "D7": 3,
                   "D8": "Yes", "D8_detail": "Kitchen door still misaligned", "D9": "" } },
    { "unit": "R-21", "project": "Playa", "stage": "pre", "community": "Playa",
      "answers": { "C1.1": 3, "C1.2": 2, "C2": 3, "C3": "Monthly" } }
  ],
  "communities": [
    { "community": "Playa",
      "answers": { "E1.1": 5, "E1.2": 4, "E1.3": 5, "E1.4": 4, "E1.5": 4, "E1.6": 3, "E1.7": 4,
                   "E2.1": 4, "E2.2": 4, "E2.3": "N/A",
                   "E3.1": 5, "E3.2": 4, "E3.3": 4, "E3.4": 3, "E3.5": 2,
                   "E4.1": 4, "E4.2": 4, "E4.3": 5, "E4.4": 3,
                   "E5.1": 3, "E5.2": 3, "E5.3": 2, "E6": 4, "E7": "N/A", "E8": 4,
                   "E9": ["Security & traffic", "Other"], "E9_other": "More shade at the beach" } }
  ]
}
```

## Suggested Marketing Cloud data extensions

The payload is nested, so it fits best in three data extensions joined on `Ref`. In every one, write `Text` for each answer field: rating fields can hold `"N/A"`, and multi-select fields can be stored joined with `;`.

1. **`CX_Survey_2026_Responses`**: one row per `ref`, the primary key.
   - Response fields: `Ref`, `CustomerId`, `Status`, `EndedEarly`, `SectionsDone` (joined with `,`), `SectionsTotal`, `Lang`, `QuestionSet`, `Demo`, `LinkIssues`, `FirstSubmittedAt`, `LastSubmittedAt`.
   - Answer fields: all `customer` keys, with dots replaced by underscores (`B1_1` and so on).
   - **Upsert** on every send.
2. **`CX_Survey_2026_Units`**: one row per `ref` and `unit`, with that pair as the primary key.
   - Unit fields: `Ref`, `CustomerId`, `Unit`, `Project`, `Stage`, `Community`.
   - Answer fields: `C1_1`, `C1_2`, `C2`, `C3`, `D0` to `D9`, `D8_detail`.
3. **`CX_Survey_2026_Communities`**: one row per `ref` and `community`, with that pair as the primary key.
   - Response fields: `Ref`, `CustomerId`, `Community`.
   - Answer fields: `E1_1` to `E1_7`, `E2_1` to `E2_4`, `E3_1` to `E3_5`, `E4_1` to `E4_4`, `E5_1` to `E5_3`, `E6`, `E7`, `E8`, `E9`, `E9_other`.

**Follow-up triggers** (from the design doc). Each one applies only if `A8 = "Yes"`:

- **Any module:** `A1` from 0 to 6, or any rating of 1 or 2, opens a ticket for that module's owner.
- **Part D:** `D8 = "Yes"` opens a ticket for the Operation Team.
- **Part E (Community):** low scores go to Ahmed Salem for Playa and Peter for Seashell.
