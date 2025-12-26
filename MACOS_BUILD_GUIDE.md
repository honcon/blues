# 🎮 macOS Build Guide

This guide explains how to compile the **Blues Brothers / Prehistorik 2** engine on macOS.

---

## 📋 Prerequisites

- macOS with Apple Silicon (M1/M2/M3) or Intel
- [Homebrew](https://brew.sh) installed
- Xcode Command Line Tools (`xcode-select --install`)

---

## 🛠️ Step 1: Install Dependencies

```bash
brew install sdl2 libmodplug binutils
```

| Package | Purpose |
|---------|---------|
| **sdl2** | Graphics, audio, input |
| **libmodplug** | MOD music playback |
| **binutils** | GNU linker tools (for `objcopy`) |

---

## 🛠️ Step 2: Create macOS Makefile

The original `Makefile` uses `objcopy --localize-hidden` which is **not compatible** with macOS Mach-O format.

Create a new file `Makefile.macos`:

```makefile
SDL_CFLAGS := `sdl2-config --cflags`
SDL_LIBS   := `sdl2-config --libs`

MODPLUG_CFLAGS ?= -I/opt/homebrew/opt/libmodplug/include
MODPLUG_LIBS ?= -L/opt/homebrew/opt/libmodplug/lib -lmodplug

BB := decode.c game.c level.c objects.c resource.c screen.c sound.c staticres.c tiles.c unpack.c
JA := game.c level.c resource.c screen.c sound.c staticres.c unpack.c
P2 := bosses.c game.c level.c monsters.c resource.c screen.c sound.c staticres.c unpack.c

BB_SRCS := $(foreach f,$(BB),bb/$f)
JA_SRCS := $(foreach f,$(JA),ja/$f)
P2_SRCS := $(foreach f,$(P2),p2/$f)
SRCS := $(BB_SRCS) $(JA_SRCS) $(P2_SRCS)
OBJS := $(SRCS:.c=.o)
DEPS := $(SRCS:.c=.d)

CPPFLAGS += -Wall -Wextra -Wno-unused-parameter -Wno-sign-compare -Wpedantic -MMD $(SDL_CFLAGS) $(MODPLUG_CFLAGS) -I. -g

all: blues

game_bb.o: CPPFLAGS += -fvisibility=hidden

game_bb.o: $(BB_SRCS:.c=.o)
	ld -r -o $@ $^

game_ja.o: CPPFLAGS += -fvisibility=hidden

game_ja.o: $(JA_SRCS:.c=.o)
	ld -r -o $@ $^

game_p2.o: CPPFLAGS += -fvisibility=hidden

game_p2.o: $(P2_SRCS:.c=.o)
	ld -r -o $@ $^

blues: main.o mixer.o sys_sdl2.o util.o game_bb.o game_ja.o game_p2.o
	$(CC) $(LDFLAGS) -Wl,-all_load -o $@ $^ $(SDL_LIBS) $(MODPLUG_LIBS)

clean:
	rm -f $(OBJS) $(DEPS) *.o *.d blues

-include $(DEPS)
```

> **Note for Intel Mac:** Change paths from `/opt/homebrew/` to `/usr/local/` for MODPLUG paths.

---

## 🛠️ Step 3: Compile

```bash
make -f Makefile.macos
```

To clean and rebuild:

```bash
make -f Makefile.macos clean && make -f Makefile.macos
```

You'll see many warnings - these are expected (legacy C style code).

---

## ✅ Verification

```bash
./blues
# Output: "No data files found"
```

If you see this message, the build succeeded!

---

## 🎮 Running the Game

### Download Demo Data

**Prehistorik 2 Demo:**
```bash
curl -L -o /tmp/pre2.zip "http://cd.textfiles.com/ccbcurrsh1/demos/pre2.zip"
unzip /tmp/pre2.zip -d /tmp/pre2_data
```

**Blues Brothers Demo:**
```bash
# Download from: https://archive.org/details/TheBluesBrothers_1020
```

### Running

```bash
./blues --datapath=/tmp/pre2_data/
```

Or place data files in the same directory as the executable and run without arguments:
```bash
./blues
```

### Command Line Options

| Option | Description |
|--------|-------------|
| `--datapath=PATH` | Directory containing data files |
| `--level=NUM` | Start at level |
| `--fullscreen` | Enable fullscreen |
| `--scale=N` | Scale factor (default: 2) |
| `--cheats=MASK` | Cheats: 1=no-hit, 2=∞ lives, 4=∞ energy |

---

## 🔧 Troubleshooting

### "sdl2-config: command not found"
```bash
brew install sdl2
```

### "libmodplug/modplug.h file not found"
```bash
brew install libmodplug
# Ensure paths in Makefile.macos are correct
```

### "objcopy: No such file or directory"
Use `Makefile.macos` which bypasses this step.

### Duplicate symbols error
This occurs if you try to link .o files directly without `ld -r`. Use `Makefile.macos`.

---

## 📁 Required Data Files

| Game | Files |
|------|-------|
| Blues Brothers | `*.BIN, *.CK1, *.CK2, *.SQL, *.SQV, *.SQZ` |
| Jukebox Adventure | `*.EAT, *.MOD` |
| Prehistorik 2 | `*.SQZ, *.TRK` |

The program auto-detects which game to run based on the files present.

---

## 🎯 TL;DR - Quick Install

```bash
# 1. Dependencies
brew install sdl2 libmodplug

# 2. Build
make -f Makefile.macos

# 3. Run
./blues --datapath=/path/to/game/data
```

---

*Guide created for macOS Sequoia / Apple Silicon*
