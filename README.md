# SessionLint

**A linter and handoff validator for music production sessions.**

SessionLint analyzes exported audio files and project handoff folders before they are sent to a mixing engineer, mastering engineer, producer, collaborator, or client.

It catches technical problems, organizational mistakes, ambiguous filenames, alignment issues, missing assets, and other common handoff problems before they become someone else's problem.

> Lint your session before you send it.

---

## Why SessionLint?

Music project handoffs are often messy.

A typical folder may look like this:

```text
final_bass.wav
final_bass2.wav
vocals FINAL.wav
vocals new.wav
Audio 17.wav
guitar_good.wav
track(6).wav
```

And even when the files look fine, there may still be hidden problems:

- one stem starts 1.2 seconds late;
- one file has a different sample rate;
- a vocal export clips;
- two files are probably duplicates;
- stereo and mono exports are inconsistent;
- filenames do not explain what the files contain;
- the rough mix is missing;
- BPM or key information is missing;
- a reference track was forgotten;
- an outdated version was accidentally included.

These problems waste time and create unnecessary back-and-forth between artists, producers, and engineers.

Software developers have linters.

Music production should have one too.

---

## Example

```bash
sessionlint ./MySong
```

Output:

```text
SESSIONLINT

28 audio files analyzed

TECHNICAL
✓ All files use 48 kHz sample rate
✓ All files use 24-bit audio
✓ 27 files share the same start position
✗ BackingVocal_03.wav starts 1.248 s late
⚠ LeadVocal.wav reaches -0.02 dBFS
✓ No corrupted audio files detected

ORGANIZATION
⚠ 6 ambiguous filenames
⚠ Possible duplicate files:
  vocal_final.wav
  vocal_final2.wav

HANDOFF
✓ Stem durations are consistent
✗ Rough mix not found
✗ Session notes not found
⚠ BPM not specified

READY SCORE

72 / 100

Status:
NOT READY FOR HANDOFF
```

---

## Goals

SessionLint should answer one simple question:

> **Is this session ready to be handed to another person?**

The goal is not to judge music creatively.

SessionLint does not decide whether:

- the vocal sounds good;
- the arrangement is interesting;
- the mix is aesthetically correct;
- the guitar tone is appropriate.

It checks things that can be verified objectively.

---

## What SessionLint checks

### Audio integrity

- file readability;
- duration;
- sample rate;
- bit depth;
- channel count;
- peak level;
- digital clipping;
- silence;
- corrupted or malformed files.

### Alignment

- common start position;
- consistent duration;
- suspicious offsets;
- missing or unexpectedly short exports.

### File consistency

- mixed sample rates;
- mixed bit depths;
- inconsistent channel formats;
- unusual encoding.

### Naming

Detect filenames such as:

```text
Audio 12.wav
final2.wav
new vocal.wav
track copy.wav
```

and flag them as ambiguous.

SessionLint may suggest clearer alternatives without automatically renaming anything.

### Duplicate detection

Detect:

- exact duplicates;
- renamed duplicates;
- near-identical audio exports.

### Handoff completeness

Optional project rules may require:

- rough mix;
- references;
- session notes;
- BPM;
- key;
- lyrics;
- stems;
- metadata.

---

## Handoff Profiles

Different workflows require different rules.

SessionLint should support configurable profiles.

Example:

```yaml
profile: mixing

audio:
  require_same_sample_rate: true
  require_same_bit_depth: true
  require_common_start: true

recommended:
  - rough_mix
  - session_notes
  - bpm

naming:
  reject_ambiguous_names: false
```

Possible profiles:

```text
mixing
mastering
collaboration
archive
custom
```

---

## Clean Handoff Packages

Future versions of SessionLint should be able to turn a validated session into a structured package.

```bash
sessionlint pack ./MySong
```

Example:

```text
MySong_MixHandoff/
├── README.md
├── manifest.json
├── rough_mix.wav
├── references/
├── stems/
│   ├── drums/
│   ├── bass/
│   ├── guitars/
│   ├── vocals/
│   └── other/
└── notes/
```

SessionLint should never modify the original folder without explicit permission.

---

## Project Manifest

A generated manifest may contain:

```json
{
  "project": "My Song",
  "sample_rate": 48000,
  "bit_depth": 24,
  "bpm": 124,
  "key": "D major",
  "files": 28,
  "profile": "mixing"
}
```

This creates a machine-readable description of a music handoff.

---

## Philosophy

SessionLint follows several principles:

### Objective over subjective

Only report findings that can be supported by evidence.

### Explain every warning

Bad:

```text
Your session is messy.
```

Good:

```text
BackingVocal_03.wav starts 1.248 seconds later than 27 other stems.
```

### Never destroy work

Analysis should be read-only by default.

Renaming, moving, converting, or deleting files must require explicit user action.

### Configurable, not dogmatic

There is no universal music-production workflow.

A mastering handoff and a mixing handoff have different requirements.

### Fast enough to use every time

Running SessionLint should feel like running a code linter before a commit.

---

## Planned Architecture

```text
Audio Files
    ↓
Scanner
    ↓
Metadata Extraction
    ↓
Audio Analysis
    ↓
Rule Engine
    ↓
Findings
    ↓
CLI / Report / Handoff Package
```

The analysis engine should remain independent from the CLI and future GUI.

---

## Proposed Tech Stack

- Python
- Typer
- Pydantic
- NumPy
- SciPy
- soundfile
- FFmpeg / ffprobe where appropriate

Additional DSP libraries should only be added when they solve a concrete problem.

---

## Roadmap

### v0.1 — Audio Scanner

- recursively scan a folder;
- detect supported audio files;
- read metadata;
- report sample rate;
- bit depth;
- channels;
- duration;
- peak level.

### v0.2 — Consistency Rules

- sample-rate consistency;
- bit-depth consistency;
- channel consistency;
- duration analysis;
- missing/corrupted files.

### v0.3 — Alignment

- detect shared start/end behavior;
- identify suspicious offsets;
- analyze leading silence.

### v0.4 — Naming & Duplicates

- ambiguous filename detection;
- exact duplicate detection;
- near-duplicate audio detection.

### v0.5 — Handoff Profiles

- mixing;
- mastering;
- collaboration;
- custom configuration.

### v0.6 — Pack

Generate clean, structured handoff packages.

### Later

- desktop UI;
- Logic Pro integration;
- Ableton Live integration;
- Pro Tools integration;
- Reaper integration;
- intelligent stem classification;
- project manifests;
- archive validation.

---

## Non-Goals

SessionLint is not:

- a DAW;
- a mixing assistant;
- an automatic mastering service;
- an AI music generator;
- a creative quality scorer;
- a replacement for communication between collaborators.

It exists to make technical handoffs cleaner and more reliable.

---

## Status

SessionLint is currently in early development.

The first milestone is intentionally small:

> Scan a folder of audio exports and produce a trustworthy technical consistency report.

---

## License

To be defined.

---

## Contributing

Contributions, test sessions, real-world handoff examples, and workflow suggestions are welcome.

Especially useful are examples of recurring problems encountered by:

- mixing engineers;
- mastering engineers;
- producers;
- recording engineers;
- artists;
- session musicians.
