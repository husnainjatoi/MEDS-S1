# `meds_s1_sba_ctrl`

| | |
|---|---|
| **Status** | COMPLETE |
| **Owner** | [husnainjatoi](https://www.github.com/husnainjatoi) |
| **Backup** | [MuhammadYousaf79](https://github.com/MuhammadYousaf79) |
| **Project** | R-06 |
| **Spec** | SPEC §13, INTERFACES.md §7 |
| **Source** | `rtl/debug/meds_s1_sba_ctrl.sv` |
| **Testbench** | `verif/unit/tb_meds_s1_sba_ctrl.sv` |

## Purpose

FSM to manage System Bus Access (SBA) execution for the Debug Module. It bridges the 64-bit Debug Module domain to the 256-bit AXI4 crossbar backbone via an AXI Upsizer, supporting 8-bit, 16-bit, 32-bit, and 64-bit accesses.

## Interface contract

| Signal | Dir | Width | Meaning | Contract |
|---|---|---|---|---|
| `clk_i` | in | 1 | clock | single domain |
| `rst_ni` | in | 1 | reset | async assert, sync de-assert |
| `dmi_en_i` | in | 1 | DMI enable | high for active DMI request |
| `dmi_wr_en_i` | in | 1 | DMI write enable | determines read/write context |
| `dmi_rd_en_i` | in | 1 | DMI read enable | determines read/write context |
| `dmi_addr_i` | in | 8 | DMI address | targets SBA registers (0x38, 0x39, 0x3C) |
| `sbreadonaddr_q` | in | 1 | read on addr | config flag to auto-trigger reads |
| `sbreadondata_q` | in | 1 | read on data | config flag to auto-trigger reads |
| `sbautoincrement_q` | in | 1 | auto increment | flag to pulse `inc_sbaddress_o` |
| `clear_errors_i` | in | 1 | clear errors | recovers from `ERROR_HALT_E` |
| `sbbusy_o` | out | 1 | SBA busy | high when AXI transaction is active |
| `set_sberror_o` | out | 1 | set SBA error | pulses on AXI error response |
| `set_sbbusyerror_o`| out | 1 | set busy error | pulses if DMI request hits while busy |
| `inc_sbaddress_o` | out | 1 | increment addr | pulses upon successful transaction |
| `axi_*` (group) | in/out | - | AXI4 subset | handshake signals to the Upsizer |

**Handshake:** Standard AXI4 valid/ready handshaking handles the bus transaction (`axi_arready`, `axi_awready`, `axi_wready`).
**Latency:** Variable depending on the AXI4 backbone arbitration and memory response.
**Backpressure:** Asserts `sbbusy_o` during transactions. DMI requests arriving while busy trap to an error state.
**Reset state:** Enters `IDLE_E`, all outputs default to `0`.

## Parameters

| Parameter | Default | Legal range | Effect |
|---|---|---|---|
| None | | | |

## Behaviour

![System Bus Access Controller FSM](../../rtl/debug/sba_ctrl_fsm.svg)

The FSM implements four states: `IDLE_E`, `AXI_ADDR_DATA_E`, `AXI_RESP_E`, and `ERROR_HALT_E`. Transactions are initiated automatically based on writes to `SBDATA0` (0x3C) or configurations like `sbreadonaddr_q` / `sbreadondata_q`. During `AXI_ADDR_DATA_E` and `AXI_RESP_E`, `sbbusy_o` is asserted. Any concurrent DMI requests trigger a busy error. Upon a successful AXI response (`AXI_OKAY`), `inc_sbaddress_o` is driven if configured to auto-increment.

## Exceptions and errors

Raises `set_sberror_o` if the AXI transaction returns anything other than `AXI_OKAY` (e.g., `SLVERR`). Raises `set_sbbusyerror_o` if a DMI request arrives while the FSM is already processing a transaction. In both cases, the controller drops to `ERROR_HALT_E` and waits for `clear_errors_i`.

## Verification status

| Layer | Status | Where |
|---|---|---|
| Lint | | |
| Unit test | 18 checks | `verif/unit/tb_meds_s1_sba_ctrl.sv` |
| Co-simulation | not applicable | |
| Formal | | |

## Known limitations

128-bit accesses are not supported; data is mapped exclusively to `sbdata0` and `sbdata1`.

## Open questions

None.