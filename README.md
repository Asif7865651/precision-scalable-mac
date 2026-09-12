# Scalable-Precision MAC Unit

A parameterized Multiply-Accumulate (MAC) unit in Verilog that supports **runtime-selectable
operand precision** — INT4, INT8, INT16, and INT32 — through a single `prec_sel` control input,
built and simulated in Xilinx Vivado.

## Why scalable precision?

A fixed-width MAC either wastes power/area on small operands or can't handle large ones. This
design lets the same hardware pack down to smaller signed operand widths (sign-extended back up
to the full datapath width) when full 32-bit precision isn't needed — useful in ML inference
accelerators where different layers/quantization levels need different precision.

## Architecture

```
         a[31:0], b[31:0], prec_sel[1:0]
                     |
                     v
           +-------------------+
           |   precision_ctrl  |   selects & sign-extends the active
           |                   |   operand width based on prec_sel
           +-------------------+
                     |
              a_eff, b_eff (sign-extended to MAX_W)
                     |
                     v
           +-------------------+
           |     mac_core       |  2-stage pipeline:
           |                     |   stage 1: a_eff * b_eff
           |                     |   stage 2: accumulate (if acc_en)
           +-------------------+
                     |
                     v
              y[2*MAX_W:0]  (signed accumulator output)
```

`prec_sel` encoding:

| `prec_sel` | Precision |
|---|---|
| `2'b00` | INT4  (4x4 -> sign-extended) |
| `2'b01` | INT8  (8x8 -> sign-extended) |
| `2'b10` | INT16 (16x16 -> sign-extended) |
| `2'b11` | INT32 (full width) |

## Repo layout

```
.
├── rtl/
│   ├── mac_top.v          # top-level: wires precision_ctrl + mac_core
│   ├── mac_core.v         # pipelined signed multiply-accumulate
│   └── precision_ctrl.v   # precision select / sign-extension mux
├── tb/
│   ├── mac_top_tb.v        # top-level testbench
│   └── mac_core_tb.v       # mac_core-only testbench
└── README.md
```

## Simulation results

Behavioral simulation was run in Vivado 2025.2 across fixed-precision and mixed-precision
(including signed) test vectors, covering:

- Fixed-precision MAC accumulation
- Full scalable-precision sweep (INT4 -> INT32)
- Signed-operand handling verification

## Tools used

- Xilinx Vivado 2025.2 (behavioral simulation, synthesis)

## Status

- [x] `precision_ctrl`: precision select + sign extension — implemented, simulated
- [x] `mac_core`: 2-stage pipelined MAC — implemented, simulated
- [x] `mac_top`: integration — implemented, simulated
- [ ] Synthesis / implementation results (add if you run these)
