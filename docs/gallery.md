# The HTML gallery

The gallery is a self-contained `gallery.html` page generated from the images in your output
folder. It lets you browse, inspect, compare and rate your generations in a browser, with no
server and no dependencies.

## Generating the gallery

```powershell
./New-GPTImage.ps1 -Gallery
```

This builds `gallery.html` in the images folder (`defaultOutputFolder`) and exits. It makes
no API calls and uses no model. Regenerate it whenever you want to include newly produced
images; the gallery version is shown in the regeneration log line.

## Browsing and viewing

The gallery presents your images as thumbnails and includes:

- a **lightbox** with mouse-wheel **zoom** (from 0.25x to 8x) and pan, plus a zoom
  indicator;
- light and dark **themes**;
- **sorting** by creation date, modification date, name or size;
- **search and filters**;
- **extended per-image info**: pixel dimensions, aspect ratio, DPI, color depth, and
  transparency.

## Model comparison

The gallery includes model-comparison views, useful together with [benchmark mode](bench.md):
after generating the same prompt across several models, you can line up the results visually
and judge quality differences directly.

## Ratings

You can rate images directly in the gallery. The ratings are exported to a
`gallery_ratings.json` file in the images folder through the browser's **File System Access
API**, which is available in Chromium-based browsers (Chrome and Edge). Other browsers can
display the gallery but cannot write the ratings file.

For a rating to be attributed to the correct model, the image must have been generated with
`-Metadata True`, so that its model is recorded in the `.json` sidecar. Without metadata the
gallery cannot know which model produced a given image.

## Feeding ratings back into selection

The ratings you collect are not just informational: `-Auto Quality` can read them. When a
model has accumulated enough gallery votes (a threshold number), the script blends those
votes with the model's quality rating using a normalized weighted average, so your own
visual judgment influences which model `-Auto Quality` picks. See the
[command-line reference](cli.md) and [managing models](models.md).

## Currency

The gallery displays only the currency you configured in `defaultCurrency` (see the
[configuration reference](config.md) and the [cost management guide](costs.md)), keeping the
cost figures consistent with the rest of the tool.

## Browser support summary

- **Viewing, zoom, sorting, filters, comparison**: any modern browser.
- **Writing ratings** to `gallery_ratings.json`: Chromium-based browsers (Chrome/Edge),
  because they implement the File System Access API.
