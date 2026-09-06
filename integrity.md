# Integrity Checks

This file holds the project's integrity checks, split out of [CLAUDE.md](CLAUDE.md) to keep that file smaller. Everything else about the project — the ontologies, file layout, naming conventions, and architectural patterns these checks refer to — is documented in `CLAUDE.md`.

Files inside any directory named `under-development/` (at any depth) are works-in-progress and must be **excluded from all integrity checks** below.

After any change to a graph (its `mia.member`/`mia.topic` entry or `### Graph NN` body section) or a cell DataBook, verify the following.

Check numbers are stable identifiers, never renumbered: **Checks 17 and 22 no longer exist** — both were retired, and their numbers are deliberately left unused rather than reassigned.

**Check 1 — Diagram ↔ files ↔ EXAMPLE.md coverage**: Every numbered graph circle in any of the 12 cell diagrams (`example/images/`) must have (a) a corresponding embedded graph section — a `mia.member`/`mia.topic` entry plus its `### Graph NN` body section — inside a cell-databook file under `example/Cells/`, and (b) a row in one of the tables in the **Graphs** section of `EXAMPLE.md`. Conversely, every row in those tables must correspond to a numbered circle in a diagram and an embedded graph that actually exists. If a circle exists in a diagram but has no embedded graph or `EXAMPLE.md` row, create them to match the diagram.

**Check 2 — Graph id naming convention**: Every `mia.member[]`/`mia.topic[]` entry's own `id` value's local-name (the string after the final `/`) — across all cell-databooks in `example/Cells/` — must follow the flat pattern `graph-<NN>`, where `<NN>` is a zero-padded two-digit number matching the graph's own diagram label, `### Graph NN` body heading, and `<a id="graph-NN">` anchor. If an id does not match this pattern, flag it rather than silently renaming — the entry's `id` also doubles as the graph's own named-graph identity (`{id}#graph`), so changing it is a bigger operation than a file rename ever was.

**Check 3 — `member`/`topic` entry well-formedness**: Each `mia.member`/`mia.topic` entry carries a graph's full metadata directly (`id`, `claimant`, `subject`, and optionally `template`) rather than a bare local-name reference into a separate list, so there's no cross-list consistency left to check — but a malformed entry (e.g. a stray bare string left over from a hand edit, or a missing required field) would otherwise go unnoticed. For every cell-databook under `example/Cells/`, verify that every `mia.member`/`mia.topic` entry is a mapping (never a bare string), that its `id` matches the `graph-<NN>` id pattern (see Check 2), and that both `claimant` and `subject` are present — both required per `shacl/cell-shacl.ttl`'s `:SCGraphShape`; `template` stays optional (see Check 28 for when it's actually required). Run:

```python
import glob, re, yaml

GRAPH_ID_RE = re.compile(r'^http://www\.example\.org/mia/graphs/graph-\d{2}$')

def frontmatter(path):
    text = open(path, encoding='utf-8').read()
    m = re.match(r'^---\n(.*?)\n---', text, re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else None

def as_list(v):
    return [] if v is None else (v if isinstance(v, list) else [v])

errors = 0
for path in sorted(glob.glob('example/Cells/**/*.databook.md', recursive=True)):
    if 'under-development' in path.split('/'):
        continue
    fm = frontmatter(path)
    if not fm or fm.get('type') != 'cell-databook':
        continue
    mia = fm.get('mia', {}) or {}
    for field in ('member', 'topic'):
        for entry in as_list(mia.get(field)):
            if not isinstance(entry, dict):
                print(f'{path}: {field} entry is not a mapping: {entry!r}')
                errors += 1
                continue
            if not GRAPH_ID_RE.match(entry.get('id') or ''):
                print(f"{path}: {field} entry id {entry.get('id')!r} does not match graph-<NN> id pattern")
                errors += 1
            if not entry.get('claimant'):
                print(f"{path}: {field} entry {entry.get('id')!r} missing claimant")
                errors += 1
            if not entry.get('subject'):
                print(f"{path}: {field} entry {entry.get('id')!r} missing subject")
                errors += 1
if not errors:
    print('All cell-databooks: every member/topic entry is a well-formed graph mapping.')
```

If a malformed entry is found, fix it directly (turn a stray bare string back into a full mapping, correct the id, or add the missing `claimant`/`subject`).

**Check 4 — No orphan Persons**: Every `persona:Person` individual other than `:Self` must be reachable via `BFO_0000115` (has member part) from a Social Network individual linked to another `persona:Person` via `persona:hasSocialNetwork`. `:Self` is always the root and needs no incoming link. Since graphs are embedded graph sections across every cell-databook under `example/Cells/**`, this check's scope is the merged whole-tree dump (EXAMPLE.md's "Merged whole-tree dump"), which spans every embedded graph. **Exception**: a `persona:Person` referenced only via a professional/service-designation property (e.g. `persona:hasPrimaryCarePhysician`) rather than social-network membership is exempt — it represents a service relationship (e.g. a physician), not a social connection, so it has no social network to be reachable from. Example: `:Jane_Starostina` (graph #25), Sophia Walker's primary care physician.

**Check 5 — Validation documentation completeness**: The `## Validation` section of `EXAMPLE.md` must document a single validator, `helpers/validate.py`, run from the repo root as `python3 helpers/validate.py`. There are no "tiers" — that two-tier split (a global merged-data run plus a per-graph run) was retired because merging every cell into one default graph unions facts that the self-containment convention deliberately keeps per-graph, manufacturing violations no real query would see (the same reasoning as the [Named graph scoping of `BFO_0000115`](CLAUDE.md#key-architectural-patterns) note). The script walks every cell-databook under `example/Cells/` (skipping `under-development/`), validating **each cell in isolation from every other cell** — no two cells' data ever reach the same `shacl validate` call — and gives each cell two passes:

1. **Cell pass** — data is the cell's whole content at once: every one of its embedded graphs' Turtle (via `databook_graphs.iter_graph_blocks()`) plus the `cell:` triples synthesized from that cell's own `mia.*` frontmatter (via `databook_graphs.process_cell_databook()`, shared with `helpers/yaml-to-rdf.py`). Shapes are the four general shapes files merged into one graph with `owl:imports` stripped: `shacl/cell-shacl.ttl`, `shacl/persona-shacl.ttl`, `shacl/organization-shacl.ttl`, `shacl/agent-shacl.ttl`. The graph Turtle must be in this data, not just the frontmatter triples — `:MemberCellShape`/`:SCGraphShape` constrain `creator`/`owner`/`claimant` with `sh:or ( [sh:class p:Person] [sh:class o:Organization] … )`, and those individuals are typed only in the graph Turtle. This pass is the only thing that validates `cell-shacl` against real instance data.
2. **Template pass** — each graph carrying a `template:` value, checked on its own against the shape that value names. Since `cell:template`'s range is `sh:NodeShape` (`cell.ttl`), the value already names the shape directly, with no label-to-shape resolution; the script's `SHAPE_TO_FILE` table only maps that shape's local name to which physical `*-shacl.ttl` file defines it, since `pshapes:` shapes are split across `shacl/persona-shacl.ttl` and `shacl/contactinfo-shacl.ttl`. A graph with no `template:` value is skipped. Each resolved shape is scoped so it can't fire outside the one graph being checked: every other shape co-located in the same physical file is deactivated, and — only for `ContactInfoShape` (`sh:targetClass persona:Person`, the one class broad enough to risk an incidental same-type individual, e.g. a bare `:Self` reasserted by the self-containment convention) — the shape is re-targeted (`sh:targetNode`) at only the *substantive* `persona:Person` individual(s) in that graph.

The two passes use **different base merges**, both built once per run: `cat-templates.ttl` is in the template pass's base but deliberately excluded from the cell pass's, so `cell-shacl`'s `:CellShape` can't fire on the 102 `ctpl:*TemplateCell` individuals — generic class-level content bound to no real person. The script exits non-zero on any violation or unresolved shape, so it doubles as a CI-style gate.

`EXAMPLE.md` must also document the **merged whole-tree dump** (`helpers/extract-all.py` + `helpers/yaml-to-rdf.py` + `riot`) separately from validation, with the warning that the general shapes must never be run against it. That dump exists for the questions that genuinely need the union — Check 4's cross-cell reachability, and loading the example into a triplestore — not for SHACL. If the algorithm changes, update `EXAMPLE.md`'s Validation section and `helpers/validate.py` to match.

**Check 6 — PNG file location**: The diagram PNG for every embedded graph (each `mia.member`/`mia.topic` entry across every cell-databook under `example/Cells/`) must be stored directly in `example/graphs/images/` (flat, no subfolders — not `images/example/`) — this location is unchanged by the graph/cell merge; only the graphs' own `.databook.md` files were removed, not this images directory. Files in `under-development/` are excluded. **Exempt**: the minimal `:Self`-stub `member` graph Check 21 requires on a purely organizational scaffold cell — whose box draws no graph shapes at all, per Check 10h/Check 1's scope — needs no diagram PNG and no `EXAMPLE.md` row of any kind, not even `*(todo)*` (Check 7); it was never intended to be visualized, unlike every other embedded graph. `example/Cells/Cells(person).databook.md`'s `graph-38` and the other scaffold-cell stub graphs (e.g. `graph-52` on `Work`, `graph-61` on `Vehicles`) fall under this exemption.

**Check 7 — PNG filename convention**: Every diagram PNG in `example/graphs/images/` must use the same base filename as the graph's own `mia.member[]`/`mia.topic[].id` local-name (the string after the final `/`), with `.png` appended. For example, id local-name `graph-14` → `graph-14.png`. If the PNG does not yet exist, the `EXAMPLE.md` Diagram cell must be marked `*(todo)*` rather than left blank — except for a scaffold-cell stub graph covered by Check 6's exemption, which gets no `EXAMPLE.md` row at all, not even `*(todo)*`.

**Check 8 — No broken image links in `README.md`/`EXAMPLE.md`/`APP-BEHAVIOR.md`**: Every PNG path referenced in `README.md`, `EXAMPLE.md`, or `APP-BEHAVIOR.md` (both `<img src="...">` tags and `[view](...)` table links) must resolve to an actual file on disk. Run:

```bash
python3 -c "
import re, os
content = open('README.md').read() + open('EXAMPLE.md').read() + open('APP-BEHAVIOR.md').read()
pngs = [m.group(1) for m in re.finditer(r'src=[\"\\'](.*?\.png)[\"\\']', content)]
pngs += [m.group(1) for m in re.finditer(r'\]\((example/[^\s\"\']+\.png)\)', content)]
missing = [p for p in sorted(set(pngs)) if not os.path.exists(p)]
[print('MISSING:', p) for p in missing] or print('All PNG refs OK')
"
```

If any `MISSING:` lines appear, either add the file or update the link.

**Check 9 — Cell id naming convention**: This check applies only to `example/Cells/` — the user's own instance tree — since there is no separate canonical-instance file tree: the canonical tree is the `cat:CategoryScheme` SKOS concept scheme in `category.ttl` itself, with class-level templates in `cat-templates.ttl`. A cell-databook's `id:` is deliberately independent of its filename — the filename stays the folder's own verbatim name per the Cell DataBook Filename Convention, but the `id:` value is a flat, opaque, globally-unique identifier, following the same reasoning and pattern as the [Graph ID Naming Convention](CLAUDE.md#graph-id-naming-convention)'s `graph-<NN>`: encoding the folder's name and catType into the id would risk a collision the moment two different folders elsewhere in the tree shared both a name and a catType, and nothing in the repo actually depends on the id's string *structure* — it's purely a self-contained RDF subject identifier for that one cell, never cross-referenced by another cell, a graph, or a catalog file. Every `id:` value — across all cell-databooks in `example/Cells/` — must follow the flat pattern `http://www.example.org/mia/cells/cell-<NN>`, where `<NN>` is a zero-padded two-digit number, assigned once at creation and never reused or renumbered. Every `<NN>` must be globally unique across the whole tree. Run:

```python
import glob, re

pattern = re.compile(r'^http://www\.example\.org/mia/cells/cell-(\d{2})$')
seen = {}
errors = 0
for path in sorted(glob.glob('example/Cells/**/*.databook.md', recursive=True)):
    if 'under-development' in path.split('/'):
        continue
    text = open(path).read()
    m = re.search(r'^id:\s*(\S+)', text, re.MULTILINE)
    fid = m.group(1).strip() if m else ''
    pm = pattern.match(fid)
    if not pm:
        print(f'MALFORMED  {path}  id={fid!r} does not match http://www.example.org/mia/cells/cell-<NN>')
        errors += 1
        continue
    nn = pm.group(1)
    if nn in seen:
        print(f'DUPLICATE  cell-{nn}  used by both {seen[nn]!r} and {path!r}')
        errors += 1
    else:
        seen[nn] = path
print('Check 9: OK' if errors == 0 else f'Check 9: {errors} issue(s) found')
```

If a malformed id is found, fix it to match the pattern. If a duplicate `<NN>` is found, assign the newer cell the next unused number — never renumber an existing cell's id, since (like a graph id) it may already be referenced by an external peer over the PDN. This rule has no exceptions for `example/Cells/`, fictional as its data is — treat every id there exactly as if a real external PDN peer might already hold a reference to it.

Note: this check's `^id:\s*(\S+)` regex is anchored at true line-start with no leading whitespace, so it only ever matches a file's own top-level `id:` line — a nested, indented `mia.member[]`/`mia.topic[].id` value never matches this anchor and is intentionally out of scope here (a graph's `id` is not expected to relate to its owning cell file's name at all; see Check 2 for that). This imposes a requirement on any script that writes `mia.member`/`mia.topic`: never emit an unindented `id:` at column 0.

**Check 10 — Example cell diagrams are authoritative**: The 12 cell diagrams in `example/images/` are the authoritative source of truth for the example cell tree. When any discrepancy is found between a diagram and the DataBook files, the diagram wins — update the DataBooks to match, not the other way around. Each diagram box corresponds to a cell in `example/Cells/` (a folder holding its one `cell-databook` directly inside it, box label = the cell's own folder name, mirrored in its cell-databook's `title:`). After any change to `example/Cells/` DataBooks or to the 12 diagrams, verify all of the following:

- **10a — Every cell box has a cell DataBook**: Every cell box shown in any of the 12 diagrams must have a corresponding cell in `example/Cells/` whose cell-databook's `title:` matches the box label. If a box has no DataBook, create the cell (folder + cell DataBook).

- **10b — Every cell DataBook has a diagram box**: Every cell's cell-databook in `example/Cells/` (except the top-level `example/Cells/` folder's own cell-databook, `Cells(person).databook.md`, which is the invisible root) must appear as a visible box in at least one of the 12 diagrams. If a DataBook has no corresponding box, either add it to the appropriate diagram or delete that cell's DataBook.

- **10c — A cell box's graph shapes match its DataBook's `member`/`topic` links exactly, both in count and in type**: Shape, not fill, is what distinguishes `cell:member` from `cell:topic` — a circle for a `member` graph, a square for a `topic` graph (fill/outline color is a separate, independent fact showing who claimed that graph — see Check 15's legend). For every cell box that draws graph shapes at all (a purely organizational scaffolding box draws none — see Check 10h): (1) the total number of shapes attached to it (circles plus squares together) must equal the total number of that cell-databook's own `mia.member` + `mia.topic` entries — no more, no fewer; (2) every circle's own numbered label must correspond to a `mia.member` entry, and every square's own numbered label must correspond to a `mia.topic` entry — never the reverse. This is a visual check (no automated image parsing), but the script below prints every cell's own member/topic graph numbers, split by field, for direct cross-reference against whichever diagram box is being checked — e.g. `pets.png`'s "Ginger" box shows one circle (`Self [36]`) and one square (`Ginger [37]`), matching `Ginger(pets).databook.md`'s own one member (graph-36) and one topic (graph-37); its "Medical" box shows two circles (`Self [33]`, `Paula [57]`) and one square (`Ginger [32]`), matching `Medical.databook.md`'s two members (graph-33, graph-57) and one topic (graph-32). Run:

```python
import re, glob, yaml

def frontmatter(path):
    text = open(path, encoding='utf-8').read()
    m = re.match(r'^---\n(.*?)\n---', text, re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else None

def local_names(entries):
    entries = entries if isinstance(entries, list) else ([entries] if entries else [])
    return [e['id'].rsplit('/', 1)[-1] for e in entries]

for path in sorted(glob.glob('example/Cells/**/*.databook.md', recursive=True)):
    if 'under-development' in path.split('/'):
        continue
    fm = frontmatter(path)
    if not fm or fm.get('type') != 'cell-databook':
        continue
    mia = fm.get('mia', {}) or {}
    members = local_names(mia.get('member'))
    topics = local_names(mia.get('topic'))
    print(f"{fm.get('title')!r:30} member(circle)={members}  topic(square)={topics}")
```

- **10d — Numbered graph circles/squares have matching embedded graphs**: Every numbered graph circle or square (e.g. `[10]`, `[17]`) shown in a diagram must correspond to a `mia.member`/`mia.topic` entry (equivalently, a `### Graph NN` body section) in some cell-databook under `example/Cells/` whose id contains that number (e.g. `(10)`, `(17)`).

- **10e — Child arrows match folder nesting**: Every downward child arrow from cell box A to cell box B in a diagram must correspond to B's folder being a direct filesystem subfolder of A's folder (i.e. B is a descendant cell of A) — child links are derived purely from folder nesting, not any `child:` YAML field. Conversely, every direct-subfolder relationship between two cells must be reflected by a visible child arrow in the diagram.

- **10g — Black parenthetical category-label text matches `cell:category`'s concept label**: As the second line of a cell box's content (there is no blue Subject text above it any more — cell diagrams don't render a cell's subject at all, since it's derivable from the graph circles already drawn rather than an independently stored fact; see README's Representative Cells section), a cell box may carry a black parenthetical giving its `cell:category` concept's `skos:prefLabel` (from `category.ttl`) in human-readable form. It follows the exact same compression rule as the Cell DataBook Filename Convention's `<local>(<catType>)` filename form: shown only when that label differs from the box's own folder-name label, and omitted entirely when the two are identical. This text must match the co-located cell-databook's actual `mia.category` concept's `skos:prefLabel`, verbatim — no more and no fewer words, never invented or abbreviated further. This is a visual check (no script) — e.g. `companies.png`'s "Google" and "ATT" boxes both show `(Companies)`, matching their shared `mia.category: "cat:Companies"` (label "Companies"); `gov-state.png`'s "Birth Certificate" and "Drivers License" boxes show no parenthetical at all, correctly compressed since each folder was renamed to match `cat:BirthCertificate`'s/`cat:DriversLicense`'s own label exactly; `gov-federal.png`'s "SSN" and "Passport" boxes are the same way, compressed against `cat:SSN`'s/`cat:Passport`'s own labels; `home.png`'s "Boston" and "Paradise" boxes both show `(Home)`, matching their shared `mia.category: "cat:Home"` (label "Home") — Boston reuses `cat:Home` directly rather than the "Previous" scaffold's own `cat:Previous`, since a previous residence is still just a Home category-wise, the current/previous distinction being a temporal fact on the address data itself, not a separate category; `things.png`'s "Things" box shows no parenthetical at all, correctly compressed since `cat:Things`'s label already equals the folder name "Things"; `things.png`'s "RAV4" box shows `(Vehicles)`, matching `cat:Vehicles`'s label; `travel.png`'s "Trips" box shows no parenthetical, correctly compressed since `cat:Trips`'s label already equals the folder name "Trips", and its "Kyoto Trip 2027" box shows `(Trips)`, matching `cat:Trips`'s label (reused from its immediate parent, the same "child folder reuses its parent's category" pattern RAV4/Ginger already use).

- **10h — Black curly-brace `{NN}` label matches the cell's own `cell-<NN>` id**: As the last line of a cell box's content (immediately below the Category text, or combined with it on one line, e.g. `(SSN) {6}`), a cell box carries a small black `{NN}` label in curly braces — the cell's own number. **Every** cell box carries one, scaffolding cells (`Work`/`Acme`/`Employees`/`Vehicles`/`Travel`/`Trips`/`Companies`/`Finances`/`Home`/`Previous`/`State`/`Federal`/`People`/`Others`/`Pets`/`Affiliations` and the like) included, alongside its Category text where that isn't compressed away (Check 10g). What a scaffolding cell's box omits is only its **graph shapes**: every one of these cells carries real (stub) `member` content, and most carry a deliberately-empty `topic` as well, but neither is drawn as a circle or square — a visual simplification, not a sign the cell lacks content. Check 10c's shape-count rule is therefore scoped to cell boxes that draw graph shapes at all: a scaffolding box drawing none is conformant no matter how many `mia.member`/`mia.topic` entries its cell-databook actually holds. Wherever `{NN}` appears, it must equal the zero-padded two-digit `<NN>` from the co-located cell-databook's own `id: http://www.example.org/mia/cells/cell-<NN>` (see Check 9). Don't confuse this with the numbered graph circles' `[NN]` labels (Check 10d) — those are graph numbers in square brackets attached to a circle; this is the cell's own number in curly braces attached to the box itself. This is a visual check (no automated OCR), but the script below prints every folder's actual `cell-<NN>` for quick cross-reference against whichever diagram is being checked — e.g. `people.png`'s "Bob Johnson" box shows `{16}` and its "Fred Flintstone" box shows `{17}`; `people2.png`'s "Sophia Walker" (Immediate Family) box shows `{12}`, "Health & Wellness" shows `{13}`, "Jane Starostina" shows `{14}`, and "Medical Appointment" shows `{15}`; `companies.png`'s "Google"/"ATT" boxes show `{3}`/`{2}`; `finances.png`'s "Citibank" box shows `{4}`; `gov-state.png`'s "Government"/"State" boxes show `{25}`/`{28}`, and its "Birth Certificate"/"Drivers License" boxes show `{10}`/`{9}`; `gov-federal.png`'s "Government"/"Federal" boxes show `{25}`/`{26}`, and its "SSN"/"Passport" boxes show `{6}`/`{5}`; `home.png`'s "Home"/"Previous" boxes show `{48}`/`{49}`, and its "Boston"/"Paradise" boxes show `{7}`/`{8}`; `things.png`'s "Things" box shows `{11}`; `affiliations.png`'s "Boston Hub Society" box shows `{1}`; `work.png`'s "Paula"/"Alice Walker" boxes show `{19}`/`{18}`; `pets.png`'s "Ginger" box shows `{41}`, its "Medical" box shows `{40}`, and its "Care & Feeding" box shows `{42}`; `things.png`'s "RAV4" box shows `{44}`; `travel.png`'s "Kyoto Trip 2027" box shows `{47}`. Run:

```python
import glob, re

for path in sorted(glob.glob('example/Cells/**/*.databook.md', recursive=True)):
    if 'under-development' in path.split('/'):
        continue
    text = open(path).read()
    m = re.search(r'^id:\s*http://www\.example\.org/mia/cells/(cell-\d{2})', text, re.MULTILINE)
    if m:
        print(f'{m.group(1)}  {path}')
```

- **10i — Fill color and folder-name-text color match real data**: Every cell across all 12 diagrams carries exactly two independent, mechanically-checkable colors, matching Check 10a's Person/Organization/Custom fill-swatch legend (see Check 15's identical rule for `cat-cell-graph.png`): a **fill** color, applied to the cell's own DataBook box (a separately-drawn folder icon, where a diagram draws one, stays plain white and never carries fill — see Check 20's `folder-mapping.png`) — tan if the folder's `mia.category` resolves to `cat:Person`, light blue if `cat:Organization`, purple/Custom if the cell has no category at all — and a folder-**name-text** color (green/"Predefined" if the folder's `title:` equals the category concept's own `skos:prefLabel` verbatim, plain black/"User-defined" otherwise — always black for a no-category/Custom cell). This is a visual check (no automated pixel/OCR comparison) — use Check 20's script (identical rule, just applied to a different set of diagrams) to compute the correct fill/text color for every real folder and cross-reference against whichever diagram is being checked — e.g. `people.png`: "People"/"Others" tan fill + green text; "Bob Johnson"/"Fred Flintstone" tan fill + black text; `work.png`: "Paula"/"Alice Walker" light-blue fill + black text; `people2.png`: "Sophia Walker"/"Jane Starostina" tan fill + black text; "Medical Appointment" tan fill + green text (folder name "Medical Appointment" now matches category `cat:MedicalAppointment`'s own label verbatim); `pets.png`: "Pets"/"Medical"/"Care & Feeding" tan fill + green text (folder name matches category label in each case), "Ginger" tan fill + black text (category `cat:Pets` label "Pets" ≠ folder name "Ginger"); `things.png`: "Vehicles" tan fill + green text (folder name matches category label), "RAV4" tan fill + black text (category `cat:Vehicles` label "Vehicles" ≠ folder name "RAV4"); `travel.png`: "Travel"/"Trips" tan fill + green text (folder name matches category label in each case), "Kyoto Trip 2027" tan fill + black text (category `cat:Trips` label "Trips" ≠ folder name "Kyoto Trip 2027"). No real example cell currently uses the Custom (no-category, `(custom)` filename) case — every current cell has a category — so no example diagram box is expected to show purple fill yet.

The 12 diagrams are: `example/images/people.png`, `example/images/people2.png`, `example/images/work.png`, `example/images/companies.png`, `example/images/finances.png`, `example/images/gov-state.png`, `example/images/gov-federal.png`, `example/images/home.png`, `example/images/things.png`, `example/images/affiliations.png`, `example/images/pets.png`, `example/images/travel.png`. There is no `health.png` — its content, e.g. Health & Wellness/Medical/Provider, lives in `people2.png` instead. `pets.png` shows the "Ginger" cell (cell-41, `{41}`, graphs `[36]`/`[37]`) and its two sibling child cells "Medical" (cell-40, `{40}`, graphs `[32]`/`[33]`/`[57]`) and "Care & Feeding" (cell-42, `{42}`, graphs `[58]`/`[59]`/`[60]`) as content boxes under the `Pets → Ginger → {Medical, Care & Feeding}` folder nesting — "Medical" holds Ginger's medication content directly (no more nested "Medications" sub-cell, folded away when `cat:PetsMedications` was merged into `cat:PetsMedical`), and "Care & Feeding" is its new sibling for Ginger's day-to-day care instructions (`cat:PetsCareAndFeeding`). `things.png` shows the "Things" cell (cell-11, `{11}`, graph `[22]`) and its child scaffold box "Vehicles" (no rendered content, per the bare-scaffolding-folder treatment above) — "Vehicles"'s own child cell, "RAV4" (cell-44, `{44}`, graphs `[62]`/`[63]`), is drawn as a content box in this same diagram, one level further down (`Things → Vehicles → RAV4`). `travel.png` shows two bare scaffold boxes, "Travel" (cell-45) and its child "Trips" (cell-46), neither rendered with content (per the bare-scaffolding-folder treatment above) — "Trips"'s own child cell, "Kyoto Trip 2027" (cell-47, `{47}`, graphs `[66]`/`[67]`/`[68]`/`[69]`/`[70]`/`[91]`), is drawn as a content box one level further down (`Travel → Trips → Kyoto Trip 2027`), with three member circles ("Self" `[66]`, "Agent" `[67]`, "Dave" `[68]`) and three topic squares sharing one subject ("Kyoto" `[69]`/`[70]`/`[91]`, claimed by Self/Agent/Dave respectively) — the first worked example of an `a:Agent` cell member (see README.md's [Agent Ontology](README.md#agent-ontology)) and, with one topic value per member, the first worked example reaching `cell:topic`'s real upper bound (Check 25).

**Check 11 — Physical folder structure IS the tree of cells in `example/Cells/`**: This check applies only to `example/Cells/` — the user's own instance tree — since there is no separate canonical-instance file tree to mirror. There is no `mia.child`/`mia.cell` YAML list to cross-check the tree against either, so this check has no independently-asserted list to "mirror" at all; it collapses to a pure filesystem sanity check with no YAML frontmatter parsing at all. A folder is a **cell** ("marker dir") iff it directly contains exactly one `*.databook.md` file (the only DataBook type in a user's instance tree is `cell-databook`, so no `-cell` marker is needed to identify one) — that file is simultaneously the cell's real (or placeholder) content and its tree-node marker (cell.ttl's folder ownership boundary rule). A folder can never legally hold more than one cell-databook: a `cell:Cell` is self-contained, and letting two cells share a folder would risk a single file in that folder becoming ambiguously part of both — this check flags any such folder as an error. A plain filesystem folder with no cell-databook of its own is not a cell at all — that word stays reserved for a folder that does have one. Cell naming is not standardized — a cell's own folder name may be the category's display label, a role-based label, or anything else — but a cell-databook's own filename is always the folder's exact verbatim name (see the Cell DataBook Filename Convention), so this check's per-folder marker test and its filename root are one and the same string. A bare, marker-less pass-through directory between two marker dirs is legal, matching README.md's own definition of a "regular filesystem folder" ("A folder without a matching cell DataBook is simply a regular file system folder, not a cell — **even if it contains nested cells of its own**"): such a folder is legal anywhere in the tree, including between two marker dirs, as long as it isn't otherwise empty (i.e. something beneath it eventually has a cell-databook). Run:

```python
import os

def check_tree(root):
    marker_dirs = set()
    cell_counts = {}
    for dirpath, _, filenames in os.walk(root):
        cells = [f for f in filenames if f.endswith('.databook.md')]
        if cells:
            rel = os.path.relpath(dirpath, root)
            marker_dirs.add(rel)
            cell_counts[rel] = cells

    def parent_of(reldir):
        if reldir == '.':
            return None
        p = os.path.dirname(reldir)
        return p if p != '' else '.'

    errors = []

    # (A bare, marker-less pass-through directory between two marker dirs is
    #  legal — README.md's own definition of a "regular filesystem folder"
    #  explicitly allows this ("even if it contains nested cells of its
    #  own") — so no ancestor-chain check is needed here.
    #  Rule 1 below (empty/placeholder detection) already covers the only
    #  real failure mode: a bare folder with nothing but other bare folders
    #  under it, all the way down.)

    # 1. Any subfolder with no cell-databook anywhere under it at all is
    #    either an empty/placeholder folder (flag, don't delete) or plain
    #    non-cell content living inside a cell's own folder (fine,
    #    not an error) — only flag when it's otherwise empty.
    for dirpath, dirnames, filenames in os.walk(root):
        rel = os.path.relpath(dirpath, root)
        for entry in sorted(dirnames):
            full = os.path.join(dirpath, entry)
            sub_rel = os.path.join(rel, entry) if rel != '.' else entry
            if sub_rel not in marker_dirs and not any(
                fn.endswith('.databook.md') for _, _, fns in os.walk(full) for fn in fns
            ):
                errors.append(f'EMPTY/PLACEHOLDER FOLDER (no databook.md anywhere under it): {sub_rel!r} under {rel!r}')

    # 2. A cell's folder holds exactly one cell-databook — a cell is
    #    self-contained, so more than one sharing a folder is always an
    #    error (it risks a single file ambiguously belonging to both).
    for d, cells in sorted(cell_counts.items()):
        if len(cells) > 1:
            errors.append(f'TOO MANY CELLS: {d!r} has {len(cells)} cell-databooks (expected exactly 1): {sorted(cells)}')

    return errors

for root in ['example/Cells']:
    errors = check_tree(root)
    print(f'{root}: ' + (f'{len(errors)} issue(s) found:' if errors else 'OK — folder structure IS the tree of cells, no gaps.'))
    for e in errors:
        print(' -', e)
```

If a `TOO MANY CELLS` issue is found, move the extra file(s) out to their own new folder — a folder may hold only one cell-databook, so a second cell belongs in its own new folder, not alongside the first. An empty/placeholder folder is not necessarily an error — flag it to the user rather than deleting it, since it may be a deliberate placeholder for content not yet added. Cell-databook files routinely carry substantial body content (one `### Graph NN` section per embedded graph) — this is expected and not itself a violation of this check, which only validates folder nesting, not file size or content.

**Check 12 — `cell.ttl` matches `images/cell-ontology/cell.png`**: `Cell` shows only `category` (to a `skos:Concept` box, drawn **0..1**, matching `cell:category`'s actual cardinality). `cell:category`'s range is the classificatory `skos:Concept` (scoped to `cat:CategoryScheme`, category.ttl) — there is no tree-position class for it to be confused with, so it does not conflict with `cell:Cell`'s "no link to a tree position" design (see `cell:Cell`'s own `rdfs:comment`) — it records what kind of thing a cell is, not where it lives, and needs no `owl:imports category.ttl` (referenced by name only, mirroring `cell:creator`'s identical pattern). The diagram shows three arrows off `Cell` with no counterpart in `cell.ttl` — `note` (to a placeholder box, `(markdown file) 1..1`), `attachment` (to a placeholder box, `(file) 0..N`), and `chat` (to a placeholder box, `(a chat stream) 1..1` — every cell always has a chat stream, possibly empty) — this is a deliberate, accepted exception to 12a/12b below: README.md already describes all three properties' intended semantics (see the Documentation-only Properties section) as planned, but `cell.ttl` itself has no `cell:note`/`cell:attachment`/`cell:chat` declaration yet — none of the three is ever reified as a triple in any real graph. `Cell` (abstract, blue) carries `category` only (plus the still-open `note`/`attachment`/`chat` arrows above). `Cell` splits into two disjoint kinds (`owl:disjointWith` — a cell is always exactly one, never both): `TemplateCell` (abstract, blue, a reusable class-level template) carries `isTopicCell` (to an `xsd:boolean` box) and `memberGraphShape`/`topicGraphShape` (each to its own `sh:NodeShape` box, 0..N) — `category`, drawn off the shared `Cell` box above, applies to `TemplateCell` too (every real template carries one, naming the concept it's a template for); `MemberCell` (concrete, black, an actual cell instantiated in a user's own tree) carries `member` (to a `cell:SCGraph` box, 1..N, no fixed upper bound), `creator` (to a union of `p:Person`/`o:Organization`, 1..1), and `owner` (to the same union of `p:Person`/`o:Organization` box `creator` already points to, 1..N, no fixed upper bound). There is no `shape` arrow off `MemberCell` at all — `cell:shape` was removed (a `MemberCell`'s validation shape is now derived via a reverse lookup on its own `category` value rather than stored) — and no cell-level `subject` arrow off `MemberCell` either — that property was removed (a cell's own subject is now derived from `member`/`topic`'s own `cell:subject` values rather than stored on the cell itself; `cell:subject` itself is a real, surviving property, just scoped to `cell:SCGraph` rather than `MemberCell` — see Check 13). One class hangs directly off `MemberCell` as its child: `TopicCell` (concrete, black — a mixin combining with `MemberCell` for a cell that also carries at least one `topic` value), which alone carries `topic` (to a `cell:SCGraph` box, 1..N — required, at least one, once a cell is typed `TopicCell` at all; OWL/SHACL itself sets no maximum, but the real upper bound — one `topic` value per cell member, each with a distinct claimant — is Check 25, not something this diagram draws). `TemplateCell` has no subclasses of its own, and no individual is ever typed both `TemplateCell` and `MemberCell` (or any `MemberCell`-lineage class, including `TopicCell`) — every template individual in `cat-templates.ttl` is typed solely `TemplateCell`. No arrow points from `Cell`, `TemplateCell`, or `MemberCell` to any tree-position box at all — `cell:category`'s arrow points to `skos:Concept` instead, the classificatory hierarchy. This diagram is the ontology-level (not example-tree) picture of `cell:Cell`'s structure — the member-composition hierarchy and its content-linking properties. Unlike Check 10 (example diagrams, where the diagram always wins), this check does not presume which side is authoritative when the two disagree — surface the discrepancy and ask:

- **12a** — every property arrow shown off `Cell` (`category`) has a corresponding `cell:` property in `cell.ttl` with `rdfs:domain cell:Cell` (the diagram's `note`, `attachment`, and `chat` arrows are the three accepted exceptions — see above: all three are planned properties, not yet added to `cell.ttl`). Every arrow off `TemplateCell` (`isTopicCell`, `memberGraphShape`, `topicGraphShape`) has `rdfs:domain cell:TemplateCell`; every arrow off `MemberCell` (`creator`, `owner`, `member`) has `rdfs:domain cell:MemberCell`; the one arrow off `TopicCell` (`topic`) has `rdfs:domain cell:TopicCell`. No `shape` arrow should appear off `MemberCell` at all — `cell:shape` doesn't exist (a `MemberCell`'s validation shape is derived via `category`, never stored). No cell-level `subject` arrow should appear off `MemberCell` either — that property doesn't exist (see Check 12's own note above); `cell:subject` itself is real, but scoped to `cell:SCGraph`, not `MemberCell` (see Check 13). Each arrow's target type in the diagram must match the property's `rdfs:range` — `member`'s and `topic`'s are both `cell:SCGraph`, `isTopicCell`'s is `xsd:boolean`, `creator`'s and `owner`'s are both the union of `p:Person`/`o:Organization` (the same target box), `category`'s is `skos:Concept` itself (value is a named category-concept individual, scoped via `skos:inScheme cat:CategoryScheme` — no class-value punning, since category.ttl's tree is SKOS, not OWL classes), `memberGraphShape`'s and `topicGraphShape`'s are both `sh:NodeShape` (two separate arrows, each 0..N — not one shared 0..1 arrow). No `note` arrow should appear off `Cell` at all.
- **12b** — every `cell:` property defined in `cell.ttl` appears as an arrow in the diagram, under the box matching its domain — `Cell`, `TemplateCell`, `MemberCell`, or `TopicCell` (catches new properties added to the ttl but never drawn, or drawn under the wrong box).
- **12c** — the class hierarchy `Cell` → `TemplateCell`/`MemberCell` (both `owl:disjointWith` one another), and separately `MemberCell` → `TopicCell`, shown in the diagram matches `cell.ttl`'s actual `rdfs:subClassOf` relationships (by class local name, not just position). `TopicCell` must be drawn as a direct child of `MemberCell`, not of `Cell` — `TopicCell` combines with `MemberCell` directly, not via any intermediate member-count class.

**Check 13 — `cell.ttl`'s `cell:Graph`/`cell:SCGraph` terms match `images/cell-ontology/graph.png`**: `cell:Graph`/`cell:SCGraph` and their `template`/`subject`/`claimant` annotation properties — the graph-DataBook classification vocabulary, defined in `cell.ttl` since a graph DataBook only ever exists to be linked from a cell — are diagrammed separately from `cell.ttl`'s member-composition hierarchy (Check 12's `cell.png`), in this dedicated `graph.png`. The diagram shows `Graph`/`SCGraph`. `Graph` shows only `template` (targeting `sh:NodeShape`, matching `cell:template`'s actual `rdfs:range`); `SCGraph` (subClassOf `Graph`) shows `subject` (targeting `xsd:anyURI` — any resource IRI, not necessarily `p:Person`/`o:Organization`) and `claimant` (targeting `p:Person`/`o:Organization`/`a:Agent`, not `i:PDNidentifier`) — no `about-by` arrow. No leaf subtype boxes appear below `SCGraph` — `SCGraph` has no subclasses. This diagram is the ontology-level picture of `cell:Graph`'s structure. After any change to `cell.ttl`'s `cell:Graph`/`cell:SCGraph` terms or to this diagram, verify:

- **13a** — every property arrow shown off `Graph` in the diagram (`template`) has a corresponding `cell:` property in `cell.ttl` with `rdfs:domain cell:Graph`, and its target type matches the property's `rdfs:range`.
- **13b** — every property arrow shown off `SCGraph` in the diagram (`subject`, `claimant`) has a corresponding `cell:` property with `rdfs:domain cell:SCGraph`; `claimant`'s target in the diagram must match its actual `rdfs:range` — a union of `p:Person`/`o:Organization`/`a:Agent`, not `i:PDNidentifier`; `subject`'s target must match its actual `rdfs:range` — any resource IRI (`xsd:anyURI`), not a Person/Organization union. No `about-by` arrow should appear — `cell.ttl` defines no such property.
- **13c** — every `cell:` property with domain `cell:Graph` or `cell:SCGraph` defined in `cell.ttl` appears in the diagram under the correct box (catches new properties added to the ttl but never drawn, or drawn under the wrong box).
- **13d** — no subclasses appear below `SCGraph` — `cell.ttl` defines none for it. If any appear here or in `cell.ttl`, reconcile them.

**Check 14 — `category.ttl` matches `images/category-ontology/category.png`**: This diagram is the ontology-level picture of `category.ttl`'s SKOS concept scheme, not an OWL class hierarchy: `cat:CategoryScheme` (a `skos:ConceptScheme` box) carries `hasTopConcept` arrows to `cat:Person`/`cat:Organization`, each a plain `skos:Concept` box (there is no `cat:Category` class anywhere in the diagram — every box is an individual, not a class), each with representative narrower-concept examples reachable via `broader` arrows drawn concept → concept (Affiliations/People/Work → Person; Suppliers/People (org) → Organization). There is no `templateCell` arrow anywhere — that property was removed; a templated concept's reusable content is found from `cat-templates.ttl`'s side instead (see Check 12), not drawn on this diagram at all. No `Folder`/`CategoryDefined`/`UserDefined` boxes and no `child`/`cell`/`category`/`catType`/`label` arrows should appear anywhere. This diagram does not presume which side is authoritative when the two disagree — surface the discrepancy and ask. After any change to `category.ttl` or to this diagram, verify:

- **14a** — the only property arrows in the diagram are `hasTopConcept` (off `CategoryScheme`, to `Person`/`Organization`) and `broader` (off every other concept, to its own parent concept) — matching `category.ttl`'s actual `skos:hasTopConcept`/`skos:broader` triples. No `templateCell` arrow should appear anywhere — that property no longer exists in `category.ttl` at all. No `catType`, `child`, `cell`, `category`, `label`, or `narrower` arrow should appear anywhere either — `category.ttl` asserts only `broader` (child → parent), never the inverse `narrower`, and defines none of the `Folder`/`CategoryDefined`/`UserDefined` classes such arrows would live on. No `memberGraphShape`/`topicGraphShape` arrow should appear either — those are `cell.ttl` properties (see Check 12), never `category.ttl` ones. There must be no `Canonical` box and no `copiedFrom` arrow anywhere — `category.ttl` defines neither.
- **14b** — every `skos:` property actually asserted in `category.ttl` (`hasTopConcept`, `broader`, `inScheme`, `topConceptOf`, `prefLabel`) appears as an arrow or label in the diagram, under the box matching its domain (catches a new relationship added to the ttl but never drawn). `category.ttl` defines no `cat:` property of its own any more.
- **14c** — the concept structure shown in the diagram — `CategoryScheme` → `hasTopConcept` → `Person`/`Organization` → `broader`-linked narrower concepts — matches `category.ttl`'s actual `skos:hasTopConcept`/`skos:broader` triples (by concept local name, not just position). There is no class hierarchy to check any more — `category.ttl` defines no `owl:Class` other than the ontology header's implicit `owl:Ontology` typing, and no `Folder` hierarchy at all.

**Check 15 — `images/cat-cell-graph.png` matches example usage**: The legend is a single box (title "Key") — there is no longer a separate "Category" legend box. It holds, in order: three fill-color swatches — Person (tan), Organization (light blue), None (Custom) (purple/lavender); a compact two-line folder-name-text formula, "Name = Category or" (blue "Name =", green "Category") / "User-defined + (Category)" (bold black) — green text means the folder's name is copied verbatim from its category's own `skos:prefLabel`, black means the user gave it a different name (shown alongside its `(Category)` parenthetical), and a Custom (no-category) folder's name is always black, never green, since it has no label to match; three claim-color swatches — a green-filled swatch labeled "Other" (claimed by someone other than the self), a gray-filled swatch labeled "Delegate" (claimed by an invited `a:Agent` — see README.md's [Agent Ontology](README.md#agent-ontology)), and a dashed/outlined swatch labeled "Self" (claimed by the user); a circle labeled "Member" and a square labeled "Topic" — **shape**, not fill, is what distinguishes `c:member` from `c:topic` in this diagram, a change from the older circles-only design; fill/outline (green/gray/dashed-outline) is then layered on independently to show who claimed that graph, so a `c:topic` square can carry any of the three colors just as a `c:member` circle can; and a single "Cell" swatch (one uniform rounded-corner, single-border style) — the member-count border-style distinction this legend once carried (three separate "3+-Member Cell"/"2 Member Cell"/"1 Member Cell" entries) was retired project-wide (see Check 10f) — a cell's member count is no longer a visually-checkable fact anywhere in this diagram. None of the legend's names are OWL classes — `category.ttl` defines no `Folder`/`CategoryDefined`/`UserDefined` class; Custom stays a pure filename/display convention. Every circle/square carries an explicit subject-name label (e.g. "Bob", "Self", "BHS") baked directly into the shape — this is how a viewer still learns who is involved, without a separate Subject annotation. There is no "Subject" heading grouping these any more, and no blue per-box Subject text — a cell box no longer displays its subject at all, since it's derivable from the circles/squares already drawn rather than an independently stored fact (see README's Representative Cells section). This diagram illustrates representative cell/category associations generically, not tied to a specific example instance — six boxes, each with a single-line folder-name header (no separate `catType`/`label` split), fill color on the cell's own box (this diagram draws no separate folder icon at all, unlike `folder-mapping.png` — see Check 20). None of the six currently illustrates the gray/Delegate state — that is instead demonstrated by real data in `travel.png`'s "Kyoto Trip 2027" cell (see Check 10's own diagram list):
  - `Medical Appointment` (tan/`Person` fill, green text — folder name matches category `(Medical Appointment)`'s own label exactly, two squares "Med. Appt mt." — one green/Other, one white/Self, demonstrating that `cell:topic` isn't capped at one value — its real cap is the cell's own member count (Check 25), reached here at two — plus two circles "Self" (white) and "Bob" (green); two members)
  - `Friends` (purple/Custom fill, black text, no category at all, shown as `()`, two circles: a white "Self" member — the cell's required `member` entry, per Check 21 — and a green "Fred", since Fred is the derived subject but not a member; no squares; two members)
  - `Employee` (light-blue/`Organization` fill, green text — folder name matches category `(Employee)`'s own label exactly, one white "Self" circle, no squares; one member)
  - `Bob Johnson` (tan/`Person` fill, black text — category `(Others)` ≠ folder name, four circles — two white/Self, two green/Other, all four `c:member` link types filled; no squares; two members)
  - `BHS` (tan/`Person` fill, black text — category `(Affiliations)` ≠ folder name, three circles (Self white, Bob green, BHS green — its three `c:member`) plus one green square (BHS's own organization profile, linked via `cell:topic`) — illustrative only, not tied to real cell-01 data (which has no `cell:topic` value at all, see Check 18); three or more members)
  - `People` (tan/`Person` fill, green text — no category parenthetical shown, correctly compressed since the category's label already equals the folder name, one white "Self" circle, no squares; one member)

  Each cell box shows no icon of any kind — no folder icon and no separate "note", "attachment", or "chat" icon (see Check 12's `cell:note`/`cell:attachment`/`cell:chat` planned-property note, which concerns `cell.png` only, not this diagram) — just the filled box itself. Re-verify each box's circles/squares remain a valid illustration of the properties and cardinalities described in the Cell and Graph Ontology sections of `README.md` after any change to those properties.

**Check 16 — IRI roots: `mee.foundation/ontologies` for foundational files, `www.example.org` for example data**: Every foundational ontology and SHACL shapes file — `persona.ttl`, `cell.ttl`, `category.ttl`, `cat-templates.ttl`, `pdn-identity.ttl`, `organization.ttl`, `agent.ttl`, every `*-shacl.ttl` companion (each living in a `shacl/` subfolder directly below the ontology file it validates — the foundational ones under the repo-root `shacl/`, each `other/*.ttl` peer ontology's own under `other/shacl/`, one recursive glob covers all of them), and every `other/*.ttl` peer ontology (`other/pets.ttl`, `other/vehicles.ttl`, `other/identity-documents.ttl`, globbed the same way) — must declare its `owl:Ontology` IRI under `http://mee.foundation/ontologies/`. There is no separate canonical category/cell DataBook tree to check — the canonical tree's IRI roots are covered by `category.ttl`/`cat-templates.ttl` themselves. Every DataBook under `example/Cells/` (excluding `under-development/`) represents Alice's own example instance data, so both its own `id:` and every `mia.member[]`/`mia.topic[].id` value it carries must be grounded under `http://www.example.org/` — `https://` is deliberately rejected here, not just accepted alongside it, since every identifier in the example tree (cell ids and graph ids alike) was standardized on the plain `http://` scheme for consistency; a stray `https://` is exactly the kind of drift this check exists to catch. Run:

```python
import os, re, glob, yaml

FOUNDATIONAL_TTL = [
    'persona.ttl', 'cell.ttl', 'category.ttl', 'cat-templates.ttl',
    'pdn-identity.ttl', 'organization.ttl', 'agent.ttl',
] + sorted(glob.glob('**/shacl/*.ttl', recursive=True)) + sorted(glob.glob('other/*.ttl'))
# Every *-shacl.ttl companion lives in a shacl/ subfolder directly below the
# ontology file it validates — shacl/ at the repo root for the foundational
# ones, other/shacl/ for other/pets.ttl's and other/vehicles.ttl's own shapes
# — so the recursive **/shacl/*.ttl glob alone covers all of them, present and
# future, with no need to list each by name.

errors = 0
for path in FOUNDATIONAL_TTL:
    if not os.path.exists(path):
        continue
    text = open(path).read()
    m = re.search(r'^<(http[^>]+)>\s+rdf:type\s+owl:Ontology', text, re.MULTILINE)
    if not m:
        print(f'NO owl:Ontology IRI FOUND: {path}')
        errors += 1
        continue
    if not m.group(1).startswith('http://mee.foundation/ontologies/'):
        print(f'WRONG ROOT (expected mee.foundation): {path} -> {m.group(1)}')
        errors += 1

def frontmatter(path):
    text = open(path, encoding='utf-8').read()
    m = re.match(r'^---\n(.*?)\n---', text, re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else None

def check_cell_tree_id_roots(pattern, expected_prefixes):
    global errors
    for path in sorted(glob.glob(pattern, recursive=True)):
        if 'under-development' in path.split(os.sep):
            continue
        fm = frontmatter(path)
        if not fm:
            continue
        iri = fm.get('id')
        if iri and not any(str(iri).startswith(p) for p in expected_prefixes):
            print(f'WRONG ID ROOT: {path} -> {iri}')
            errors += 1
        mia = fm.get('mia', {}) or {}
        entries = mia.get('member') or []
        entries = entries if isinstance(entries, list) else [entries]
        topic = mia.get('topic') or []
        entries += topic if isinstance(topic, list) else [topic]
        for graph in entries:
            tid = graph.get('id') if isinstance(graph, dict) else None
            if tid and not any(tid.startswith(p) for p in expected_prefixes):
                print(f'WRONG GRAPH ID ROOT: {path} -> {tid}')
                errors += 1

check_cell_tree_id_roots('example/Cells/**/*.databook.md', ['http://www.example.org/'])

print('OK — no IRI-root violations found.' if errors == 0 else f'{errors} violation(s) found.')
```

If a violation is found, rename the offending file's `owl:Ontology`/`id:` IRI to the correct root, and update every catalog entry and cross-reference that pointed at the old IRI to match (see Check 5's validation commands, which also hardcode these IRIs).

**Check 18 — `topic`'s presence is what types a cell `cell:TopicCell`, and a cell's subject is derived from it plus `member`**: There is no independently-asserted cell-level subject property (nor an `mia.subject` field) — who or what a cell's relationship is about is computed, not stored, by a simple two-branch rule, from its linked graphs' own `cell:subject` values: **if the cell has any `topic` entries** (i.e. it's typed `cell:TopicCell`), the full set of distinct `cell:subject` values among them is the cell's subject (e.g. `Medical Appointment.databook.md`, a two-member cell: `member` holds Dave's and Self's graphs, `topic` holds Sophia's graph — subject is `:Sophia_Walker`); **otherwise** the subject is the full set of distinct `cell:subject` values among `member` — the cell's own active members (e.g. `Bob Johnson(others).databook.md`, a two-member cell with no `topic`: subject is `:Self` and `:Bob_Johnson` together; `Fred Flintstone(others).databook.md`, likewise: `:Self` and `:Fred_Flintstone` together). `Boston Hub Society(affiliations).databook.md` shows the first branch taking over from the second: as a three-member cell it would derive `:BHS`, `:Bob_Johnson`, and `:Self` together, but its one manually-added `topic` — BHS's own organizational profile, claimed by BHS — narrows the derived subject to `:BHS` alone. `cell:topic` is required (at least one value) once a cell is typed `cell:TopicCell` at all (`shacl/cell-shacl.ttl`'s `:TopicCellShape`) — every real example cell today happens to carry exactly one `topic` value, but the rule and this check both generalize to any number, up to Check 25's real upper bound (the cell's own member count). A `topic` subject is explicitly **allowed** to duplicate a `member` subject — this is the normal shape for every pattern-(2) `isTopicCell:true` template whose `cell:member` content is just the generic `ContactInfoShape` business-card stub (claimed by and about `:Self`) while the cell's real content lives in `cell:topic`, also about `:Self` (e.g. `Google(companies).databook.md`, `SSN.databook.md`, `Paradise(home).databook.md`): the stub `member` isn't a distinct party the derived subject needs to separately surface, so the derivation switching to the `topic` set alone (ignoring `member`) is intended, not a masking bug. This check therefore only reports each cell's derived subject for reference — it does not flag member/topic subject overlap as a violation. Run:

```python
import re, yaml, glob

def frontmatter(path):
    text = open(path, encoding='utf-8').read()
    m = re.match(r'^---\n(.*?)\n---', text, re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else None

for f in glob.glob('example/Cells/**/*.databook.md', recursive=True):
    fm = frontmatter(f)
    if not fm:
        continue
    mia = fm.get('mia', {}) or {}
    if not mia.get('member'):
        continue
    # mia.member/topic entries now carry their own subject directly —
    # no separate mia.graphs list to look up.
    pt = mia.get('member') or []
    pt = pt if isinstance(pt, list) else [pt]
    ot = mia.get('topic') or []
    ot = ot if isinstance(ot, list) else [ot]
    pt_subs = {t.get('subject') for t in pt}
    ot_subs = {t.get('subject') for t in ot}
    derived = ot_subs if ot else pt_subs
    print(f'derived subject={sorted(s for s in derived if s)} {f}')
print()
print('Derived subjects listed above for every cell — member/topic subject overlap is allowed, not a violation.')
```

**Check 19 — Cell-databook `title:` is the cell's name and matches its own folder's OS name**: `title:` is defined as the cell's own name — it is always exactly the name of the filesystem folder that holds the cell-databook, and the two are kept in sync (a folder rename means updating `title:` to match, never the reverse); `title:` is never an independent display-name override of the folder's name. The Cell DataBook Filename Convention already requires a cell-databook's *filename root* to be an exact copy of its folder's own name, but that convention is about the filename — not the separate `title:` YAML field, which several other checks (notably Check 10a's box-label match) treat as authoritative for what a cell "is called." The invariant: for every cell-databook under `example/Cells/`, `title:` must equal `os.path.basename` of the folder it directly lives in, verbatim (same case/spacing/punctuation rule as the filename convention — no kebab-casing, no paraphrasing). Run:

```python
import os, re, yaml

def frontmatter(path):
    text = open(path, encoding='utf-8').read()
    m = re.match(r'^---\n(.*?)\n---', text, re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else None

errors = 0
for dirpath, _, filenames in os.walk('example/Cells'):
    if 'under-development' in dirpath.split(os.sep):
        continue
    cells = [f for f in filenames if f.endswith('.databook.md')]
    for fname in cells:
        path = os.path.join(dirpath, fname)
        fm = frontmatter(path)
        if not fm:
            continue
        title = fm.get('title')
        folder_name = os.path.basename(dirpath)
        if title != folder_name:
            print(f'MISMATCH  {path}  folder={folder_name!r}  title={title!r}')
            errors += 1
print('OK — title: matches its own folder name for every cell-databook.' if errors == 0 else f'{errors} mismatch(es) found.')
```

If a mismatch is found, the folder name is authoritative — update `title:` to match it exactly, even when the existing `title:` reads more naturally (e.g. an honorific like `Dr. Jane Starostina` vs. folder `Jane Starostina`, or an expansion like `AT&T` vs. folder `ATT`): `title:` is not an independent display-name override, so it cannot legitimately diverge from the folder's own name — the mismatch is drift, not a deliberate choice, since the folder name is also what Check 10a's diagram-box match keys off. If the *folder's* name is what's actually wrong (e.g. it should have been named `AT&T` all along), rename the folder itself instead, then update the cell-databook's filename and `title:` together to match the new folder name.

**Check 20 — `images/folder-mapping.png` folder colors match real data**: This diagram has no dedicated check of its own until now (unlike `cat-cell-graph.png`'s Check 15). Every cell shown in this diagram (and in `cat-cell-graph.png`, and in all 12 example diagrams — Check 10i) carries exactly two independent, mechanically-checkable colors: a **fill** color, applied to the cell's own Cell DataBook box — since `c:category` is a fact asserted in the DataBook's own YAML frontmatter, not on some separate notion of "the folder," the DataBook box is what carries the fill; the folder icon drawn alongside it is always plain white and never carries fill — (tan if the cell's `mia.category` resolves to `cat:Person`, light blue if `cat:Organization`, purple/Custom if the cell has no category at all) and a folder-**name-text** color (green/"Predefined" if the cell's `title:` equals the category concept's own `skos:prefLabel` verbatim, plain black/"User-defined" otherwise — and always black for a no-category/Custom cell, since there's no label to match). A cell with no category is only legal if its cell-databook's filename carries the literal `(custom)` disambiguator (see the Cell DataBook Filename Convention) — the two facts (no `mia.category` and a `(custom)` filename) must always agree; either alone without the other is an error. This is a visual check (no automated pixel/OCR comparison), but the script below computes the correct fill and text color for every real cell, for direct cross-reference against whichever diagram box is being checked — e.g. this diagram's "Fred Flintstone" box (category `cat:Others`, folder name "Fred Flintstone") should be tan fill + black text; "People" (category `cat:People`, folder name "People") should be tan fill + green text; the "Friends" box (no category, filename `Friends(custom).databook.md`) should be purple fill + black text. Run:

```python
import glob, re, yaml

text = open('category.ttl').read()
concepts, labels = {}, {}
# category.ttl's tree is a SKOS concept scheme (skos:broader, child -> parent,
# same direction rdfs:subClassOf used to be), not an OWL class hierarchy —
# cat:Person/cat:Organization are top concepts with no skos:broader value of
# their own, so that group is optional in the pattern below.
pattern = re.compile(
    r'cat:([A-Za-z]+(?:\\\(org\\\))?) rdf:type skos:Concept\s*;\s*'
    r'skos:prefLabel "([^"]+)"@en\s*;\s*'
    r'(?:skos:broader cat:([A-Za-z]+(?:\\\(org\\\))?)\s*;\s*)?'
)
for m in pattern.finditer(text):
    child = m.group(1).replace('\\(', '(').replace('\\)', ')')
    labels[child] = m.group(2)
    if m.group(3):
        concepts[child] = m.group(3).replace('\\(', '(').replace('\\)', ')')

def ancestry_root(cls):
    if cls in ('Person', 'Organization'):
        return cls
    seen = set()
    while cls in concepts and cls not in seen:
        seen.add(cls)
        parent = concepts[cls]
        if parent in ('Person', 'Organization'):
            return parent
        cls = parent
    return concepts.get(cls, cls)

for path in sorted(glob.glob('example/Cells/**/*.databook.md', recursive=True)):
    if 'under-development' in path.split('/'):
        continue
    text2 = open(path, encoding='utf-8').read()
    m = re.match(r'^---\n(.*?)\n---', text2, re.DOTALL)
    fm = yaml.safe_load(m.group(1)) if m else None
    if not fm:
        continue
    mia = fm.get('mia', {}) or {}
    category = mia.get('category')
    title = fm.get('title')
    is_custom_filename = path.endswith('(custom).databook.md')
    if not category:
        if not is_custom_filename:
            print(f'INCONSISTENT: {path} has no mia.category but its filename does not carry (custom)')
            continue
        print(f'{title:35s} category={"(none)":28s} label={"":24s} fill={"purple/Custom":24s} text=black/UserDefined')
        continue
    if is_custom_filename:
        print(f'INCONSISTENT: {path} carries a (custom) filename but has mia.category={category!r}')
        continue
    local = category.split(':', 1)[1]
    root = ancestry_root(local)
    fill = 'tan/Person' if root == 'Person' else ('light-blue/Organization' if root == 'Organization' else f'UNKNOWN ROOT ({root})')
    label = labels.get(local, '???')
    text_color = 'green/Predefined' if title == label else 'black/UserDefined'
    print(f'{title:35s} category={category:28s} label={label:24s} fill={fill:24s} text={text_color}')
```

**Check 21 — `:Self` must be a member of every cell in the user's own tree**: A cell-databook under `example/Cells/` — the user's own instance tree — can only ever have gotten there one of two ways: (1) the user created it themselves, in which case they (`:Self`) are trivially a member, or (2) someone else shared it with the user, in which case the share necessarily made `:Self` a member (a cell can't be "shared with" someone without them becoming a member of it). Either way, `:Self` must be one of the cell's active members — i.e. `:Self` must be the `cell:subject` of at least one of that cell's `member` — for **every** cell in `example/Cells/`, regardless of how many members the cell has or what the cell's derived subject (Check 18) is. This is strictest for a cell with only one `member` entry: that entry's subject must be `:Self`, full stop — never the cell's `topic` subject (see Check 18's placement rule above), even when the cell's derived-from-`topic` subject is a third party (e.g. `Jane_Starostina`, `Sophia_Walker`, `Ginger`) and no other graph happens to exist yet. Cells with more members have more room, so `:Self` just needs to be one of the members alongside whichever other real members the cell has (already satisfied by every existing example, e.g. Bob Johnson, Fred Flintstone, Medical Appointment, Boston Hub Society). This is not itself an OWL/SHACL-expressible constraint (same reasoning as Check 18 — it requires dereferencing each `member` value's own `subject`, not just counting or matching cardinalities), so it's checked here instead. Run:

```python
import re, yaml, glob

def frontmatter(path):
    text = open(path, encoding='utf-8').read()
    m = re.match(r'^---\n(.*?)\n---', text, re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else None

violations = 0
for f in glob.glob('example/Cells/**/*.databook.md', recursive=True):
    if 'under-development' in f.split('/'):
        continue
    fm = frontmatter(f)
    if not fm:
        continue
    mia = fm.get('mia', {}) or {}
    if not mia.get('member'):
        continue
    pt = mia.get('member') or []
    pt = pt if isinstance(pt, list) else [pt]
    subs = [t.get('subject') for t in pt]
    if not any(s == ':Self' for s in subs):
        violations += 1
        print(f'VIOLATION member-subjects={subs} (no :Self) {f}')
print('All cells have :Self as a member.' if violations == 0 else f'{violations} violation(s) found.')
```

If a violation is found, add a new minimal graph claimed by and about `:Self` (following the pattern in `Medical.databook.md` under `Pets/Ginger/`, `Jane-Starostina(primary-care-physician).databook.md`, or `Health & Wellness.databook.md` — a single `designated by` → `GivenName` triple is enough), assign it the next free `graph-<NN>`, put it in `member`, and move whatever was in that slot to `topic` instead.

If a diagram box's fill or text color doesn't match this script's output for the corresponding real folder, the diagram wins (per Check 10's own rule) — update `mia.category`/`title:`/filename only if the *data* is actually wrong, otherwise redraw the box.

**Check 23 — `cell:owner`'s two subset invariants: creator ⊆ owner, and owner ⊆ member-subjects**: There is no OWL/SHACL-expressible way to check either invariant, since both require dereferencing values or comparing across fields rather than counting/matching a single path's cardinality — the same reasoning as Checks 18/21. For every cell-databook under `example/Cells/` (excluding `under-development/`) with a `mia.owner` value: (a) `mia.creator`'s own value must appear among `mia.owner`'s values — the creator is always at least the cell's initial owner, and this repo's example data never demonstrates a promotion, so today `mia.owner` always equals exactly `[mia.creator]`, but the check only requires creator ⊆ owner, not equality, so a future example demonstrating promotion (owner as a proper superset of creator) would still pass; (b) every value in `mia.owner` must equal the `cell:subject` of at least one of that cell's `mia.member` graphs — an owner must always be one of the cell's actual members, never a non-member. Run:

```python
import re, yaml, glob

def frontmatter(path):
    text = open(path, encoding='utf-8').read()
    m = re.match(r'^---\n(.*?)\n---', text, re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else None

violations = 0
for f in glob.glob('example/Cells/**/*.databook.md', recursive=True):
    if 'under-development' in f.split('/'):
        continue
    fm = frontmatter(f)
    if not fm:
        continue
    mia = fm.get('mia', {}) or {}
    owner = mia.get('owner') or []
    owner = owner if isinstance(owner, list) else [owner]
    if not owner:
        continue
    creator = mia.get('creator')
    if creator and creator not in owner:
        violations += 1
        print(f'VIOLATION {f}: creator {creator!r} not in owner {owner!r} (creator must be a subset of owner)')

    mt = mia.get('member') or []
    mt = mt if isinstance(mt, list) else [mt]
    member_subs = {t.get('subject') for t in mt}
    bad_owners = [o for o in owner if o not in member_subs]
    if bad_owners:
        violations += 1
        print(f'VIOLATION {f}: owner value(s) {bad_owners} not among member subjects {sorted(s for s in member_subs if s)}')
print('All cells satisfy the owner subset invariants.' if violations == 0 else f'{violations} violation(s) found.')
```

If a violation is found: for (a), add the missing creator value to `mia.owner`. For (b), either add a `mia.member` graph whose subject matches the owner value, or remove that value from `mia.owner` if it doesn't actually belong.

**Check 24 — All of a cell's `topic` graphs must share one common subject**: Check 18's derivation rule takes "the full set of distinct `cell:subject` values among [`topic` entries]" as the cell's subject precisely because a `cell:TopicCell`'s `topic` graphs are always different claims *about the same thing* — a person, pet, vehicle, or trip — described from possibly multiple parties' viewpoints, never a bucket of unrelated subjects glued into one set. So in practice this "set" must always collapse to a single value: for every cell-databook under `example/Cells/` (excluding `under-development/`) whose `mia.topic` holds two or more entries, every one of those entries' `cell:subject` values must be identical (e.g. `Kyoto Trip 2027(trips).databook.md`'s three topic graphs — `graph-69`, Alice's own basic claim, `graph-70`, her travel agent's drafted itinerary, and `graph-91`, Dave's own contribution — all resolve to subject `:Kyoto_Trip_2027`: different claimants, same subject). A cell with zero or one `topic` entries trivially satisfies this. This is not itself an OWL/SHACL-expressible constraint (same reasoning as Checks 18/21/23 — it requires dereferencing each `topic` value's own `subject`, not just counting or matching cardinalities), so it's checked here instead. Run:

```python
import re, yaml, glob

def frontmatter(path):
    text = open(path, encoding='utf-8').read()
    m = re.match(r'^---\n(.*?)\n---', text, re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else None

violations = 0
for f in glob.glob('example/Cells/**/*.databook.md', recursive=True):
    if 'under-development' in f.split('/'):
        continue
    fm = frontmatter(f)
    if not fm:
        continue
    mia = fm.get('mia', {}) or {}
    ot = mia.get('topic') or []
    ot = ot if isinstance(ot, list) else [ot]
    if len(ot) < 2:
        continue
    subs = {t.get('subject') for t in ot}
    if len(subs) > 1:
        violations += 1
        print(f'VIOLATION {f}: topic subjects {sorted(s for s in subs if s)} are not all equal')
print('All cells with 2+ topic entries share one subject.' if violations == 0 else f'{violations} violation(s) found.')
```

If a violation is found: reconsider whether all of the offending `topic` graphs really belong under this one cell — a `topic` graph whose subject doesn't match the others likely belongs in a different (or new) cell instead.

**Check 25 — `cell:topic`'s real upper bound is the cell's own member count: one value per member, each with a distinct claimant**: `cell:topic` carries no OWL/SHACL-expressible maximum — `shacl/cell-shacl.ttl`'s `:TopicCellShape` asserts only `sh:minCount 1` — but in practice its upper bound is exactly the cell's own member count, since every `cell:topic` value represents one of the cell's own members making their own claim about the cell's shared topic (see README.md's Topic Cell section: "the maximum for an N member cell is N ... when every cell member creates its own `c:SCGraph` whose subject is the topic and whose claimant is themselves"). Concretely, two things must both hold for every cell-databook under `example/Cells/` (excluding `under-development/`) that carries a `mia.topic` value: (a) each `topic` entry's `cell:claimant` must be one of that same cell's own `mia.member` subjects — a `topic` graph is never claimed by a non-member; and (b) no two `topic` entries may share the same claimant — each member gets at most one claim on the cell's topic. Together these two facts are what actually cap the count at the member count, rather than any cardinality restriction. This is not itself an OWL/SHACL-expressible constraint (same reasoning as Checks 18/21/23/24 — it requires dereferencing each `topic` value's own `claimant`, and each `member` value's own `subject`, not just counting or matching cardinalities), so it's checked here instead. Run:

```python
import re, yaml, glob

def frontmatter(path):
    text = open(path, encoding='utf-8').read()
    m = re.match(r'^---\n(.*?)\n---', text, re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else None

violations = 0
for f in glob.glob('example/Cells/**/*.databook.md', recursive=True):
    if 'under-development' in f.split('/'):
        continue
    fm = frontmatter(f)
    if not fm:
        continue
    mia = fm.get('mia', {}) or {}
    ot = mia.get('topic') or []
    ot = ot if isinstance(ot, list) else [ot]
    if not ot:
        continue
    mt = mia.get('member') or []
    mt = mt if isinstance(mt, list) else [mt]
    member_subs = {t.get('subject') for t in mt}
    topic_claimants = [t.get('claimant') for t in ot]
    non_member = [c for c in topic_claimants if c not in member_subs]
    if non_member:
        violations += 1
        print(f"VIOLATION {f}: topic claimant(s) {non_member} are not among this cell's own member subjects {sorted(s for s in member_subs if s)}")
    seen, dup = set(), set()
    for c in topic_claimants:
        (dup.add(c) if c in seen else seen.add(c))
    if dup:
        violations += 1
        print(f'VIOLATION {f}: topic claimant(s) {sorted(dup)} repeat — each cell:topic value must be claimed by a distinct member')
print('All cells satisfy the topic-claimant invariants (member-only, distinct).' if violations == 0 else f'{violations} violation(s) found.')
```

If a violation is found: for a `topic` claimed by a non-member, either add that claimant as a new `member` entry first, or reconsider whether the claim really belongs in this cell's `topic` at all. For a repeated claimant across two `topic` entries, merge the two graphs' content into one — a single member can only make one claim about the cell's topic, not two.

**Check 26 — `cell:template`'s declared shape must resolve to an `rdf:type` actually asserted in the graph's own embedded Turtle body**: `cell:template` (`cell.ttl`) is a real `owl:AnnotationProperty` (domain `cell:Graph`, range `sh:NodeShape`, cardinality 0..N), synthesized into RDF from each `mia.member[]`/`mia.topic[].template` YAML value by `helpers/yaml-to-rdf.py`. But SHACL validation of a templated graph never actually reads this synthesized triple — each per-template shape (e.g. `:PassportShape`) fires purely via its own `sh:targetClass`, matching whatever `rdf:type` is asserted directly in the graph's body (e.g. `:Alice_US_Passport rdf:type identitydocuments:Passport`), completely independent of `template:`. So nothing else cross-checks that a graph's declared `template:` shape's own `sh:targetClass` actually names a class asserted on an individual in its own body — a typo'd or stale `template:` value would go undetected, silently decoupled from what SHACL is actually validating. This is not itself an OWL/SHACL-expressible constraint (same reasoning as Checks 18/21/23/24/25 — it requires dereferencing the graph's own embedded Turtle content, not just its YAML frontmatter), so it's checked here instead. For every cell-databook under `example/Cells/` (excluding `under-development/`) with a `mia.member[]`/`mia.topic[].template` value, resolve that shape CURIE to its own `sh:targetClass` (scanning `shacl/persona-shacl.ttl`/`shacl/contactinfo-shacl.ttl` and every `other/shacl/*-shacl.ttl` file for the matching shape, the same scan Check 27 already runs), then verify that same graph's own embedded Turtle body asserts `rdf:type` (directly, on some individual) to that resolved class. **Exempt**: `pshapes:ContactInfoShape` — per `helpers/validate.py`'s own module docstring, this is the one broad, class-wide shape (`sh:targetClass persona:Person`) that `helpers/validate.py` re-targets, at actual validation time, at only the *substantive* `persona:Person` individual(s) present in a graph, precisely because it's legitimate for a `cell:member`-list graph (required to carry this shape unconditionally, per Check 30) to contain none at all — e.g. graph-01 (`Boston Hub Society(affiliations).databook.md`) is BHS's own self-claimed member stub, typing `:BHS` only as `o:Organization`, never `persona:Person`, and graph-27 (`Citibank(banking-payments).databook.md`) is Alice's own note about Citibank as an institution, typing `:Citibank` the same way — both correctly, vacuously satisfy `ContactInfoShape` in real SHACL validation with zero `persona:Person` individuals present. Every other shape (e.g. `idocshapes:PassportShape`, `bankingshapes:DebitCardShape`) targets a narrow, specific document/account class that a declaring graph always does instantiate for real (e.g. the real Citibank debit-card graph asserts `rdf:type cco:ent00000051` directly, alongside `banking:DebitCard`), so no other exemption is needed. Run:

```python
import re, yaml, glob
from databook_graphs import as_list, split_frontmatter, extract_graph_block

def read(path):
    return open(path, encoding='utf-8').read()

# --- shape CURIE -> sh:targetClass CURIE, scanned across every file that can
# define a shape under a given prefix (same scan Check 27/28 already run).
SHAPES_FILES = {
    'pshapes': ['shacl/persona-shacl.ttl', 'shacl/contactinfo-shacl.ttl'],
    'petshapes': ['other/shacl/pets-shacl.ttl'],
    'vehicleshapes': ['other/shacl/vehicles-shacl.ttl'],
    'idocshapes': ['other/shacl/identity-documents-shacl.ttl'],
    'mashapes': ['other/shacl/medical-appointments-shacl.ttl'],
    'sashapes': ['other/shacl/service-accounts-shacl.ttl'],
    'bankingshapes': ['other/shacl/banking-shacl.ttl'],
    'residenceshapes': ['other/shacl/residences-shacl.ttl'],
    'itineraryshapes': ['other/shacl/itineraries-shacl.ttl'],
    'oshapes': ['shacl/organization-shacl.ttl'],
}
target_class = {}
for prefix, paths in SHAPES_FILES.items():
    for path in paths:
        for m in re.finditer(r':(\w+)\s*\n\s*a sh:NodeShape\s*;.*?sh:targetClass\s+(\S+?)\s*;', read(path), re.DOTALL):
            target_class[f'{prefix}:{m.group(1)}'] = m.group(2)

def load(path):
    fm_text, _, body = split_frontmatter(read(path))
    return yaml.safe_load(fm_text), body

def strip_comments(text):
    return '\n'.join(re.sub(r'#.*$', '', line) for line in text.split('\n'))

violations = 0
for f in glob.glob('example/Cells/**/*.databook.md', recursive=True):
    if 'under-development' in f.split('/'):
        continue
    fm, body = load(f)
    if not fm:
        continue
    mia = fm.get('mia') or {}
    for g in as_list(mia.get('member')) + as_list(mia.get('topic')):
        if not isinstance(g, dict) or not g.get('template'):
            continue
        raw, gid = g['template'], g['id']
        templates = raw if isinstance(raw, list) else [raw]  # template: may be a scalar or a list
        block = None
        for shape in templates:
            if shape == 'pshapes:ContactInfoShape':
                continue  # broad, vacuously-satisfiable shape exemption — see prose above
            resolved = target_class.get(shape)
            if resolved is None:
                violations += 1
                print(f"VIOLATION {f}: graph {gid} declares template {shape!r}, no sh:targetClass found for it")
                continue
            if block is None:
                block = extract_graph_block(body, f"{gid}#graph")
                if block is None:
                    print(f'VIOLATION {f}: no turtle block found for {gid} (declares template {shape!r})')
                    violations += 1
                    break
                text = strip_comments('\n'.join(block))
            found = any(
                resolved in [t.strip() for t in m.group(1).split(',')]
                for m in re.finditer(r'rdf:type\s+(.*?)[;.]', text, re.DOTALL)
            )
            if not found:
                violations += 1
                print(f"VIOLATION {f}: graph {gid} declares template {shape!r} (resolves to {resolved!r}) but no individual in its own turtle block is asserted rdf:type {resolved!r}")
print('All template-declaring graphs assert a matching rdf:type.' if violations == 0 else f'{violations} violation(s) found.')
```

If a violation is found: fix whichever side is wrong — either the `template:` YAML value (if the graph's own asserted `rdf:type` is correct and `template:` names the wrong shape), or the body's `rdf:type` (if `template:` reflects the intended classification and the body was never updated to match).

**Check 27 — a `member` graph's `template:` value, and a template-driven `topic` graph's, must be among its cell's own `TemplateCell`'s `memberGraphShape`/`topicGraphShape` values**: Check 26 only compares a graph's `template:` value against its own body content — this check goes one level higher, comparing it against what the cell's *category* itself declares as authoritative. For every cell-databook under `example/Cells/` (excluding `under-development/`) whose `mia.category` value names a concept with a matching `cell:TemplateCell` in `cat-templates.ttl` (reverse lookup, per Check 18/README's Lazy Instantiation pattern — a category with no such template has nothing to check here; a defensive branch, not a live case today, since Check 29 guarantees every `category.ttl` concept except `cat:Person`/`cat:Organization` has one): every `mia.member`-list graph carrying a `template:` value must have that value be one of the TemplateCell's own `cell:memberGraphShape` value(s); and, **only when that TemplateCell carries `cell:isTopicCell true`**, every `mia.topic`-list graph carrying a `template:` value must be one of its `cell:topicGraphShape` value(s) — direct CURIE-set membership, since `cell:template`'s range is `sh:NodeShape` (`cell.ttl`), the same range `cell:memberGraphShape`/`cell:topicGraphShape` already carry, so no `sh:targetClass` resolution and no named exemptions are needed any more. A `topic` on an `cell:isTopicCell false` cell is **not** checked against `cell:topicGraphShape` at all: any `cell:MemberCell` may gain up to one manually-added `topic` (see Check 31), and the user picks that topic's template themselves — from the app's full list of shapes, not from anything the cell's category declares — so the category's own `TemplateCell` has no authority over it. Where such a template does carry a `cell:topicGraphShape` alongside `cell:isTopicCell false` (`cat:ImmediateFamily` today), that value is a *hint* — the template the app offers first for a manually-added topic — never a restriction. This is not itself an OWL/SHACL-expressible constraint (same reasoning as Checks 18/21/23/24/25/26 — it requires dereferencing values across `cat-templates.ttl`, not just cell-databook YAML), so it's checked here instead. Run:

```python
import re, yaml, glob

def read(path):
    return open(path, encoding='utf-8').read()

# --- cat-templates.ttl: category local name -> {'member': [...], 'topic': [...]} shape CURIEs ---
category_shapes = {}
for block in re.split(r'\n\n(?=ctpl:)', read('cat-templates.ttl')):
    if 'rdf:type cell:Cell, cell:TemplateCell' not in block:
        continue
    # [^\s;]+ (not just \w+) so an org-side category's escaped parens
    # (cat:BankingPayments\(org\)) aren't truncated to the same local name
    # as its person-side sibling (cat:BankingPayments) — \w+ alone would
    # collide the two and silently overwrite one's entry with the other's.
    m_cat = re.search(r'cell:category\s+cat:([^\s;]+)', block)
    if not m_cat:
        continue
    cat_local = m_cat.group(1).replace('\\(', '(').replace('\\)', ')')
    def shape_values(prop, block):
        # Handles both a single value (cell:memberGraphShape pshapes:X ;)
        # and a comma-separated list (cell:memberGraphShape pshapes:X , pshapes:Y ;) —
        # cell:memberGraphShape/cell:topicGraphShape are 0..N, so a template may
        # carry more than one value under one predicate.
        m = re.search(prop + r'\s+([^;]+);', block)
        return [v.strip() for v in m.group(1).split(',')] if m else []
    m_itc = re.search(r'cell:isTopicCell\s+(true|false)\s*\.', block)
    category_shapes[cat_local] = {
        'member': shape_values('cell:memberGraphShape', block),
        'topic': shape_values('cell:topicGraphShape', block),
        'is_topic_cell': bool(m_itc) and m_itc.group(1) == 'true',
    }

def frontmatter(path):
    text = read(path)
    m = re.match(r'^---\n(.*?)\n---', text, re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else None

violations = 0
for path in sorted(glob.glob('example/Cells/**/*.databook.md', recursive=True)):
    if 'under-development' in path.split('/'):
        continue
    fm = frontmatter(path)
    if not fm:
        continue
    mia = fm.get('mia', {}) or {}
    category = mia.get('category')
    if not category:
        continue
    shapes = category_shapes.get(category.split(':', 1)[1])
    if not shapes:
        continue  # no TemplateCell for this category — nothing to check

    for field in ('member', 'topic'):
        # A topic on an isTopicCell:false cell is manually added — the user picks
        # its template freely, so the category's TemplateCell has no say. Skip it.
        if field == 'topic' and not shapes['is_topic_cell']:
            continue
        entries = mia.get(field) or []
        entries = entries if isinstance(entries, list) else [entries]
        allowed = set(shapes[field])
        for entry in entries:
            gid = entry.get('id')
            raw = entry.get('template')
            templates = raw if isinstance(raw, list) else ([raw] if raw else [])
            for template in templates:
                if not allowed:
                    violations += 1
                    print(f"VIOLATION {path}: {field} graph {gid} declares template {template!r} but cell's TemplateCell has no cell:{field}GraphShape value(s)")
                elif template not in allowed:
                    violations += 1
                    print(f"VIOLATION {path}: {field} graph {gid} declares template {template!r}, not among cell's TemplateCell's cell:{field}GraphShape value(s) {sorted(allowed)}")
print("All member/topic template: values match their TemplateCell's shape value(s)." if violations == 0 else f'{violations} violation(s) found.')
```

If a violation is found: either the graph's `template:` value is wrong (fix it to match the shape actually reachable via the cell's `category`), or the `cell:memberGraphShape`/`cell:topicGraphShape` value(s) on the corresponding `cat-templates.ttl` individual are wrong/missing (add the correct one), or — if the graph is genuinely a different kind of content than its category's template expects — reconsider whether it belongs in `member` vs. `topic`, or under this category at all.

**Check 28 — every `cell:member` graph of a categorized cell must carry every `c:template` value its category's `TemplateCell` requires**: Check 27 only checks a `template:` value once one is already present — it never flags a `mia.member`-list graph that's missing one outright. This check closes that gap: the mapping is unconditional — whenever a `cell:TemplateCell` of category X carries one or more `cell:memberGraphShape` values, **every** real cell of category X must have every one of its `mia.member`-list graphs carry those exact shape CURIEs as `c:template` value(s) too, with no exception for which shape it happens to be (see `APP-BEHAVIOR.md`'s Lazy Instantiation section) — direct CURIE-set membership, since `cell:template`'s range is `sh:NodeShape` (`cell.ttl`), the same range `cell:memberGraphShape` already carries, so no label resolution and no named exceptions are needed any more. For every cell-databook under `example/Cells/` (excluding `under-development/`) whose `mia.category` value names a concept with a matching `cell:TemplateCell` in `cat-templates.ttl` that carries at least one `cell:memberGraphShape` value: every `mia.member`-list graph must carry a `template:` value (or list of values) that includes each of those shape CURIEs. A category with no `cell:TemplateCell` at all has nothing to check here — a defensive branch, not a live case today, since Check 29 guarantees every `category.ttl` concept except `cat:Person`/`cat:Organization` has one. This is not itself an OWL/SHACL-expressible constraint (same reasoning as Checks 18/21/23/24/25/26/27 — it requires dereferencing values across `cat-templates.ttl`, not just cell-databook YAML), so it's checked here instead. Run:

```python
import re, yaml, glob

def read(path):
    return open(path, encoding='utf-8').read()

category_shapes = {}
for block in re.split(r'\n\n(?=ctpl:)', read('cat-templates.ttl')):
    if 'rdf:type cell:Cell, cell:TemplateCell' not in block:
        continue
    # [^\s;]+ (not just \w+) — see Check 27's identical fix for why: \w+
    # would collide cat:BankingPayments with cat:BankingPayments\(org\).
    m_cat = re.search(r'cell:category\s+cat:([^\s;]+)', block)
    if not m_cat:
        continue
    cat_local = m_cat.group(1).replace('\\(', '(').replace('\\)', ')')
    def shape_values(prop, block):
        m = re.search(prop + r'\s+([^;]+);', block)
        return [v.strip() for v in m.group(1).split(',')] if m else []
    category_shapes[cat_local] = shape_values('cell:memberGraphShape', block)

def frontmatter(path):
    text = read(path)
    m = re.match(r'^---\n(.*?)\n---', text, re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else None

violations = 0
for path in sorted(glob.glob('example/Cells/**/*.databook.md', recursive=True)):
    if 'under-development' in path.split('/'):
        continue
    fm = frontmatter(path)
    if not fm:
        continue
    mia = fm.get('mia', {}) or {}
    category = mia.get('category')
    if not category:
        continue
    member_shapes = category_shapes.get(category.split(':', 1)[1])
    if not member_shapes:
        continue  # no TemplateCell (or none with a memberGraphShape) for this category
    required = set(member_shapes)
    if not required:
        continue
    entries = mia.get('member') or []
    entries = entries if isinstance(entries, list) else [entries]
    for entry in entries:
        gid = entry.get('id')
        raw = entry.get('template')
        have = set(raw if isinstance(raw, list) else ([raw] if raw else []))
        missing = required - have
        if missing:
            violations += 1
            print(f"VIOLATION {path}: member graph {gid} is missing required c:template value(s) {sorted(missing)}")
print("All c:member graphs carry every c:template value their category's TemplateCell requires." if violations == 0 else f'{violations} violation(s) found.')
```

If a violation is found: add the missing `template:` value(s) to the graph's own `mia.member[]`/`mia.topic[]` entry, and verify (per Check 26) that the graph's own individual — usually its `:Self` claim, per the required minimal `GivenName` stub — actually satisfies the shape's constraints; if it doesn't yet, add the missing content rather than just the template value.

**Check 29 — every category concept has a matching `cell:TemplateCell`**: Every `skos:Concept` in `category.ttl`'s `cat:CategoryScheme`, except the two SKOS top concepts `cat:Person`/`cat:Organization` themselves (no real leaf cell is ever instantiated as bare "Person" or "Organization" — every example cell uses a narrower concept), must have a matching `cell:TemplateCell` in `cat-templates.ttl` carrying that same `cell:category` value. This is the reverse direction of Check 27/28's own reverse lookup (`?tc cell:category cat:X`): those checks skip a category outright when it has no `TemplateCell` at all, so nothing previously caught a category that should have one but doesn't. This is not itself an OWL/SHACL-expressible constraint (same reasoning as Checks 18/21/23/24/25/26/27/28 — it requires cross-referencing every concept in `category.ttl` against every individual in `cat-templates.ttl`, not just parsing one file's own YAML or Turtle in isolation), so it's checked here instead. Run:

```python
import re

def read(path):
    return open(path, encoding='utf-8').read()

cat_text = read('category.ttl')
pattern = re.compile(
    r'cat:([A-Za-z]+(?:\\\(org\\\))?) rdf:type skos:Concept\s*;\s*'
    r'skos:prefLabel "([^"]+)"@en',
)
concepts = {}
for m in pattern.finditer(cat_text):
    local = m.group(1).replace('\\(', '(').replace('\\)', ')')
    concepts[local] = m.group(2)

EXCLUDE = {'Person', 'Organization'}

ct_text = read('cat-templates.ttl')
existing = set()
for block in re.split(r'\n\n(?=ctpl:)', ct_text):
    if 'rdf:type cell:Cell, cell:TemplateCell' not in block:
        continue
    # [^\s;]+ (not just \w+) — see Check 27's identical fix for why: \w+
    # would collide cat:BankingPayments with cat:BankingPayments\(org\).
    m_cat = re.search(r'cell:category\s+cat:([^\s;]+)', block)
    if m_cat:
        existing.add(m_cat.group(1).replace('\\(', '(').replace('\\)', ')'))

missing = sorted(set(concepts) - EXCLUDE - existing)
violations = 0
for m in missing:
    violations += 1
    print(f"VIOLATION: cat:{m} ({concepts[m]!r}) has no matching cell:TemplateCell in cat-templates.ttl")
print("Every category concept (except cat:Person/cat:Organization) has a matching TemplateCell." if violations == 0 else f'{violations} violation(s) found.')
```

If a violation is found: add a new `ctpl:XTemplateCell` individual to `cat-templates.ttl` for the missing category, following the standard pattern for a category with no document type of its own — `cell:memberGraphShape pshapes:ContactInfoShape` and `cell:isTopicCell false`, no `cell:topicGraphShape` — unless the category genuinely has its own document/record type, in which case follow the pattern of an existing `isTopicCell: true` template instead (see `cat-templates.ttl`'s row in core-files.md). Adding a new `TemplateCell` also brings the category into scope for Check 27/28 and `helpers/validate.py` — re-run those afterward, since any real cell already using that category will now need its own `mia.member` graph(s) tagged with the matching `c:template` value (Check 28) and may need minimal `GivenName` content added if it doesn't already have any (Check 26 and the template pass).

**Check 30 — every `cell:TemplateCell` individual carries `cell:memberGraphShape pshapes:ContactInfoShape`**: Every one of the 18 (now more) `cell:memberGraphShape` values in `cat-templates.ttl` happens to be the identical `pshapes:ContactInfoShape` — a `c:member` graph is always validated as a basic business-card profile, regardless of category, while any category-specific content lives in `c:topic` instead (see `APP-BEHAVIOR.md`'s Lazy Instantiation section). This is asserted directly and explicitly on every individual, on purpose — not hoisted onto the `cell:TemplateCell` class itself via an OWL restriction, since nothing in this project's own validation pipeline runs a reasoner to materialize such an entailment (`helpers/validate.py` validates literal asserted triples only, via `riot`/`shacl validate`), and Check 27/28/29 above all rely on literally finding this triple in each individual's own block. This check exists to catch a new `TemplateCell` added without it (e.g. by hand, skipping the standard pattern), which SHACL itself wouldn't catch either (`shacl/cell-shacl.ttl`'s `:TemplateCellShape` allows zero or more `cell:memberGraphShape` values, no minimum). This is not itself an OWL/SHACL-expressible constraint (same reasoning as Check 29 above), so it's checked here instead. Run:

```python
import re

def read(path):
    return open(path, encoding='utf-8').read()

ct_text = read('cat-templates.ttl')
violations = 0
for block in re.split(r'\n\n(?=ctpl:)', read('cat-templates.ttl')):
    if 'rdf:type cell:Cell, cell:TemplateCell' not in block:
        continue
    m_name = re.search(r'^(ctpl:\w+)', block)
    name = m_name.group(1) if m_name else '(unknown)'
    # ^\s*cell:memberGraphShape anchors to a real triple's own line — avoids
    # matching the same words if they appear mid-sentence inside an
    # rdfs:comment (e.g. ctpl:UserDefinedTemplateCell's own prose, which
    # happens to describe this exact property in its explanatory text).
    m_shapes = re.search(r'^\s*cell:memberGraphShape\s+([^;]+);', block, re.MULTILINE)
    values = [v.strip() for v in m_shapes.group(1).split(',')] if m_shapes else []
    if 'pshapes:ContactInfoShape' not in values:
        violations += 1
        print(f"VIOLATION: {name} does not carry cell:memberGraphShape pshapes:ContactInfoShape")
print("Every TemplateCell carries cell:memberGraphShape pshapes:ContactInfoShape." if violations == 0 else f'{violations} violation(s) found.')
```

If a violation is found: add `cell:memberGraphShape pshapes:ContactInfoShape` to the offending `TemplateCell` individual — every `TemplateCell`, with no exception, carries it.

**Check 31 — every cell whose category's `TemplateCell` has `cell:isTopicCell true` carries a `topic` value, and an `cell:isTopicCell false` cell carries at most one**: `cell:isTopicCell` (`cat-templates.ttl`) says what Lazy Instantiation's clone of a category is expected to look like — `true` means a real cell of that category is expected to carry `cell:topic` content (and be typed `cell:TopicCell`), `false` means none is expected at instantiation. This check tests that expectation in one direction only: `isTopicCell true` with no `topic` value on a real cell is always a violation — either the cell is missing content its category promises, or the template is stale. The converse is **not** a violation: **any** `cell:MemberCell` may gain up to one manually-added `cell:topic`, whatever its category's template says — the user picks that topic's own template from the app's full list of shapes (see APP-BEHAVIOR.md's [Adding a Topic](APP-BEHAVIOR.md#adding-a-topic)), and the cell is then typed `cell:TopicCell` all the same (e.g. cell-12, `Sophia Walker(immediate-family).databook.md` — a `cat:ImmediateFamily` cell, `isTopicCell false`, whose Contact Info topic about Alice's daughter Sophia was added by hand because Sophia has no instance of the app and so cannot be one of the cell's members). Neither the category's `TemplateCell` nor Check 27 constrains which shape that manually-added topic uses; where such a template does carry a `cell:topicGraphShape` alongside `cell:isTopicCell false` (`cat:ImmediateFamily` today), it is a hint at the template to offer first, not a restriction — legal, and constrained by Check 32 only in the `true` case.

The one thing this check does enforce in that direction is the count: **up to one** manually-added topic, so a cell whose category's template says `isTopicCell false` carrying two or more `topic` values is a violation. (An `isTopicCell true` cell's topics are template-driven instead, and are capped by Check 25's member-count bound rather than by this rule.) This is not itself an OWL/SHACL-expressible constraint (same reasoning as Checks 18/21/23/24/25/26/27/28/29/30 — it requires cross-referencing each cell's own `category` against `cat-templates.ttl`'s `isTopicCell` value, not just parsing one file's own YAML or Turtle in isolation), so it's checked here instead. Run:

```python
import re, yaml, glob

def read(path):
    return open(path, encoding='utf-8').read()

# category local name -> isTopicCell boolean, parsed from cat-templates.ttl
category_is_topic_cell = {}
for block in re.split(r'\n\n(?=ctpl:)', read('cat-templates.ttl')):
    if 'rdf:type cell:Cell, cell:TemplateCell' not in block:
        continue
    # [^\s;]+ (not just \w+) — see Check 27's identical fix for why: \w+
    # would collide cat:BankingPayments with cat:BankingPayments\(org\).
    m_cat = re.search(r'cell:category\s+cat:([^\s;]+)', block)
    if not m_cat:
        continue  # e.g. ctpl:UserDefinedTemplateCell, which carries no cell:category
    cat_local = m_cat.group(1).replace('\\(', '(').replace('\\)', ')')
    m_topic = re.search(r'cell:isTopicCell\s+(true|false)\s*\.', block)
    category_is_topic_cell[cat_local] = (m_topic.group(1) == 'true') if m_topic else None

def frontmatter(path):
    text = open(path, encoding='utf-8').read()
    m = re.match(r'^---\n(.*?)\n---', text, re.DOTALL)
    return yaml.safe_load(m.group(1)) if m else None

violations = 0
for path in sorted(glob.glob('example/Cells/**/*.databook.md', recursive=True)):
    if 'under-development' in path.split('/'):
        continue
    fm = frontmatter(path)
    if not fm:
        continue
    mia = fm.get('mia', {}) or {}
    category = mia.get('category')
    cat_local = category.split(':', 1)[1] if category else None
    if cat_local is None:
        continue  # UserDefined/Custom cell — no TemplateCell to check against
    expected = category_is_topic_cell.get(cat_local)
    if expected is None:
        continue  # no matching TemplateCell (Check 29 already flags this) — nothing to compare
    topics = mia.get('topic') or []
    topics = topics if isinstance(topics, list) else [topics]
    if expected:
        # isTopicCell true demands a topic (count capped by Check 25 instead).
        if not topics:
            violations += 1
            print(f"VIOLATION {path}: category cat:{cat_local} has isTopicCell=true but cell has no topic value")
    elif len(topics) > 1:
        # A manually-added topic on an isTopicCell false cell is legal (see
        # cell-12), but at most one may be added by hand.
        violations += 1
        print(f"VIOLATION {path}: category cat:{cat_local} has isTopicCell=false but cell carries {len(topics)} topic values (at most one may be added manually)")
print("Every isTopicCell:true cell carries a topic value; no isTopicCell:false cell carries more than one." if violations == 0 else f'{violations} violation(s) found.')
```

If a violation is found, the cell is missing a `topic` value its category promises — add that content. This includes a purely organizational scaffold cell that reuses a templated category from a nested descendant cell (e.g. `Companies`/`Google`/`ATT` all sharing `cat:Companies`), which this check does **not** exempt: every real cell of an `isTopicCell true` category, scaffold or leaf, must carry its own `topic` — though it may be empty. If the category genuinely should not carry topic content at all, flip its `TemplateCell` to `isTopicCell false` instead; the shape it names via `cell:topicGraphShape` may stay, now describing a topic a user might add by hand. Paula Walker's and Alice Walker's Acme cells reuse `cat:Employees` directly, the same "child folder reuses its parent's category" pattern `Pets`/`Ginger` already use — there is no narrower `cat:Employee` category for either.

**Check 32 — every `cell:TemplateCell` with `cell:isTopicCell true` carries at least one `cell:topicGraphShape` value**: `cell:isTopicCell true` promises that a real cell of this category is expected to carry `cell:topic` content (Check 31) — but without a `cell:topicGraphShape` telling the template pass what to validate that content against, this promise is unenforceable and the template is incomplete. This is the mirror image of Check 30 (every `TemplateCell` carries `cell:memberGraphShape`, unconditionally) — this one instead conditions on `isTopicCell`, since `cell:topicGraphShape` itself is optional whenever `isTopicCell` is `false`: usually absent (the ordinary, non-topic case), but legitimately present on a category where a user may add a topic by hand (`cat:ImmediateFamily` — see Check 31), which this check therefore never flags. A `topicGraphShape` here doesn't have to govern a real reified document type — see `cat:HealthWellness`/`cat:PetsCareAndFeeding`'s own shapes, each formalizing a real but modest content pattern (physical characteristics; a pet's identifying properties, all made optional) with no `template:` value ever asserted on the actual topic graph itself — the shape's existence satisfies this check regardless of whether any real graph's `template:` field currently points at it. `cat:PrimaryCarePhysician`'s own topic shape (a physician's specialty, also modest) is the one exception that does have a real `template:` value asserted — Dr. Jane Starostina's own topic graph (graph-25) carries both `pshapes:PrimaryCarePhysicianShape` and `pshapes:ContactInfoShape`. This is not itself an OWL/SHACL-expressible constraint (same reasoning as Checks 18/21/23/24/25/26/27/28/29/30/31 — it requires reading two different properties off the same `cat-templates.ttl` individual and comparing them, not a single path's cardinality), so it's checked here instead. Run:

```python
import re

def read(path):
    return open(path, encoding='utf-8').read()

violations = 0
for block in re.split(r'\n\n(?=ctpl:)', read('cat-templates.ttl')):
    if 'rdf:type cell:Cell, cell:TemplateCell' not in block:
        continue
    m_name = re.search(r'^(ctpl:\w+)', block)
    name = m_name.group(1) if m_name else '(unknown)'
    m_topic_cell = re.search(r'cell:isTopicCell\s+(true|false)\s*\.', block)
    is_topic_cell = m_topic_cell and m_topic_cell.group(1) == 'true'
    if not is_topic_cell:
        continue
    has_shape = re.search(r'^\s*cell:topicGraphShape\s+', block, re.MULTILINE)
    if not has_shape:
        violations += 1
        print(f"VIOLATION: {name} has cell:isTopicCell true but no cell:topicGraphShape value")
print("Every isTopicCell:true TemplateCell carries a topicGraphShape." if violations == 0 else f'{violations} violation(s) found.')
```

If a violation is found: either add a `cell:topicGraphShape` value to the offending `TemplateCell` (a new or existing SHACL shape describing its real topic content, even a modest one with every property optional), or flip `cell:isTopicCell` back to `false` if the category shouldn't be a `TopicCell` at all — never leave the mismatch standing.
