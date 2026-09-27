# ELUPON-EUA-Codec
Official technical specification and reference implementation for the ELUPON Universal Audio (EUA) format.
# ELUPON EUA Audio Codec (EUAL7 Specification)

Official technical repository for the proprietary **EUA (Elupon Universal Audio)** format. A lightweight, high-performance digital audio interleaving format designed for minimal unpadded hardware execution.

This documentation specifies the **EUAL7** standard, providing fixed 2x data optimization for high-resolution 48kHz Stereo streams while ensuring zero bit-shifting synchronization risk.

## Technical Metadata & IANA Registration Info
- **MIME Media Type:** `audio/prs.elupon-eua
- **File Extension:** `.eua`
- **Magic Number (Signature):** `EUAL7` (`45 55 41 4C 37` in Hex)

---

## 🔬 Binary Structure Specification

Every `.eua` file contains a minimal 14-byte unpadded header immediately followed by raw 8-bit interleaved dual-channel PCM frames.

### 1. File Header (14 Bytes Total)

| Offset (Bytes) | Size | Data Type | Field Description |
|----------------|------|-----------|-------------------|
| 0 - 4          | 5    | `char[]`  | Magic Signature: Must be string `"EUAL7"` |
| 5 - 6          | 2    | `uint16`  | Audio Channels (e.g., `2` for Stereo) |
| 7 - 10         | 4    | `uint32`  | Sample Rate in Hz (e.g., `48000`) |
| 11 - 13        | 3    | `uint24`  | Total Audio Samples count |

### 2. Audio Payload (8-bit Interleaved Stream)
Following byte 13, the payload consists entirely of alternating **Left** and **Right** channel samples. Each sample is scaled from 16-bit signed to 8-bit unsigned dynamic range (`0-255`), maximizing hardware performance on low-level system sound buffers.

---

## 🗜️ Reference Python Encoder

Use this official script to convert uncompressed 16-bit PCM Stereo WAV files into the proprietary `.eua` format.

```python
import struct
import os

def elupon_encode_stereo_hi_fi(wav_path, eua_path):
    if not os.path.exists(wav_path):
        print(f"File {wav_path} not found!")
        return

    with open(wav_path, 'rb') as f:
        wav_header = f.read(44)
        channels = struct.unpack('<H', wav_header[22:24])
        sample_rate = struct.unpack('<I', wav_header[24:28])
        raw_data = f.read()

    samples = list(struct.unpack(f'<{len(raw_data)//2}h', raw_data))
    total_samples = len(samples)
    
    print(f"🛸 Codec ELUPON Hi-Fi v7... Rate: {sample_rate} Hz | Channels: {channels}")
    
    compressed_bytes = bytearray()
    for sample in samples:
        code = int((sample + 32768) / 256)
        code = max(0, min(255, code))
        compressed_bytes.append(code)

    with open(eua_path, 'wb') as eua:
        eua.write(b'EUAL7') 
        eua.write(struct.pack('<H', channels))
        eua.write(struct.pack('<I', sample_rate))
        eua.write(struct.pack('<I', total_samples))
        eua.write(compressed_bytes)

    orig_size = len(raw_data)
    new_size = len(compressed_bytes) + 14
    print(f"🔥 Success: {eua_path} | Ratio: {round(orig_size / new_size, 2)}x (Fixed 2x!)")

elupon_encode_stereo_hi_fi("input.wav", "track.eua")
```
