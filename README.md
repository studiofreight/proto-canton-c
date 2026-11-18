# Canton C: scroll piece, ending at the pulled-back view

depth_k_follow_packet_v9_max_fast_round_smooth_aa_pullback_end.html is the approved scroll piece
(depth_k_follow_packet_v9_max_fast_round_smooth_aa.html) with one change:

- Scrolling down plays exactly as approved: the C, the line, the grid wall, the camera riding along a packet of light,
  then the camera pulling back to the wide view. The piece ends there.
- Scrolling back up from the pulled-back view no longer drops the camera back into the packet ride: it goes straight
  back to the wall, then the line and the C.
- Once you've scrolled back up to the line, the next scroll down plays the full sequence again, packet ride included.

Everything else (scroll length and speed, blocks, tubes, connectors, light, focus blur, edge smoothing) is unchanged.

## What changed from the approved page

In `draw()`, a flag `pulledBack` turns on once the pull back is past halfway (`zp > 0.5`) and turns off when the wall
breaks up again (`fieldTo < 0.5`). While it's on, the packet ride's target (`chaseTo`) is 0, and the ride eases out.

## Files

| File | What it is |
| --- | --- |
| depth_k_follow_packet_v9_max_fast_round_smooth_aa_pullback_end.html | The page (three.js 0.170.0, loaded from the jsDelivr CDN) |
| c_web.bin | The blocks, the tubes and all the motion |
| c_block.webp, c_tube.webp | The block and tube materials |
| c_piece_l0.bin to c_piece_l4.bin | The grid wall's tubes and connectors, in five levels of detail |

The page loads c_web.bin from the address in its `data-src` attribute (`<div class="c3d" data-src="c_web.bin">`),
and every other file from the same folder as that address. Keep the files together.

## Running locally

The page loads its files with fetch, so it needs a web server (opening the file directly won't work):

    python -m http.server 8000

then http://localhost:8000/depth_k_follow_packet_v9_max_fast_round_smooth_aa_pullback_end.html

## Before embedding in Webflow

- The intro text (`.intro`), the options panel (`.panel`) and the status text (`.hud`) are test scaffolding and can be
  removed; the settings they hold are fixed in the page's `PRESETS` / `CFG` objects.
- To stop the scroll bounce at the top and bottom of the page, the whole page scrolls inside a full-window box
  (`#scroller`, with `html, body { overflow: hidden }`) instead of the browser window. On a Webflow page the window
  should scroll as usual: remove `#scroller` and that `html, body` rule, and in `progress()` replace the `#scroller`
  height with `innerHeight`. Whether to keep the no-bounce behaviour (`overscroll-behavior: none` on `html, body`) is
  a site-wide choice.
- The scroll length is `CFG.pace.track` (225vh) on the `.c3d-track` section.
- On phones and tablets (`CFG.mobile`: `(max-width: 768px), (pointer: coarse)`) the pixel ratio is capped at 1.5
  (2 elsewhere).
- The focus blur and edge smoothing are full-screen passes after the scene is drawn (the `post` material).
- The page has not been profiled on laptops or phones.

## Performance changes

Same page as the approved one, with these changes:

- The focus blur's blur-amount pass (CoC) runs at half the resolution of the scene render. Blur sizes are scaled to match.
- On phones and tablets, 32 blur samples per pixel instead of 56. Desktop keeps 56, so the blurred areas stay as smooth as the original.
- The blur shaders are compiled while the page loads, not on the first frame of the C bending.
- On phones and tablets (`CFG.mobile`), the scene render used by the focus blur runs at 75% of the canvas. On desktop it stays at full resolution, so in-focus edges and the block texture are as sharp as the original.
- The page declares `<meta name="viewport" content="width=device-width, initial-scale=1">`. Without it a phone lays the page out 980 px wide, and the scene drew about 6.3 times more pixels (4.7 MP instead of 0.74 MP on a 390 px wide phone). A Webflow page already defines its own viewport, so this only affects the standalone page.

The pixel ratio caps (2, and 1.5 on phones) are unchanged.

GPU time per frame, 1440×900 at 2x on an M5 Pro (ms, lower is better):

| | C | Line | Grid wall | Pulled back |
|---|---|---|---|---|
| Before | 0.9 | 13.7 | 9.0 | 10.3 |
| After | 0.8 | 6.6 | 5.5 | 6.2 |

Same frame on desktop, before and after: the average pixel differs by 0.1 to 0.4 out of 255, and no pixel differs by more than 24. On phones (390×844, 75% scene render) the average difference is 0.2 to 0.6.

## Notes for the Webflow embed

- The ten text edits of the current Webflow loader still find their lines in this file, unchanged.
- The adaptive resolution patch in that loader caps the frame time at 0.1 s and only counts frames under 0.1 s, so on a device below 10 FPS it never steps down.
- The viewport tag only matters for this standalone page; a Webflow page already defines its own.
