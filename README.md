# sfn protocol

sfn is a lightweight protocol for sending files over a TCP connection. It is designed to be simple, portable, and easy to implement on new platforms. The most typical use case for sfn would be sending a bunch of files over a LAN where maximum speed is required, although it works just as well over the internet with a bit of port forwarding.

### Protocol revisions

L1 is the most basic one, L3 has checksum support, L4 has faster checksum support, L5 has directories, executable flag, and checksums are now optional. They are all backward-compatible: every new revision includes support for existing opcodes all the way back to L1.

### Structure

Protocol TCP stream consists of chunks, sent one after another, each started by chunk opcode. Each client **must** send `0x02` (DONE) when it doesn't intend to send any more chunks.

|           |     |           |               |
| --------- | --- | --------- | ------------- |
| [Chunk 1] | ... | [Chunk N] | `0x02` (DONE) |

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

### Choosing a default opcode

At the time of writing, most sfn implementations support L4, so it currently makes sense to implement L5 and then default to FILE_WITH_MD5 with options to upgrade to FILE_L5 / downgrade to FILE.

# Plans for future versions

* None yet, L5 seems good enough. Maybe parallel transfers?
