```markdown
# LaneForge

A single-file browser tool for aligning a [lanelet2](https://github.com/fzi-forschungszentrum-informatik/Lanelet2) vector map to its point cloud and repairing the routing graph before you feed it to Autoware[cite: 3].

Open `laneforge.html` in a browser[cite: 3]. There is no build step, no server and no dependencies[cite: 3]. Your files are read locally by the page — nothing is uploaded anywhere[cite: 3].

![LaneForge: the Routing pane, with lanelets coloured by connectivity and direction chevrons along each one](screenshot.png)[cite: 3]

---

## Why

An HD map arriving from a mapping tool or a simulator export usually needs two kinds of work before it is usable[cite: 3]:

1. **Registration** — the vector map sits a metre or two off the point cloud, or is rotated slightly[cite: 3].
2. **Routing** — lanelets that should follow one another do not, because their boundaries do not quite share the same nodes; some lanelets are drawn backwards; some have their left and right boundaries labelled the wrong way round[cite: 3].

Both are tedious to fix in a text editor and hard to see in a generic viewer[cite: 3]. LaneForge shows the two layers on top of each other, lets you nudge one onto the other, then shows the routing graph as colour and lets you repair it lanelet by lanelet[cite: 3].

---

## Quick start

1. Open `laneforge.html`[cite: 3].
2. **Align** tab → load your `.pcd` and your `.osm` (click the slots, or drag the files onto the page)[cite: 3].
3. **Autoware frame** → load your `map_projector_info.yaml`, or tick **generate** if you have none (see below — skipping this is why a map can look aligned here and still be off in Autoware)[cite: 3].
4. Drag in the view to slide the vector map onto the cloud[cite: 3]. Arrow keys nudge by the selected step[cite: 3].
5. **Routing** tab → inspect the graph, fix what is broken[cite: 3].
6. **Export** — the corrected `.osm`, plus `map_projector_info.yaml` when generating[cite: 3].

The page opens on a small synthetic demo scene so every control works before you load anything[cite: 3].

---

## Align

The point cloud is the fixed reference; the vector map moves[cite: 3].

- **Drag** in the view moves the map[cite: 3]. Middle-drag, right-drag, `Space`-drag or `Alt`-drag pans the view[cite: 3].
- **Arrow keys** nudge by the step selected (0.05 / 0.25 / 1 / 5 m)[cite: 3]. `Shift` multiplies by ten[cite: 3].
- **ΔX / ΔY / yaw** can be typed directly[cite: 3]. Yaw rotates about the map's own centre[cite: 3].
- Cloud brightness, opacity, point size and lane width are adjustable; the cloud is coloured by height[cite: 3].

Once you leave the Align tab the offset is **locked**, so a stray click hours into a routing session cannot move the map[cite: 3]. Arrow keys say so rather than silently doing nothing[cite: 3].

Large clouds are subsampled for display (3 M points; the ratio is shown)[cite: 3]. Panning drops to a coarser raster and returns to full detail when you stop[cite: 3].

Supported `.pcd`: `ascii`, `binary` and `binary_compressed` (LZF), with `x`/`y`/`z` in any field layout[cite: 3].

---

## Autoware frame (`map_projector_info.yaml`)

Autoware does not use the positions this page draws[cite: 3]. It rebuilds each node's metric position itself, following `map_projector_info.yaml` (`autoware_map_loader`), and loads the point cloud unchanged in that same frame[cite: 3]:

| `projector_type` | where Autoware puts a node |
| --- | --- |
| `Local` | at its `local_x` / `local_y` tags — lat/lon ignored[cite: 3] |
| `MGRS` | its lat/lon projected to UTM, then taken **inside its own 100 km square** (0–100 000 m) — local tags ignored[cite: 3] |
| `LocalCartesianUTM` | its lat/lon in UTM, minus `map_origin` — local tags ignored[cite: 3] |
| `TransverseMercator` | its lat/lon in transverse Mercator about `map_origin` — local tags ignored[cite: 3] |
| `LocalCartesian` | east/north of `map_origin` — local tags ignored[cite: 3] |

With no yaml at all, Autoware assumes **MGRS**[cite: 3]. So a map aligned by its `local_x`/`local_y` tags only lines up in Autoware when the yaml is `Local`; with any other type the alignment is lost, by however far the file's lat/lon disagree with its local tags — metres, or tens of kilometres for MGRS[cite: 3]. Vector Map Builder works the same way: it places the map from lat/lon in an MGRS square and the cloud as-is[cite: 3].

**You have the yaml.** Load it[cite: 3]. The map is then drawn exactly the way Autoware builds it, so what lines up here lines up there[cite: 3]. Export writes lat/lon through that projector (and `local_x`/`local_y` to the same values)[cite: 3]. A map that crosses an MGRS square boundary is reported, because Autoware's MGRS projector tears it apart there[cite: 3].

**You don't.** Tick **generate** and pick a type; the `.osm` and a matching `map_projector_info.yaml` download together[cite: 3]. The map stays in its own frame while you align, and export writes coordinates that put every node where you left it[cite: 3]:

- **Local** (default) — for a cloud from CARLA or another simulator[cite: 3]. Autoware reads `local_x`/`local_y`, added to every node where missing[cite: 3]. The cloud is never modified[cite: 3]. Lat/lon-based features (GNSS localisation) are off with this type[cite: 3].
- **LocalCartesianUTM / TransverseMercator / LocalCartesian** — the same frame, but georeferenced at the origin you enter; lat/lon are written to match[cite: 3].
- **MGRS** — only when every node sits between 0 and 100 000 m[cite: 3]. A cloud centred near 0,0, as CARLA's is, has negative coordinates that MGRS cannot hold, and export refuses with the reason rather than writing a map that lands 100 km away[cite: 3].

The projections reproduce `autoware_lanelet2_extension` / `lanelet2_projection` and agree with PROJ to well under a millimetre[cite: 3].

One thing no alignment can fix: if the file's lat/lon were produced with a *different* projection from the yaml, the map is slightly scaled relative to the cloud (a few decimetres over a few hundred metres)[cite: 3]. Translation and yaw cannot remove a scale error[cite: 3]. If the file also has correct `local_x`/`local_y`, generate a projector instead of trusting the old lat/lon[cite: 3].

The browser console (F12) logs a `[LaneForge]` line for each load and export: the frame used, the projector, and a sample node before and after[cite: 3].

---

## Routing

### Health

Every lanelet is drawn in its own colour[cite: 3]:

| colour | meaning |
| --- | --- |
| green | has both a next and a previous[cite: 3] |
| yellow | has one side only[cite: 3] |
| red | isolated[cite: 3] |

Click a band in the panel to isolate just those lanelets in the view[cite: 3]. Small chevrons run along each lanelet in its direction of travel, faint on everything once you zoom in and bright on the selection, so a backwards lanelet stands out against the flow around it[cite: 3].

### Repair candidates

A scan lists pairs whose joints are within the gate (capped at **0.50 m**, both boundary ends must qualify)[cite: 3]. Rows are sorted closest-first; clicking one flies to it, ticking it applies the link[cite: 3]. **Missing a link** filters to lanelets that actually lack a connection; **All in gate** shows every join in range[cite: 3].

### Reversed?

A third filter tries each lanelet reversed on its own and lists only those that **gain** links, ranked by how many[cite: 3]. It counts joins that would form outright plus gaps the reversal would bring inside the gate, so a lanelet that is both backwards *and* short of its neighbour is still found[cite: 3].

This matters because reversing on a hunch is destructive: a lanelet that already runs the right way loses its links when flipped[cite: 3]. The scan never proposes one of those, and reversing a lanelet that still has links asks for confirmation first[cite: 3].

### Links

Select a lanelet and its `next` and `previous` lists become editable[cite: 3]. Type an id, or leave the box empty and press **+ next** to arm pick-from-map, then click the lanelet you mean — a banner tells you what it is waiting for and `Esc` cancels[cite: 3]. Adding a successor sets the other lanelet's predecessor at the same time; a lanelet can hold as many of each as the junction needs[cite: 3].

Every lanelet id is offered, sorted nearest-first with its real joint distance[cite: 3]. Nothing is refused — the map may simply be wrong and need it — but a link past the gate quotes exactly how far the geometry will jump and how many boundary ways it will drag, and only applies on a second, deliberate press[cite: 3].

### Direction and handedness

These are two separate things, and confusing them is the most common source of trouble[cite: 3]:

- **⇄ Reverse** flips the direction of travel[cite: 3]. That swaps which end is the start *and* which boundary is on the left[cite: 3].
- **⇋ Swap L/R** only relabels the boundaries[cite: 3]. Geometry does not move[cite: 3].

When two lanelets already touch but their left/right labels disagree, the reported gap is exactly one lane width — a distance that is really a labelling error[cite: 3]. LaneForge detects this (the crossed pairing is closer than the straight one) and **relabels instead of moving anything**, telling you so[cite: 3].

### Shape

**◇ Edit shape** puts a handle on every boundary vertex of the selected lanelet[cite: 3]. Drag to bend the lane; the boundary, the lane width and the centreline recompute live[cite: 3]. Nodes shared with a neighbouring lanelet move for both — that is one node in the file, and it is what keeps the boundary continuous[cite: 3].

### Speed limits

Select one lanelet, shift-click to add more, or drag a box with the left button to take a whole zone, then apply one speed to all of them[cite: 3]. Entry is in km/h by default with an mph toggle; **the file is always written in km/h as a bare number**, which is how lanelet2 reads an untagged value[cite: 3]. Reading is more permissive: a bare number, `mph` or `m/s` are all understood[cite: 3].

### Create and delete

**✎ New lanelet** draws a centreline; `Enter` finishes, `Backspace` drops a point, `Esc` cancels[cite: 3]. The result is a real lanelet — nodes, two boundary ways and the relation — added straight into the model, so you can immediately link it, reverse it, reshape it or set its speed[cite: 3].

**🗑 Delete** removes a lanelet from the graph and drops its relation on export[cite: 3]. Its boundary ways are left in place, because a neighbouring lane usually shares them[cite: 3].

---

## How lanelet2 connectivity actually works

Worth knowing, because it explains everything the tool does[cite: 3].

**There is no `next` or `previous` field.** Lanelet B follows lanelet A when B's two boundaries *begin at the same node ids* that A's boundaries *end at*[cite: 3]. Connectivity is node identity, not proximity[cite: 3].

Three consequences[cite: 3]:

- **A link is a merge.** Joining two lanelets means making one junction out of two, which moves the follower's start onto the leader's end[cite: 3]. Within a few centimetres that is a harmless snap[cite: 3]. Across metres it teleports geometry[cite: 3].
- **Adjacent lanes share boundary ways.** One node is commonly the endpoint of three to five boundary ways, so a merge drags every line that rides on it — including lanelets that were not part of the link[cite: 3]. This is correct for a small snap and destructive for a large one[cite: 3].
- **A single edge often cannot be removed.** Once several lanelets share one junction they are all each other's neighbours[cite: 3]. Removing one edge means taking the junction apart, so the tool removes the merge responsible and says which[cite: 3].

LaneForge watches for the failure mode this creates: if a lane's two boundaries are ever pulled onto the same point, a warning appears naming the affected lanelets[cite: 3]. It does not block you — it flashes, and `Ctrl+Z` or **Undo all** puts the geometry back exactly[cite: 3].

---

## Export

**Export corrected .osm** writes a new file next to your original[cite: 3]. Everything not edited passes through byte-for-byte[cite: 3].

- **Geometry.** Both `lat`/`lon` and the `local_x`/`local_y` tags are rewritten together, so whichever one your pipeline reads they stay in agreement[cite: 3]. With a projector (loaded or generated) lat/lon are computed through it; see **Autoware frame**[cite: 3].
- **Without a projector.** Node lat/lon receive the *delta* rather than being recomputed, so an untouched node keeps its original value exactly[cite: 3]. Metres-per-degree is solved from the file itself by least squares over its own `lat`/`lon` ↔ `local_x`/`local_y` pairs, and the residual is reported[cite: 3]. Files without local coordinates are projected from lat/lon about an origin that is written into the export as a comment, so re-opening it keeps the alignment[cite: 3].
- **Routing.** Links rewrite the relevant boundary way endpoints[cite: 3]. Reversals turn the way node order and swap the member roles; where a boundary is shared with a lanelet that is *not* reversed, the reversed lanelet gets its own copy so the neighbour is untouched[cite: 3].
- **`next` / `previous` tags** can optionally be written on each relation[cite: 3]. They are informational — lanelet2 derives routing from shared nodes, not from tags[cite: 3].

Every edit is stored as intent and replayed from the untouched file, so undo is exact and the export always derives from the original rather than from an accumulated state[cite: 3].

---

## Keyboard and mouse

| | |
| --- | --- |
| left drag | move the map (Align) · box-select (Routing)[cite: 3] |
| left click | select a lanelet[cite: 3] |
| middle / right / `Space` / `Alt` drag | pan the view[cite: 3] |
| wheel | zoom about the cursor[cite: 3] |
| shift-click | add or remove from the selection[cite: 3] |
| arrows | nudge the offset (Align only)[cite: 3] |
| `Shift` + arrows | nudge ×10[cite: 3] |
| `Ctrl+Z` / `Ctrl+Shift+Z` / `Ctrl+Y` | undo / redo (250 steps)[cite: 3] |
| `Esc` | cancel a pick, a draft or the selection[cite: 3] |
| `Enter` / `Backspace` | finish / un-do a point while drawing[cite: 3] |

Selection is by containment: click anywhere inside a lane and you get that lane, at any zoom[cite: 3]. Lanes do not overlap, so neighbouring carriageway lanes stay separable[cite: 3].

---

## Notes and limits

- Undo covers links, cuts, reversals, swaps, vertex moves, speeds, deletions, creations and the alignment offset[cite: 3]. Creating and deleting are each other's inverse[cite: 3].
- Closing or reloading with unsaved edits prompts for confirmation[cite: 3].
- Light and dark themes; the choice is remembered and follows your OS setting on first run[cite: 3].
- Elevation is not edited[cite: 3]. `ele` tags pass through untouched[cite: 3].
- Lane-change (adjacency) relations are not edited[cite: 3]. Reversing a lanelet whose boundary is shared gives it a private copy of that way, which does end the sharing for that pair[cite: 3].
- A newly drawn lanelet starts isolated, and its ends are unlikely to fall inside the 0.50 m gate[cite: 3]. Draw it so its ends land on the lanelets it should join, or drag the end vertices onto them with **Edit shape** first, then link with no movement[cite: 3].
- Very large point clouds are limited by browser memory[cite: 3]. A 230 MB / 19 M point cloud loads comfortably on a desktop browser[cite: 3].

---

## Converting to OpenDRIVE (.xodr) for CARLA

LaneForge exports corrected `.osm` files in the Lanelet2 format. To use this map in simulators like CARLA, it must be converted to the OpenDRIVE (`.xodr`) standard using Tier IV's official `autoware_lanelet2_to_opendrive` tool. 

Vanilla JavaScript running inside the browser cannot execute the heavy C++ and Python dependencies required to generate complex parametric polynomials (`ParamPoly3`) for the curves. You must run the conversion locally alongside your browser.

### Option 1: Using Docker (Recommended)
The converter relies on complex system-level dependencies. Building it natively often leads to frustrating static library link conflicts (such as `ld` linker errors with system-level Python archives). Using Docker provides a clean, isolated container that guarantees the tool runs perfectly without polluting your host system.

#### 1. Install Docker
Use the official convenience script to install Docker and grant your user permissions.
```bash
curl -fsSL [https://get.docker.com](https://get.docker.com) -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker

```

#### 2. Install the Converter

Clone the Tier IV repository and build the container.

```bash
git clone [https://github.com/tier4/autoware_lanelet2_to_opendrive.git](https://github.com/tier4/autoware_lanelet2_to_opendrive.git)
cd autoware_lanelet2_to_opendrive
docker compose --profile convert build convert

```

#### 3. Convert the Map

Use `map.projector_type=Local` to force the converter to strictly trust the `local_x` and `local_y` tags written by LaneForge, preventing MGRS grid distortions.

```bash
docker run --rm \
  -v "$PWD:/io" \
  -v "$HOME/laneforge:/maps" \
  l2o-convert:local \
  map=example \
  map.projector_type=Local \
  target=carla \
  input_map_path=/maps/map_corrected.osm \
  output_map_path=/maps/map_carla.xodr

```

#### 4. Reclaim File Ownership

Because Docker runs as the root user, the new files will be locked. Run this to take back ownership:

```bash
sudo chown $USER:$USER ~/laneforge/map_carla.xodr ~/laneforge/map_carla.mapping.json

```

---

### Option 2: Automated Local Watcher Script (No Docker)

If you prefer to avoid Docker, you can configure your native Python environment to continuously watch your folder and automatically generate the OpenDRIVE file the exact second you hit "Export" in LaneForge.

#### 1. Fix the Local Library Conflict

Hide the static archive so CMake links to the correct shared libraries during the build.

```bash
sudo apt update
sudo apt install -y python3.10-dev libpython3.10
sudo mv /usr/local/lib/libpython3.10.a /usr/local/lib/libpython3.10.a.bak

```

#### 2. Install the Converter using `uv`

```bash
git clone [https://github.com/tier4/autoware_lanelet2_to_opendrive.git](https://github.com/tier4/autoware_lanelet2_to_opendrive.git)
cd autoware_lanelet2_to_opendrive
uv cache clean
uv sync --dev

```

#### 3. Create the Watcher Script

Save the following code as `watch_and_convert.py` in your `~/laneforge/` directory.

```python
import os
import time
import subprocess

WATCH_DIR = os.path.expanduser("~/laneforge")
CONVERTER_DIR = os.path.expanduser("~/laneforge/autoware_lanelet2_to_opendrive")
processed = set()

print(f"Watching {WATCH_DIR} for new _corrected.osm files...")

while True:
    for file in os.listdir(WATCH_DIR):
        if file.endswith("_corrected.osm") and file not in processed:
            osm_path = os.path.join(WATCH_DIR, file)
            xodr_path = osm_path.replace(".osm", ".xodr")
            
            print(f"\nNew map detected: {file}")
            print("Converting to OpenDRIVE...")
            
            cmd = [
                "uv", "run", "python", "-m", "autoware_lanelet2_to_opendrive.main",
                f"input_map_path={osm_path}",
                f"output_map_path={xodr_path}",
                "map=example",
                "map.projector_type=Local",
                "target=carla"
            ]
            
            subprocess.run(cmd, cwd=CONVERTER_DIR)
            processed.add(file)
            print(f"Done! Saved as {xodr_path}")
            
    time.sleep(2)

```

#### 4. Run the Watcher

Run the script in the background while you use LaneForge in your browser. Whenever you click "Export corrected .osm", the script will detect the download and instantly build your `.xodr` file next to it.

```bash
python3 ~/laneforge/watch_and_convert.py

```

---

## Licence

MIT — see [LICENSE](LICENSE).

```

```
