# Clean contest session import

## Disclaimer
```diff
- First-off:
- YOU ARE DOING THIS AT YOUR OWN RISK. I DO NOT TAKE RESPONSIBILITY FOR ANYTHING, INCLUDING DATA LOSS.
- NEVER ask the wavelog core-dev team for help if this stuff ever breaks your wavelog.
- You knew what you were doing when you followed this guide.
```
## Why?
With wavelog 3.0.0 new contesting gets introduced. When you import old contest sessions, the importer adds very descriptive comments. Example:

```
Importiert aus Logbuch
(ADIF: EA-RTTY, Jahr: 2026) [Original: EA-RTTY]
```
But having "Importiert aus Logbuch" or "imported from logbook" in your local language as text in each and every contest session comment is superflous. So we can use a MySQL statement to clean all this up and shorten the comment a bit. Since we know when we updated to 3.0.0 we know that any QSO before that must have come through the importer. 

## Preview the changes

Run this to preview the changes. Replace "importiert aus Logbuch" with your chosen language.

```sql
SELECT
  `comment` AS alt,
  TRIM(
    REGEXP_REPLACE(
      `comment`,
      '^Importiert aus Logbuch[[:space:]]*\\(ADIF: ([^)]*)\\)[[:space:]]*(\\[Original: [^]]+\\])?[[:space:]]*$',
      '\\1 \\2'
    )
  ) AS neu
FROM contest_session
WHERE `comment` REGEXP '^Importiert aus Logbuch[[:space:]]*\\(ADIF:';
```

## Do the change

Run this statement. Of course, also replace your own language version. It will persist the changes you saw in the preview.

``` sql
UPDATE contest_session
SET `comment` = TRIM(
  REGEXP_REPLACE(
    `comment`,
    '^Importiert aus Logbuch[[:space:]]*\\(ADIF: ([^)]*)\\)[[:space:]]*(\\[Original: [^]]+\\])?[[:space:]]*$',
    '\\1 \\2'
  )
)
WHERE `comment` REGEXP '^Importiert aus Logbuch[[:space:]]*\\(ADIF:';
```
