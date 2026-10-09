# `meds_s1_abs_ctrl`

| | |
|---|---|
| **Status** | COMPLETE |
| **Owner** | [husnainjatoi](https://www.github.com/husnainjatoi) |
| **Backup** | [MuhammadYousaf79](https://github.com/MuhammadYousaf79) |
| **Project** | R-06 |
| **Spec** | SPEC §13, INTERFACES.md §7 |
| **Source** | `rtl/debug/meds_s1_abs_ctrl.sv` |
| **Testbench** | `verif/unit/tb_meds_s1_abs_ctrl.sv` |

## Purpose

FSM to manage abstract command execution, interacting with debug registers. It isolates the abstract command execution logic from the top-level debug module, ensuring proper sequencing, error trapping, and state recovery when interfacing with the core.

## Interface contract

| Signal | Dir | Width | Meaning | Contract |
|---|---|---|---|---|
| `clk_i` | in | 1 | clock | single domain |
| `rst_ni` | in | 1 | reset | async assert, sync de-assert |
| `cmd_en_i` | in | 1 | command enable | initiates command execution |
| `cmdtype_i` | in | 8 | command type | valid when `cmd_en_i` is high |
| `postexec_i` | in | 1 | post execution | traps to error if requested |
| `cmderr_status_i` | in | 1 | error status | waits for debugger to clear before recovering |
| `debug_halted_i` | in | 1 | core halted | core must be halted for register execution |
| `debug_reg_ready_i` | in | 1 | reg access ready | handshakes completion of register access |
| `aarpostincrement_i` | in | 1 | auto-increment | read on completion |
| `abs_en_o` | out | 1 | abstract enable | high during execution or error wait |
| `debug_reg_en_o` | out | 1 | reg access enable | high during register execution |
| `busy_o` | out | 1 | busy flag | high during active execution, low in error wait |
| `set_cmderr_o` | out | 1 | set command error | pulses when invalid command is requested |
| `inc_regno_o` | out | 1 | increment register | Mealy output on completion |

**Handshake:** Execution waits for `debug_reg_ready_i` before returning to IDLE.
**Latency:** Variable depending on core register access time.
**Backpressure:** Asserts `busy_o` to prevent new commands from being issued while executing.
**Reset state:** Enters `IDLE_E`, all outputs default to `0`.

## Parameters

| Parameter | Default | Legal range | Effect |
|---|---|---|---|
| None | | | |

## Behaviour

![Abstract Command Controller FSM](../../rtl/debug/abs_cmd_fsm.svg)

The FSM implements three states: `IDLE_E`, `EXEC_REG_E`, and `ERROR_WAIT_E`. It transitions from IDLE to EXEC_REG only if `cmdtype_i == 8'h0`, the core is halted, and `postexec` is not requested. If an unsupported command is requested, it traps to `ERROR_WAIT_E` and waits for `cmderr_status_i` to clear. `inc_regno_o` is driven as a Mealy output immediately upon `debug_reg_ready_i` assertion.

## Exceptions and errors

Raises `set_cmderr_o` if an invalid command type is issued or if a command is issued while the core is not correctly halted. The caller must clear the error status via the DMI register to allow the FSM to return to IDLE.

## Verification status

| Layer | Status | Where |
|---|---|---|
| Lint | | |
| Unit test | 10 checks | `verif/unit/tb_meds_s1_abs_ctrl.sv` |
| Co-simulation | not applicable | |
| Formal | | |

## Known limitations

Only `cmdtype_i == 8'h0` (register access) is currently supported; any other command traps directly to an error state.

## Open questions

None.