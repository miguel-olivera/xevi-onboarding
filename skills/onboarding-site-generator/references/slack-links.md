# Resolving people to Slack links

Every named person in the page (buddy, peopleGroups, module `poc`) should
link to their real Slack profile, not an email or a guess.

1. Use the Slack connector's user search (e.g. `slack_search_users`) to find
   each person by name and get their member ID (`U0...`).
2. Build the profile URL as `https://deliveryhero.enterprise.slack.com/team/<ID>`.
3. If someone can't be found (name typo, contractor not on this Slack, etc.),
   don't fabricate an ID. Leave the person out or flag it to whoever
   requested the page — a broken/wrong Slack link is worse than no link.

**Known limitation:** this opens the person's Slack *profile*, not a direct
message — the visitor still has to click "Message" once. Getting a true
one-click DM link would need the workspace's Team ID or the DM channel ID,
neither of which is obtainable without actually sending a message first.
`/app_redirect?channel=<ID>` was floated as an alternative during the first
build of this page but was never verified against a real browser — worth
testing if this becomes a recurring complaint.

## Buddy avatar

The buddy's photo is embedded as a `data:image/...;base64,...` URI so the
final page has zero external requests (`buddy.avatarDataUri` in
`data-schema.md`). If the same person has already been someone's buddy in a
previous onboarding page, you can lift the data URI straight out of that
file's `hero__buddy-avatar` `background-image` instead of re-encoding a photo.

## Photos for people in the page (the `PHOTOS` map)

Every person row and point of contact shows a round photo when their Slack ID
is a key in the `PHOTOS` map inside `assets/template.html` (key = Slack member
ID, value = `data:image/jpeg;base64,...`); otherwise it falls back to initials.
After resolving IDs, check which ones are missing from `PHOTOS` and fill the gaps:

1. Get the photo URL from Slack: `slack_search_users` with
   `response_format: "detailed"` returns a `Profile Pic` link
   (`https://avatars.slack-edge.com/...`). `slack_read_user_profile` does NOT
   include it. If a name search returns other people (common names), search by
   a distinctive word plus the title (e.g. keywords `["Director","Subscription"]`)
   and pick the right ID. Searching by email returned nothing.
2. Download each into its own scratch folder (`curl -sfL`), and shrink/convert
   with `sips -s format jpeg -Z 160 in --out ID.jpg` (about 8-12 KB each).
3. Add `"<ID>":"data:image/jpeg;base64,<...>",` to `PHOTOS` in the template
   (skip IDs already there), then rebuild the page. Keep the page a single
   offline file: never reference the Slack URL directly.
4. If a person has no real photo (Slack returns a gravatar/default avatar),
   leave them out and let initials show.
