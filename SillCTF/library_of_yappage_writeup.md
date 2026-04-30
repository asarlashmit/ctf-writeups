# Library of Yappage

Type: Crypto

## Summary

The long `0-9a-z` string was not a direct ciphertext to decrypt into plaintext. The challenge title and the journal clue pointed to the [Library of Babel](https://libraryofbabel.info/) location format:

- `W:4` -> wall 4
- `S:3` -> shelf 3
- `V:25` -> volume 25
- `64` inside the triangle -> page 64

The provided short URL led to a public OneDrive folder named `Library of Yappage` containing `journalnonsense.png`, which supplied those coordinates.

## Recon

The short URL was protected by Cloudflare for normal `curl`, but it still exposed the final redirect target with a browser-like request:

```bash
python - <<'PY'
import requests
ua='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36'
r=requests.get('https://www.shorturl.at/i8ZUn', headers={'User-Agent': ua}, allow_redirects=False)
print(r.headers['location'])
PY
```

That redirect pointed to a public SharePoint folder. The folder page contained `journalnonsense.png`, and the image showed the Library of Babel coordinates.

## Solve

Use the challenge string as the hex/book id and open the Babel page:

```text
book.cgi?<ciphertext>-w4-s3-v25:64
```

In practice:

```bash
python - <<'PY'
import requests

ct = "32poj8wulkindvnv9mbg7dqkfymcjnb9h7j5u22nkv4fdrmxi5ojhhe89ibeefbrcntpm003zp561uykl5cvid1yixovurdhu5x4rttbkgrjlrrg9hdgl3vjn7af8w1kmhmi4d08b3fxj1xzhnz8zszcyqrujkvrtf8fgud036ounnkw8wzjhtzgpp20zdxo09ic0xwzhpl5fhzhg1i7zr4pbwn6pf5dcse3bsmdyl7yljzuerf91nuu6jlxyciidsxi4l8ifm1ef1g2vn2kc9izsq1pipnfkashiqmml4qxaln16mq7re4y1imyxxieezkd8dezwdg4lmnrvultv07dt42a8609qj8q17tn9iphe4umv20am6s8n5pfb1ldzx7mfh8hnw14dbwd8lur4fvs4l7wjw49bb6kwhz41glkmxpre9l38pgyemmz3h5yol6tm5w6674cbcju6jiq4apt3xlfn3t5vatjv2zabw0mheymy2rqqcuzvofavh8xuo2hlu5h2j166qdbkmmqpy84h0ymp0jow6f9a2q5amymjsr0pjzqghpa5y1aikfhyik1xthizhq6m6w5p956szirqzpa3n2ydoxnokbn5p0jdm17oihchtfhipo9vhv1id9ge9dn07tu8gift32nfzy6qi03kirn4a9ag06v6sa3lu98vgf46g81b27bg5woo14i823abes6g89vajpkobrgtv68wph0lkd5h5plkqxg8zi8td3j4mct5eppjk01hd1vvkb5hhj4yaakg4szonqflhuwms0cnrswxlzwpu0i42v7araneh0of63ji4k04bmal16gc4w4l8ps9yxz0u7mdn9xz6cf362fkhlan8o22ofpoynmi6jlv945d3ue8c5hwdbepalf98ynvoauqdzwfaguwd8b9n3uxxhgy0zfp3x5hastrk3dvf4000iwj4gxvvufh3u0ithcw27nn68c0j9ldn96ridiesay6bt8swhdbktseqvami4eqmrqpuf74oh6yptqcs80nd0w7n5kf24j13mswfrzhn3992434etfycj1428dcf5a83t79t1fok6b2k0dy6kapqd8ruyr841jt1xf3sya2ah0nvw5c4h4byf7qguceeophpc6r36y2auwu2w4luz7vy6d8ep8kypcwwswlce3xxf090vhym8a405sw539cqpq5o2t78xada3nuv0jxoorcjprv1tj3mykjwovzaciptb0cxnkzjxdqsrxif3ikh7e5d48atsrf4mhx8rlaepvkx1a6lsjs2fqa0vb9vo0pz6izp3bx73ibgem9yrb3rnreamd0kshxbb5c65pft9n148jw8tyew08lz7b0x19qsw98xl5nugloaqqz856l23yofm3b1wov07mqcm5zz3mn5gudb5ipu5jlwk61s6doktqe32kzjw9xatn02vnzftf3nfdw252wcxgprjcvfzyb68ivqfflxag9leo2dpty816h1op15yylakmp9vqzwzrxylun92x0pn0nhb90p3nn7j2lgwqdav0zns0pwakxiafulvykmfr0qtxudowt7of3fy20b5gqqj5dca0cpu5ik1f9q6gmxtn60exk9pxg1cbsfo8itu1kigx8czaktjq8l8q6k9mt6oyw155pqfpgyobn5s46lglg804pxxa82dmd089poafw7yy9wge3d36amlt7jg8bjjkbwrs7xhpn672zlbo1k5if0vww4gylvr3m7j5xjv31l4tuqqrm6wvma3e3ylmmws7vcvyg1243dzg8w1c98gzj491y04q9c29rdi6kwcqfozwltac45gsftw2xx1c6tcvl8by2gzxqa3gqv0thm01aefjkb08kphz5tb22bke39le9sjsntr31x0nubzbp71j332vf2my84vgd7ayq26wcug4cpg239dubw5thtm4s1udbr5zsuvr7fqzdapjfgt7d3ps0713rxgiikf8ywbpcmxv020bi0iqo20s114h1ubodqwnsxkhjsfu1h0qvjnh8bt0g59qw63xuqvzhfb4vi3undlwhqkuwlkyturb6ql51uyc2es0yl2p2ekm87jsipy9nh6rposl6pew1foilbzr56guq1uzj22afwsoibmodfpsunw5q2cvvh21hv97t6onqv3x5m58ta54aimyaal5ml67kec2t3jqg856gwx73t5ul6mco3jc2kmi6l8fls9j15sdoitoojm9h3f3jqsounbha74xbap87m5gr6kxc1yk120lihkvdcfwyoa9e6neyoa8bditaowmnxnmj3boknxwjh06xjczb582mhxiein5k2bd1zblkqlz512g8fguzqszfj66ka0nd0q01x71q5j24itemx8omm6t92oxpf89z972yvqolty5vog2o73rk7ii9djcp5knmsd2ehejjjob2xl3i285slpco7w3lwp86i0qpov8f4tpfk2zzf2fg5u31k3rcyrdowkkmy0gpoudqzfqpfp6jfymu7af3xiuin2r7feg36a6vn5jhbluw6c25h0oy6wddtgbyagju6wgwclbqbwbaq4jcxysyspfwrvw4dkyhzf3sa77eom20ieyhm2gom6xxhlnhiimlrbxb4uzrx2530rur3xvkb9smupc1c8oblfs6avyssy5b2b5p6tf900ifdldpbu74jspryus517adf2jd5us3v2a8v09du9n8sx77ca7oc8fhh7i5s3v2pmmxv0nwvm2nzq363l4akiyywjxt9dfjyofatvegfezefxu5pd86vjgda0n9qea93j75f0q1senimaatkffduups4ch13sk8a3dtdhnd8idtzl8q2ilqwcsa5qnv6xk47yavybghrf1jhier0w9uko8o75xcxdby05qiqulk72l7nwju42051zsk5yclitk4hxl0vb3cxnlygi38gj1y84wkf9s5xyc0c0x7tt6fre0wlmp5k4d071vai6xm03f1y9porf9ux26g9iho3gcj32ssnz25h42205cq4y4wpbqqhs6smceom81bgnzul54avj4k7gdfwgxgubllz98yef1etyrrefn15d6dvskwyrost3009lcjxf8wut5d4rwpwwuhqr4jo0xb26prx1e4z1lmdkwur81qhejyqrav4dcyp1jcl03bkojyzkqm69u8sxzflkdung31gzcu71xsuq6ljxkt2n4xz'\nurl = 'https://libraryofbabel.info/book.cgi?' + ct + '-w4-s3-v25:64'\nprint(requests.get(url, timeout=30).status_code)\nPY\n```

The single page still looked like random Babel text, so the next step was to download the whole volume:

```bash
python - <<'PY'\nimport requests,re\nhtml = open('babel_page.html', encoding='utf-8').read()\nfields = dict(re.findall(r'<INPUT[^>]+name\\s*=\\s*\"([^\"]+)\"[^>]+value\\s*=\\s*\"([^\"]*)\"', html, re.I))\nr = requests.post('https://libraryofbabel.info/download.cgi', data=fields, timeout=60)\nopen('babel_volume.txt', 'wb').write(r.content)\nprint(r.status_code)\nPY\n```

Searching the downloaded volume found the only relevant hit:

```text
sillyctf iheartyapping
```

Because Babel pages cannot represent braces, the intended CTF flag format is the normal prefix plus braces around the recovered body.

## Flag

```text
sillyCTF{iheartyapping}
```
