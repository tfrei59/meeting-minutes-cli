# Meeting Minutes CLI

A command-line tool to record meetings, transcribe audio locally with Whisper, and generate meeting minutes automatically.

**100% local processing** — No cloud APIs, no subscriptions, no data leaving your machine.

## Features

✅ **Record meetings** via your microphone or system audio  
✅ **Transcribe locally** with OpenAI Whisper (no APIs)  
✅ **Generate minutes** from transcript + agenda  
✅ **Multiple output formats** — txt, vtt (timestamped), srt, json  
✅ **Organize automatically** — Timestamped directories per meeting  
✅ **Zero dependencies** (beyond ffmpeg, Whisper, Hermes)  

## Installation

### 1. Install Dependencies

```bash
# Install Whisper and audio tools
uv pip install --system openai-whisper pydub
sudo apt-get install -y ffmpeg sox libsndfile1

# Or on macOS
brew install ffmpeg sox
```

### 2. Download Whisper Model (One-Time)

```bash
python3 -c "import whisper; whisper.load_model('base')"
```

### 3. Install the Script

```bash
git clone https://github.com/tfrei59/meeting-minutes-cli.git
cd meeting-minutes-cli

# Make it executable
chmod +x meeting-minutes

# Add to PATH (option A: copy to ~/bin)
cp meeting-minutes ~/bin/

# Or (option B: create symlink)
ln -s $(pwd)/meeting-minutes ~/bin/meeting-minutes

# Or (option C: add alias to ~/.bashrc)
echo "alias meeting-minutes='$(pwd)/meeting-minutes'" >> ~/.bashrc
source ~/.bashrc
```

## Usage

### Basic Usage

```bash
# Record a meeting with agenda (prompts for it)
meeting-minutes "team-standup"

# Record a general meeting summary (no agenda - just press Ctrl+D)
meeting-minutes "impromptu-discussion"

# With a pre-written agenda file
meeting-minutes "quarterly-planning" ~/Documents/agenda.txt

# Examples
meeting-minutes "upper-nine-condo"
meeting-minutes "board-meeting" agenda.md
```

### Workflow

**With Agenda:**
1. **Run the script** with meeting name
2. **Enter agenda** (Ctrl+D when done)
3. **Script records** your meeting (Ctrl+C to stop)
4. **Whisper transcribes** the audio
5. **Minutes template generated** using your agenda

**Without Agenda (General Summary):**
1. **Run the script** with meeting name
2. **Press Ctrl+D immediately** (skip agenda step)
3. **Script records** your meeting (Ctrl+C to stop)
4. **Whisper transcribes** the audio
5. **Minutes template generated** ready to fill in

### Output

All files saved to: `~/meetings/TIMESTAMP-meeting-name/`

```
meetings/20261006-213000-upper-nine-condo/
├── audio.mp3              # Original recording
├── audio.txt              # Full transcript
├── audio.vtt              # Timestamped (for video players)
├── audio.srt              # Subtitle format
├── audio.json             # Structured with confidence scores
├── agenda.txt             # Your meeting agenda
└── minutes.md             # Minutes template (edit & share)
```

## Whisper Models

Choose which model to use:

| Model  | Speed        | Accuracy | Size   | RAM  |
|--------|--------------|----------|--------|------|
| tiny   | ~1 min/hr    | 40%      | 39 MB  | ~1 GB|
| **base**   | **~2-3 min/hr**  | **80%**      | **140 MB** | **~1 GB**|
| small  | ~5-10 min/hr | 85%      | 244 MB | ~2 GB|
| medium | ~20-30 min/hr| 90%      | 769 MB | ~5 GB|
| large  | ~60+ min/hr  | 95%      | 2.9 GB | ~10 GB|

**Use base by default.** Change with:

```bash
WHISPER_MODEL=small meeting-minutes "important-meeting"
WHISPER_MODEL=large meeting-minutes "critical-meeting"
```

## Customization

### Change default Whisper model

Edit `meeting-minutes`:

```bash
WHISPER_MODEL="${WHISPER_MODEL:-medium}"  # Default: medium instead of base
```

### Change meetings directory

```bash
export MEETINGS_DIR="/path/to/my/meetings"
meeting-minutes "standup"
```

### Change audio quality

Edit `meeting-minutes` line ~70:

```bash
# Lower quality (smaller files, faster transcription)
ffmpeg -f pulse -i default -acodec libmp3lame -q:a 9 ...

# Higher quality (larger files, better transcription)
ffmpeg -f pulse -i default -acodec libmp3lame -q:a 0 ...
```

## Examples

### Daily Standup (15 minutes)

```bash
$ meeting-minutes "daily-standup"
Enter agenda: (quick notes, press Ctrl+D)
Updates from team members
Blockers discussion
(Ctrl+D)
Recording... (press Ctrl+C when done)
✓ Transcript ready in ~/meetings/20261006-093000-daily-standup/
```

### Weekly Planning (1 hour)

```bash
meeting-minutes "weekly-planning" ~/Documents/planning-agenda.txt
# Uses agenda from file, records, transcribes
```

### Condo Association Meeting (Like Your Upper Nine Example)

```bash
meeting-minutes "upper-nine-condo" ~/Documents/Upper\ Nine\ Oct\ 6\ 2026.pdf
# Creates polished minutes from transcript
```

## Tips

1. **Test audio setup before big meetings:**
   ```bash
   ffmpeg -f pulse -i default -t 10 -acodec libmp3lame -q:a 4 test.mp3
   ```

2. **Provide detailed agenda** — Helps Whisper understand context

3. **Quiet environment** — Background noise reduces transcription quality

4. **Review transcript** before finalizing minutes:
   ```bash
   cat ~/meetings/TIMESTAMP-meeting-name/audio.txt
   ```

5. **Edit minutes with your notes:**
   ```bash
   nano ~/meetings/TIMESTAMP-meeting-name/minutes.md
   ```

6. **Convert to PDF:**
   ```bash
   libreoffice --headless --convert-to pdf ~/meetings/TIMESTAMP-meeting-name/minutes.md
   ```

## Troubleshooting

### `whisper: command not found`

```bash
# Activate the virtual environment
source ~/.hermes-whisper/bin/activate
whisper --version
```

### No audio captured

```bash
# List available audio devices
ffmpeg -f pulse -list_devices true -i ""

# Try ALSA instead of PulseAudio
ffmpeg -f alsa -i hw:0 -t 10 test.mp3
```

### Slow transcription

- Use smaller model: `WHISPER_MODEL=tiny meeting-minutes "standup"`
- Shorter meetings: <30 minutes
- Enable GPU (if available): requires CUDA setup

### Poor transcription quality

- Use larger model: `WHISPER_MODEL=medium meeting-minutes "meeting"`
- Improve audio quality (reduce background noise)
- Provide detailed agenda for context

## Privacy & Security

✅ **100% Local** — All audio processing on your machine  
✅ **No APIs** — No cloud dependencies or subscriptions  
✅ **No Data Collection** — Audio never leaves your computer  
✅ **Open Source** — Full transparency on what's happening  

## Requirements

### System Dependencies

- **bash** (≥ 4.0) — Shell scripting
- **ffmpeg** (≥ 4.0) — Audio recording and encoding
- **Python 3** (≥ 3.8) — Required for Whisper

### Python Packages

- **openai-whisper** (≥ 20230314) — Local speech-to-text transcription
- **pydub** (≥ 0.25.1) — Optional, for advanced audio processing

### Optional

- **sox** — Advanced audio manipulation (alternative to ffmpeg for some operations)
- **Hermes Agent** — For workflow integration and advanced automation
- **libreoffice** — For converting Markdown minutes to PDF
- **GPU support** (NVIDIA CUDA) — For faster transcription (5-10x speedup)

### Operating System Support

- ✅ **Linux** — Primary platform (Debian, Ubuntu, Fedora, Arch, etc.)
- ✅ **macOS** — Fully supported (Intel & Apple Silicon)
- ✅ **Windows** — Supported via WSL2 (Windows Subsystem for Linux)

### Hardware Requirements

**Minimum (Works, but slow):**
- CPU: 2-core processor
- RAM: 4 GB
- Storage: 500 MB for Whisper base model

**Recommended (Smooth experience):**
- CPU: 4-core processor (or higher)
- RAM: 8 GB or more
- Storage: 1 GB available for Whisper model + meeting files
- GPU: Optional but recommended (NVIDIA RTX or similar for 5-10x speedup)

### Audio Hardware

- **Microphone** — Any USB or built-in microphone
- **PulseAudio or ALSA** — Linux audio system (usually pre-installed)
- **Speakers** — For monitoring during recording (optional)

## License

MIT — Feel free to fork, modify, and share improvements!

## Contributing

Found a bug? Have an idea? Issues and PRs welcome:

```bash
git clone https://github.com/tfrei59/meeting-minutes-cli.git
cd meeting-minutes-cli
# Make your changes
git commit -m "Describe your change"
git push
```

## Changelog

### v1.0.0 (2026-10-06)
- Initial release
- Record, transcribe, generate minutes
- Whisper integration
- Multiple output formats
- Minutes template generation

---

**Questions?** Open an issue or reach out!
