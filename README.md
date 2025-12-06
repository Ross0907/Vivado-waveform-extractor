<img src="images/logo.svg" alt="Logo" width="1000">

# Vivado Waveform Extractor

Extract simulation waveform data from Xilinx Vivado XSim to VCD, CSV, JSON, or Excel formats.

## Quick Start

### 1. Generate VCD from Vivado

In Vivado: `Tools` → `Run Tcl Script` → select `extract_waveform.tcl` (while simulation window is open)

**Auto-logging (recommended):** Just use normal `run` commands - VCD captures automatically!
```tcl
run 100us              # Auto-logs to waveform.vcd
run -all               # Works with testbenches too
```

**Manual capture:** For more control (restarts simulation, applies forces)
```tcl
capture "all"          # For testbench (runs until $finish)
capture "100us"        # For manual testing
```

The VCD file is saved to `vcd_output/` next to the script.

### 2. Convert VCD to Other Formats (Optional)

```bash
python vcd_converter.py                      # GUI
python vcd_converter.py waveform.vcd --json  # CLI
```
## GUI

![VCD Converter GUI](images/gui_screenshot.png)

## Requirements

**Tcl Script:** Xilinx Vivado 2020.x+ with active XSim simulation

**Python Converter:** Python 3.6+ (tkinter included). Excel export auto-installs `openpyxl` on first use.

## Project Structure

```
├── extract_waveform.tcl    # Tcl script for Vivado XSim
├── vcd_converter.py        # Python converter (CLI + GUI)
├── vcd_converter.pyw       # GUI only (double-click, no console window)
├── vcd_output/             # VCD files (auto-created)
└── converted_output/       # CSV/JSON/Excel (auto-created)
```

---

## Command Reference

### Tcl Commands (Vivado)

| Command | Description |
|---------|-------------|
| `capture "<time>"` | Restart, apply forces, run for `<time>` duration save VCD |
| `capture "all"` | Run until testbench `$finish` |
| `autolog on/off` | Enable/disable auto-logging |
| `stop_auto_log` | Save and close current VCD |
| `force /path hex FF` | Force signal (remembered across restarts) |
| `show_forces` | List recorded forces |
| `clear_forces` | Clear all forces |
| `signals` | List all signals |
| `snapshot` | Export current values to CSV |

### Python CLI

```
python vcd_converter.py <input.vcd> [-o output] [--csv|--json|--excel] [--hex|--int|--signed|--smag|--bin] [--us|--ns|--ps]
```

| Option | Description |
|--------|-------------|
| `-o <file>` | Output path |
| `--csv/--json/--excel` | Output format (default: csv) |
| `--hex` | Values as hexadecimal (default) |
| `--int` | Values as unsigned integers |
| `--signed` | Values as signed two's complement |
| `--smag` | Values as signed magnitude |
| `--bin` | Values as binary strings |
| `--us/--ns/--ps` | Time unit (default: us) |

> **Excel Graphing Tip:** Use `--int`, `--signed`, or `--smag` for Excel export if you want to create graphs. These formats store actual numbers. Hex and binary are stored as text (to preserve formatting like leading zeros) and cannot be graphed directly.

---

## Theory & Implementation

### Why VCD?

Vivado XSim supports **VCD (Value Change Dump)**, an IEEE standard (IEEE 1364) for waveform data. VCD is event-driven: it only records timestamps when signals *change*, making it compact.

### How the Tcl Script Works

The script uses three XSim commands:
- `open_vcd <file>` - Opens VCD file for writing
- `log_vcd *` - Registers all signals for logging (must be called *before* running)
- `close_vcd` - Finalizes the file

**Auto-logging** hooks into the `run` command to start VCD capture automatically. The `capture` command provides more control: it restarts simulation, reapplies any forces, then captures.

### VCD Format

```
$timescale 1ps $end           ← Timestamps in picoseconds
$var wire 8 " data [7:0] $end ← Signal declaration (ID=", 8-bit)
#0                            ← Time = 0
b00000000 "                   ← data = 0
#5000                         ← Time = 5ns
b00000001 "                   ← data = 1
```

### Python Converter

The converter parses VCD in two passes:
1. **Header** - Extract signal declarations (name, width, ID mapping)
2. **Values** - Track changes over time, build complete timeline

Since VCD only records *changes*, the converter maintains current state and outputs complete snapshots at each timestamp.

---

## Troubleshooting

- **No signals in VCD:** Run simulation at least once before loading the script.
- **Empty VCD file:** Ensure simulation ran. Check write permissions.
- **Forces not applied:** Use exact signal paths. Run `signals` to see available paths.

## License

MIT
