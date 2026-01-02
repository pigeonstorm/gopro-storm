# GoPro Storm ⚡️

Backup your GoPro from the terminal using a lightweight Ruby script. Avoid buggy official apps and OS-native software—this script automatically imports your files and organizes them into clean, week-based folders using EXIF metadata.

## Features

- **Automatic volume detection**: Scans common mount points for GoPro SD cards
- **Multi-volume support**: Import from multiple SD cards simultaneously
- Automatically detects GoPro files by extension and filename patterns
- Extracts creation date from EXIF metadata
- Organizes files into directories named `week_<week>_<year>`
- Preserves file metadata during copying
- Handles duplicate filenames gracefully
- Comprehensive error handling and reporting
- **Beautiful progress bar** showing current file and overall progress
- **Dry-run mode** (`--dry-run`) to preview operations without copying files

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
sudo cp gopro_import /usr/local/bin/gopro_import
```

### Basic Usage

```bash
ruby gopro_import                           # Auto-detect GoPro volumes
ruby gopro_import /path/to/sd/card          # Import from specific volume
ruby gopro_import /path/to/sd/card /path/to/destination
```

### Advanced Usage

```bash
ruby gopro_import --dry-run                           # Preview auto-detected volumes
ruby gopro_import --volumes /Volumes/A,/Volumes/B     # Import from multiple volumes
ruby gopro_import --dry-run --volumes /Volumes/A,/Volumes/B /path/to/destination
```

### Options

- `--volumes V1,V2`: Specify volumes to scan (comma-separated list)
- `--auto-detect`: Auto-detect GoPro volumes (default when no source paths given)
- `--dry-run`: Preview what would be done without actually copying files
- `--limit N`: Limit number of files to process in dry-run mode (for fast testing)
- `--help`, `-h`: Show help message

### Arguments

- `source_path`: Path(s) to SD card(s) or directory containing GoPro files
  - If not specified, auto-detects GoPro volumes
  - Multiple paths can be specified as separate arguments
- `destination_path`: Directory where organized files should be copied (optional, defaults to current directory)

### Examples

```bash
# Auto-detect and import from all GoPro volumes
ruby gopro_import.rb

# Preview auto-detected volumes
ruby gopro_import --dry-run

# Import from specific SD card
ruby gopro_import /Volumes/UNTITLED

# Import from multiple volumes
ruby gopro_import /Volumes/UNTITLED /Volumes/GOPRO2

# Import from multiple volumes using --volumes option
ruby gopro_import --volumes /Volumes/UNTITLED,/Volumes/GOPRO2

# Import to a specific directory
ruby gopro_import /Volumes/UNTITLED /Users/me/Videos/GoPro

# Preview import from multiple volumes to specific directory
ruby gopro_import --dry-run --volumes /Volumes/A,/Volumes/B /Users/me/GoPro

# Import from a directory on your hard drive
ruby gopro_import /Users/me/Desktop/GoPro_Files

# Show help
ruby gopro_import --help
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
week_01_2024/     # Week 1 of 2024
week_15_2023/     # Week 15 of 2023
week_52_2022/     # Week 52 of 2022
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
- Duplicate filenames are handled by adding numeric suffixes
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
