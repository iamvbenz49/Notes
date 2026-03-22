# Varint Encoding in Databases — Notes

---

## The Problem

Standard integers (32-bit or 64-bit) always use their full space, even for tiny numbers.

- Storing `21` in a 64-bit integer wastes **59 bits**
- Most real-world values in databases are small — IDs, counters, offsets, deltas

Varint (Variable-length Integer Encoding) fixes this by using **only as many bytes as the number actually needs**.

---

## Where Varints Are Used

Almost every major database and protocol uses varints:

- **Redis**
- **Cassandra**
- **LevelDB / RocksDB**
- **MySQL**
- **Protobuf** (used in gRPC communication)

---

## How Varints Work

### Core Idea

Each byte is split into two parts:

```
[ 1 continuation bit | 7 data bits ]
```

- **Continuation bit = 1** → more bytes follow
- **Continuation bit = 0** → this is the last byte

Small numbers fit in 1 byte (0–127). Larger numbers spill into more bytes, but only as many as needed.

### Byte Capacity

| Bytes Used | Max Value |
|---|---|
| 1 byte | 127 |
| 2 bytes | 16,383 |
| 3 bytes | 2,097,151 |
| 4 bytes | 268,435,455 |

---

## Encoding Example: 292

Binary of 292:
```
100100100  (9 bits — too big for 1 byte)
```

Split into 7-bit groups (right to left):
```
0000010 | 0100100
```

Add continuation bits:
```
Byte 1:  1 0100100  (continuation = 1, more bytes follow)
Byte 2:  0 0000010  (continuation = 0, last byte)
```

Result: **292 stored in 2 bytes** instead of the usual 4 or 8.

---

## Decoding

Decoding is the exact reverse:

1. Read bytes one by one
2. Check the continuation bit — if `1`, keep reading; if `0`, stop
3. Strip the continuation bit from each byte, keeping the 7 data bits
4. Concatenate all data bits (in reverse order) to reconstruct the original number

### Example: Decoding back to 292

```
Read byte 1:  1 0100100  → continuation = 1, data = 0100100
Read byte 2:  0 0000010  → continuation = 0, data = 0000010

Concatenate (byte 2 first, byte 1 second):
0000010 | 0100100 = 100100100 = 292 ✅
```

---

## Trade-offs

| | Detail |
|---|---|
| **Gain** | Less disk space, smaller network payloads, faster transmission |
| **Loss** | Extra CPU cost to encode and decode (bit shifting and masking) |

Worth it in most cases — I/O and network are far more expensive than CPU in database workloads.

---

## Implementation (Golang)

```go
// Encode an int64 as varint bytes
func encodeInt64(n int64) []byte {
    var buf []byte
    for {
        b := byte(n & 0x7F) // take 7 bits
        n >>= 7
        if n != 0 {
            b |= 0x80 // set continuation bit
        }
        buf = append(buf, b)
        if n == 0 {
            break
        }
    }
    return buf
}

// Decode varint bytes back to int64
func decodeInt64(buf []byte) int64 {
    var result int64
    var shift uint
    for _, b := range buf {
        result |= int64(b&0x7F) << shift // extract 7 data bits
        if b&0x80 == 0 {                 // check continuation bit
            break
        }
        shift += 7
    }
    return result
}
```

**Key operations:**
- `& 0x7F` — masks out the top bit, keeping only the 7 data bits
- `| 0x80` — sets the continuation bit to signal more bytes follow
- `>>= 7` — shifts the number right by 7 to process the next chunk
- `<< shift` — places each 7-bit chunk in the correct position during decode
