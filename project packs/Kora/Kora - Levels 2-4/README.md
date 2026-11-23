# Kora Workplace Supply Ltd. - Free Project Pack

Analyze Financial, People and Operations data across four years (2022–2025). Choose one of two business problems in each of the three project guides. The same synthetic dataset supports SQL Server, PostgreSQL, MySQL and SQLite. Reviewer solutions are not included.

- [Full project guide](Guides/Kora-Workplace-Supply-Free-Project-Pack.pdf) and [editable text](Guides/Kora-Workplace-Supply-Free-Project-Pack.md)
- [Database setup guide](Guides/Kora-Database-Setup-Guide.md)
- [Full data dictionary](Reference/Data_Dictionary.md) and [CSV dictionary](Reference/Data_Dictionary.csv)
- [Database schema diagrams](Reference/Database_Schema.pdf) and [all relationships](Reference/Relationships.csv)
- [Financial guide](Guides/Kora-Financial-Analysis-Free-Project-Guide.md), [People guide](Guides/Kora-People-Analysis-Free-Project-Guide.md), [Operations guide](Guides/Kora-Operations-Free-Project-Guide.md)

## Download the complete pack

GitHub's upload path for this pack requires four archive parts. Download all four `.part01`–`.part04` files from this folder, then combine them in numerical order. On macOS/Linux:

```bash
cat Kora-Levels-2-4-Four-Database-Pack.zip.part0{1,2,3,4} > Kora-Levels-2-4-Four-Database-Pack.zip
sha256sum -c Archive_SHA256.txt
unzip Kora-Levels-2-4-Four-Database-Pack.zip
```

On Windows PowerShell:

```powershell
$parts = 1..4 | ForEach-Object { 'Kora-Levels-2-4-Four-Database-Pack.zip.part{0:D2}' -f $_ }
$dest = [IO.File]::Create('Kora-Levels-2-4-Four-Database-Pack.zip')
try { foreach ($part in $parts) { $source = [IO.File]::OpenRead($part); try { $source.CopyTo($dest) } finally { $source.Dispose() } } } finally { $dest.Dispose() }
Get-FileHash Kora-Levels-2-4-Four-Database-Pack.zip -Algorithm SHA256
```

Compare the Windows hash with `Archive_SHA256.txt`, then extract the ZIP. The archive contains all CSVs, four engine loaders, two ready SQLite database files, guides, schema and SQL starters. SQLite passed row-count and integrity checks; live loading on the other three engines still needs local validation.
