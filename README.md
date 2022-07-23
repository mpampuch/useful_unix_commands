# Useful Unix Commands

# How to work with and free up ports


Things that I know work on Unix

```
# find out process ID of process using a specific port
fuser PORT_OF_INTEREST_HERE/tcp 

# find out which user started that process
ps -o user= -p PROCESS_ID_HERE

# terminate process and free up port 
# (obviously only do this if the process belongs to you. Don't mess with anyone elses work)
kill PROCESS_ID_HERE
```

Tricks that worked I know work on Mac (https://askubuntu.com/questions/447820/ssh-l-error-bind-address-already-in-use)

```
lsof -ti:5901 | xargs kill -9
# lsof -ti:5901 to find whatever is using port 5901.
# Pass the whole thing to kill -9 to kill whatever was using port 5901.
```

# Managing Processes

- If you have a script that executes multiple commands (e.g. my `dl_mult_sras.sh` script), you can't use `top` to kill the process because it'll just kill the sub process (in this case a bunch of fastq-dump commands)
```
# dl_mult_sras.sh outputs
fastq-dump --gzip --skip-technical --readids --read-filter pass --dumpbase --split-3 --clip --outdir /Volumes/ubdata/mpampuch/diatomicbase_data/files_sorted/transcriptomics_data/cruz_de_carvalho_2016_PRJNA278661 SRR1917245
fastq-dump --gzip --skip-technical --readids --read-filter pass --dumpbase --split-3 --clip --outdir /Volumes/ubdata/mpampuch/diatomicbase_data/files_sorted/transcriptomics_data/cruz_de_carvalho_2016_PRJNA278661 SRR1917246
fastq-dump --gzip --skip-technical --readids --read-filter pass --dumpbase --split-3 --clip --outdir /Volumes/ubdata/mpampuch/diatomicbase_data/files_sorted/transcriptomics_data/cruz_de_carvalho_2016_PRJNA278661 SRR1917247
fastq-dump --gzip --skip-technical --readids --read-filter pass --dumpbase --split-3 --clip --outdir /Volumes/ubdata/mpampuch/diatomicbase_data/files_sorted/transcriptomics_data/cruz_de_carvalho_2016_PRJNA278661 SRR1917248
fastq-dump --gzip --skip-technical --readids --read-filter pass --dumpbase --split-3 --clip --outdir /Volumes/ubdata/mpampuch/diatomicbase_data/files_sorted/transcriptomics_data/cruz_de_carvalho_2016_PRJNA278661 SRR1917249
fastq-dump --gzip --skip-technical --readids --read-filter pass --dumpbase --split-3 --clip --outdir /Volumes/ubdata/mpampuch/diatomicbase_data/files_sorted/transcriptomics_data/cruz_de_carvalho_2016_PRJNA278661 SRR1917250
fastq-dump --gzip --skip-technical --readids --read-filter pass --dumpbase --split-3 --clip --outdir /Volumes/ubdata/mpampuch/diatomicbase_data/files_sorted/transcriptomics_data/cruz_de_carvalho_2016_PRJNA278661 SRR1917241
fastq-dump --gzip --skip-technical --readids --read-filter pass --dumpbase --split-3 --clip --outdir /Volumes/ubdata/mpampuch/diatomicbase_data/files_sorted/transcriptomics_data/cruz_de_carvalho_2016_PRJNA278661 SRR1917242
# etc
```
- if you kill the process with `top` or `ctrl + C`, it'll just move onto the next script
- to kill all the commands, you need to check the process tree with `pstree` and `pgrep` and kill the root process
```
$ pstree -ps `pgrep dl_mult_sras.sh`
systemd(1)───bash(5936)───jupl(6058)───jupyter-lab(7011)───bash(944289)───dl_mult_sras.sh(948367)───fastq-dump(949367)───{fastq-dump}(949370)
```
- in this case you would kill 948367 to kill the entire pipeline
- but killing is also more complicated than just kill
	- see https://askubuntu.com/questions/520107/how-to-kill-a-script-running-in-terminal-without-closing-terminal-ctrl-c-doe?newreg=d4336963af664502988de90ee4e5c1f7 for more details

## What you should try is this in this order

```
kill -INT -YOUR_PID_HERE 
```
- This sends `SIGINT` signal
	- Same as doing `ctrl + C`

- If this doesn't work, try

```
kill -TERM -YOUR_PID_HERE 
```
- This sends `SIGTERM` signal
	- This is what I had to do to shutdown my `dl_mult_sras.sh` pipeline
	- If you do ``pstree -ps `pgrep dl_mult_sras.sh` ``  again you should see a whole big mess of trees
		- thought I messed something up when I did this, but this is what you want

- Finally, the last resort is 

```
kill -KILL -YOUR_PID_HERE 
```
- This sends `SIGKILL` signal
	- once again, this is only a last resort



# Check space used in your home directory 

```
cd ~
du -h ./ | grep [0-9]G
```

# Get command histroy without the numbers
```
history | awk '{$1="";print substr($0,2)}'
```

# Get full file path of a file
`readlink -f file_name`

# monitor a file that is updating
```bash
tail -f updating_file
# exit with ctrl + C

# works well with nohup
# ie
nohup ./final_blast_update.sh features_list.txt > final_blast_update.log 2> final_blast_update.errlog &
tail -f final_blast_update.log
```

# Extract sequence from fastq file given read name

```gunzip -cd all_gac_reads.fastq.gz | awk '/read_id/{getline; print; exit}'```

- gunzip if file is compressed

- make sure to have -cd options on gunzip!! Very important.

- ' ' starts a program in awk

- / / denotes a character search in awk

- replace read_id with your read id

- exit stops scanning the file and exits once the first instance of your command gets printed

- getline just eats the line and goes on to the next.
    - So ie if you want like the 2nd line after (ie. grep -A 2 equivalent, do awk '/blah/{getline; getline; print; exit}'

# delete empty lines using awk
`awk 'NF' file`

Will turn

```
xxxxxx


yyyyyy


zzzzzz
```

into

```
xxxxxx
yyyyyy
zzzzzz
```
How does this work? Since NF stands for "number of fields", those lines being empty have 0 fiedls, so that awk evaluates 0 to False and no line is printed; however, if there is at least one field, the evaluation is True and makes awk perform its default action: print the current line.

# Call a shell within a shell
- use `sh -c`
```bash
# e.g. This is useful for checking the number of columns in your tables across multiple files at once
ls -1 *POST* | xargs -I{} sh -c "awk '{print NF \" \" FILENAME}' {}| sort | uniq"
```
<img width="645" alt="image" src="https://user-images.githubusercontent.com/80661840/179791992-34ce42b8-5345-402e-8819-1b8853b29c3e.png">

# Grab multiple stout's while using subshells and edit a desired one while using xargs
- Use `xargs -I` and use different symbols to grab and store information throughout your command (i.e. `xargs -I{}` and `xargs -I%`)
- To act upon the stout of the first one, use `echo` in the subshell
- e.g. I used this trick to rename multiple files with the `sed` command. `sed` needs to act on the stout of the first command which was `ls -1`. I couldn't do `xargs -I{} sh -c "sed 's/txt_2_change/new_txt/g' {}"` because that would perform sed on the contents **inside** the file, not on the filename itself (in this case stout of ls -1). `xargs -I{} sh -c "echo'{}' | sed 's/txt_2_change/new_txt/g'"` enabled this to work
- Final command was 
```bash 
ls -1 *WITH* | xargs -I{} sh -c "echo '{}' | sed 's/\.blast/PRE_INDEXING_FIX_BOTH_PLUS_AND_MINUS\.blast/g' | xargs -I% mv {} %"
```


# Nested parameter expansion in bash

- bash doesn't do nested parameter expansion. 
- Have to preface a parameter expansion with `eval` and `\$` is what the eval command is for

```
eval variable=\${${var}[j]} 
```

- This is useful for when you wanna unpack a list
- For example, for when you wanna the same command on a bunch of slightly modified commands in a loop

```
for ((j=0; j < $len; i++)); do
eval sra=\${${list}[i]} 
fastq-dump --gzip --skip-technical --readids --read-filter pass --dumpbase --split-3 --clip --outdir /Volumes/ubdata/mpampuch/diatomicbase_data/files_sorted/transcriptomics_data/ $sra 
done
```

# ls -1 and xargs

- Using this set up `ls -1 | xargs (some_command)`, you can perform an action on each file that is in a directory 
- Also, `xargs -I{}` allows you to not append xargs arguments to end of command but rather wherever you put the {}
	- for example
```
ls -1 | xargs readlink -f | xargs -I{} cp {} /Volumes/ubdata/mpampuch/
```

Copies all files from the current directory into the mpampuch directory

- Another good example is to copy a lot of directories into all sub directories
- ie.
```console 
# Original directory structure

# Final directory strucutre
(mark) mpampuch@agrajag:/Volumes/ubdata/mpampuch/REMAPPING_P
IPELINE/test_from_start_to_finish/sequence_length_check/feat
ure_folders$ ls -1 | xargs readlink -f | xargs -I{} cp -r ..
/failed_length_check/ ../passed_length_check/ {}
(mark) mpampuch@agrajag:/Volumes/ubdata/mpampuch/REMAPPING_P
IPELINE/test_from_start_to_finish/sequence_length_check$ rm 
-r passed_length_check/ feature_folders/ failed_length_check
(mark) mpampuch@agrajag:.$ tree
.
├── cds
│   ├── failed_length_check
│   └── passed_length_check
├── chromosomes
│   ├── failed_length_check
│   └── passed_length_check
├── direct_repeats
│   ├── failed_length_check
│   └── passed_length_check
├── exons
│   ├── failed_length_check
│   └── passed_length_check
├── five_prime_utrs
│   ├── failed_length_check
│   └── passed_length_check
├── genes
│   ├── failed_length_check
│   └── passed_length_check
├── introns
│   ├── failed_length_check
│   └── passed_length_check
├── lnc_rnas
│   ├── failed_length_check
│   └── passed_length_check
├── mrnas
│   ├── failed_length_check
│   └── passed_length_check
├── nc_rna_genes
│   ├── failed_length_check
│   └── passed_length_check
├── nc_rnas
│   ├── failed_length_check
│   └── passed_length_check
├── pseudogenes
│   ├── failed_length_check
│   └── passed_length_check
├── pseudogenic_transcripts
│   ├── failed_length_check
│   └── passed_length_check
├── regions
│   ├── failed_length_check
│   └── passed_length_check
├── repeat_regions
│   ├── failed_length_check
│   └── passed_length_check
├── rnas
│   ├── failed_length_check
│   └── passed_length_check
├── sequence_features
│   ├── failed_length_check
│   └── passed_length_check
├── sno_rnas
│   ├── failed_length_check
│   └── passed_length_check
├── sn_rnas
│   ├── failed_length_check
│   └── passed_length_check
├── supercontigs
│   ├── failed_length_check
│   └── passed_length_check
├── three_prime_utrs
│   ├── failed_length_check
│   └── passed_length_check
└── trnas
    ├── failed_length_check
    └── passed_length_check
```

# Turn a multi-line FASTA sequence into a single-line FASTA sequence

Make sure you have seqtk installed, use `seqtk seq`

`seqtk seq multi-line.fasta > single-line.fasta`

# Change FASTA headers by key value pair file

Need the seqkit package
```
seqkit replace -w 0 -p "(.+)" -r '{kv}' -k kv.txt annotated_seqs_file_safe_unique.fa > annotated_seqs_metadata_unique.fa
```
- Flags
	- -w = line width line width when outputing FASTA format (0 for no wrap) (default 60)
	- -p, --pattern string = search regular expression
	- -r, --replacement string = replacement. supporting capture variables.  e.g. $1 represents the text of the first submatch. ATTENTION: for *nix OS, use SINGLE quote NOT double quotes or use the \ escape character. Record number is also supported by "{nr}".use ${1} instead of $1 when {kv} given!*


# Extract Sequences from GFF file 
- Need bedtools
- Need .GFF file, Reference.FASTA file

```
bedtools getfasta [OPTIONS] -fi <input FASTA> -bed <BED/GFF/VCF>

# Example
bedtools getfasta -s -fi Phaeodactylum_tricornutum.ASM15095v2.dna.toplevel_old_reformatted.fa -bed clean6.gff -fo anotated_seqs.fa
```
- Flags
	- -s = Force strandedness. If the feature occupies the antisense strand, the sequence will be reverse complemented. *Default: strand information is ignored.*
	- -fo = Specify an output file name. By default, output goes to stdout.
- **bedtools substracts 1bp from the START of the sequence in the FASTA header**
- GFF
	- <img width="480" alt="image" src="https://user-images.githubusercontent.com/80661840/174163506-32f6dbeb-68ee-467e-9fcb-6aa0e1d2ea11.png">
- bedtools FASTA
	- <img width="480" alt="image" src="https://user-images.githubusercontent.com/80661840/174163447-1e443554-8fb0-43a4-b542-20b3379aeb86.png">

# Apply a command to only the first file in an AWK script

- use `'NR==FNR {action; next} {action on file2}' file1 file2`  

- Explaination (https://stackoverflow.com/questions/32481877/what-are-nr-and-fnr-and-what-does-nr-fnr-imply)
	- FNR refers to the record number (typically the line number) in the current file,
	- NR refers to the total record number.
	- The operator == is a comparison operator, which returns true when the two surrounding operands are equal.
		- This means that the condition NR==FNR is only true for the first file, as FNR resets back to 1 for the first line of each file but NR keeps on increasing.
		- This pattern is typically used to perform actions on only the first file.

The next inside the block means any further commands are skipped, so they are only run on files other than the first.

# Check if sequence in FASTA has pattern

# Count Number of Occurances of Pattern in the sequences within FASTA files
