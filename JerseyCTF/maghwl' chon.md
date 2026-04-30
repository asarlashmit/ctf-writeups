# maghwl' chon

## Category
Crypto

## Flag
`jctf{we_are_klingons}`

## Solution
The PDF contained one page of scrambled text. Extracting it with `mutool` gave:

```text
m_`_gu_e__lb_yt_D__`oIfg}lyag_yh`S_St_.`oajIQna``{a_uIthma__uSuar_o.
Sa'aab_.tnD'SlaaSySturjoom__b_Qy``_l_aDuhnn_eultm_jm_bblrq'eejhmbIv
HHgm_vhleHmweoIueu!jlcIau'_aa_aQ.nhnoaIjrr'H'_yptnHQ__
```

The challenge text gives the cipher hint:

- 7 Klingon ambassadors -> use 7 rows.
- "hit with a box 23 times" -> Caesar Box cipher. The 23 is the Julius Caesar clue.

After removing whitespace, the ciphertext length is 189, which is `7 * 27`. Put the text in a 7-row box and read down the columns:

```python
from pathlib import Path

ct = ''.join(Path('note.txt').read_text().split())
rows = 7
cols = len(ct) // rows
grid = [ct[i * cols:(i + 1) * cols] for i in range(rows)]
pt = ''.join(grid[r][c] for c in range(cols) for r in range(rows))
print(pt)
```

This produces readable Klingon transliteration:

```text
matlhHa_ghanHa`_mang__yaS_magh_ye_Qu`_Suv._Sutlhne_Sutlh_Surmen_taj_Hol_rojmab._omwI_`om_ejyo._bortaS_bIr_jablu'DI'_reH_QaQqu'_nay'!_`ab`ejyo`_`elpI`._jctf{tlhIngan_maH}_DabuQlu'DI'_yISuv__
```

The flag-looking part is `jctf{tlhIngan_maH}`, but that is still Klingon. The phrase `tlhIngan maH` translates to "we are Klingons." Submitting the translated phrase in the expected JerseyCTF flag format gives:

```text
jctf{we_are_klingons}
```
