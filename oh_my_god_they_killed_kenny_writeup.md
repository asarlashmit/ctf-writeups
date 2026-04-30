# Oh My God, They Killed Kenny!

Type: Unknown / OSINT

## Summary

This challenge is an OSINT pivot through the real-world town that inspired `South Park`.

The key steps were:

1. Identify the real-life town as `Fairplay, Colorado`.
2. Identify the local museum as the `South Park City Museum`.
3. Find the museum's steam locomotive and determine its type.
4. Map that locomotive type to the railroad named by museum-historical sources.

## Solve

The South Park City Museum in Fairplay has a page for its train exhibit. That page says the museum's engine is:

```text
Porter Mogul #6
```

and also says it is:

```text
the same type of engine used on the Denver and South Park Line
```

Source:

`https://southparkcity.snappages.site/narrow-guage-train`

To pin down the full railroad name, the National Register documentation for the South Park City Museum gives the stronger wording:

```text
This engine, known as a Porter Mogul #6, was the same type used by the Denver, South Park, and Pacific Railroad.
```

Source:

`https://npgallery.nps.gov/NRHP/GetAsset/6396b459-f7a5-4995-8f73-995cedec30a9`

That identifies the railroad as:

```text
Denver, South Park, and Pacific Railroad
```

Lowercasing the railroad name for the flag gives:

```text
sillyCTF{denver, south park, and pacific railroad}
```

## Notes

Some museum and rail-history sources render the same railroad name as `Denver, South Park & Pacific Railroad`. The underlying railroad is the same, but the NPS wording above is the most direct primary-source phrasing for this challenge.

## Flag

```text
sillyCTF{denver, south park, and pacific railroad}
```
