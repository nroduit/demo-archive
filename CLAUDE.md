# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a **data/content repository**, not a software project. It holds the DICOM
sample datasets behind the [Weasis website demo](https://nroduit.github.io/en/demo/),
served as a static site over GitHub Pages from `https://nroduit.github.io/demo-archive/`.

There is **no build, test, lint, or run step**. Work here means adding/curating DICOM
files and keeping the XML manifests that index them in sync. The bulk of the repo is
binary DICOM data (files with no extension) and JPEG thumbnails.

## Layout

Three independent dataset roots, each with the same pattern (manifest + data + thumbnails):

- `demo/` — the main flat collection (~900 DICOM files). Each DICOM file is named by its
  bare `SOPInstanceUID` (no extension) and sits directly in `demo/`. Thumbnails live in
  `demo/thumb/`. Many small topic-specific manifests live here, one per viewer feature being
  demonstrated (e.g. `color.xml`, `video.xml`, `rt.xml`, `seg.xml`, `sr.xml`, `ko.xml`, the
  `pr-*.xml` presentation-state samples, `pixel-*.xml`, `compression*.xml`).
- `3d/` — `head-neck.xml` plus `3d/head-neck/` containing files named by `SOPInstanceUID`.
- `Lumbar/` — `mf.xml` plus per-series subfolders (`3-PlaneLoc/`, `SagT2frFSES/`, …) of
  `IM-XXXX-XXXX.dcm` files, with thumbnails in `Lumbar/thumb/`.

Loose root files (`CT0081.dcm`, `us-palette.dcm`) are standalone samples not tied to a manifest.

## Manifest format (the only "code" here)

Manifests are **Weasis XML manifests**, namespace `http://www.weasis.org/xsd/2.5`. They are
the contract the viewer reads to discover and download studies. Structure is a strict nesting:

```
manifest > arcQuery > Patient > Study > Series > Instance
```

Key attributes when editing or adding entries:

- `arcQuery@baseUrl` — absolute URL prefix the viewer resolves downloads against
  (e.g. `https://nroduit.github.io/demo-archive/demo/`).
- `Instance@DirectDownloadFile` — path resolved relative to `baseUrl`. In `demo/` this is the
  bare `SOPInstanceUID` (file sits in `demo/`); in `Lumbar/` it is a subpath like
  `/3-PlaneLoc/IM-0001-0001.dcm`.
- `Series@DirectDownloadThumbnail` — path to the JPEG thumbnail, resolved the same way.
- UIDs must be consistent: `Instance@SOPInstanceUID` matches the data file's UID, and
  `SeriesInstanceUID` / `StudyInstanceUID` group instances correctly.

When adding a sample: drop the DICOM file (named by its SOPInstanceUID for `demo/`) and its
thumbnail into the right folder, then add the matching `Instance`/`Series`/`Study` node to the
manifest so the path, UIDs, and `baseUrl` resolve to the new file.

## Conventions

- Patient data in samples is anonymized (synthetic `PatientName`/`PatientID`); keep it that way.
- The git history is data-management commits ("Add samples", "Update rt.xml", "fix order issue");
  match that style — describe what dataset/manifest changed.
- `baseUrl` values are tied to the published GitHub Pages URL; don't change them unless the
  hosting location actually moves.