# QPSK Costas Loop for Carrier Synchronization on AD9361 + ZedBoard

This repository contains the complete Vivado project for a **QPSK Costas loop** implemented in Verilog HDL for carrier phase and frequency synchronization. The design targets the **AD9361 RF transceiver + ZedBoard (Zynq-7020)** SDR platform.

## Overview

A Costas loop recovers the carrier from a suppressed-carrier modulated signal such as QPSK. The loop estimates the phase error between the incoming I/Q samples and a local oscillator, filters that error, and drives a numerically controlled oscillator (NCO) so the local carrier locks to the received one.

## Features

- QPSK Costas loop written in Verilog HDL (top module: `qpsk_costas_loop`)
- LUT-based sine/cosine NCO (`sine_lut.mem`, `cosine_lut.mem`)
- Parameterized proportional-integral loop filter (`Kp`, `Ki`) to tune loop bandwidth and convergence
- Fixed-point arithmetic suitable for FPGA implementation
- Python-generated I/Q test vectors and lookup tables
- Testbench (`tb_qpsk_costas_loop`) that streams I/Q data from `.mem` files to verify lock behavior

## Target Platform

| Item | Detail |
|---|---|
| Board | ZedBoard (`digilentinc.com:zedboard:part0:1.1`) |
| FPGA | Xilinx Zynq-7000 `xc7z020clg484-1` |
| RF front end | Analog Devices AD9361 |
| Tool | Vivado 2024.1 |

## Repository Structure

```
costas_verification/
├── costas_verification.xpr          # Vivado project file
├── costas_verification.srcs/
│   ├── sources_1/
│   │   ├── new/costas.v             # Costas loop design (qpsk_costas_loop)
│   │   └── imports/python/          # Memory files generated in Python
│   │       ├── sine_lut.mem
│   │       ├── cosine_lut.mem
│   │       ├── I_data*.mem          # In-phase test vectors
│   │       └── Q_data*.mem          # Quadrature test vectors
│   └── sim_1/
│       └── new/tb_costas.v          # Testbench (tb_qpsk_costas_loop)
└── README.md
```

## Parameters

| Parameter | Description |
|---|---|
| `Kp` | Proportional gain of the loop filter |
| `Ki` | Integral gain of the loop filter |

Increasing the gains widens the loop bandwidth and speeds up lock acquisition, at the cost of more noise and less stability. Tune them for your symbol rate and expected frequency offset.

## How to Run the Simulation

1. Open `costas_verification.xpr` in **Vivado 2024.1**.
2. Make sure the `.mem` files are listed under Design Sources, since the design and testbench read them at simulation time.
3. Set `tb_qpsk_costas_loop` as the simulation top module.
4. Click **Run Simulation > Run Behavioral Simulation**.
5. Run the simulation long enough for the loop to lock (the testbench finishes on its own, after roughly 4.27 ms of simulated time in the logged runs).
6. Inspect the I/Q outputs, phase error, and NCO signals in the waveform window.

Note: the testbench memory arrays are large, so Vivado may refuse to add them to the wave window. To raise the limit, run:

```tcl
set_property display_limit <new_limit> [current_wave_config]
```

## Test Data

The I/Q input files and sine/cosine lookup tables are generated with Python scripts and exported as `.mem` files. The project includes several data sets, such as `I_data_100.mem`, `I_data_500.mem`, and `I_data_R_64_1_0.mem` with matching `Q_data` files. Each has different test conditions.

## Synthesis and Implementation

1. Open the project in Vivado 2024.1.
2. Run **Synthesis**, then **Implementation**, then **Generate Bitstream** for the `xc7z020clg484-1` device.
3. Review the timing, utilization, and power reports.

## Results

Add your measured results here, for example:

- Lock time: _TBD_
- Steady-state phase error: _TBD_
- Supported symbol rate: _TBD_
- Resource utilization (LUTs / FFs / DSPs / BRAM): _TBD_

## Future Work

- On-board validation with live AD9361 RF data
- Integration with the Zynq processing system through AXI
- Symbol timing recovery ahead of the Costas loop
- Adaptive loop gains for faster acquisition

## Author

Saurav Kumar
