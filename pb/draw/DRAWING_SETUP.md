# Winner drawing setup

The new page is `draw.html`. It intentionally stays separate from the public
fundraiser page and is marked `noindex`.

## Apps Script change

1. Open the Apps Script project currently deployed at the URL used by
   `jar.html`.
2. Copy the helper functions from `apps-script-donor-addon.gs` into that
   project.
3. In the existing `doGet(e)`, store its current response object in a variable
   named `response` and add this before the response is serialized:

```javascript
if (e && e.parameter.include === 'donors') {
  response.donors = getDrawingDonors_();
}
```

4. If the donor data is not on the spreadsheet's first tab, set
   `DRAWING_SHEET_NAME` to that tab's exact name.
5. Deploy a new version of the existing web app. Keep the same deployment so
   its `/exec` URL does not change.

The helper recognizes these headings (case-insensitive):

- Donor: `Entrant`, `Donor`, `Donor Name`, `Name`, `Contributor`, or `Contributor Name`
- Entries: `Entries`, `Entry`, `Entry Count`, `Number of Entries`, or
  `Raffle Entries`

Duplicate donor names are combined. Blank names, zero entries, negative
entries, non-numeric entries, and the configured `Nick and Michele` entrant
are excluded.

## Expected JSON

The existing progress fields remain unchanged. A request with
`?include=donors` additionally returns:

```json
{
  "title": "Peanut Butter Love",
  "goal_lbs": 159,
  "total_lbs": 229,
  "entries_count": 20,
  "updated": "2026-10-10T22:48:17.447Z",
  "donors": [
    { "name": "Example Donor", "entries": 4 }
  ]
}
```

## Drawing behavior

The browser first picks one ticket using `crypto.getRandomValues()`, then maps
that ticket to a donor according to entry count. The cycling label is only the
reveal animation; it cannot change the already-selected result. The button is
disabled while a draw is running, and reduced-motion preferences are honored.
