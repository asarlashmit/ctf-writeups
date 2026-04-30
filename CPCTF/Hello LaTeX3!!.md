# Hello LaTeX3!!

## Challenge

The challenge provides an `expl3` LaTeX snippet with one missing command:

```tex
\seq_new:N \l_my_char_code_seq
\tl_new:N  \l_my_result_tl

\cs_new_protected:Npn \my_convert_clist_to_string:n #1
  {
    \???:Nn \l_my_char_code_seq { #1 }
    \tl_clear:N \l_my_result_tl
    \seq_map_inline:Nn \l_my_char_code_seq
      {
        \tl_put_right:Nx \l_my_result_tl { \char_generate:nn { ##1 } { 12 } }
      }
    \tl_use:N \l_my_result_tl
  }
```

## Solution

The variable `\l_my_char_code_seq` is declared as a sequence with `\seq_new:N`.
The input argument `#1` is a comma-separated list of character codes, and the next
operation maps over `\l_my_char_code_seq`.

Therefore, the missing instruction must convert the comma list into a sequence.
In `expl3`, the function for this is:

```tex
\seq_set_from_clist:Nn
```

The challenge source already includes the `:Nn` signature after the placeholder:

```tex
\???:Nn
```

So the missing placeholder itself is:

```text
seq_set_from_clist
```

## Flag

```text
CPCTF{seq_set_from_clist}
```
