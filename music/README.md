# 🎵 Music Tools Taskfile

This Taskfile provides a collection of music-related utilities, focusing on
audio format conversions and processing tasks.

## 📋 Prerequisites

- [FFmpeg](https://ffmpeg.org) installed
- [Task](https://taskfile.dev) installed (`brew install go-task/tap/go-task`)

## 🎯 Available Tasks

### Audio Conversion Tasks

#### wav-to-mp3

Converts WAV file(s) to MP3 format with configurable bitrate. Supports both
single file and batch operations.

```bash
# Single file conversion
task wav-to-mp3 INPUT="~/Music/song.wav"

# Single file with custom settings
task wav-to-mp3 INPUT="~/Music/song.wav" OUTPUT="~/Music/converted.mp3" BITRATE=320k

# Batch conversion (current directory)
task wav-to-mp3

# Batch conversion (specific directory)
task wav-to-mp3 DIR="~/Music/wav-files" BITRATE=256k
```

Variables:

- `INPUT`: Path to the WAV file to convert (optional - if not provided, runs in
  batch mode)
- `OUTPUT`: Custom output filename (optional - defaults to same name with .mp3
  extension)
- `DIR`: Directory to scan for WAV files in batch mode (optional - defaults to
  current directory)
- `BITRATE`: MP3 encoding bitrate (optional - defaults to 192k)
- `BATCH`: Force batch mode even with INPUT specified (optional - defaults to false)

#### batch-wav-to-mp3

Convenience alias for batch WAV to MP3 conversion.

```bash
# Convert all WAV files in current directory
task batch-wav-to-mp3

# Convert all WAV files in specific directory with custom bitrate
task batch-wav-to-mp3 DIR="~/Music/wav-files" BITRATE=320k
```

Variables:

- `DIR`: Directory to scan for WAV files (optional - defaults to current directory)
- `BITRATE`: MP3 encoding bitrate (optional - defaults to 192k)

## 📝 Example Usage

### Format Conversion Examples

1. **Converting a single audio file:**

```bash
task wav-to-mp3 INPUT="~/Downloads/My Recording.wav"
```

1. **Converting with maximum MP3 quality:**

```bash
task wav-to-mp3 INPUT="~/Music/master.wav" BITRATE=320k
```

1. **Batch converting an album:**

```bash
task batch-wav-to-mp3 DIR="~/Music/Albums/New Album" BITRATE=256k
```

1. **Converting to a specific location:**

```bash
task wav-to-mp3 INPUT="~/Downloads/audio.wav" OUTPUT="~/Music/Converted/audio.mp3"
```

## 🎚️ Audio Quality Reference

### MP3 Bitrates

- `128k`: Acceptable quality, smallest file size
- `192k`: Good quality, balanced file size (default)
- `256k`: Very good quality, larger files
- `320k`: Maximum MP3 quality, largest files

## 🔧 Extending the Taskfile

You can extend these tasks in your own Taskfile by importing this template:

```yaml
version: "3"

includes:
  music:
    taskfile: ./music.yml
    optional: true

tasks:
  # Convert and organize music files
  convert-album:
    desc: "Convert WAV album to MP3 and organize"
    vars:
      ALBUM: "{{.ALBUM}}"
      ARTIST: '{{.ARTIST | default "Unknown Artist"}}'
    cmds:
      - mkdir -p "~/Music/{{.ARTIST}}/{{.ALBUM}}"
      - task: music:wav-to-mp3
        vars:
          DIR: "{{.DIR}}"
          BITRATE: 320k
      - mv {{.DIR}}/*.mp3 "~/Music/{{.ARTIST}}/{{.ALBUM}}/"
      - echo "Album converted and organized!"

  # Process audio for podcast
  podcast-prep:
    desc: "Convert and normalize audio for podcast"
    cmds:
      - task: music:wav-to-mp3
        vars:
          INPUT: "{{.INPUT}}"
          BITRATE: 128k # Lower bitrate for speech
      - echo "Podcast audio prepared!"
```

## 🚀 Planned Features

Future tasks that could be added to this music toolkit:

- **flac-to-mp3**: Convert FLAC files to MP3
- **mp3-to-wav**: Convert MP3 back to WAV for editing
- **normalize-audio**: Normalize audio levels across files
- **extract-audio**: Extract audio from video files
- **merge-audio**: Combine multiple audio files
- **split-audio**: Split audio files by silence or duration
- **audio-metadata**: Read/write audio file metadata
- **batch-rename**: Rename audio files based on metadata

## 🔍 Important Notes

- All tasks preserve original files (non-destructive operations)
- File paths with spaces and special characters are fully supported
- Tilde (`~`) expansion works for home directory paths
- Batch operations process only the specified directory (not subdirectories)
- Tasks include progress indicators for long operations
- Error handling ensures clear feedback when issues occur

## 🚨 Troubleshooting

1. **"ffmpeg command not found" error:**

   - Install FFmpeg using your package manager
   - macOS: `brew install ffmpeg`
   - Ubuntu/Debian: `sudo apt-get install ffmpeg`
   - Windows: Download from [ffmpeg.org](https://ffmpeg.org/download.html)

1. **"Input file not found" error:**

   - Verify the file path is correct
   - Use quotes for paths with spaces: `INPUT="~/My Music/song.wav"`
   - Try using absolute paths instead of relative ones

1. **No files found in batch mode:**

   - Ensure files have the correct extension (`.wav`, case-sensitive)
   - Check read permissions for the directory
   - Specify directory explicitly: `DIR="./my-audio-files"`

1. **Poor audio quality after conversion:**
   - Increase the bitrate: use `BITRATE=256k` or `BITRATE=320k`
   - Check the source file quality - conversions can't improve poor sources

## 📚 Resources

- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)
- [Audio Format Comparison](https://en.wikipedia.org/wiki/Comparison_of_audio_coding_formats)
- [MP3 Bitrate Guide](https://www.adobe.com/creativecloud/video/discover/audio-bitrate.html)
