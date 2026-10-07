# Expected high-level matches

This is a sanity-check guide rather than a byte-for-byte output fixture.

## 01_dataleak.txt
Expected configured owners to appear:
- Acme Industries
- Northstar Bank
- BlueRail Logistics
- CloudBox SaaS (third party of Acme Industries)
- PayBridge (third party of Northstar Bank)
- TrackOps (third party of BlueRail Logistics)

`evilacme-industries.example` and `example.org` must not match.

## 02_blocks.log (block mode)
- BLK-001 -> Acme Industries
- BLK-002 -> CloudBox SaaS / Acme Industries third-party tree
- BLK-003 -> Northstar Bank + PayBridge
- BLK-004 -> no output
- BLK-005 -> BlueRail Logistics + TrackOps

## 03_records.csv
Matches: Acme Industries, CloudBox SaaS, Northstar Bank, PayBridge.

## 04_dump.sql
Matches: BlueRail Logistics, TrackOps, Acme Industries.

## 05_records.json
Matches: Northstar Bank, PayBridge, CloudBox SaaS.

## nested/06_nested.txt
Matches: BlueRail Logistics and TrackOps. Confirms recursive discovery.

## 07_workbook.xlsx / 08_workbook.xlsm
Each workbook contains matches for all three direct customers and all three third parties, plus deliberately unrelated values that must not match.
