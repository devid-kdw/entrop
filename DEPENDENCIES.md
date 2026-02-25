# ENTROP — Dependencies & Setup

## System Requirements

| Requirement | Minimum | Recommended |
|---|---|---|
| macOS | 13.0 (Ventura) | 14.0+ (Sonoma) |
| Chip | Apple Silicon (M1) | M2 / M3 |
| Xcode | 15.0 | 15.4+ |
| CMake | 3.24 | 3.28+ |
| C++ Standard | C++20 | C++20 |
| Git | 2.30 | Latest |

## Setup Steps

### 1. Install Xcode Command Line Tools
```bash
xcode-select --install
```

### 2. Install CMake (if not present)
```bash
brew install cmake
```

### 3. Clone with submodules
```bash
git clone --recursive https://github.com/your-org/entrop-clap.git
cd entrop-clap
```

### 4. Add CLAP SDK as submodule (first time setup)
```bash
git submodule add https://github.com/free-audio/clap.git external/clap
git submodule add https://github.com/free-audio/clap-helpers.git external/clap-helpers
git submodule update --init --recursive
```

### 5. (Optional) Add JUCE for GUI
```bash
git submodule add https://github.com/juce-framework/JUCE.git external/JUCE
```

## Runtime Dependencies

| Dependency | Type | License | Install |
|---|---|---|---|
| **CLAP SDK** | Header-only C API | MIT | Git submodule |
| **clap-helpers** | Header-only C++ wrappers | MIT | Git submodule |
| **Apple Accelerate** | System framework | Apple | Ships with macOS |
| **JUCE** (optional) | GUI framework | GPL / Commercial | Git submodule |

> **No external runtime dependencies.** Everything is either header-only or ships with macOS.
> This is a hard constraint — see Developer Plan Section 10.

## Verification

Run these commands to verify your environment is ready:

```bash
# C++ compiler (must support C++20)
clang++ --version

# CMake
cmake --version

# Xcode tools
xcode-select -p

# Git
git --version

# Apple Accelerate (should list the framework)
ls /System/Library/Frameworks/Accelerate.framework

# Homebrew (optional but recommended)
brew --version
```

## Target Hosts for Testing

| DAW | Version | Priority |
|---|---|---|
| Bitwig Studio | 5.x | Primary |
| REAPER | 7.x | Primary |
| Ableton Live | 12+ (CLAP support) | Secondary |
