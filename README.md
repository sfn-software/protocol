

#### 0x01 (`FILE` chunk)


#### 0x03 (`FILE_WITH_MD5` chunk)

|        |                                             |                               |                                                       |               |
| ------ | ------------------------------------------- | ----------------------------- | ----------------------------------------------------- | ------------- |
| `0x03` | Filename (UTF-8),<br />end with `0x0A` (LF) | Size (64 bits, little endian) | MD5 sum (ASCII hex string),<br />end with `0x0A` (LF) | File contents |


















# sfn protocol

Protocol TCP stream consists of chunks, one after another, each started by chunk opcode. Each client **must** send `0x02` (DONE) when it doesn't intend to send chunks anymore.

|           |     |           |               |
| --------- | --- | --------- | ------------- |
| [Chunk 1] | ... | [Chunk N] | `0x02` (DONE) |

## Protocol revisions

L1 is the most basic one, L3 has checksum support, L4 has faster checksum support. They are backward-compatible: every new revision includes support for existing opcodes all the way back to L1.

## Opcodes

Protocol revision|Opcode|Name|Description
-----------------|------|----|-----------
L1 | `0x01` | FILE |
L1 | `0x02` | DONE | Signals that no more opcodes will be sent.
L3 | `0x03` | MD5_WITH_FILE | _(outdated)_
L4 | `0x04` | FILE_WITH_MD5 |

When an unknown opcode is encountered, implementations are expected to:

1) show a warning to the user;
2) stop reading incoming data (but finish sending files if there are any left).

## FILE

|        |                                                      |                               |               |
| ------ | ---------------------------------------------------- | ----------------------------- | ------------- |
| `0x01` | Filename (UTF-8),<br />terminated by `\n` (LF, 0x0A) | Size (64 bits, little endian) | File contents |


## MD5_WITH_FILE

```
OPCODE:   0x03
FILENAME: utf-8 string terminated by \n
FILESIZE: 64-bit unsigned integer, little endian
MD5:      ascii hexadecimal string, terminated by \n
DATA:     exactly FILESIZE bytes
```

## FILE_WITH_MD5

```
OPCODE:   0x04
FILENAME: utf-8 string terminated by \n
FILESIZE: 64-bit unsigned integer, little endian
DATA:     exactly FILESIZE bytes
MD5:      ascii hexadecimal string, terminated by \n
```
