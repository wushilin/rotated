# rotated
Pipe your standard out to this, and it will rotate for you.
It support rotate by size with max backup index

# Usage - Just pipe your output to this rotated

```bash
# Rotate to test.log and keep 20 backups, each file target size 100MiB
./my-program | rotated -o test.log -k 20 -s 100MiB
```

The default keep is 20 (files).

The default size limit is 100MiB.

You can override them with arguments

# Supported arguments

## Required output file name
`-out filename || --out filename || --out=filename || -o filename`

Specify the output file. 

Back file will named like `filename.1`, `filename.2`,..., up to `filename.<max-backups>`

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

`-s 100k` (100000)

`--size 100K` (102400)

`--size=1GiB` (1073741824)

`-s 45M` (45 MiB 47185920 bytes)


File will only be rotated after its size exceeded this number.

Actual file size may slightly exceed this number due to IO buffering.


## Optional Max backup index. Default: 20
`-k <number> || --keep <number> || --keep=<number> || -keep <number>` 

How many files to keep, excluding the current output file.

File beyond this will be deleted.

# Other notes
This file operates at binary mode. So it is guaranteed no bytes is lost in translation.

You can even use this as a split utility.

The following command will split the big file to smaller chunks. 
Earlier part will be written first.

`cat huge-file | ./rotated -o small-file-prefix -s 100MiB -k 9999999`
