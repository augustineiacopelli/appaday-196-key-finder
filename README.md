# Key Finder | AppADay 196

Key Finder works out how the tables in a database fit together. It profiles every column, proposes primary keys and foreign keys with a confidence and its reasons, and avoids the classic trap of matching row numbers to row numbers. Confirm the links, see the diagram, and tap two tables to get the JOIN.

Live at https://augustineiacopelli.github.io/appaday-196-key-finder/ and part of the [AppADay portfolio](https://augustineiacopelli.github.io/appaday/). Day 196 is the third of six daily builds that assemble into the Query Desk milestone on Day 200.

## Using it

Tap **Load sample** for a made up college with ten tables and no keys declared, or use **Open** to pick a project saved on this device (projects from CSV Intake and Query Bench appear there automatically) or any `.sqlite` file. Loading does not analyze anything. A project that has tables but no analysis yet shows one button, **Analyze**.

Analyze reads every column, then proposes a key for each table and the links between tables. A project that already carries keys or links opens straight to them without analyzing again.

**Keys** shows one row per table: the confirmed key, or otherwise the best proposal, with its label (strong, likely or weak), its score and the reasons behind it. Other candidates sit behind a disclosure, each with **Use this key**. A table with nothing that could serve as a key says No key found.

**Links** has three groups. Proposed holds the strong and likely links waiting for a decision, with **Confirm all strong** in its header. Maybe holds the weak ones. Decided holds what you confirmed or rejected, and **Undo** sends a decision back to Proposed. Each link shows the two columns, its label, whether it is many to one or one to one, an Optional tag when the column has empty values, the count of orphan rows when some values have no match, and the reasons behind a disclosure. When another table fits about as well, a Could also be line offers **Use this instead**, which confirms the alternate and rejects the first guess. Decisions are kept when you analyze again.

**Add a link by hand**, at the bottom of Links, takes four pickers. The target column defaults to that table's key, and the orphan count is previewed before you save. A link added by hand is saved as confirmed and can be removed.

**Diagram** draws one box per table with its key column marked PK and its link columns marked FK. Confirmed links are solid lines, proposed strong and likely links are dashed, and a dot marks the parent end. Drag a box to arrange the diagram; positions are saved inside the project file. Tap a table, then a second one, to get the JOIN. Tapping the same table again clears the selection. With a keyboard, Tab to a box, press Enter or Space to select it, and use the arrow keys to nudge it.

The JOIN sheet explains the path in words and shows the SQL. It uses confirmed links first; when they do not connect the two tables, it tries the strong and likely proposals too and says so, with **Confirm these links**. **Keep unmatched rows** switches to LEFT JOIN. **Copy**, **Save as query** (stored in the project) and **Open in Query Bench** (which opens the project in App 195 with the SQL in its editor) sit below.

On a phone, Keys, Links and Diagram are three views behind a segmented control and the JOIN is a bottom sheet. From 720 px up the lists sit in a left column beside the diagram and the JOIN is a side panel. Settings, behind the gear, holds **Reset layout**, the saved projects and the file size limits.

## How it is built

One `index.html`, vendored from the Query Desk workspace in [appaday-194-csv-intake](https://github.com/augustineiacopelli/appaday-194-csv-intake). The head carries the shell CSS from Query Bench (accent `#289664` on a dark neutral ground) and sql.js 1.16.0 loaded by relative path from the portal repository at `../appaday/vendor/sql.js-1.16.0/`. The first script block holds the shared modules `QD:CORE`, `QD:KEYS` and `QD:CLINIC`, written by `tools/vendor.js` and never edited here. The second holds the interface.

Analyze runs `QDKeys.analyze` and then `QDKeys.apply`, which writes proposals into the project's `_qd_keys` and `_qd_relations` tables and never overrides a decision already made. The lists and the diagram always render from what is stored in the project, not from the analysis object. The JOIN comes from `QDKeys.joinPath` and `QDKeys.joinSQL`. Box positions are a JSON map stored with `setMeta` under `diagram_positions`, written only when a drag ends. The diagram is an SVG drawn at world size inside a scrolling container, so there is no zoom or pan; boxes use Pointer Events with pointer capture, and a non passive touchstart on a box keeps iOS from scrolling the page mid drag.

Not included by design: composite foreign keys entered by hand, automatic layout beyond the starting grid, zoom and pan, choosing among alternate join paths, and trimming `SELECT *` to drop duplicate columns.

## Checks

`test/e2e_key_finder.py` in the 194 repository drives this app in Chromium at 375 px: the sample loads ten tables and waits for Analyze, which proposes exactly the fourteen expected links and the ten `<table>_id` keys; a rejected link stays rejected after analyzing again; Confirm all strong leaves thirteen confirmed; the diagram draws ten boxes and thirteen solid lines; a dragged box keeps its place after a reload; tapping enrollments and then campuses writes a JOIN through students; Save as query stores it; Open in Query Bench runs it there; and two unrelated tables built in the page give no links and two keys. `node tools/vendor.js --check` verifies the vendored modules match `core/`.
