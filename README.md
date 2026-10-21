# Arctic Splash

Kodi image resource add-on `resource.images.arctic.splash`, version 1.0.6.

## Weather cards

`resources/weather/` contains 49 full-bleed 320 x 720 portrait JPEG cards:
one distinct Japanese illustration for each Kodi weather fanart code `0` through
`47`, plus `na` for unavailable conditions. Scenes include Mount Fuji, cherry
blossoms, Tokyo streets, Kyoto temples, mountain villages, gardens and coasts.
The artwork depicts the code's precipitation, visibility, wind, cloud cover,
temperature or day/night condition in the same painted style as the splash.

Use unpadded code filenames, matching Kodi's standard fanart codes:

`special://home/addons/resource.images.arctic.splash/resources/weather/32.jpg`

This follows the direct add-on path already used by the layered splash and
avoids resource-handler loading failures observed for the cards and fallback.

Unavailable weather uses `weather/na.jpg`. `resources/weather/manifest.json`
records all code meanings, scenes, individual weather prompts and the common
style prompt. The mapping follows the
[official Kodi weather codes](https://kodi.wiki/view/Weather_addons#Weather_Codes).
The cards are generated individually using built-in imagegen with the Tokyo
alley illustration from this project as a style reference. They are opaque RGB
JPEGs at quality 95 with no chroma subsampling, borders or overlay labels.

`tools/weather-cards-preview.jpg` provides a labeled contact sheet. Run
`python tools/prepare_weather_cards.py` to validate coverage, dimensions and
distinct artwork and recreate the contact sheet (Pillow required). To prepare
new built-in imagegen outputs, pass `--sources sources.json`, containing a list
of objects with `code` and absolute original image `path` fields. Existing cards
are preserved; resizing uses proportional center fitting and Lanczos sampling.

## Watercolour weather cards

`resources/weather-watercolour/` contains a separate set of 49 full-bleed
320 x 720 portrait cards for the same codes `0` through `47` and `na`.
The original illustrated set remains in `resources/weather/` with its existing
filenames and artwork intact. Each style has its own manifest and contact sheet.

The watercolours use broad transparent washes, soft edges, paper texture and
simplified architecture and foliage, matching the approved Sunny / code 32
draft. The shared prompt and per-code scene and weather requirements are recorded
in `resources/weather-watercolour/manifest.json`. Every card is authored with
built-in imagegen; code 32 uses the approved draft itself.

The forecast cards use the watercolour set, including its `na.jpg` fallback:

`special://home/addons/resource.images.arctic.splash/resources/weather-watercolour/32.jpg`

`tools/weather-watercolour-preview.jpg` shows the full labeled set. Validate it
and regenerate the preview with
`python tools/prepare_weather_cards.py --style watercolour`.
Pass `--sources sources.json` to prepare newly generated originals. The
preparation tool defaults to the original illustrated set, so both styles
can be maintained separately without overwriting existing cards.

## Layered splash

The 616 registered 1920 x 1080 image layers are in `resources/layers/`. The
manifest defines four seasons, five weather patterns and 24 hourly states.
Composite each state's nine images in manifest order using normal alpha-over.
Scene lighting and reflections are authored in the image assets. The skin adds
animated weather on separate layers.

Large layers use quality 95 WebP with lossless alpha. Celestial, glow and
conditions overlays retain lossless PNG; sky plates retain JPG. Images total
approximately 101 MiB. `compile="false"` preserves these compressed files.

The skin declares this add-on as a required dependency and reads images from:

`special://home/addons/resource.images.arctic.splash/resources/layers/`

Kodi's image resource `resource://` handler only publishes PNG/JPG, including
in Kodi 21 and current source. Reading the add-on's files through `special://`
allows WebP decoding without expanding the artwork back to PNG. Install this
add-on normally in Kodi's user add-ons directory. No executable code is needed.

The skin selects season, hour and weather. Custom user image paths continue to
use the existing slideshow and do not use these layers.

Animated weather uses `resources/effects/rain.png`, a 240 x 540 transparent tile
drawn at 480 x 1080. The skin reuses it in four strips at different speeds with
two vertically adjacent copies per strip, following the holiday falling-texture
pattern. Rebuild it with `python tools/generate_weather_effects.py` (Pillow).

Snow weather reuses the rain cloud and lake plates with a separate animated
`resources/effects/snow.png` tile (300 x 560, about 14 KiB). The weather tile
reuses the existing winter snowflake sprite at varied sizes, angles and opacity,
with filled crystal branches and blurred alpha edges for soft, fluffy flakes.
Four strips fall over 16–22 seconds with independent sine sways and a small
vertical lilt; overscan keeps their edges covered. The same generator rebuilds
both precipitation tiles. Snow and mixed snow conditions select this effect in
every season, and the splash preview includes a snow option.

Thunderstorms use rain cloud/lake plates between flashes. The existing storm
clouds and lake plates appear together in a short double flash every ten seconds,
reusing the authored lightning bolts and matching reflections. A blue-white wash
reuses the skin's existing white texture to briefly light the scene. Static rain
conditions plates are replaced by the animated tile. No animation frames or
additional full-screen images are required.

All weather conditions reuse their hourly cloud plate in a slow,
continuous rightward loop behind the mountains (6.4 pixels per second). Original and horizontally
mirrored copies meet at matching edges; two identical pairs cover the full
3840-pixel loop so the restart has exactly the same image as the end. A faint,
vertically flipped and compressed copy moves in sync over the lake, below the
foreground trees. The shoreline stays fixed. The cloud
animation adds no image assets or animation frames. Foreground haze uses the
same loop at half the speed during hazy conditions.

Clear and hazy cloud plates use a repaired continuous cloud bank, with the old
mountain and foliage cutouts filled for scrolling. One generated transparent
daytime master supplies both weather sets and their eight existing hourly
palettes; hazy retains 40% cloud opacity. These replace the existing WebP plates
without adding runtime layers. The [repair prompt](tools/cloud-repair-prompt.md)
records the source and offline rebuild command.

Lake ripples reuse the existing hourly lake texture with two tiny 4 x 1080
diffuse masks, `resources/effects/lake-odd.png` and `lake-even.png`. The masks
select alternating horizontal bands with feathered joins, widening toward the
viewer and fading out near the distant bank. Two native skin slide animations
move these bands three pixels in opposite directions, beneath the trees and
time-of-day grade. The base lake and shoreline stay registered; no new lake
artwork or animation frames are needed. Rebuild the masks with
`python tools/generate_lake_masks.py` (Pillow).

Each season uses one small transparent sprite shape in
`resources/effects/foliage/`. Spring petals and autumn leaves were extracted from
the detached pieces in their foreground plates; those pieces are removed from
the static artwork. Summer has no detached pieces, so its sprite is clipped from
an attached green leaf while its foreground stays unchanged. Each shape has eight
tiny PNG lighting variants using the original scene's night and twilight grades;
there are no animation frames or additional full-screen textures. Winter uses
a soft, fluffy snow particle at 64 x 64 pixels, derived from the generated
snowflake with the same branch filling and alpha blur used by falling snow;
its eight lighting variants total approximately 31 KiB. Cleaning the
foreground and lighting plates offsets the sprites' storage cost.

The skin reuses each seasonal sprite in six particles with different sizes,
diagonal paths and timings. A smooth lilt periodically lifts each particle against
its gentle descent, while rotation and foreshortening add flutter. Particles fade
before their paths restart. They sit above the static lighting plate and use the
matching hourly sprite so moving pieces retain their correct lighting anywhere
on screen. Winter reuses the same motion for snowflakes. Rain, snow and thunderstorms
hide these seasonal particles. The preview's season selector updates them while
the window remains open.

Rebuild the sprites and cleaned plates with `python tools/extract_foliage.py`
(Pillow and Git). The script reads the recorded original Git revision, or accepts
`--source-dir` pointing to the original three foreground WebP files. It replaces
only the WebP alpha chunk in both the foreground and time-of-day plates,
preserving the compressed RGB data and verifying that every other decoded pixel
remains identical. Where a removed piece reveals terrain, the script repairs its
lighting alpha against the original scene's land grade.

Rebuild only the winter lighting variants with
`python tools/extract_foliage.py --snowflake-source tools/snowflake-source.png`.
Both snow generators read this preserved original so repeated rebuilds do not
progressively blur the runtime artwork.
The [snowflake prompt](tools/snowflake-prompt.md) records its image-generation source.
