# GelSVG sample files

Documents for GelSVG `schema_version` 1, `schema_revision` 0 (the draft of
2026-10-27), each with what a reader that follows the specification must make
of it. They are for checking an implementation against the specification:
read each file, and compare what your reader reports with `expect` in
`cases.json`.

## The files

- `files/valid/`: documents with nothing wrong. Three single panels saved by
  MyWB 0.7.0 (`Flag`, `HA`, `V5`: three regions of one synthetic membrane,
  each with its source pixels and metadata embedded), the `HA` panel saved
  without its source pixels, the `HA` panel's content on a small part of its
  source images (`compact-HA`, the base of the variants that need embedded
  pixels, to keep them small), that panel with a band quantification, and
  the three panels drawn together as one assembly (with and without source
  pixels).
- `files/container/`, `png/`, `json/`, `core/`, `figure/`, `assembly/`: each
  file is a valid file with one thing changed. `cases.json` says which file
  it was made from and what was changed.
- `files/drawing/`: valid single panels, made from a part of the same
  membrane, that each use some of the drawing rules (§4, §6.2, §7.2): turns
  and mirrors, a fine rotation, several channels in colour, size matching
  both ways, 16-bit sources, the average of RGB, contrasts at their edges, and
  a lane quantification. Drawing each rendered image again, and measuring
  each quantification again, must give exactly what the file holds.
- `files/drawing/near-halves-*`: a fine rotation and size matching by odd
  ratios (enlarging and shrinking), whose source pixels are set so that some
  values lie within about 1e-19 of a half. A drawing that does not follow
  the arithmetic of §4.3 exactly (terms added in another order, the kernel
  written another way, a maths library's cos and sin, the passes of size
  matching swapped or rounded between, a fused multiply-add, a reciprocal
  instead of a division) differs from these files in some pixels.
  `hard-variants.txt` counts the pixels each of these loose ways changes.

## What `expect` says

- `document`: whether the file is a GelSVG document at all (§1.1, §1.2).
- `panels`, in document order (§1.2):
  - `read`: `yes`; `unreadable` (§2.6, rule 5); or `newer_version` (§9, the
    content is not read).
  - `checks`: `rendered_hash` (§6.3), `drawn_from` (§6.4) and `svg_hash`
    (§6.5), each `matched`, `differs`, `unknown` (recorded but cannot be
    checked) or `absent` (not recorded). `svg_hash` applies only to a reader
    that checks it.
  - `channels`: for each channel of each source image (`"s1/gray"`), whether
    its channel image is `embedded` or `not_embedded` once read (§5.3).
  - `source_metadata`: for each source image that points to source
    metadata, whether it is `present` or `absent` once read (§5.4).
  - `quantifications`: `current`, `stale` or `unknown` (whether it is stale
    cannot be checked), one per quantification kept (§7.4).
  - `redraw`: drawing the rendered image again from the embedded channel
    images (§6.2, with the reference interpolation of §4.3) gives exactly the
    carrier PNG's pixels (`identical`), or does not (`differs`).
  - `remeasure`: measuring each value's `region` again (§7.2, §7.3) gives
    exactly the recorded `raw`.
  - `reports`: what the reader reports besides check results, by the kinds
    of §2.6 (`broken`, `damaged`, `unknown`, `reinterpreted`, `unreadable`,
    `declined`, `assemblies`). A reader must make every report whose
    `required` is true, may make those whose `required` is false (the
    specification allows the same event to be told both ways), and should
    make no report of a kind not listed. `about` says in words what each
    report concerns; how a reader words it is its own business.
- `assemblies`: each assembly read, with its `members`, the panels found (by
  index) and the `member` numbers missing (§1.7).
- `reports`: reports about the document as a whole (not a document,
  assemblies).

## Licence

The sample files, `cases.json` and `hard-variants.txt` are dedicated to the
public domain under CC0 1.0 (see `LICENSE`). They may be copied into
implementations and their tests without attribution.

## Cases

| File | What was changed | Sections |
|---|---|---|
| `valid/single-Flag.gel.svg` | MyWB 0.7.0 saved the 'Flag' document: one panel with its source pixels and metadata embedded. | 1, 3, 5, 6 |
| `valid/single-HA.gel.svg` | MyWB 0.7.0 saved the 'HA' document: one panel with its source pixels and metadata embedded. | 1, 3, 5, 6 |
| `valid/single-V5.gel.svg` | MyWB 0.7.0 saved the 'V5' document: one panel with its source pixels and metadata embedded. | 1, 3, 5, 6 |
| `valid/single-HA-light.gel.svg` | MyWB 0.7.0 saved the 'HA' document without its source pixels: nothing is embedded. | 1.4, 5.2, 5.3 |
| `valid/compact-HA.gel.svg` | The HA panel's content on a 250 x 600 part of its source images, saved as PNG files with a text chunk and read as source files, written and drawn by the sample builder: the small base of the variants that need embedded pixels. | 1, 5, 6 |
| `valid/quantified-HA.gel.svg` | A band quantification of the HA row was added: the inverted aligned image (fine rotation -1.03°, reference interpolation) summed over each band's box. Written again by the sample builder, which kept the picture and did not draw the SVG, so the panel has no svg_hash. | 4.3, 7 |
| `valid/assembly-Flag-HA-V5.gel.svg` | The three panels drawn together as one assembly (one &lt;g>), lanes lined up and one samples table; the SVG's hash recorded in all three. | 1.7, 6.5 |
| `valid/assembly-light.gel.svg` | The same assembly, its panels without source pixels. | 1.7 |
| `container/utf-16.gel.svg` | The whole SVG re-encoded as UTF-16 (with a byte order mark). | 1.1 |
| `container/gzip.gel.svgz` | The SVG compressed with gzip (.svgz). | 1.1 |
| `container/doctype-entity.gel.svg` | A document type declaration whose internal subset declares an entity (never used). | 1.1 |
| `container/doctype-external.gel.svg` | A document type declaration that only points to an external DTD. Read as if absent (the DTD is not fetched); canonical XML leaves it out, so svg_hash still matches. | 1.1, 6.5 |
| `container/xlink-href.gel.svg` | The carrier PNG moved to xlink:href, its data URI written as 'data:image/PNG ; name=panel.png ; base64,' with the base64 broken into lines. Still the same panel; the SVG differs, so svg_hash differs. | 1.2, 6.5 |
| `container/defs-use.gel.svg` | The carrier PNG's image moved into &lt;defs> and drawn with &lt;use>. Still one panel; svg_hash differs. | 1.2, 1.6 |
| `png/itxt-crc.gel.svg` | The CRC of the iTXt holding the panel JSON no longer matches. | 1.3, 2.6 |
| `png/idat-crc.gel.svg` | The CRC of the first IDAT no longer matches: only the rendered image is unusable. | 1.3, 6.3, 6.5 |
| `png/gels-crc.gel.svg` | The CRC of the geLs holding the blot's channel image no longer matches; its payload is intact, so the header's SHA-256 recovers it. | 1.4 |
| `png/gels-payload.gel.svg` | One byte of the blot's channel image payload changed (the chunk's CRC left as it was): the chunk cannot be recovered and the blot's channel is not embedded. | 1.4, 5.3 |
| `png/truncated.gel.svg` | The carrier PNG cut short inside its last geLs (the source metadata), so IEND is missing too. | 1.3, 1.4, 5.4, 6.5 |
| `png/two-itxt.gel.svg` | The iTXt holding the panel JSON appears twice. | 1.3, 2.6 |
| `png/gels-codec-3-revision-0.gel.svg` | An extra geLs with codec 3, which this version does not define, in a panel of revision 0. | 1.4, 2.6, 9 |
| `png/gels-codec-3-revision-1.gel.svg` | An extra geLs with codec 3, which this version does not define, in a panel of revision 1. | 1.4, 2.6, 9 |
| `png/gels-type-damaged.gel.svg` | One bit of the type of the geLs holding the blot's channel image changed, turning 'geLs' into 'GeLs': a critical chunk whose CRC does not match. The rendered image is unusable, and the blot's channel image, not found, is reported as damage, not as a writer's error. | 1.3, 5.3 |
| `png/bad-idat-beside-damaged-gels.gel.svg` | The image data's zlib stream spoilt with its CRC made to match (an error made in writing), and, apart from it, a geLs whose CRC no longer matches (recovered). The damaged geLs did not stop the walk, so it does not explain the image data: the panel is unreadable. | 1.3, 1.4, 2.6 rule 5 |
| `json/repeated-key.gel.svg` | The key 'note' appears twice in 'core'. | 2.1, 2.6 |
| `json/nan.gel.svg` | orientation.rotation written as NaN. | 2.1, 2.6 |
| `json/null-optional.gel.svg` | core.annotation written as null, and the writer's hashes taken over the core as written (with the null). | 2.1, 2.6, 2.7 |
| `json/empty-optional.gel.svg` | core.note written as an empty string; hashes taken over the core as written. | 2.4, 2.6, 2.7 |
| `json/integer-as-decimal.gel.svg` | orientation.turn written as 0.0 (not an integer). drawn_from is left as it was: canonical JSON writes 0.0 as 0, so the hash of the core does not change. | 2.2, 2.6 rule 4, 2.7 |
| `json/crlf.gel.svg` | The named row's name holds CR LF; hashes taken over the name as written. | 2.5, 2.7 |
| `json/unknown-key-revision-0.gel.svg` | A key this version does not define (core.sources[0].wavelength_nm) in a panel of revision 0; hashes taken over the core as written (with the key). | 2.6 rule 1, 2.7, 9 |
| `json/unknown-key-revision-1.gel.svg` | A key this version does not define (core.sources[0].wavelength_nm) in a panel of revision 1; hashes taken over the core as written (with the key). | 2.6 rule 1, 2.7, 9 |
| `json/newer-version.gel.svg` | schema_version 2. | 9 |
| `json/no-revision.gel.svg` | schema_revision left out: read, and taken as a newer revision. | 3, 9 |
| `json/closed-object-unknown-key.gel.svg` | An unknown key in the catalog entry of the blot's channel image, a closed object: the entry is broken, so the blot's channel is not embedded. Its geLs is used by the entry as written, so it is not reported again (§5.3). | 2.6 rule 1, 5.3 |
| `json/channel-image-kind-revision-1.gel.svg` | The catalog entry of the blot's channel image has a kind this version does not define, in a panel of revision 1: the entry is not used, and its geLs is still used by it as written, so the geLs is not reported as unused. | 2.6 rule 2, 5.3 |
| `core/band-unknown-lane.gel.svg` | The band in lane 1, row r2 refers to lane 'zz', which does not exist. | 2.6 rule 3, 5.13 |
| `core/band-same-cell.gel.svg` | A second band in the cell of lane 3, row r2, written after the first. | 2.6 rule 3, 5.13 |
| `core/lane-number-repeated.gel.svg` | Lane '2' has number 1, as lane '1' (earlier in the array) does. | 2.6 rule 3, 5.11 |
| `core/contrast-reversed.gel.svg` | The reference channel's contrast written max first. | 4.5, 5.6, 2.6 rule 4 |
| `core/band-shape-revision-0.gel.svg` | The band in lane 4, row r2 has shape 'ellipse', in a panel of revision 0. | 2.6 rules 1-3, 5.13 |
| `core/band-shape-revision-1.gel.svg` | The band in lane 4, row r2 has shape 'ellipse', in a panel of revision 1. | 2.6 rules 1-3, 5.13 |
| `core/free-form-metadata.gel.svg` | The blot's source image given source metadata in a free-form format (an https:// URL this reader does not know). It is kept, its hashes checked, and nothing is reported. | 5.4 |
| `figure/svg-text-edited.gel.svg` | The row name drawn in the SVG changed from 'HA' to 'HA-tag' in an editor. | 6.5 |
| `figure/rendered-repainted.gel.svg` | A patch of the rendered image brightened and the PNG written again around the same JSON and geLs. | 6.3, 6.5, 6.6 |
| `figure/core-edited.gel.svg` | The named row renamed in the JSON only (a script), the figure not drawn again. Names are left out of the quantification's hash, so the quantification stays current. | 2.7, 6.4, 7.4 |
| `figure/quantification-stale.gel.svg` | The band in lane 1, row r2 moved 4 pixels right in the JSON only: the figure and the quantification are both out of date. | 6.4, 7.4 |
| `figure/computed-on-sha512-revision-0.gel.svg` | The quantification's computed_on written as a 'sha512:' hash, a kind this version does not define, in a panel of revision 0. | 2.2, 2.6 rules 1-3, 7.4 |
| `figure/computed-on-sha512-revision-1.gel.svg` | The quantification's computed_on written as a 'sha512:' hash, a kind this version does not define, in a panel of revision 1. | 2.2, 2.6 rules 1-3, 7.4 |
| `figure/drawn-from-sha512-revision-0.gel.svg` | figure.drawn_from written as a 'sha512:' hash in a panel of revision 0. | 2.2, 6.1, 6.4 |
| `figure/form-written-wrong.gel.svg` | The only blot element given a colour, so the core asks for an RGB picture, while the picture stays grey; hashes taken over the core as written, so rendered_hash and drawn_from both match. | 6.2 |
| `figure/form-redrawn.gel.svg` | The grey picture replaced by the same picture as RGB after it was drawn: its form no longer matches the core, and rendered_hash differs, so it is a check result, not a report. | 6.2, 6.3 |
| `figure/size-unchecked.gel.svg` | crop one pixel narrower than the picture, and rendered_hash a 'sha512:' hash (revision 0), so nothing shows the picture was changed. | 2.2, 6.2 |
| `figure/channel-image-replaced.gel.svg` | The blot's channel image replaced by one with a patch blacked out, its catalog entry and geLs made consistent, but its pixel_hash left as it was. | 5.3, 6.6 |
| `figure/svg-hash-method-unknown.gel.svg` | figure.svg_hash_method has a value this version does not define, in a panel of revision 0. | 2.6 rule 1, 6.1, 6.5 |
| `assembly/copies.gel.svg` | The assembly's &lt;g> copied beside itself: two copies, each in its own &lt;g>. | 1.7 |
| `assembly/copies-ungrouped.gel.svg` | The same two copies with both assembly &lt;g> elements ungrouped (each panel keeps its own &lt;g>): the copies cannot be told apart. | 1.7 |
| `assembly/members-disagree.gel.svg` | Member 2 says the assembly has 4 members; the others say 3. | 1.7 |
| `assembly/missing-member.gel.svg` | Member 3's group deleted from the figure. | 1.7 |
| `assembly/newer-version-member.gel.svg` | Member 2 is a panel of schema_version 2. | 1.7, 9 |
| `drawing/turn-flip.gel.svg` | Turned 90° and mirrored (stage 2, multiples of 90° only). | 4.2, 4.3, 6.2 |
| `drawing/fine-rotation-invert.gel.svg` | A fine rotation of 12.5°, drawn inverted: the corners the rotation leaves empty are 0, black, before inversion, and so white in the picture. | 4.3, 4.5, 6.2 |
| `drawing/two-channels.gel.svg` | Two channels drawn in colour, whose sums pass 1 in green and are clamped. The second, half the size and stored upside down, is turned 180°, mirrored and matched to the reference's size (enlarging), and then both turn by -2.5°. | 4.2, 4.3, 6.2 |
| `drawing/shrink-to-match.gel.svg` | The second channel, twice the size, is shrunk to the reference's size: the weight function widens as it shrinks. | 4.3, 6.2 |
| `drawing/uint16-out16.gel.svg` | A 16-bit source drawn as a 16-bit picture. | 5.10, 6.2 |
| `drawing/uint16-out8.gel.svg` | The same 16-bit source drawn as an 8-bit picture (rounding to 8 bits, ties to even). | 5.10, 6.2 |
| `drawing/average.gel.svg` | An RGB source drawn by the average of its three channels, whose image is not embedded; the element records its pixel_hash. | 4.4, 5.6, 6.2 |
| `drawing/contrast-equal.gel.svg` | A contrast whose minimum and maximum are equal: a two-level picture. | 4.5, 6.2 |
| `drawing/contrast-outside.gel.svg` | A contrast that reaches outside 0-255, applied as it is. | 4.5, 5.6, 6.2 |
| `drawing/quantified-lanes.gel.svg` | A lane quantification of a panel turned by -3.2°, measured inverted. The first and last regions reach outside the aligned image, whose parts there are not measured, and into the corners the rotation leaves empty, which count as the maximum (most of the first lane's raw). | 4.3, 7.2, 7.3 |
| `drawing/near-halves-rotation.gel.svg` | A fine rotation by 7.001°, an angle whose cos or sin a maths library rounds differently from §4.3, of a source with 24 output pixels set within about 1e-19 of a half. A lane quantification over the whole picture sums them. | 4.3, 6.2, 7.2 |
| `drawing/near-halves-enlarge.gel.svg` | Size matching by an odd ratio (53 x 77 to 84 x 120, enlarging), with 24 output pixels of the second channel set within about 1e-19 of a half. | 4.3, 6.2 |
| `drawing/near-halves-shrink.gel.svg` | Size matching by an odd ratio (131 x 187 to 84 x 120, shrinking), with 24 output pixels of the second channel set within about 1e-19 of a half. | 4.3, 6.2 |
