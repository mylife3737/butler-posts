# Cynthia Butler Campaign — Social Posting Dashboard

Reference for anyone operating this folder.

## Files

- **`postingstrategy.html`** — main dashboard. Open in browser directly, no build.
  Contains `allPosts` array (~112 entries), banners, ticker, today card, buttons that copy pre-written post text to clipboard.
- **`postlist.html`** — flat table view. Read-only companion. Time column shows when to post + Nextdoor timing hint.
- **`HANDOVER.md`** — this file.

Both HTML files are single-file: Tailwind CDN + vanilla JS. No install, no server.

## Post schema (`allPosts` entries)

```js
{
  id: 42,
  sortDate: '2026-07-23',   // YYYY-MM-DD, ISO
  date: 'Jul 23',           // display
  type: 'personal' | 'group' | 'both' | 'event',
  nd: 'public' | 'group',   // optional — Nextdoor version
  tip: 'short label',
  instr: 'operator instruction',
  text: 'the post copy for FB',
  ndText: 'the post copy for Nextdoor',   // optional
  venue: '...', time: '...'               // event fields
}
```

**Always use straight quotes in string literals.** Smart/curly quotes (`'` `"` `"`) inside `allPosts` cause `SyntaxError` and blank the page.

## Best times to post (source of truth)

Cheat sheet: `~/.claude/paste-cache/b0ba64f77efc95f5.txt`

| Day | FB window | ND window |
|---|---|---|
| Mon | 12–1 PM | — |
| Tue | 9 AM–12 PM (also 5–6 PM) | — |
| Wed | 9 AM–12 PM (also 5–6 PM) | — |
| Thu | 9 AM–12 PM (also 5–6 PM) | **5–7 PM** |
| Fri | 10 AM–12 PM | **5–7 PM** |
| Sat | 8–10 AM | — |
| Sun | Soft reach (10 AM) | — |

District 75 retiree peak: 7–10 AM. Professionals: 12 PM + 7–9 PM. Young voters (Reels): 8–11 PM.

`postlist.html`'s `getPostTime()` / `getTimeDisplay()` already encode this — no need to recompute.

## Nextdoor rule (strict)

- Post **Thu or Fri only**, **5–7 PM**.
- **1–2 posts per week max.**
- **Zero national talking points.** House District 75 issues only or Nextdoor flags as spam.
- Any post with `nd:` on the wrong day → `postlist.html` shows a green `ND → Thu/Fri 5–7 PM` hint next to the time. Move it or drop the ND version.

## Facebook rule

- 1–2 main feed posts/day.
- Reels mix: 40% policy nuggets · 30% precinct walking · 20% CTA · 10% comment replies.
- Reels: 15–30s, vertical, hook in first 3s, **captions mandatory**.

## Compliance (2026)

- Any AI-generated realistic audio/video → label **"AI Info"**.
- **Ad blackout starts Oct 27, 2026** — no new political ads after that.
- Every ad must show **"Paid for by Cynthia Butler Campaign."**
- 2FA on every campaign account. Verify identity via Meta Business Suite before boosting anything.

## Editing `postingstrategy.html`

1. Find `const allPosts = [` (~line 500-ish).
2. Add / edit entries. Straight quotes only.
3. Save.
4. **Re-sync `postlist.html`** with the script below.

## Re-sync script (run after any `allPosts` change)

```python
import re
with open('/home/isthismylife37/posting schedule/postingstrategy.html', 'r') as f:
    lines = f.readlines()
start = end = None
for i, line in enumerate(lines):
    if 'const allPosts = [' in line and start is None: start = i
    if start is not None and line.strip() == '];' and end is None: end = i; break
block = ''.join(lines[start:end+1]).strip()
with open('/home/isthismylife37/posting schedule/postlist.html', 'r') as f:
    src = f.read()
out = re.sub(r'const allPosts = \[.*?\];', lambda m: block, src, flags=re.DOTALL)
with open('/home/isthismylife37/posting schedule/postlist.html', 'w') as f:
    f.write(out)
```

The `lambda m: block` matters — a plain string replacement lets `\n` sequences in post text get interpreted as escapes.

## Common gotchas

- **Blank page after edit** → check the browser console. 99% of the time it's a smart quote or a missing comma in `allPosts`.
- **`postlist.html` out of date** → you forgot the sync script.
- **Wrong day showing green ND hint** → that's the point; move the ND version to Thu/Fri.
- **Event posts** always time as 7:00 AM. `id:56` and `id:107` are pinned to 7:00 PM (evening events).
