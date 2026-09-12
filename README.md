# Scalable-Precision MAC Unit

A parameterized Multiply-Accumulate (MAC) unit in Verilog that supports **runtime-selectable
operand precision** — INT4, INT8, INT16, and INT32 — through a single `prec\\\\\\\_sel` control input,
built and simulated in Xilinx Vivado.

## Why scalable precision?

A fixed-width MAC either wastes power/area on small operands or can't handle large ones. This
design lets the same hardware pack down to smaller signed operand widths (sign-extended back up
to the full datapath width) when full 32-bit precision isn't needed — useful in ML inference
accelerators where different layers/quantization levels need different precision.

## Architecture

```
         a\\\\\\\[31:0], b\\\\\\\[31:0], prec\\\\\\\_sel\\\\\\\[1:0]
                     |
                     v
           +-------------------+
           |   precision\\\\\\\_ctrl  |   selects \\\\\\\& sign-extends the active
           |                   |   operand width based on prec\\\\\\\_sel
           +-------------------+
                     |
              a\\\\\\\_eff, b\\\\\\\_eff (sign-extended to MAX\\\\\\\_W)
                     |
                     v
           +-------------------+
           |     mac\\\\\\\_core       |  2-stage pipeline:
           |                     |   stage 1: a\\\\\\\_eff \\\\\\\* b\\\\\\\_eff
           |                     |   stage 2: accumulate (if acc\\\\\\\_en)
           +-------------------+
                     |
                     v
              y\\\\\\\[2\\\\\\\*MAX\\\\\\\_W:0]  (signed accumulator output)
```

`prec\\\\\\\_sel` encoding:

|`prec\\\\\\\_sel`|Precision|
|-|-|
|`2'b00`|INT4  (4x4 -> sign-extended)|
|`2'b01`|INT8  (8x8 -> sign-extended)|
|`2'b10`|INT16 (16x16 -> sign-extended)|
|`2'b11`|INT32 (full width)|

## Repo layout

```
.
├── rtl/
│   ├── mac\\\\\\\_top.v          # top-level: wires precision\\\\\\\_ctrl + mac\\\\\\\_core
│   ├── mac\\\\\\\_core.v         # pipelined signed multiply-accumulate
│   └── precision\\\\\\\_ctrl.v   # precision select / sign-extension mux
├── tb/
│   ├── mac\\\\\\\_top\\\\\\\_tb.v        # top-level testbench
│   └── mac\\\\\\\_core\\\\\\\_tb.v       # mac\\\\\\\_core-only testbench
└── README.md
```

## Simulation results

Behavioral simulation was run in Vivado 2025.2 across fixed-precision and mixed-precision
(including signed) test vectors, covering:

* Fixed-precision MAC accumulation
* Full scalable-precision sweep (INT4 -> INT32)
* Signed-operand handling verification

## Tools used

* Xilinx Vivado 2025.2 (behavioral simulation, synthesis)

## Status

* \[x] `precision\\\\\\\_ctrl`: precision select + sign extension — implemented, simulated
* \[x] `mac\\\\\\\_core`: 2-stage pipelined MAC — implemented, simulated
* \[x] `mac\\\\\\\_top`: integration — implemented, simulated

