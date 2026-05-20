# Powerlimit for 60m

## Disclaimer
```diff
- First-off: YOU ARE DOING THIS AT YOUR OWN RISK. I DO NOT TAKE RESPONSIBILTY FOR ANYTHING, INCLUDING DATA LOSS.
```
## Why?
On the 60m band, users in Germany have a power limit of 15W EIRP, which comes to about 9W PEP on my setup. 

While I do set this powerlimit on my Transceiver like the rule-abiding citizen that I am, I sometimes forget to change it in the WSJT-X logging window (and sometimes forget to change it back for the other bands).

## What does this do

This trigger looks at each now inserted row. 

If the band is 60m, it looks at the given power. If it is above 9W, it sets 9W. If it's below, take the users input.

Since this works on the database level, it does not matter where the data comes from. It works if you enter it, it works if you import an ADIF file, it works even for Waveloggate or similar sources.

Important: This works regardless of the logged in user!!!

## Installation

Run this statement in your DB-IDE of choice. Include the DELIMITER statement on the top and the bottom.

``` sql
DELIMITER //

CREATE TRIGGER trg_bi_hrd_contacts_v01_60m_tx_pwr
BEFORE INSERT ON TABLE_HRD_CONTACTS_V01
FOR EACH ROW
BEGIN
  IF UPPER(NEW.COL_BAND) = '60M'
     AND NEW.COL_TX_PWR > 9.0 THEN

    SET NEW.COL_TX_PWR = 9.0;

  END IF;
END//

DELIMITER ;
```

## Deinstallation

If you wanna get rid of it and go back to standard behaviour, run this:

``` sql
DROP TRIGGER IF EXISTS trg_bi_hrd_contacts_v01_60m_tx_pwr;
```
