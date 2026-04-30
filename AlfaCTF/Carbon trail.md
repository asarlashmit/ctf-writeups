# Carbon trail

## Type

Reverse engineering / terminal UI exploitation.

## Vulnerability

`carbon.elf` is an unstripped x86-64 ELF. The login requires username
`sam_altman` and compares the submitted password against a randomized hidden
`admin_password`.

The input cursor handlers are the weakness. Left and right arrow keys update
`username_cursor` and `password_cursor` without bounds checks. `field_insert_char`
uses the cursor value for the write before checking whether it is in range.

Relevant layout:

- `admin_password` at `0x408380`
- `password` at `0x4083e0`
- distance from `password` back to `admin_password`: `0x60` bytes

By switching to the password field, moving left 96 times, and typing a chosen
15-byte password, the input writes over `admin_password`. Moving right 81 times
returns the cursor to the start of the real password field, where the same
15-byte value is entered normally. The username is set to `sam_altman`, so the
login succeeds.

After login, the flag is only displayed when the final guardrail is off and the
carbon metric is below the threshold. The guardrails screen starts with the
cursor on the final rule, so pressing `g`, space, then `d` toggles it off and
returns to the dashboard, where the flag appears.

## Exploit

```bash
python3 solve.py
```

## Flag

`alfa{nO_codEX_70_Run_nO_Fuel_7O_bURN_no_cARBoN_T0_EmiT}`
