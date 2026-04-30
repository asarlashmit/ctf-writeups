# Where in the City? Writeup

## Challenge

We are given a photo of a cyclist warming up at a city intersection and a hint:

- it was part of a multi-stage city cycling event
- the first stage was a race against the clock

Flag format: `VishwaCTF{intersection_name}`

## Recon

The image itself gives two strong clues:

1. The billboards are clearly Indian.
2. The left billboard shows performer names like `Osman Mir`, `Rahul Sharma`, and `Rahul Deshpande`, which matches Pune cultural-event advertising and points to Pune.

## Matching the event

Using the hint, the relevant event is the **Pune Grand Tour 2026**.

Multiple sources describe its opening prologue as an individual time trial starting at **Goodluck Chowk**:

- Indian Express: the prologue was an ITT starting at Goodluck Chowk on FC Road.
- Official race stages page: prologue location includes Goodluck Chowk / Deccan Gymkhana / Deccan Bus Stop.
- News coverage and race summaries also repeatedly mention the prologue beginning at Goodluck Chowk.

This matches the challenge hint exactly: a multi-stage city cycling event whose opening segment was a time trial.

## Conclusion

The intersection is:

`goodluck_chowk`

## Flag

`VishwaCTF{goodluck_chowk}`

## References

- https://indianexpress.com/article/cities/pune/traffic-changes-in-place-for-prologue-of-pune-grand-tour-2026-cycling-race-10479751/
- https://punegrandtour.in/race-stages/
- https://timesofindia.indiatimes.com/sports/more-sports/others/pune-grand-tour-kicks-off-with-high-speed-prologue-harshveer-singh-leads-indian-challenge/articleshow/126687918.cms
