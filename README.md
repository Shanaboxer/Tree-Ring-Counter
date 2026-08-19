# Dendro Bench

A browser tool for counting tree rings. You draw a line across a photograph of a cross-section, it marks every ring boundary the line crosses, and **you correct it by hand** where it gets things wrong.

No install, no server, no upload. One HTML file.

**[Open the tool →](https://shanaboxer.github.io/Tree-Ring-Counter/ring-counter.html)**

---

## What problem this solves

Most ring-counting scripts count peaks in a brightness profile along a radius. That fails on real discs for three reasons:

- Rings tighten toward the bark, so any fixed peak-spacing threshold is wrong at one end of the radius or the other.
- Outer rings are fainter, so a fixed contrast threshold silently drops them.
- Cracks, rays and the pith add peaks that aren't rings.

This tool flattens the profile against a *local* baseline and *local* spread, so a faint 3 px ring near the bark is judged on the same scale as a bold 12 px ring near the pith. Every local minimum then becomes a candidate boundary scored by prominence, and one slider sets the cut-off.

It will still be wrong sometimes. That is why the marks are editable.

---

## Getting started

**Use it online** — open the link above. Nothing to install.

**Or run it from your own machine** — go to **Code → Download ZIP**, unzip, and double-click `ring-counter-standalone.html`. It works with no internet connection.

> Don't click the **Raw** button and expect a working tool. GitHub serves raw HTML as plain text, so you will see the source code instead of the page. Download the file first.

A sample cross-section is built in, so you can try everything before loading your own photo.

---

## How to use it

### 1. Load a photo

Drag an image onto the specimen panel, or press **Load image**. Any format your browser can open.

### 2. Draw the measuring line

Drag from the pith to just inside the bark. Only what the line touches is measured. A magnifier follows the cursor so you can land on the pith precisely.

The line doesn't have to be a full radius — measure any stretch you like, and count sections separately if the growth rate changes sharply along the radius.

### 3. Zoom in and check the marks

- **Scroll** to zoom at the cursor
- **Shift-drag** or **middle-drag** to pan
- **Double-click**, or the **Fit** button, to reset

Above 250% the image stops being smoothed so you see real pixels. This is the point of the whole exercise: put your eye on the marks and see whether they sit on ring boundaries.

### 4. Add and remove rings by hand

This is the part that matters. The software proposes, you decide.

- **Click a mark to delete it.** Useful for false rings — an intra-annual density fluctuation laid down in a dry spell looks like a boundary but isn't a year.
- **Click bare wood on the line to add a mark.** It snaps to the nearest dark boundary within reach, so you don't have to hit the pixel exactly.

Zoom in first for fiddly regions — the click tolerance scales with zoom, so at 800% you can place marks pixel by pixel.

The count updates immediately. **Your corrections** in the readout tracks what you changed (`+3 / −1`), your edits are carried into both exports, and the saved image is stamped *hand-corrected* so nobody mistakes it for a raw machine count.

Edits survive changing any setting. They are only lost if you redraw the line, or press **Reset marks** to return to the automatic result.

### 5. Save the result

- **Save marked image** — the original photo with every mark drawn on it, at original resolution or 2×/3× for legible ring numbers. Marks are drawn as brackets with a gap in the middle, so the ring being marked stays visible and the count can be audited later. A caption records the count, the settings, and whether you corrected it.
- **Download CSV** — one row per ring: number, distance along the line, ring width, and image coordinates.

---

## Controls

| Control | What it does |
| --- | --- |
| **Sensitivity** | The prominence cut-off. Lower finds fainter boundaries and more false positives. Start at 0.45 and move it while watching the marks. |
| **Narrowest ring to look for** | Minimum spacing between marks, in pixels. This is a hard floor — set it below your tightest rings or they cannot be found. |
| **Widest ring to look for** | Upper bound for the ring-width estimator. Only needs raising for very wide-ringed specimens. |
| **Cross-check fan** | Repeats the measurement on 9 nearby angles. The spread across them is your uncertainty. Set to 0 to switch off. |
| **Contrast channel** | Which colour information to measure. *Wood* is blue-weighted and best for conifers, where latewood is dense and orange. Try the others on stained, weathered or hardwood samples. |

---

## The limit no setting can move

**A ring needs to be about three pixels wide in your source photograph before its boundary survives as a dip in the signal at all.** Below roughly two pixels per ring, the camera and the JPEG encoder have already averaged the boundaries away, and there is nothing left in the file to find.

The test is simple: zoom to 800% and look. If a band reads as smooth wood, no sensitivity setting will recover it. The fix is a bigger photograph of that band, ideally of a sanded face.

This bites hardest on the outer rings of old, slow-grown trees — exactly where the years pile up. A 500 px photo of a stump with 110 rings cannot be counted by any software, including this one.

**Practical guidance:** photograph a sanded surface in flat, even light, fill the frame, and shoot as large as your camera allows. If the outer band is very tight, take a second close-up of just that band and measure it separately.

---

## How accurate is it?

Tested against a hand count on a weathered stump, split into three segments:

| | Hand count | This tool | Pixels per ring |
| --- | --- | --- | --- |
| Segment 1 | 18 | **18** | 6.9 |
| Segment 2 | 23 | **23** | 5.2 |
| Segment 3 | 69 | 26 | 2.2 |

Exact agreement where the rings are resolved. Segment 3 fails, and it fails for the reason above — at 2.2 px per ring the boundaries are not in the image. That segment reads as blank wood at 8× magnification.

Treat the automatic count as a first pass, always check it at zoom, and expect to correct the innermost few years by eye: the first two or three rings sit in a handful of pixels around the pith, where nothing can separate them.

---

## How it works

1. The path is resampled with cubic interpolation at quarter-pixel spacing. Linear interpolation acts as a low-pass filter and flattens the shallow minima of narrow rings.
2. A Morlet wavelet ridge estimates how wide the rings are locally. This sets the baseline window — it does **not** do the counting. A smooth rhythm cannot represent a narrow ring wedged between wide ones, so counting its cycles silently drops rings.
3. The signal is flattened against a Gaussian local baseline and local spread. Gaussian, not boxcar: a boxcar filter rings at the ring frequency and fills in exactly the dips being hunted.
4. Every local minimum becomes a candidate, scored by prominence — how far the signal must climb to escape it on both sides. The sensitivity slider is the cut-off on that score.
5. Marks closer together than the minimum spacing are resolved in favour of the stronger one.

---

## Privacy

Your photographs never leave your device. The tool reads files with `FileReader`, works on the pixels in memory, and produces downloads with `Blob`. There is no server, no upload, no analytics, no storage. You can verify this by disconnecting from the internet — everything still works.

---

## Building and hosting

Static files only. GitHub Pages, Cloudflare Pages and Netlify all host this for free with HTTPS.

Two build targets from the same source:

- `index.html` + `sample.jpg` — for hosting. Smaller HTML, image cached separately.
- `ring-counter-standalone.html` — sample image base64-embedded, for downloading and running offline.

**Both are necessary.** Over `file://`, browsers treat every local file as its own origin, so a downloaded page loading a sibling image taints the canvas and `getImageData` throws a `SecurityError`. The sample must be embedded in the standalone build. Uploaded images are unaffected — `FileReader` returns a `data:` URL, which does not taint the canvas.

---

## Known limitations

- **False rings cannot be told from true rings by any photograph.** A sharp minimum may be an intra-annual density fluctuation. Delete those by hand where you can identify them.
- **The innermost years are unresolvable.** Count them by eye.
- **Cracks and rays running along the measuring line will suppress the count.** Move the line to a clean radius.
- **Very large photographs may exhaust memory on mobile.** iOS Safari caps total canvas area at roughly 16.7 megapixels.
- **One line at a time.** No cross-dating, and no multi-radius averaging beyond the cross-check fan.

This is a measuring aid, not a dendrochronology lab. For research-grade work, use a sanded core, a stereo microscope, and cross-dating against a reference chronology.

---

## Licence

MIT — see [LICENSE](LICENSE).

<!-- Pick a licence before publishing. Without a LICENSE file the default is
     "all rights reserved": people may read the code but have no permission to
     use or modify it. Delete this comment once you have decided. -->
