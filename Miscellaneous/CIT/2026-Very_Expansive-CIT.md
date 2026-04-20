# Very Expansive Writeup

## Challenge Info

- **Challenge:** Very Expansive
- **Category:** OSINT / Misc
- **Points:** 980
- **Flag format:** `CIT{Name_of_Place}`

## Prompt

> Where in the wide, wide world of sports is this, beratna? Good place for Dusters, mi pensa.

Example:

```text
CIT{Grand_Canyon}
```

## Initial Analysis

The prompt contains a few unusual words:

```text
beratna
mi pensa
Dusters
```

These words are the main hints. They are associated with **Belter Creole** from *The Expanse*.

The challenge title, **Very Expansive**, also points toward *The Expanse* universe.

## Key Clue: Dusters

In *The Expanse*, **Dusters** is slang for people from **Mars**.

So the prompt is basically asking:

```text
Where is a sports-related place on Mars in The Expanse?
```

## Finding the Location

After identifying the reference to *The Expanse*, I looked for Martian locations connected to sports.

One strong match is **Mariner Valley**.

Mariner Valley is a location on Mars in *The Expanse*, and it is known for sports culture, especially football matches. This fits the prompt's phrase:

```text
wide, wide world of sports
```

It also fits:

```text
Good place for Dusters
```

because Dusters are Martians.

## Flag Construction

The challenge requires the flag in this format:

```text
CIT{Name_of_Place}
```

The location is:

```text
Mariner Valley
```

Replacing the space with an underscore gives:

```text
Mariner_Valley
```

## Flag

```text
CIT{Mariner_Valley}
```


---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)

