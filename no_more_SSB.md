# Never see "SSB" as a Mode again

## Disclaimer
```diff
- First-off: YOU ARE DOING THIS AT YOUR OWN RISK. I DO NOT TAKE RESPONSIBILTY FOR ANYTHING, INCLUDING DATA LOSS.
```
## Why?
I don't like seeing "SSB" in my log. I wanna have "USB" or "LSB" in there. Why? I don't know. My inner monk doesn't like that.
Unfortunately, if you import ADIFs from contests or just... other logs, it will take whatever that log thinks it's right. Including "only SSB" without the submode, which forces me to think about changing that manually via Logbook advanced.

This is a MySQL trigger that forces the Submode depending on the band if none is set. 

## What does this do

This trigger looks at each now inserted row. 

If it has a mode of SSB, but no submode, it will set USB, unless the TX-Band is 160m, 80m or 40m. 

If you set a submode specifically (even the wrong one), it will leave it alone and take your input as is.

Since this works on the database level, it does not matter where the data comes from. It works if you enter it, it works if you import an ADIF file, it works even for Waveloggate or similar sources.

Important: This works regardless of the logged in user!!!

## Installation

Run this statement in your DB-IDE of choice. Include the DELIMITER statement on the top and the bottom.

``` sql
DELIMITER //

CREATE TRIGGER trg_bi_hrd_contacts_v01_submode
BEFORE INSERT ON TABLE_HRD_CONTACTS_V01
FOR EACH ROW
BEGIN
  IF NEW.COL_MODE = 'SSB'
     AND NULLIF(TRIM(NEW.COL_SUBMODE), '') IS NULL THEN

    IF NEW.COL_BAND IN ('160m', '80m', '40m') THEN
      SET NEW.COL_SUBMODE = 'LSB';
    ELSE
      SET NEW.COL_SUBMODE = 'USB';
    END IF;

  END IF;
END//

DELIMITER ;
```

## Deinstallation

If you wanna get rid of it and go back to standard behaviour, run this:

``` sql
DROP TRIGGER IF EXISTS trg_bi_hrd_contacts_v01_submode;
```

## Updating existing rows

If you also wanna update all existing rows to get rid of submode-less SSB forever, you can run this statement. This will apply the same logic as the trigger to all existing database rows. IRREVERSIBLE!

``` sql
UPDATE TABLE_HRD_CONTACTS_V01
SET COL_SUBMODE =
  CASE
    WHEN COL_BAND IN ('160m', '80m', '40m') THEN 'LSB'
    ELSE 'USB'
  END
WHERE COL_MODE = 'SSB'
  AND NULLIF(TRIM(COL_SUBMODE), '') IS NULL;
```
