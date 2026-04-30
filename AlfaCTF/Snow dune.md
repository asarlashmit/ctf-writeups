# Snow dune

## Type
Web

## Root cause
The challenge does not require recovering the color at all.

In [`snowpile/snowpile/app/main/views.py`](/home/kali/ctf-orc/ctf/snow-dune/snowpile/snowpile/app/main/views.py:186), `/api/check_color` verifies the submitted color, but it always includes the flag in the JSON response:

```python
correct = guessed_color.upper() == current_color.upper()
r.delete(_pending_color_key(g.player_id))

if correct:
    t = get_latest_submission_tokens(g.player_id)
    try_record_scoreboard(g.username, t)

flag = "alfa{XXXXXXXXX_CENSORED_XXXXXXXXX}"

return jsonify({
    'correct': correct,
    'guessed_color': guessed_color,
    'flag': flag
})
```

The frontend only displays `response.flag` when `response.correct` is true, but the backend leaks it unconditionally.

## Exploit
1. Log in with any username.
2. Call `/api/check_prompt` once with any prompt. This creates a pending color for the session.
3. Call `/api/check_color` with any wrong value, for example `#000000`.
4. Read `flag` directly from the JSON response.

## Local verification
I reproduced the flow locally on April 25, 2026 with the provided source:

```text
login 200 {'redirect': '/'}
check_prompt 200 {'job_id': 'a401f9fc-03ea-42dd-b3d7-7ecc2580e634', 'message_id': '446fdf239e9b4d8f86f24847752d596d', 'queued': True}
check_color 200 {'correct': False, 'flag': 'alfa{XXXXXXXXX_CENSORED_XXXXXXXXX}', 'guessed_color': '#000000'}
```

## Automation
Use [`solve.py`](/home/kali/ctf-orc/ctf/snow-dune/solve.py) against a working instance:

```bash
python3 solve.py https://snowdune-e15mar.alfactf.ru
```

## Blocker
On April 25, 2026, the provided host `https://snowdune-e15mar.alfactf.ru/` returns a generic nginx `404 Not Found`, so the real uncensored flag could not be retrieved from the live service in this environment.
