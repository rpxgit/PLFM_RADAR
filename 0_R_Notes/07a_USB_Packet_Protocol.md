---
type: reverse-engineering-note
status: active
domain: protocol
confidence: mixed
canonical: true
---

# USB Packet Protocol

> **Related notes:** [[07_Communication_Protocols]] · [[06_Software_GUI]]
> **Status:** Read-only Forensic Snapshot

---

## 1. USB Transport Architecture

The system communicates between the Host PC and the FPGA via an **FTDI FT2232H** chip operating in **245 Synchronous FIFO mode**. 
There is no CDC/Virtual COM port implementation on the high-speed data path. The host utilizes raw USB Bulk endpoints to push and pull data from the FTDI chip.

| Interface | Transfer Type | Max Packet | Purpose | Evidence |
| --------- | ------------- | ---------- | ------- | -------- |
| Channel A | Bulk (Synchronous FIFO) | 65536 bytes (chunking) | Full duplex Host ↔ FPGA | `radar_protocol.py: FT2232HConnection` |

---

## 2. Host → Device Command Protocol

The host configuration interface uses a fixed 4-byte (32-bit) payload. Commands are stateless and act as direct memory-mapped register writes into the FPGA logic.

### 2.1. Command Packet Layout

| Offset | Size | Field | Type | Endianness | Meaning | Evidence |
| -----: | ---: | ----- | ---- | ---------- | ------- | -------- |
| 0 | 1 | `Opcode` | Unsigned Int | Big-Endian | Action/Register ID | `radar_protocol.py: build_command()` |
| 1 | 1 | `Address` | Unsigned Int | Big-Endian | Sub-address (unused/0x00) | `radar_protocol.py: build_command()` |
| 2 | 2 | `Value` | Unsigned Int | Big-Endian | Data payload | `radar_protocol.py: build_command()` |

### 2.2. Command Catalogue (Opcode Map)

*Note: Opcodes strictly match `usb_cmd_opcode` cases in the Verilog code.*

| Command ID | Name | Hardware Effect | Confidence |
| ---------- | ---- | --------------- | ---------- |
| `0x01` | RADAR_MODE | Sets 2-bit radar operating mode. | Confirmed |
| `0x02` | TRIGGER_PULSE | Triggers a one-shot radar sweep. | Confirmed |
| `0x03` | DETECT_THRESHOLD | Sets 16-bit CFAR minimum threshold. | Confirmed |
| `0x10` | LONG_CHIRP | Sets long chirp duration (cycles). | Confirmed |
| `0x24` | CFAR_MODE | Selects CFAR algorithm type. | Confirmed |
| `0x28` | AGC_ENABLE | Enables/disables digital AGC in FPGA. | Confirmed |
| `0x30` | SELF_TEST_TRIGGER | Initiates FPGA internal diagnostic. | Confirmed |
| `0xFF` | STATUS_REQUEST | Requests a 26-byte status response frame. | Confirmed |

---

## 3. Device → Host Data Protocol

The FPGA streams two distinct packet types back to the host: **Radar Data Frames** and **Status Responses**. Both use a simple start/stop framing byte mechanism.

### 3.1. Radar Data Frame (11 Bytes)

Streamed continuously at high speed during acquisitions.

| Offset | Size | Field | Type | Endianness | Meaning |
| -----: | ---: | ----- | ---- | ---------- | ------- |
| 0 | 1 | `Header` | Magic | N/A | Always `0xAA` |
| 1 | 2 | `Range Q` | Signed Int16 | Big-Endian | IQ Range Data |
| 3 | 2 | `Range I` | Signed Int16 | Big-Endian | IQ Range Data |
| 5 | 2 | `Doppler I`| Signed Int16 | Big-Endian | IQ Doppler Data |
| 7 | 2 | `Doppler Q`| Signed Int16 | Big-Endian | IQ Doppler Data |
| 9 | 1 | `Flags` | Bitfield | N/A | Bit [7]: Frame Start. Bit [0]: CFAR Detection. |
| 10 | 1 | `Footer` | Magic | N/A | Always `0x55` |

### 3.2. Status Response (26 Bytes)

Returned only in response to a `0xFF` STATUS_REQUEST command.

| Offset | Size | Field | Type | Endianness | Meaning |
| -----: | ---: | ----- | ---- | ---------- | ------- |
| 0 | 1 | `Header` | Magic | N/A | Always `0xBB` |
| 1 | 4 | `Word 0` | Bitfield | Big-Endian | `0xFF` + Mode + Stream Ctrl + Threshold |
| 5 | 4 | `Word 1` | Bitfield | Big-Endian | Long Chirp/Listen cycles |
| 9 | 4 | `Word 2` | Bitfield | Big-Endian | Guard/Short Chirp cycles |
| 13 | 4 | `Word 3` | Bitfield | Big-Endian | Short Listen cycles + Chirps/Elev |
| 17 | 4 | `Word 4` | Bitfield | Big-Endian | AGC Gain/Saturation metrics + Range Mode |
| 21 | 4 | `Word 5` | Bitfield | Big-Endian | Self-Test Detail & Flags |
| 25 | 1 | `Footer` | Magic | N/A | Always `0x55` |

---

## 4. Framing & Recovery Behavior

1.  **Alignment Search:** The Python parser processes incoming 64KB USB chunks linearly. It locates the `0xAA` (Data) or `0xBB` (Status) headers, looks ahead by the known packet size, and verifies the `0x55` footer.
2.  **Invalid Packet Handling:** If the footer is missing or invalid, the parser discards the header and advances by 1 byte to resynchronize the stream.
3.  **Buffer Overruns:** If the GUI processing thread blocks the `RadarAcquisition` polling thread, the FT2232H hardware FIFO will fill up (4KB on-chip), eventually causing the FPGA to stall or drop packets.

---

## 5. Unknowns / Required Investigation

| ID | Unknown | Impact | Next Steps |
| -- | ------- | ------ | ---------- |
| USB-01 | MCU Native USB | Does the MCU have an independent USB CDC connection (`USBHandler.cpp`) used for debugging or calibration, independent of the primary FTDI data plane? | Analyze `main.cpp` for `MX_USB_DEVICE_Init()` and `CDC_Receive_FS()` handlers. |


## Related Notes

- [[07_Communication_Protocols]]
- [[04b_USB_Interface]]
- [[06_Software_GUI]]
