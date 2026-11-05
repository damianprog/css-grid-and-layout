# CSS Grid & Layout Patterns

Deliberate practice log. For each exercise: the goal, the pattern behind it, what I got wrong, what fixed it, and drills to repeat it without looking.

---

## Pattern: Photo collage with spanning items (explicit line placement)

**Source:** [Chapter 3 – You Try It: Basic Grid Layouts](https://grid-flexbox.css.education/ch3.html), Mars collage
**Date:** 2026-11-03

### Goal

Rebuild a 980px-wide, centered collage of 5 images without reordering the HTML:

```
+-----------------------------+
|        1 landscape          |   row 1
+-------------------+---------+
|                   | 4 rocks |   row 2
|      2 Mars       +---------+
|                   | 5 sand  |   row 3
+-------------------+---------+
|        3 panorama           |   row 4
+-----------------------------+
  col 1     col 2     col 3
lines: 1 ---- 2 ---- 3 ---- 4
```

### Process that worked

1. **Draw the grid lines on the target picture first.** Count the tracks: 3 columns, 4 rows. Number the _lines_ (1 to 4 for 3 columns, 1 to 5 for 4 rows), not the cells.
2. **Map each image to its HTML position.** The HTML order is not the visual order: image 3 is the panorama, which goes at the very bottom. Check what each `src` actually is before placing it.
3. **Place each item with `grid-column` / `grid-row` start/end lines.**
4. **Only then fix sizing:** image width, row heights, gap, centering.

### Final code

```css
.collage {
  display: grid;
  gap: 1rem;
  width: 980px;
  margin: auto;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(4, auto);
}

img {
  width: 100%;
}

.collage img:nth-child(1) {
  grid-column: 1/4;
  grid-row: 1/2;
} /* landscape */
.collage img:nth-child(2) {
  grid-column: 1/3;
  grid-row: 2/4;
} /* Mars, spans 2 rows */
.collage img:nth-child(3) {
  grid-column: 1/4;
  grid-row: 4/5;
} /* panorama */
.collage img:nth-child(4) {
  grid-column: 3/4;
  grid-row: 2/3;
} /* rocks */
.collage img:nth-child(5) {
  grid-column: 3/4;
  grid-row: 3/4;
} /* sand */
```

### Mistakes I made, and the lesson from each

| Mistake                                                                  | Why it was wrong                                                                                                                                            | Lesson                                                                                                                         |
| ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Gave Mars only `grid-column: 1/3`                                        | Columns control width only. Mars also has to span 2 rows.                                                                                                   | Spanning in two directions needs both `grid-column` and `grid-row`.                                                            |
| Gave the 3rd image `grid-column: 1/4` because I assumed it was the rocks | I guessed the image from its position in the HTML instead of checking what it showed.                                                                       | Find out _which_ image each child is before placing it.                                                                        |
| Left out `width: 100%` on `img`                                          | An image keeps its natural width (the panorama is 7010px). A `1fr` track can't shrink below its content, so the grid grew past 980px.                       | Images in grid cells usually need `width: 100%` (or `max-width: 100%`).                                                        |
| Used `grid-template-rows: repeat(4, 1fr)`                                | When the container has no fixed height, `1fr` rows all become as tall as the tallest one. The short landscape row ended up with empty black space under it. | Use `auto` rows when each row's height should come from its content. Use `fr` rows only when there is a known height to share. |

### Key concepts

- **Line numbers, not cells.** `grid-column: 1/4` means "from line 1 to line 4", which covers 3 columns. N tracks have N+1 lines.
- **Visual order ≠ source order.** Explicit `grid-row` placement lets an element that is 3rd in the HTML show up last on screen. The HTML stays in its logical order.
- **`margin: auto` centers only when the box has a width.** Without `width: 980px` the block would fill the whole page and there would be nothing to center.
- **`gap`** creates the space between tracks, so items need no margins.

### Retrieval drills (answer without looking)

1. A grid has 4 columns. Which `grid-column` value makes an item span the whole width?
2. An item should take up the middle two of 4 rows. What is its `grid-row`?
3. Why does a 7000px image blow up a `1fr` column, and what single declaration fixes it?
4. The container has no height and the rows are `repeat(3, 1fr)`. One row holds a tall item. What happens to the other rows?
5. What happens if I give items only `grid-column` and no `grid-row`? Where does auto-placement put them?
6. Rewrite the collage using `span` instead of end lines (e.g. `grid-row: 2 / span 2`).

### Next reps

- Rebuild this collage from scratch in under 10 minutes without looking at these notes.
- Build the same layout again with `grid-template-areas` and compare which version is easier to read.
- Try `object-fit: cover` together with fixed row heights, so the rocks and sand fill the Mars rows exactly.

---

## Pattern: Feature card + nested grids at a desktop breakpoint

**Source:** `responsive-grid-layouts/` exercise, Planets and Moons cards
**Date:** 2026-11-05

### Goal

At `min-width: 900px`, make the feature card take up 2 columns and 3 rows of the page grid, and give every small card a text column next to a full-height image:

```
+---------------------+-----------+
|                     |  Earth    |   row 1
|  Our Solar System   +-----------+
|  (h2, img, p,       |  Mars     |   row 2
|   button)           +-----------+
|                     |  Jupiter  |   row 3
+----------+----------+-----------+
|  Saturn  | Neptune  |  Pluto    |   row 4
+----------+----------+-----------+
```

### Process that worked

1. **Check the outer grid first.** The placement on `.card-container` was already right, so the problem had to be inside `.feature`.
2. **Turn on the DevTools grid overlay for the suspect element.** It showed 2 columns where the CSS declared 1, which pointed to the cause.
3. **Check which rules from outside the media query still apply.** Mobile rules keep working at wider breakpoints unless they get overridden.
4. **Override inside the media query, not in the base rule,** so the narrower layouts stay as they are.
5. **Check the result after each change.** Each fix showed the next problem: the column fix exposed the wrong row order, and stretching exposed the distorted images.

### Final code

```css
.button {
  white-space: nowrap;
}

@media (min-width: 900px) {
  .card-container {
    grid-template-rows: repeat(4, 1fr);
  }

  .card {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    align-items: stretch;
    gap: 2rem;
  }

  .feature {
    grid-column: 1/3;
    grid-row: 1/4;
    grid-template-columns: 1fr;
  }

  .feature img {
    grid-column: 1/2;
  }

  .feature p:last-child {
    grid-row: 4/5;
  }

  .card img {
    margin-bottom: 0;
    object-fit: cover;
  }
}
```

### Mistakes I made, and the lesson from each

| Mistake | Why it was wrong | Lesson |
| ------- | ---------------- | ------ |
| Expected `grid-template-columns: 1fr` to guarantee one column | The base rule `.feature img { grid-column: 1/3 }` still applied. Line 3 doesn't exist in a one-column grid, so the browser added an implicit column. | `grid-template-columns` defines the explicit grid only. Items can still force extra tracks. |
| Left the button on `grid-row: 3/4` from the mobile layout | On mobile the paragraph is hidden, so 3 rows were enough. Once it was visible, the button kept row 3 and auto-placement put the paragraph in the next free row, under the button. | When a breakpoint shows a new element, recheck the fixed row numbers of the elements around it. |
| Let the button text wrap in a narrow column | The `<a>` is inline, so its text, background and padding break across lines like normal text. | Use `white-space: nowrap` for labels that have to stay on one line. |
| Images didn't fill the card height because of `align-items: start` | `start` keeps items at the top of their area at their own height. | `stretch` makes items fill their area. `align-self` does the same for a single item. |
| Stretched the images without handling their proportions | The box no longer matched the photo's aspect ratio, and the default `object-fit: fill` squeezes the photo into the box. | Pair stretched images with `object-fit: cover`, and accept that the edges get cropped. |
| Left the `1rem` image margin from the `min-width: 500px` rule | Rules from `min-width` queries keep applying at every wider breakpoint, so the margin kept the image from reaching the card padding. | Reset inherited spacing at the breakpoint where it stops being useful (`margin-bottom: 0`). |

### Key concepts

- **Implicit tracks.** If an item asks for a line that doesn't exist, the browser creates the missing track instead of ignoring the request or reporting an error.
- **Each grid numbers its own lines.** `grid-column` always refers to the grid of the direct parent. `.feature { grid-column: 1/3 }` uses the lines of `.card-container`, and `.feature img { grid-column: 1/2 }` uses the lines of `.feature`.
- **Auto-placement fills the gaps.** An item with no position takes the next free cell, so a fixed position on one item can move the others.
- **Mobile-first rules cascade upward.** A `min-width` override affects that width and wider, and leaves narrower layouts alone.
- **`object-fit` works on the content, not the box.** The box size comes from the grid, and `object-fit` decides how the photo fits inside it.

### Retrieval drills (answer without looking)

1. A grid has one column. Which line numbers exist, and what happens with `grid-column: 1/3`?
2. What would happen to `.feature` at 900px or wider if I removed `.feature img { grid-column: 1/2 }`, and why? (I couldn't answer this one the first time.)
3. Three items share a grid. One has a fixed `grid-row: 3/4`, and the other two have no position. Which rows do they land in?
4. Why does `.feature img { grid-column: 1/2 }` count lines inside `.feature` and not inside `.card-container`?
5. Why does an inline `<a>` look torn when its text wraps, and which declaration stops the wrapping?
6. What is the difference between `object-fit: fill` and `object-fit: cover`? What does each one cost?
7. How do I remove a margin only from 900px up without changing the 500-899px layouts?

### Next reps

- Rebuild the 900px layout from the mobile styles in under 15 minutes without looking at these notes.
- Replace `align-items: stretch` on `.card` with `align-self: stretch` on the image only, and compare what changes.
- In DevTools, turn `.feature img { grid-column: 1/2 }` off and on with the grid overlay enabled, then explain what moves.
