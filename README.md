# Weather graphic images for WeeWX

Generate a PHP weather image in PNG, GIF or JPEG using WeeWX report data and a
background image. The skin supplies default, banner, large banner, avatar and
custom layouts. Your web server runs `index.php` to draw the requested image.

This repository mirrors [the upstream source](https://github.com/glennmckechnie/weewx-WXgraphic). The maintained
catalog archive is **0.6.8.post20261022**. Install that fixed archive below;
the mirrored default branch can differ from its repaired payload.

The catalog archive targets WeeWX `>=5.5.2,<6` and Python `>=3.10,<3.15`.

![Example weather banner](skins/WXgraphic/examples/MessmateFarmCustom.png)

## Requirements

Use WeeWX with Cheetah report generation, a web host that executes PHP, and PHP's
GD extension. TrueType fonts need GD's FreeType support. The supplied font and
image files remain part of the skin. PyEphem in WeeWX's Python environment adds
the sunrise/sunset data used for day/night icons; the template also handles a
basic almanac without those values.

The repository does not declare a tested PHP version range. Use
`PHP_verify.php` on your own host to check its PHP/GD setup. A PHP file served
as source text means PHP execution is not configured.

## Install

Use `weectl` from the Python environment running WeeWX. With stock WeeWX 5,
install the published archive:

```sh
weectl extension install https://github.com/weewx-packages/catalog/releases/download/wxgraphic/v0.6.8.post20261022/wxgraphic-0.6.8.post20261022.tar.gz --config /path/to/weewx.conf
```

Replace `/path/to/weewx.conf` with your station configuration. You can also
download that URL and pass the local archive to the same command. Use elevated
permissions only when your station files require them.

## Publish the generated report

The installer adds `user.wxgraphic.WXgraphic` and the `WXgraphic` skin, and
enables this report in `weewx.conf`:

```ini
[StdReport]
    [[WXgraphic]]
        skin = WXgraphic
        enable = true
        HTML_ROOT = wxgraphic
        lang = en
```

Set `HTML_ROOT` to the location where WeeWX should generate the report.
Restart WeeWX yourself and let the next report run finish. Publish that output
to the PHP host using your existing FTP/rsync report or another deployment
method. Local report generation alone does not upload it.

The output contains `config.txt`, `DATA/wxgraphic_weewx.txt`, `index.php`, images,
fonts and icons. The data file comes from
[`DATA/wxgraphic_weewx.txt.tmpl`](skins/WXgraphic/DATA/wxgraphic_weewx.txt.tmpl).
It uses the configured `delimiter_dsv`, default `;`.

Check `$data_file_path` in the generated `config.txt`. It must resolve to the
published data file in the PHP server's filesystem. A remote host can have a
different path from the station's `HTML_ROOT`; set
`[WXgraphic] [[Clientraw]] data_file_path` in the skin to that server path if
needed. For an actual `clientraw.txt` source, use that file's path and the
Clientraw options instead.

Open `https://weather.example.com/wxgraphic/PHP_verify.php`, then
`https://weather.example.com/wxgraphic/`. Embed the image with:

```html
<img src="https://weather.example.com/wxgraphic/" alt="Current weather">
```

These are example URLs; use your own host and published path. The PHP endpoint
renders on request and reads the latest report file, so freshness follows the
report interval.

## Choose a layout and units

Edit the installed skin's `skin.conf`, retaining a backup. For example:

```ini
[WXgraphic]
    image_style = banner_big
    image_format = png
    additional_text = WeeWX
```

`image_style` accepts `default`, `banner`, `banner_big`, `avatar` and `custom`.
`image_format` accepts `png`, `gif` and `jpeg`. The report's WeeWX unit settings
control generated readings and labels. Color and font settings are under
`[WXgraphic] [[PHPGenerator]]`; layout details live in
[`config.txt.tmpl`](skins/WXgraphic/config.txt.tmpl). The original
[script reference](skins/WXgraphic/README.txt) explains the drawing functions,
backgrounds, fonts and image formats. Its standalone install procedure is
historical; use the fixed WeeWX archive above.

## Customize and troubleshoot

Keep customized backgrounds in both the installed skin and published output.
`CopyGenerator` copies static files on startup, so the skin is the persistent
source. Generated `config.txt` is replaced each report cycle.

To experiment directly on the host, temporarily comment out
`template = config.txt.tmpl` in `[CheetahGenerator] [[Config]]` in `skin.conf`.
Edit the generated `config.txt`, refresh the image and check the PHP error log.
When finished, transfer the changes to the installed `config.txt.tmpl` and
restore the template setting. Keep backups of both versions.

PHP errors can produce a blank image. Check PHP/GD, the generated config, data
path, fonts and images. Temporarily enabling `display_errors` in `index.php`
can help on a private development host. The diagnostic script may be removed
from the published host after use; retain the installed package assets so the
extension can still be managed normally.

The generated WeeWX feed does not populate every condition icon supported by
the original VWS/clientraw formats. Its built-in icon handling is limited to
the available data, including day/night when sunrise/sunset are present.
No forecast extension integration is established here.

## Origin and license

Glenn McKechnie adapted Anole Computer Services' Weather Graphic generator
for WeeWX. Copyright 2018-2020 Glenn McKechnie applies to the installer;
Anole's script notices and credits remain in the source. The installer states GPLv3. The root [LICENSE](LICENSE) contains GPLv2,
and bundled PHP retains its [GPLv2-or-later notice](skins/WXgraphic/license.txt).
See [changelog](changelog), [source origins](src/origins.txt), and the
[original script reference](skins/WXgraphic/README.txt). Original image and font
assets and their notices remain unchanged.
