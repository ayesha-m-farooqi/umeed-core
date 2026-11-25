# umeed-core

**An accessibility toolkit written entirely in Quantum (`.sa`).**
It helps people who find reading, speaking or controlling an app difficult, and it gives developers small, reusable building blocks for accessible software.

*Umeed* means "hope" in Urdu. This package is the logic core of Project Umeed, a voice-first accessibility app for people who cannot use a normal touchscreen.

---

## The problem

Most apps assume the user can see small text, read quickly, speak clearly and tap precisely. People with visual, hearing, speech or mobility needs are often left out.

## Who it helps

- **People with accessibility needs**, who get a profile of recommended features, text that is easier to read, a phrase board for when speaking is hard, and a calm emergency flow.
- **Developers**, who can import small functions that need no network, no database and no setup.
- **Caregivers and teachers**, who can check how readable a text is before sharing it.

## What is in the package

| Module | What it does |
|---|---|
| `lib/profile.sa` | Builds an accessibility profile and recommends features |
| `lib/text.sa` | Plain-language checker: Flesch Reading Ease, sentence length, tips |
| `lib/reading.sa` | Reading aids: one sentence per line, word wrapping, framed large text |
| `lib/phrases.sa` | A phrase board for people who find speaking hard |
| `lib/emergency.sa` | The emergency alert flow as plain logic (pending, confirmed, cancelled) |
| `lib/contacts.sa` | Trusted contacts: validation, priority order, who to tell first |
| `umeed.sa` | A menu-driven demo app that uses every module |
| `index.sa` | The package entry point. Importing it gives you every function |

## Install

```
qpm install umeed-core
```

## Quick start

Run the demo app from the project folder:

```
qrun umeed.sa
```

Use the functions in your own file:

```
# @use ./index.sa

let profile = create_profile("Sara", true, false, false, true)
for rec in recommend_features(profile) {
    print("- " + rec)
}

let report = analyze_text("Please sit down. The doctor will see you soon.")
print(report["level"])
```

## Function reference

### Profile (`lib/profile.sa`)

| Function | Returns |
|---|---|
| `create_profile(name, visual, hearing, speech, mobility)` | A dictionary with the name and four true/false needs |
| `parse_yes_no(answer)` | `true` for yes or y, `false` for no or n, `nil` for anything else |
| `recommend_features(profile)` | A list of recommended features. With no needs selected it recommends the standard experience |
| `yes_no_text(value)` | `"Yes"` or `"No"` |

### Plain-language checker (`lib/text.sa`)

| Function | Returns |
|---|---|
| `analyze_text(text)` | A dictionary: `words`, `sentences`, `syllables`, `long_words`, `avg_sentence_length`, `reading_ease`, `level` |
| `writing_tips(analysis)` | A list of short, friendly tips |
| `reading_level(score)` | A name for a Flesch score, from "Very easy" to "Very difficult" |
| `split_words(text)`, `split_sentences(text)` | Lists of cleaned words and of sentences |
| `count_syllables(word)` | An estimated syllable count |

### Reading aids (`lib/reading.sa`)

| Function | Returns |
|---|---|
| `one_sentence_per_line(text)` | A list with one sentence per entry |
| `wrap_text(text, width)` | A list of lines no longer than `width` |
| `spaced_letters(text)` | The text with a space between letters, to help with tracking |
| `framed(text, width)` | Lines for a large, upper-case phrase inside a frame |

### Phrase board (`lib/phrases.sa`)

| Function | Returns |
|---|---|
| `phrase_board()` | The list of phrases |
| `get_phrase(n)` | Phrase number `n` (starting at 1), or `nil` if it is out of range |

### Emergency flow (`lib/emergency.sa`)

An alert starts as `pending` with a 10-second countdown. If the person answers, it becomes `cancelled`. If the countdown reaches zero, it becomes `confirmed`. Alerts are never changed in place: every function returns a new alert.

| Function | Returns |
|---|---|
| `create_alert(name, kind)` | A pending alert. `kind` is `"sound"` or `"manual"`. Returns `nil` for a bad kind or an empty name |
| `tick(alert)` | The alert after one second, confirmed when the countdown ends |
| `respond_ok(alert)` | A cancelled alert, if it was still pending |
| `confirm_now(alert)` | A confirmed alert, skipping the countdown |
| `simulate_countdown(alert, responses)` | The final alert after a list of true/false answers, one per second |
| `is_final(alert)` | `true` once the alert is confirmed or cancelled |
| `alert_reason(alert)`, `alert_message(alert)` | Short sentences to show on screen or read aloud |
| `countdown_seconds()` | The countdown length (10) |

### Trusted contacts (`lib/contacts.sa`)

| Function | Returns |
|---|---|
| `create_contact(name, phone, relation, priority)` | A contact, or `nil` if the name, phone or priority is invalid. Priority 1 is called first |
| `is_valid_phone(phone)`, `clean_phone(phone)` | Phone check (7 to 15 digits) and cleaning (`"+92 300-1234567"` becomes `"+923001234567"`) |
| `sort_contacts(contacts)` | Contacts by priority. Equal priorities keep their order, and `nil` entries are skipped |
| `first_contact(contacts)` | The contact to call first, or `nil` |
| `contact_names(contacts)`, `join_names(names)` | Names in priority order, and a "A, B, C" string |
| `call_link(contact)` | A dialable link such as `"tel:+923001234567"` |
| `notify_plan(contacts, hospital)` | `{"contacts": [...], "hospital": ...}`, the list of who to tell |

## Run the tests

```
qrun tests/test.sa
qrun tests/emergency_contacts_test.sa
```

Each check prints `PASS` or `FAIL`.

## Design notes

- **Pure logic.** The package decides what should happen. It does not call anyone, send messages or open a network connection. An app using it does the actual calling and notifying.
- **No hidden state.** Functions return new values instead of changing the ones you pass in, which makes them easy to test.
- **Small and readable.** Each module is a single file you can read in a few minutes.

## Limitations

- The **syllable count is an estimate** (it counts groups of vowels), so reading scores are approximate. The score is designed for English.
- **Sentences are split at full stops, exclamation marks and question marks.** Abbreviations, decimals and file names can be split incorrectly.
- The **phrase board is a fixed English list**, and there is no speech or audio output in this package.
- The **emergency flow is a model, not an emergency service.** It must not be the only way a person can get help, and it does not replace calling the local emergency number.
- This is accessibility tooling, not a medical device, and it makes no medical claims.

## Part of Project Umeed

The full Project Umeed app (voice assistant, hospital registration, learning centre and API) is a separate project. This package contains the parts of it that can be shared as plain logic.

## License

MIT. See the `LICENSE` file.
