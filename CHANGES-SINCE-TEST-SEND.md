# CX Survey 2026: changes to `index-v2.html` since the 29 Oct test send

**For:** Ragui Chawkat (Marketing Cloud CloudPage + SurveyForce wiring)
**Prepared:** 3 Nov 2026

| | Version | Where |
|---|---|---|
| **Before** (what you wired) | Commit `f27dda1`, 22 Oct 2026. This is the version your test emails opened on Thu 29 Oct 2026 (sent 13:43 Cairo). | `cloud.onecommunity-eg.com/cx-survey-2026` (still runs this version today) |
| **After** (current) | Commit `6885bfd`, 2 Nov 2026. Question set `2026-11-02`. | `https://gdev-ai.github.io/cx-survey-static/index-v2.html` |

The full current payload spec is in [`PAYLOAD.md`](PAYLOAD.md). This note lists only what changed against the version you have.

---

## What you need to do

1. **Survey link (SFMC `SurveyQuery`): no change.** `cid`, `name` and `u=<unit>|<handed|pre>|<Playa|Seashell>` are read exactly as before, with the same validation rules and the same `linkIssues` values.
2. **Submit handler / data extension:** update the field mapping (section 2). Main points:
   - `B1.1`–`B1.3` moved from each unit to the customer level.
   - `D6` is now a number.
   - There are new keys (`B3`, `E1A.*`, `E7.1`–`E7.3`, `questionSet`, `lang`).
   - Some existing E keys now mean something different.
   - Seashell `E1.6` and `E1.7` are no longer sent.
3. **CloudPage:** replace the page with the new file. Then re-apply your four customisations and point the fonts at absolute URLs (section 3).
4. **SurveyForce:** if you mirror the questions there, update the wording and add the new questions (section 4).

---

## 1. Link and URL parameters: unchanged

The link-parsing code is the same, line for line. This also covers `?demo=1`, the `?state=` previews, `MAX_UNITS` (60), duplicate-unit handling and name tidying.

---

## 2. Payload changes

The transport is unchanged: one `ref` per customer, `partial` after each section, `complete` at the end, `endedEarly` on an early finish, and the latest payload per `ref` wins. Section ids are unchanged (`A`, `B`, `C`, `D`, `E:Playa`, `E:Seashell`, `Closing`), and `sectionsTotal` is still 3 to 7.

### 2.1 Top level: two new fields

| Key | Type | Notes |
|---|---|---|
| `questionSet` | string | `"2026-11-02"`. It changes whenever the questions change. Store it with the response so you can tell versions apart. |
| `lang` | string | `"en"` or `"ar"`: the language the customer answered in (see 4.1). |

### 2.2 Moved: Part B is now asked once per customer

| Key | Before | Now |
|---|---|---|
| `B1.1`, `B1.2`, `B1.3` | `units[i].answers` (asked for every unit) | `customer` (asked once) |

`units[].answers` now holds only Part C keys (`stage = "pre"`) or Part D keys (`stage = "handed"`).

### 2.3 New keys

| Key | Where | Type | Question |
|---|---|---|---|
| `B3` | `customer` | 1–5 | Communication, responsiveness and support from the Customer Excellence team |
| `E1A.1` | `communities[].answers` | 1–5 or `"N/A"` | Playa: Overall experience at Main Spine · Seashell: Internal retail & dining area |
| `E1A.2` | `communities[].answers` | 1–5 or `"N/A"` | Playa: Overall experience at Playa Strip · Seashell: Sports courts & facilities |
| `E1A.3` | `communities[].answers` | 1–5 or `"N/A"` | Seashell only: Children's play areas |
| `E1A.4` | `communities[].answers` | 1–5 or `"N/A"` | Seashell only: Seashell Walk (was Seashell `E1.7`) |
| `E7.1` | `communities[].answers` | 1–5 or `"N/A"` | OneCommunity App: ease of use and navigation |
| `E7.2` | `communities[].answers` | 1–5 or `"N/A"` | OneCommunity App: access and QR code features |
| `E7.3` | `communities[].answers` | 1–5 or `"N/A"` | OneCommunity App: submitting and tracking service requests |

`E7` (app overall) keeps its key and type. It is now the first row of the app question, next to `E7.1`–`E7.3`.

### 2.4 Same key, different type or meaning

These keys look the same as before, but the data is not comparable with older responses.

| Key | Before | Now |
|---|---|---|
| `D6` | **Text**, one of `"None"`, `"1–5"`, `"6–15"`, `"More than 15"` (number of defects found) | **Number 1–5**: satisfaction with the overall condition and readiness of the unit at handover |
| `D7` | Only sent when `D6` ≠ `"None"`. Speed and completeness of defect fixes. | **Always sent.** Support and follow-up after handover (1–5). |
| `D8` / `D8_detail` | Only sent when `D6` ≠ `"None"`. "Is any defect still not fixed?" | **Always sent.** "Any outstanding handover issues you'd like us to follow up on?" `Yes` still requires `D8_detail` and should still open a ticket. |
| `E1.5` (Playa) | Commercial outlets | Beachfront commercial area (now the same item in both communities) |
| `E2.1` (Playa) | Sand cleanliness and quality | Lagoons cleanliness and water quality |
| `E2.2` (Playa) | Lagoon cleanliness and water quality | Swimmable lagoon cleanliness and water quality |
| `E2.1` (Seashell) | Sand cleanliness and quality | Pool cleanliness and water quality |
| `E2.2` (Seashell) | Lifeguards | Pool lifeguards |
| `E2.3` (Seashell) | Swimmable lagoon **or pools** cleanliness and water quality | Swimmable lagoon cleanliness and water quality |
| `E3.4` | Traffic | Traffic and parking in residential areas |
| `E3.5` | Parking | Traffic and parking at the beach |

### 2.5 No longer sent

| Key | Was | Replaced by |
|---|---|---|
| `E1.6` (Seashell) | Internal commercial area, including the sports court and kids' area | Split into `E1A.1`, `E1A.2` and `E1A.3` |
| `E1.7` (Seashell) | Seashell Walk | `E1A.4` |

### 2.6 Unchanged

- Answer keys: `A1`–`A8`, `B2`, `C1.1`, `C1.2`, `C2`, `C3`, `D0`–`D5`, `D9`, `E1.1`–`E1.4`, `E2.3` (Playa), `E2.4` (Seashell), `E3.1`–`E3.3`, `E4.*`, `E5.*`, `E6`, `E8`, `E9`, `E9_other`.
- Stored values are still English. `N/A` is still sent as `"N/A"`.
- Multi-choice values are unchanged. `B2` still sends `"App"`, `"Phone call"` and `"Site visit"`, although the screen now shows "Mobile App (OneCommunity)", "Phone Call" and "Site Visit".

---

## 3. Updating the CloudPage

The CloudPage copy differs from our 22 Oct file in four places. The new file does **not** include them, so please carry them over:

1. **`CONFIG.submitUrl`** → `"https://cloud.onecommunity-eg.com/cx-survey-2026-submit"`.
2. **`send()`**: `Content-Type: "text/plain;charset=UTF-8"` (no CORS preflight). Then read the `{ ok, code, error }` body and throw an error carrying `code` when `ok` is false. `send()` is unchanged apart from this, so the old edit applies as it is.
3. **`already_submitted`**: in the three `catch (err)` blocks of `submitSection()`, `finishEarly()` and `submit()`, add
   `if (err.code === "already_submitted") return renderState("already");`
4. **The "already completed" screen.** `renderState()` now shows one language at a time, so the text must go through `L(en, ar)` instead of being stacked EN + AR. Add `already` to `titles` / `titlesAr`, and add this entry to `views`:

   ```js
   already: `<div class="state-icon ok">${icons.ok}</div>
     <p class="lead">${L("We already have your answers for this survey, so there is nothing more to do. Thank you for your time.",
       "لقد استلمنا إجاباتك على هذا الاستبيان بالفعل، فلا حاجة لأي خطوة أخرى. شكرًا لوقتك.")}</p>
     <p class="muted">${L("You can now close this page.", "يمكنك الآن إغلاق هذه الصفحة.")}</p>`
   ```

**Fonts (new).** The page no longer uses Google Fonts. It loads 8 brand font files from a relative `fonts/` folder (`url("fonts/gdev-….woff2")` in the `@font-face` rules at the top). That path will not resolve on a CloudPage. Use one of these fixes:

- Replace `fonts/` with `https://gdev-ai.github.io/cx-survey-static/fonts/`. These URLs are live and send `Access-Control-Allow-Origin: *`, so they load from your domain.
- Or upload the 8 `.woff2` files to Content Builder and use their URLs.

**Language preference.** The page now saves the visitor's language choice in `localStorage` (key `cx26-lang`). Nothing to configure.

---

## 4. Question and flow changes (for SurveyForce)

### 4.1 Flow

- **One language at a time.** The old version showed English and Arabic together on every screen. Now there is an EN / AR switch in the header. The default is Arabic when the browser's first language is Arabic, otherwise English. Switching keeps the answers already given.
- **Part B:** was a B1 page for every unit, then a B2 page. Now it is **one page, asked once**: B1, B2 and the new B3.
- **Part D:** D6, D7 and D8 are no longer conditional. All Part D questions are always shown.
- **New G Communities intro screen** before Part E, shown only to customers with a handed-over unit. It has no questions and adds nothing to the payload.
- **Part E:** now **4 pages per community** instead of 3:
  1. E1, **E1A**, E2, E3, E4
  2. E5, E6
  3. **E7, E7.1–E7.3** (the app)
  4. E8, E9
- After each section, the confirmation screen and the closing "thank you" screen have new copy. The closing screen uses both logos only if a Part E section was completed.

### 4.2 Section headings

| Section id | Before | Now |
|---|---|---|
| A | Part A · Your experience | Part A · Your Overall Experience with G Developments |
| B | Part B · Your account | Part B · Customer Excellence & Account Services |
| C | Part C · Before handover | Part C · Your Journey Before Handover |
| D | Part D · Your handover | Part D · Your Unit Handover Experience |
| E | Part E · Your community | Part E · Your Community Experience |
| Closing | Closing | Your Feedback & Suggestions |

### 4.3 Wording changes (English; the Arabic was updated to match)

| Key | Before | Now |
|---|---|---|
| Scale | 1 = Very dissatisfied · 5 = **Completely** satisfied | 1 = Very dissatisfied · 5 = **Very** satisfied |
| N/A option | Not applicable / didn't use | Not applicable |
| `A3` | How easy **is it** to get your requests and inquiries resolved with us? | How easy **has it been** to get your requests and inquiries resolved with us? |
| `A6` | If you were buying another property, how likely would you be to buy from G Developments again? | If you were to purchase another property, how likely would you be to choose G Developments again? |
| `A7` | What is the one most important thing we could improve for you as a customer? | What suggestions do you have to help us improve your overall experience with G Developments? |
| `B1.3` | Handling of your contract and document requests (**e.g.,** resale or transfer) | Handling of your contract and document requests (resale or transfer) |
| `B3` | *(new)* | How satisfied are you with the communication, responsiveness and support provided by the Customer Excellence team? |
| `D6` | Roughly how many defects or snags did you find at handover? | How satisfied were you with the overall condition and readiness of your unit at handover? |
| `D7` | How satisfied are you with how quickly and completely those defects were fixed? | How satisfied were you with the support and follow-up provided after your unit handover? |
| `D8` | Is any defect from handover still not fixed? | Are there any outstanding issues from your unit handover that you would like us to follow up on? |
| `D9` | What is the most important improvement you would recommend for unit readiness and the handover journey? | Do you have any suggestions for improving the unit handover experience? |
| `E1A` | *(new)* | How satisfied are you with the commercial and recreational areas? |
| `E2` | How satisfied are you with the swimmable zones (pools or lagoons)? | How satisfied are you with the pools and lagoons? |
| `E7` | How satisfied are you with the OneCommunity App overall? | How satisfied are you with the OneCommunity mobile app? (rows: The OneCommunity App overall · Ease of use and navigation · Access and QR code features · Submitting and tracking service requests) |

For the E1, E2 and E3 row labels, see the tables in 2.3 to 2.5. Arabic-only wording changes (no English change) were made in A4, D4, E3, E5.3 and E9, and in Arabic community names (Seashell → سيشيل). All texts are in the `Q` object at the top of the `<script>` in `index-v2.html`.

---

## Change log since the test send

| Date (Cairo) | Commit | Change |
|---|---|---|
| 29 Oct | `be99428` | One language at a time with EN/AR switch; new intro and indicators |
| 30 Oct | `2c15811` | 28 Oct question set (Unified Design 1.1); section headings; Part B once per customer |
| 30 Oct | `a3d60fa`, `0f4eaa7` | Mobile header and sticky bottom bar; Part B on one page; motion |
| 1 Nov | `ffc2505`, `ef82504`, `cb31552` | Code tidy-up; G Communities intro screen; star rating display; thank-you copy |
| 1 Nov | `8f8960b` | Self-hosted brand fonts (`fonts/`) instead of Google Fonts |
| 1 Nov | `cf14562`, `81a095f` | App feature ratings E7.1–E7.3; commercial & recreational areas (E1A); closing message |
| 2 Nov | `c3f5214`, `6885bfd` | Seashell Walk moved to E1A.4; question set `2026-11-02`; Arabic G Communities wording |
