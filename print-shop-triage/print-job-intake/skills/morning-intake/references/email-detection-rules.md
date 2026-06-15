# Email Detection Rules

## Gmail Account

- **Account**: gopostalsd@gmail.com
- **Target**: unread emails that contain new print job requests

## Known Client Senders

Match ANY of these senders (email address or domain) as high-confidence print job candidates:

| Client | Known email(s) / domain |
|--------|--------------------------|
| Harbor City Church | omar@harborcity.church |
| Kettner Exchange | nikki@kettnerexchange.com |
| Loma Club | ellis@thelomaclub.com / thelomaclub.com |
| Queenstown Bistro | atzinrdz24@gmail.com, larry@queenstownbistro.com |
| Music Box / Belly Up | marketing@musicboxsd.com, ty@musicboxsd.com |
| SDCM group (Grass Skirt, Captain's Quarters, etc.) | giancarlo@sdcm.com, bella@sdcm.com, jenny@sdcm.com |
| Gaslamp District Media (Havana, El Chingon, Prohibition) | erin@gaslampdistrictmedia.com |
| Remedy Pharmacy | samantha@remedyrx.com |
| Queenstown Public House | ryan@queenstownpublichouse.com |
| Crudo / Barra Oliba / Civico 1845 (Eduardo group) | paulojgarza17@gmail.com, pietro@civico1845.com |
| Isola | raffaele.da88@gmail.com |
| Loré Experiential | hello@loreexperiential.com |
| Bri Rahoy (Grind & Prosper) | bri@grindprosper.com |
| ARC (large format vendor) | valeria.barron@e-arc.com |

If an email comes from a domain or address not on this list, use subject/body content to judge.

## Subject Line Keywords (Print Job Signals)

Treat these subject keywords as positive indicators of a print job:

`print`, `menu`, `menus`, `poster`, `posters`, `flyer`, `flyers`, `flier`, `fliers`, `bulletin`, `bulletins`, `card`, `cards`, `sign`, `signs`, `signage`, `bookmark`, `bookmarks`, `order`, `labels`, `stickers`, `handbill`, `handbills`, `inserts`

## Positive Body Signals (New Job Request)

An email is a new print job if the body contains ANY of:
- Explicit quantities: numbers followed by units ("35 menus", "qty 100", "150 copies", "(50)")
- Explicit sizes: dimension patterns ("8.5x11", "4x6", "11x17", "8.5''x14''")
- Request language: "could we please", "we need", "I need", "please print", "can you print", "please get the following printed", "we'd like", "can we get"
- Material/finish specs: "laminated", "cardstock", "polyester", "double-sided", "front and back"
- File attachment reference: "see attach", "attached", "attached pdf", "file attached"

## Negative Signals (NOT a New Job — Skip)

An email is NOT a new job if the body is primarily:
- A thank-you or pickup confirmation: "thank you", "you're amazing", "will pick up", "picking up tomorrow"
- Billing or payment: "charge", "charges", "invoice", "billing", "payment", "claim"
- Cancellation: "scratch this", "cancel", "never mind", "we had it printed"
- A status check: "ready?", "is it ready", "when will it be ready"
- Michael's own outgoing email (From: gopostalsd@gmail.com or gopostal28@gmail.com)

## Ambiguous Cases

If an email matches positive keywords but the body is unclear or missing specs:
- Flag it in the summary report
- Do NOT create a Notion entry
- Note it for Michael to review manually

## Gmail Search Query

Use this search in Gmail to surface candidates each morning:

```
is:unread (subject:print OR subject:menu OR subject:menus OR subject:poster OR subject:flyer OR subject:flier OR subject:bulletin OR subject:card OR subject:sign OR subject:signage OR subject:bookmark OR subject:order OR subject:labels OR subject:inserts OR subject:handbill)
```

Then separately check for unread emails from known sender domains not caught by subject:

```
is:unread (from:harborcity.church OR from:kettnerexchange.com OR from:thelomaclub.com OR from:queenstownbistro.com OR from:musicboxsd.com OR from:sdcm.com OR from:gaslampdistrictmedia.com OR from:remedyrx.com OR from:queenstownpublichouse.com OR from:loreexperiential.com OR from:grindprosper.com OR from:e-arc.com)
```

Merge both result sets, deduplicate, then evaluate each one.
