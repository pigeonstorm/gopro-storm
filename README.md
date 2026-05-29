# GoPro File Import Script

This Ruby script automatically imports GoPro files from an SD card and organizes them by week and year based on their metadata. 

## Features

- **Automatic volume detection**: Scans common mount points for GoPro SD cards
- **Multi-volume support**: Import from multiple SD cards simultaneously
- Automatically detects GoPro files by extension and filename patterns
- Extracts creation date from EXIF metadata
- Organizes files into directories named `<year>_week_<week>`
- Preserves file metadata during copying
- **Skips re-copying** files that are already present and byte-for-byte identical (verified by MD5)
- Handles genuine filename collisions (same name, different content) by adding a numeric suffix
- **Move mode** (`--move`) deletes each source file once a verified copy exists in the destination
- Comprehensive error handling and reporting
- **Beautiful progress bar** showing current file and overall progress
- **Dry-run mode** (`--dry-run`) to preview operations without copying files
- **Integrity check** (`--check`) to compare MD5 checksums between the camera and the copied files
- **Safe cleanup** (`--clean`) to delete camera files only after verifying a matching MD5 copy exists

## Prerequisites

1. **Ruby**: Make sure Ruby is installed on your system
2. **ExifTool**: The script requires ExifTool to be installed on your system

### Installing ExifTool

**macOS (with Homebrew):**
```bash
brew install exiftool
```

**Ubuntu/Debian:**
```bash
sudo apt-get install libimage-exiftool-perl
```

**Windows:**
Download from: https://exiftool.org/

## Installation

1. Clone or download this repository
2. Install dependencies:
```bash
bundle install
```

## Usage

Expose the script to your PATH.

```bash
export PATH=$PATH:`pwd`
```
or
```bash
sudo cp gopro_storm /usr/local/bin/gopro_storm
```

### Basic Usage

```bash
ruby gopro_storm                                        # Auto-detect GoPro volumes
ruby gopro_storm /path/to/sd/card                       # Import from specific volume
ruby gopro_storm --volumes /path/to/sd/card /path/to/destination
```

### Advanced Usage

```bash
ruby gopro_storm --dry-run                              # Preview auto-detected volumes
ruby gopro_storm --volumes /Volumes/A,/Volumes/B        # Import from multiple volumes
ruby gopro_storm --dry-run --volumes /Volumes/A,/Volumes/B /path/to/destination
```

### Options

- `--volumes V1,V2`: Specify volumes to scan (comma-separated list)
- `--auto-detect`: Auto-detect GoPro volumes (default when no source paths given)
- `--dry-run`: Preview what would be done without actually copying files
- `--limit N`: Limit number of files to process in dry-run mode (for fast testing)
- `--since DATE`: Only process files created on or after `DATE` (`YYYY-MM-DD`)
- `--until DATE`: Only process files created on or before `DATE` (`YYYY-MM-DD`)
- `--last-week`: Only process files from the last 7 days
- `--check`: Compare MD5 checksums between the source (camera) and the destination copies. Read-only — no files are changed.
- `--clean`: Delete source (camera) files that already have a verified MD5-matching copy in the destination. Combine with `--dry-run` to preview deletions.
- `--move`: Import normally, then delete each source file once it is safely copied/verified in the destination (i.e. move instead of copy). Cannot be combined with `--check` or `--clean`. Combine with `--dry-run` to preview.
- `--help`, `-h`: Show help message

### Duplicate Handling

When a file with the same name already exists in the target week directory, the script
compares the two files (size first, then MD5):

- **Identical** → the copy is skipped (no duplicate is created). With `--move`, the source
  file is deleted since it is already safely stored.
- **Different content** → the new file is copied alongside the existing one with a numeric
  suffix (e.g. `GOPR0001_1.MP4`), so nothing is overwritten.

This makes re-running an import safe and idempotent — already-imported files are simply skipped.

### Verifying and Cleaning Up (`--check` / `--clean`)

After importing, you can verify the copies are intact and free up space on the SD card.

`--check` walks every GoPro file on the source and looks for a matching copy in the
destination (matching by filename, including numeric duplicate suffixes such as
`GOPR0001_1.MP4`), then compares MD5 checksums:

- ✔ **Verified**: a destination copy with an identical MD5 was found
- ⚠ **MD5 mismatch**: a copy exists but its checksum differs
- ✘ **Missing**: no copy was found in the destination

`--clean` performs the same comparison and then removes source files that are safe to delete:

- ✔ **Deleted**: a verified MD5-matching copy exists, so the source file is removed
- **Skip**: no copy was found in the destination, so the file is left untouched (a skip message is shown)
- ⚠ **Warning**: a copy exists but the MD5 does not match — the source file is **kept** and a warning is displayed

> **Note:** Both modes require the destination to already exist (it is never created),
> and they respect the `--since`, `--until`, and `--last-week` date filters.

```bash
# Verify every camera file has an intact copy in the destination
ruby gopro_storm --check --volumes /Volumes/UNTITLED /Users/me/Gopro

# Preview which camera files would be deleted (nothing is removed)
ruby gopro_storm --clean --dry-run --volumes /Volumes/UNTITLED /Users/me/Gopro

# Delete camera files that already have a verified copy
ruby gopro_storm --clean --volumes /Volumes/UNTITLED /Users/me/Gopro
```

### Arguments

- `source_path`: Path(s) to SD card(s) or directory containing GoPro files
  - If not specified, auto-detects GoPro volumes
  - Multiple paths can be specified as separate arguments
- `destination_path`: Directory where organized files should be copied (optional, defaults to current directory)

### Examples

```bash
# Auto-detect and import from all GoPro volumes
ruby gopro_storm

# Preview auto-detected volumes
ruby gopro_storm --dry-run

# Import from specific SD card
ruby gopro_storm /Volumes/UNTITLED

# Import from multiple volumes
ruby gopro_storm --volumes /Volumes/UNTITLED,/Volumes/GOPRO2

# Import to a specific directory (use --volumes when also giving a destination)
ruby gopro_storm --volumes /Volumes/UNTITLED /Users/me/Videos/GoPro

# Preview import from multiple volumes to specific directory
ruby gopro_storm --dry-run --volumes /Volumes/A,/Volumes/B /Users/me/GoPro

# Import from a directory on your hard drive
ruby gopro_storm /Users/me/Desktop/GoPro_Files

# Verify copies / clean up the card (see "Verifying and Cleaning Up" above)
ruby gopro_storm --check --volumes /Volumes/UNTITLED /Users/me/Videos/GoPro
ruby gopro_storm --clean --volumes /Volumes/UNTITLED /Users/me/Videos/GoPro

# Move: import, then delete each source file once it is verified in the destination
ruby gopro_storm --move --volumes /Volumes/UNTITLED /Users/me/Videos/GoPro
ruby gopro_storm --move --dry-run --volumes /Volumes/UNTITLED /Users/me/Videos/GoPro

# Show help
ruby gopro_storm --help
```

### Volume Detection

The script automatically detects GoPro volumes by looking for:
- GoPro directory structures (`DCIM/100GOPRO`, `DCIM/101GOPRO`, etc.)
- GoPro filename patterns (`GOPR0001.MP4`, `GP010001.MP4`, etc.)
- `MISC` directory (common on GoPro SD cards)

It scans common mount points:
- `/Volumes` (macOS)
- `/media` (Linux)
- `/mnt` (Linux alternative)

## File Detection

The script automatically detects GoPro files using:

- **File extensions**: `.mp4`, `.mov`, `.jpg`, `.jpeg`, `.raw`, `.gpr`
- **Filename patterns**:
  - `GOPR0001.MP4` (standard video files)
  - `GP011234.MP4` (timelapse/multi-shot)
  - `GX011234.MP4` (gyro data)
  - `GH011234.MP4` (high-resolution video)

## Directory Structure

Files are organized into directories named like:
```
2024_week_01/     # Week 1 of 2024
2023_week_15/     # Week 15 of 2023
2022_week_52/     # Week 52 of 2022
```

## Metadata Sources

The script tries to extract creation dates from the following EXIF fields (in order of preference):
1. DateTimeOriginal
2. CreateDate
3. DateTime
4. ModifyDate
5. File modification time (fallback)

## Error Handling

- Files without readable metadata fall back to file modification time
- Identical files already in the destination are skipped (verified by MD5); genuine name collisions get a numeric suffix
- All errors are reported but don't stop the import process
- A summary is provided at the end showing total files processed and any errors

## Troubleshooting

### "ExifTool not found"
Make sure ExifTool is installed and accessible in your PATH.

### "Source path does not exist"
Verify that your SD card is properly mounted and the path is correct.

### "Permission denied"
Make sure you have read access to the source and write access to the destination.

### Files not being detected
Check if your GoPro files follow standard naming conventions. The script looks for common GoPro filename patterns.

## Contributing

Feel free to submit issues or pull requests if you encounter problems or have suggestions for improvements.
