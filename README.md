# useful_unix_commands
## Based on my bioinformatics attempts

Extract sequence from fastq file given read name

```gunzip -cd all_gac_reads.fastq.gz | awk '/read_id/{getline; print; exit}'```

- gunzip if file is compressed

- make sure to have -cd options on gunzip!! Very important.

- ' ' starts a program in awk

- / / denotes a character search in awk

- replace read_id with your read id

- exit stops scanning the file and exits once the first instance of your command gets printed

- getline just eats the line and goes on to the next.
    - So ie if you want like the 2nd line after (ie. grep -A 2 equivalent, do awk '/blah/{getline; getline; print; exit}'
