# rotated
I swallow your stdout/stderr, and put them into the log files, and rotate for you.

I can also add timestamp to your log lines.

# Behavior
1. Guaranteed size not exceeding the limit for each file. Unless limit is set to a number that is smaller than the longest line (256KiB).

2. Lines exceeded 256KiB (262144 bytes) might be split into 2 files (readline is capped at 256KiB).

3. File rotation is checked pre-write. If a write would have caused a file to exceed the size, it will be rotated before writing.

4. When restarting and existing log file is found, it will be appended, however, if it is exceeding the limit, it will be rotated immediately.

5. If you enabled dated (default false), it will add timestamp for you too. (Enable with `-d || --dated || -dated`). If enabled, logging `hello, world` will write `[Fri Sep  2 13:44:03 2022] hello, world` to output file.

# Usage - Just pipe your output to this rotated

```bash
# Rotate to test.log and keep 20 backups, 
# each file target size 100MiB, 
# and do prepend timestamp for each line
./my-program | rotated -out test.log -keep 20 -size 100MiB -d

# Rotate to test.log and keep 10 backups, 
# each file target size 10000KiB, 
# and do not prepend timestamp for each line
./my-program | rotated -out test.log -keep 10 -size 10,000KiB
```

The default keep is 20 (files).

The default size limit is 100MiB.

The default flag for prepend timestamp is `No`

You can override them with arguments

# Supported arguments

## Required output file name
`-out filename || --out filename || --out=filename || -o filename`

Specify the output file. 

Backup file will named like `filename.1`, `filename.2`,..., up to `filename.<max-backups>`

## Optional file size limit. Default: 100MiB
`-size expr || --size=expr || -s expr || --size expr`

Where expr = number + [unit] with unit is optional.

When unit is omitted, unit is raw bytes.

Example: `100K`, `1024MiB`, `9,000,000`, `4_000_000GiB`

Valid units: 
```
"k" => 1000,
"K" => 1024,
"m" => 1000000,
"M" => 1048576,
"g" => 1000000000,
"G" => 1073741824,
"kB" => 1000,
"KB" => 1024,
"KiB" => 1024,
"mB" => 1000000,
"MB" => 1048576,
"MiB" => 1048576,
"gB" => 1000000000,
"GB" => 1073741824,
"GiB" => 1073741824
```

Valid usage example:

`-s 100k` (100000 bytes)

`--size 100K` (102400 bytes)

`--size=1GiB` (1073741824 bytes)

`-s 45M` (45 MiB 47185920 bytes)

Actual size of the file is always smaller or equal to the size limit,
unless the limit is smaller than 256KiB and a single line in logs exceeded this limit.


NOTE: If a line is longer than 256KiB (262144 bytes), it may be split into 2 different files.

## Optional Max backup index. Default: 20
`-k <number> || --keep <number> || --keep=<number> || -keep <number>` 

How many files to keep, excluding the current output file.

File beyond this will be untouched (if exists)

For example, you have test.log.25 and your max backup index is 24, 

then test.log.25 will never be deleted by this program.

## Optional Timestamp logging. Default: No
`-d || --dated || -dated`

Enable the timestamp logging. Timestamp will prefix a line with a date like `[Fri Sep  2 13:44:03 2022] `.

## Optional Enable Buffering. Default: No
`-b | --buffered || -buffered`

Enable buffering. May increase throughput, but you may not see you outlines immediately.

# Other notes
This file operates at binary mode. So it is guaranteed no bytes is lost in translation.

You can even use this as a split utility.

The following command will split the big file to smaller chunks. 

Earlier part will be written first.

`cat huge-file | ./rotated -o small-file-prefix -s 100MiB -k 9999999`

If you enabled dated prefix, it will not work as you expect.


## Error behavior
If the write fail, the program will exit. Common failure case is, disk full.

When the program exit, typically your upstream program will quit too (due to SIGPIPE).

You should test the behavior of your program in event of disk full.

