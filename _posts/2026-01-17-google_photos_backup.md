---
layout: post
title: "Backing Up Google Photos with Metadata (Properly)"
categories: [photos, backup, linux, terminal]
---

I wanted an **independent backup outside of Google Photos** — something I control, with metadata preserved, and without trusting a single cloud provider.

Google Takeout gives you all your photos, but the output schema format isn’t user friendly (possibly intentionally so):
dozens of `.zip` files, each containing images plus separate JSON metadata.
This is how I turned that mess into a clean `Photos/YYYY/MM` archive, (mostly) using the terminal on Linux.

---

## Requirements

- `jq` to read Google’s JSON metadata  
- `exiftool` to reapply timestamps and GPS  
- `7z` to safely extract Takeout ZIPs  

> **Note:** `unzip` often reports *zip bomb warnings* for Google Photos exports.  
> This appears to be a **false positive** caused by Google’s ZIP structure (many small files with a high compression ratios), but it’s noisy and aborts the extraction.  
> `7z` handles these archives cleanly, so I used it instead.

Unfortunately, you still have to manually download each zipped archive (which, at ~75 GB, meant a lot of files for me[^1]). I tried grabbing the links to use with `wget`, but the URLs are seemingly only generated on click, so that approach went didn't work.

---

## Step 1: Define the basic file structure

```bash
#!/bin/bash
# Safely extract Google Takeout photos from multiple zips into Photos/YYYY/MM
# Applies timestamps + GPS metadata
# No overwrites, zip-by-zip processing

set -euo pipefail
IFS=$'\n\t'

PHOTOS='./Photos'
```

---

## Step 2: Fail early if required tools are missing

```bash
require_cmd() {
    command -v "$1" >/dev/null 2>&1 || {
        echo "Required command not found: $1"
        exit 1
    }
}
```

---

## Step 3: Extract one ZIP at a time (safely)

```bash
extract_zip() {
    local zip="$1"
    local tmp="$2"

    rm -rf "$tmp"
    mkdir -p "$tmp"

    echo "Extracting $zip → $tmp"

    if ! 7z x -y "$zip" -o"$tmp" >/dev/null; then
        rc=$?
        if [ "$rc" -ne 1 ]; then
            echo "7z failed for $zip (exit code $rc)"
            exit 1
        fi
        echo "7z warning for $zip (continuing)"
    fi
}

```

---

## Step 4: Prevent filename collisions without deduplication

```bash
unique_dest() {
    local dir="$1"
    local filename="$2"

    local base="$dir/$filename"
    local dest="$base"

    if [ -e "$dest" ]; then
        local name ext i=1
        ext="${filename##*.}"
        name="${filename%.*}"
        while [ -e "$dir/${name}__${i}.${ext}" ]; do
            i=$((i + 1))
        done
        dest="$dir/${name}__${i}.${ext}"
    fi

    echo "$dest"
}
```

---

## Step 5: Reapply timestamps and GPS metadata

```bash
apply_metadata() {
    local file="$1"
    local ts="$2"
    local lat="$3"
    local lon="$4"

    exiftool -overwrite_original \
        -AllDates="$(date -d "@$ts" '+%Y:%m:%d %H:%M:%S')" \
        "$file" >/dev/null

    if [ -n "$lat" ] && [ -n "$lon" ]; then
        exiftool -overwrite_original \
            -GPSLatitude="$lat" -GPSLongitude="$lon" \
            -GPSLatitudeRef=N -GPSLongitudeRef=E \
            "$file" >/dev/null
    fi
}
```

---

## Step 6: Turn JSON metadata into organized photos

```bash
process_json() {
    local json="$1"

    local photo ts lat lon src year month target_dir dest

    photo=$(jq -r '.title // empty' "$json")
    ts=$(jq -r '.photoTakenTime.timestamp // empty' "$json")
    lat=$(jq -r '.geoData.latitude // empty' "$json")
    lon=$(jq -r '.geoData.longitude // empty' "$json")

    [ -n "$photo" ] || return
    [ -n "$ts" ] || return

    src="$(dirname "$json")/$photo"
    [ -f "$src" ] || return

    year=$(date -d "@$ts" '+%Y')
    month=$(date -d "@$ts" '+%m')
    target_dir="$PHOTOS/$year/$month"
    mkdir -p "$target_dir"

    dest=$(unique_dest "$target_dir" "$photo")

    mv "$src" "$dest"
    apply_metadata "$dest" "$ts" "$lat" "$lon"
}
```

---

## Step 7: Process each ZIP cleanly

```bash
process_zip() {
    local zip="$1"
    local tmp="./tmp_$(basename "$zip" .zip)"

    echo "=== Processing $zip ==="

    extract_zip "$zip" "$tmp"

    find "$tmp" -type f -name '*.json' | while read json; do
        process_json "$json"
    done

    rm -rf "$tmp"
    echo "Finished $zip"
    echo
}
```

---

## Step 8: Tie it all together

```bash
require_cmd jq
require_cmd exiftool
require_cmd 7z

mkdir -p "$PHOTOS"

for zip in *.zip; do
    [ -f "$zip" ] || continue
    process_zip "$zip"
done

echo "All done. Photos are in $PHOTOS/YYYY/MM"
```

---

## Result

```bash
Photos
├── 2019
│   └── 08
├── 2020
│   ├── 01
│   └── 12
└── 2021
    └── 07
```

This gave me a **true independent backup**, browsable, metadata-complete, and not tied to Google’s ecosystem anymore.

The full script can be found [here](https://github.com/sdysch/dotfiles/tree/master/setups/arch/.local/bin/extract_takout_photos.sh). Any suggestions are very welcome!

## Caveats: duplicates
In my initial Google Photos extract, I specified **only** 'Photos from 2008, Photos from 2009...Photos from 2026'. If you download everything, including albums (which really just tag photos), then you will likely get a stream of duplicates.
I did not try to check or verify this with my script. I think it could be done with a hash check in theory.


[^1]: I didn't want to specify a 50GB max file size to download and extract, so I used 10 GB chunks
