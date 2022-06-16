# useful_unix_commands

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

# Get full file path of a file
`readlink -f file_name`

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

## delete empty lines using awk
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

# Nested parameter expansion in bash

- bash doesn't do nested parameter expansion. 
- Have to preface a parameter expansion with `eval` and `\` is what the eval command is for

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
