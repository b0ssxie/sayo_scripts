# AGENTS.md

## Repo purpose

Scripts for a programmable keyboard that runs a custom VM. Scripts are plain `.txt` files using a bespoke assembly-like language.

## Key resources

- **`readme.md`** — the complete reference manual: instruction set, register map, HID keycodes, architecture constraints, and examples. Read this before writing any script.
- Scripts live in subdirectories named after the game/application they target (e.g. `三角洲/`).

## Script language rules

The language has unusual constraints an agent will likely miss:

- **Instructions are binary-encoded bytecode.** The assembly syntax is just a human-readable representation — the keyboard firmware parses it. Opcodes, operand counts, and widths are fixed per instruction.
- **No HID hex codes in scripts.** Always use the named aliases defined in readme.md (e.g. `hidkey_A`, not `0x04`; `hidmedia_Mute`, not `08`). Multiple modifier keys use `|` notation.
- **Every key press/release must be followed by a delay** (at least 1ms, recommended 5–35ms) or the host PC may miss it.
- **Exit script before releasing keys = keys stay held.** Always `EXIT` at the end.
- **Random delays are never used alone.** Always pair a fixed delay with random: `SLEEP 10` + `SLEEP_RAND 20` for 10–30ms range.
- **Modifier + normal key combos need a small delay between them** (e.g. 16ms between Alt press and Tab press).
- **`RANDOM` register is read-only** — don't write to it.
- **Script loops forever by default** (sequential execution, only stops on `SLEEP`, `WAIT_*`, or `EXIT`). For looping behavior, use `RES` at the end, not infinite jumps.
- **Toggle mode vs jog mode:** default is toggle (press to start, press again to kill). Use `MODE_JOG` first for double-click or complex input handling.
- **`RES` = `JMP 0`** — restarts the script from the beginning.
- **Preserve the release pattern:** `PRESS → SLEEP → RELEASE → SLEEP` for every key operation.

## User intent interpretation

From `readme.md` §X.1, these are things users mean that agents might misinterpret:

- "一直按" (keep pressing) = repeated press-release cycles, NOT holding a key down.
- "间隔时长" (interval) = time between start of one cycle and start of the next, not time between two operations.
- Default press duration is 35ms unless user specifies. If user wants random press time, default to 30–60ms random range.
- Always add proportional random delays everywhere when user asks for randomness.
- Dead loops need `EXIT_IF_ANYKEY` to allow breaking out.
- Must distinguish between user's physical key vs the simulated key operations in script.

## Constraints the VM imposes

- Code space: typically 4KB (some devices 256B or 64KB). Keep scripts compact.
- RAM: minimum 128B. Prefer registers over RAM.
- 16× 32-bit registers per thread (R0–R15), 4× 8-bit V-registers (V0–V3) for key parameters.
- At least 4 global registers (GL_0–GL_3) shared across threads.
- Stack grows upward, only via PUSH/POP.
- New threads occupy R12–R15. Main thread exit kills all child threads.

## File conventions

- Scripts are `.txt` files with one instruction per line.
- Labels use `.loc_XXXX:` format (hex address) or descriptive names like `label_timeout:`.
- Directories are named after the target game/application in Chinese.
- No build system, no tests, no linting — just edit and upload to the keyboard.