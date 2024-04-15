# sfn protocol

Protocol TCP stream consists of chunks, one after another, each started by chunk opcode. Each client **must** send `0x02` (DONE) when it doesn't intend to send chunks anymore.

|           |     |           |               |
| --------- | --- | --------- | ------------- |
| [Chunk 1] | ... | [Chunk N] | `0x02` (DONE) |

### Protocol revisions

L1 is the most basic one, L3 has checksum support, L4 has faster checksum support. They are backward-compatible: every new revision includes support for existing opcodes all the way back to L1.

### Opcodes

Protocol revision|Opcode|Name|Description
-----------------|------|----|-----------
L1 | `0x01` | FILE |
L1 | `0x02` | DONE | Signals that no more opcodes will be sent.
L3 | `0x03` | MD5_WITH_FILE | _(outdated)_
L4 | `0x04` | FILE_WITH_MD5 |
L5 | `0x05` | FILE_L5 |

When an unknown opcode is encountered, implementations **should:**

1) show a warning to the user;
2) stop reading incoming data (but finish sending files if there are any left).

### FILE

|        |                                                      |                               |               |
| ------ | ---------------------------------------------------- | ----------------------------- | ------------- |
| `0x01` | Filename (UTF-8),<br />terminated by `\n` (LF, 0x0A) | Size (64 bits, little endian) | File contents |


### MD5_WITH_FILE

* Outdated: checksum needs to be calculated before start of transmission, slowing things down

```
OPCODE:   0x03
FILENAME: utf-8 string terminated by \n
FILESIZE: 64-bit unsigned integer, little endian
MD5:      ascii hexadecimal string, terminated by \n
DATA:     exactly FILESIZE bytes
```

### FILE_WITH_MD5

* Faster checksumming (can be done in parallel with transmission)

```
OPCODE:   0x04
FILENAME: utf-8 string terminated by \n
FILESIZE: 64-bit unsigned integer, little endian
DATA:     exactly FILESIZE bytes
MD5:      ascii hexadecimal string, terminated by \n
```


### FILE_L5

* Fast checksumming, also it's optional now
* Directories (both sides can choose to ignore them to reduce codebase size)
* Executable flag, also optional

```
OPCODE:        0x05
FILENAME:      utf-8 string terminated by \n
FILESIZE:      64-bit unsigned integer, little endian
FILEPATH:      utf-8 string terminated by \n, may be empty, separator "/", examples: "", "folder1", "folder1/subfolder1"
IS_EXECUTABLE: one byte, 1 if yes, 0 if no or don't care
DATA:          exactly FILESIZE bytes
MD5:           ascii hexadecimal string (empty if not implemented by sender) terminated by \n
```

A file **should** be considered executable if at least one `x` is present in the Unix `rwx rwx rwx` tuple. On Windows, this field **should** be ignored.


# Plans for future versions

* None yet, L5 seems good enough. Maybe parallel transfers?
