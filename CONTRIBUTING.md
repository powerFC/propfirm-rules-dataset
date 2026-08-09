# Contributing

Corrections and additions are welcome. There is one hard rule.

## Cite the firm, not a comparison site

A correction must reference the firm's own documentation — its rules page, plan comparison table, help centre or terms. Links to review sites, affiliate blogs, YouTube videos or Discord messages are not evidence, because that is exactly where the errors this dataset exists to correct come from.

If you have first-hand experience that contradicts the firm's published rules (for example, support told you something different), that is genuinely valuable — open an issue describing it, and it will be recorded as a note rather than changed as a value.

## Opening an issue

Include:

1. Firm and plan name
2. The field that's wrong and what it should be
3. A link to the firm's own page stating it
4. The date you checked

## Opening a pull request

- Edit `data/prop-firm-rules.json` only. The README tables are generated from it by hand; they'll be updated to match.
- Update `verified_date` to the date **you** checked, and set `verified: true` only if you read it from the firm's own documentation.
- Do not fill an `unconfirmed` field with a plausible value. Leaving it unconfirmed is more useful than a guess.
- Add a line to `notes` if the plan has a quirk that doesn't fit the schema. The notes are often the most useful part of an entry.
- Validate before submitting: `python -c "import json; json.load(open('data/prop-firm-rules.json'))"`

## Adding a firm

Most useful additions, roughly in order:

1. **Funded-phase rules for firms already covered.** Payout minimums, qualifying days, payout caps, funded-stage consistency. This is where most payout denials happen and where the dataset is thinnest — see [`docs/known-gaps.md`](docs/known-gaps.md).
2. Missing plans from firms already listed.
3. New firms with a real payout history.

New firms with no payout track record can be added, but mark them clearly in `notes`. A firm that has never paid anyone is not comparable to one that has paid out for four years.

## What won't be merged

- Rankings, scores, or "best firm" opinions. This is a rules reference, not a review site.
- Referral or affiliate links.
- Rule values with no source.
- Firms that exist only as a landing page and a Discord.
