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
