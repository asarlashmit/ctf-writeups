# Civil Aviation

## Challenge Overview
- Event: CyberGeek CTF
- Category: OSINT / Aviation
- Final Flag: geek{january2022}

## Executive Summary
The solve depends on reading the departures board carefully and correlating the visible flight numbers and scheduled times with historical airline schedules. The IX135 timing is the decisive clue, which pins the photo to January 2022.

## Solution Walkthrough
### Approach
The JPEG metadata was stripped, so the solve had to come from the flight board itself. The visible board is a Delhi Terminal 3 international departures/check-in board: the rows include QR583 to Hamad/Doha, QF68 to Sydney, NH838 to Haneda/Tokyo, IX135 to Sharjah, SV761 to Riyadh, and other Gulf departures.

Key visible rows:

- QR583, Delhi to Doha, STD 17:00
- QF68, Delhi to Sydney, STD 17:55
- NH838, Delhi to Haneda/Tokyo, STD 18:45
- IX135, Delhi to Sharjah, STD 18:55, ETD 20:45

QF68 was the first strong narrowing clue because Qantas QF68 was operating Delhi to Sydney at 17:55 in early 2022. That limited the useful search window to the Delhi-Sydney Qantas service period.

The deciding clue was IX135. The board shows IX135 with a scheduled departure of 18:55. Historical Air India Express IX135 pages show that 18:55 Delhi-to-Sharjah departure in January 2022, while the checked February and March 2022 schedules had different scheduled times. Therefore the photo month and year are January 2022.

## Flag
```text
geek{january2022}
```

## References and Notes
### Sources Used
- QF68 January 2022 Delhi-Sydney history: https://www.flightera.net/en/flight/Qantas-Delhi-Sydney/QF68/Jan-2022
- QR583 January 2022 Delhi-Doha history: https://www.flightera.net/en/flight/Qatar%2BAirways-Delhi-Doha/QR583/Jan-2022
- IX135 January 2022 Delhi-Sharjah history: https://www.flightera.net/en/flight/Air%2BIndia%2BExpress/IX135/Jan-2022
- IX135 February 2022 comparison: https://www.flightera.net/en/flight/Air%2BIndia%2BExpress/IX135/Feb-2022
- IX135 March 2022 comparison: https://www.flightera.net/en/flight/Air%2BIndia%2BExpress/IX135/Mar-2022
