
තීරු අකුරු මුහුණත / Theeru Font
==============

[![Build][badge-build]][link-build]
[![Fontspector QA][badge-qa]][link-qa]
[![Release][badge-release]][link-release]

[badge-build]: https://github.com/mooniak/theeru-font/actions/workflows/build.yaml/badge.svg?branch=dev
[badge-qa]: https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fsrc.mooniak.com%2Ftheeru-font%2Fbuild.json&query=%24.qa.pass_rate&suffix=%25&label=fontspector%20QA%20(dev%20build)&color=informational
[badge-release]: https://img.shields.io/github/v/release/mooniak/theeru-font?label=release
[link-build]: https://github.com/mooniak/theeru-font/actions/workflows/build.yaml
[link-qa]: https://src.mooniak.com/theeru-font/reports/fontspector/fontspector-report.html
[link-release]: https://github.com/mooniak/theeru-font/releases

Theeru is an open-source font family for comics and graphic storytelling from Mooniak, initiated with support from comic artist Sachi Ediriweera. The font currently covers Sinhala only, designed by Kosala Senevirathne; Mooniak and Sachi plan to extend it to support other languages of the South Asian region.

***

## Get the font

- **Download:** the [Releases](https://github.com/mooniak/theeru-font/releases) page has tagged, stable releases. A pre-release named `canary` is also published there; it is a rolling build of the latest development work and may be broken, so don't use it in production.
- **Build it yourself:** see [Building the fonts from source](#building-the-fonts-from-source).

## Font data

| | |
|---|---|
| **Family name** | Sinhala Comic Fonts (internal build name `ComicSinhala`) |
| **Scripts** | Sinhala |
| **Format** | Variable TrueType (`ComicSinhala[opsz,wdth,wght].ttf`) |
| **Axes** | `wght` (Weight), `wdth` (Width), `opsz` (Optical size) |
| **Units per em** | 1000 |
| **Source format** | [Glyphs](https://glyphsapp.com/) 3 package (`sources/Theeru.glyphspackage`) |
| **License** | [SIL Open Font License 1.1](OFL.txt) |

## Repository structure

| Path | What it is |
|---|---|
| `sources/Theeru.glyphspackage` | The font sources: glyphs, masters, kerning and OpenType feature code, as a Glyphs 3 package |
| `sources/ComicSinhalaUltra.glyphs` | A supplementary Glyphs source for the Ultra weight extreme |
| `sources/config.yaml` | Build configuration read by [gftools](https://github.com/googlefonts/gftools) |
| `Makefile` | Build, test and proof targets (see below) |
| `requirements.in`, `requirements.txt` | Python dependencies: the top-level list, and the pinned set used for builds |
| `fonts/` | Build output (created by `make build`; not committed) |
| `out/` | QA and proof reports (created by `make test` and `make proof`; not committed) |
| `documentation/` | Specimen scripts, sample comic pages and sample text used to show the font in use |
| `scripts/` | Helper scripts used by the Makefile and CI |
| `fontpackage.json` | Project metadata (designer, license, channels, directory listing info) used by Mooniak's tooling |

### Documentation and comic samples

The `documentation/` folder includes comic panels by four artists, used as real-world samples of the font in use for Sinhala comics and graphic storytelling:

| Artist | Sample |
|---|---|
| Achinthya Amarakoon | ![Comic sample by Achinthya Amarakoon](documentation/comic-1-achinthya-amarakoon.jpg) |
| Akila Weerasinghe | ![Comic sample by Akila Weerasinghe](documentation/comic-2-akila-weerasinghe.jpg) |
| Jeewaka Ekanayake | ![Comic sample by Jeewaka Ekanayake](documentation/comic-3-jeewaka-ekanayake.jpg) |
| Binusha Perera | ![Comic sample by Binusha Perera](documentation/comic-4-binusha-perera.jpg) |

`documentation/sample-text.pdf` and `documentation/test fonts/` hold additional sample text and reference static builds (Regular, Medium, Bold) used while testing the design.

## Building the fonts from source

Building needs only free, open-source tools. You do **not** need Glyphs to build, only to edit the sources.

### Requirements

- **macOS or Linux** (on Windows, use [WSL](https://learn.microsoft.com/windows/wsl/)).
- **Python 3.12** with `venv` and `pip`. This is the version the project's CI uses; the dependencies in `requirements.txt` are pinned, and other Python versions may fail to install them.
- **Make** and **git**.
- Network access for the first build, to download the Python dependencies.

On Debian or Ubuntu, if `pip` fails to compile a dependency, install the system packages used by the project's CI first:

```sh
sudo apt-get install ttfautohint libcairo2-dev python3-cairo-dev pkg-config python3-dev
```

### Build

```sh
git clone https://github.com/mooniak/theeru-font.git
cd theeru-font

# Use Python 3.12 explicitly, then let make install the pinned dependencies into it
python3.12 -m venv venv
make build
```

`make build` installs the pinned dependencies from `requirements.txt` into `venv/` (first run only), then builds the fonts from the Glyphs sources using the settings in `sources/config.yaml`. Under the hood this runs [`gftools builder`](https://github.com/googlefonts/gftools), which:

1. compiles the variable font with [fontmake](https://github.com/googlefonts/fontmake) (glyphsLib → ufo2ft → fontTools),
2. post-processes it with `gftools-fix-font` and adds the `STAT` table with `gftools-gen-stat`,
3. compresses it to WOFF2 (when enabled in `sources/config.yaml`).

The results are written to `fonts/`:

```
fonts/variable/ComicSinhala[opsz,wdth,wght].ttf
```

> Each rebuild deletes and recreates `fonts/`, so don't keep your own files there.

> **Note:** `sources/config.yaml` currently lists `comic-sinhala.glyphspackage` as its source, while the Glyphs package actually present in `sources/` is `Theeru.glyphspackage`. If `make build` reports a missing source file, update the `sources:` entry in `sources/config.yaml` to point at `Theeru.glyphspackage` first.

### After editing the sources

`make` decides whether to rebuild by comparing timestamps against the `.glyphspackage` folder itself, and editing a file *inside* the package doesn't update the folder's timestamp. So after you change the sources, `make build` may say `Nothing to be done`. Force a rebuild by removing the stamp file:

```sh
rm build.stamp
make build
```

### Quality checks and proofs

These are optional and need two extra tools, installed with [`cargo binstall`](https://github.com/cargo-bins/cargo-binstall):

```sh
cargo binstall fontspector     # for `make test`
cargo binstall diffenator3     # for `make proof`
```

| Command | What it does |
|---|---|
| `make test` | Runs [Fontspector](https://github.com/fonttools/fontspector) with the Google Fonts profile and writes HTML and Markdown reports to `out/fontspector/`. It prints a summary and does not stop the build on failures; read the report to see them. |
| `make proof` | Runs [diffenator3](https://github.com/googlefonts/diffenator3)'s `diff3proof` and writes an HTML proof to `out/proof/`. |
| `make images` | Renders the PNG specimen images from the DrawBot scripts in `documentation/`. Needs to run inside a git clone that has a commit and an `origin` remote. |
| `make manifest` | Emits `out/fontstatus.json`, the per-build manifest (files, hashes, version, channel, QA verdict) used by Mooniak's tooling. |
| `make stamp` | Stamps the version/channel (canary/dev/release) into the built fonts' name table, based on the CI environment; a no-op locally. |
| `make clean` | Deletes the `venv/` folder. |

### Build badges

The badges at the top of this page come from the automated build, which runs on every push to `dev`:

- **Build** is the status of the latest [GitHub Actions run](https://github.com/mooniak/theeru-font/actions/workflows/build.yaml) (build, QA and proofs).
- **Fontspector QA** is the pass rate of the Google Fonts profile checks on that same build, read from the build manifest published with the proof site. It tracks the latest development build, not the tagged release.

You get the same data locally: `make test` also writes one small badge file per check category to `out/badges/`, in the [shields.io endpoint format](https://shields.io/badges/endpoint-badge).

### Troubleshooting

- **`pip install` fails with a build error:** check you're on Python 3.12 (`venv/bin/python --version`). If `venv/` was created with another version, delete it and recreate it with `python3.12 -m venv venv`.
- **`fontspector not found` or `diff3proof not found`:** install them with `cargo binstall` as above and make sure `~/.cargo/bin` is on your `PATH`.
- **Build fails looking for `comic-sinhala.glyphspackage`:** see the note under [Build](#build) above — point `sources/config.yaml` at `Theeru.glyphspackage`.
- **Your edits don't show up in the font:** see [After editing the sources](#after-editing-the-sources).

## Contributing

Bug reports, design feedback and pull requests are welcome in the [issue tracker](https://github.com/mooniak/theeru-font/issues). Before opening a pull request:

1. Open pull requests against the `dev` branch; `main` is only updated when a release is made.
2. Make your changes in `sources/Theeru.glyphspackage` (this needs [Glyphs 3](https://glyphsapp.com/)).
3. Rebuild (`rm build.stamp && make build`) and run `make test`.

## Credits

| Name | Contribution |
|---|---|
| Sachi Ediriweera | Project initiation and support, as a comic artist |
| [Kosala Senevirathne](https://github.com/kosalaya) ([Mooniak](https://mooniak.com)) | Designer of the Sinhala glyph set |
| [Pathum Egodawatta](https://github.com/pathumego) ([Mooniak](https://mooniak.com)) | Project coordination and build |

### Organizations

- [Mooniak](https://mooniak.com/) leads the project.

Thanks also to the comic artists whose work appears as samples in `documentation/`: Achinthya Amarakoon, Akila Weerasinghe, Jeewaka Ekanayake and Binusha Perera.

## License

Theeru is released under the [SIL Open Font License, Version 1.1](OFL.txt). The license is also available with a FAQ at <https://openfontlicense.org>.
