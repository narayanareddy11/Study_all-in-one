### Bash shell scripts

- Execute bash commands from a file
- Automate sequences of shell commands

#### Shebang line:
- Species which interpreter should run the code

#### variable
- x=10
- If x already existed, it is assigned the new value
- filenames=”notes.txt picture.jpg movie.mov”
- Values containing spaces: use quotes
- echo “${foo}bar”
- Use $HOME instead of ~
- read -p “Type your name: “ name
- Use -x option in hashbang line
- use “set -x” to enable and “set +x” to disable

####  [[ Expression ]]
- Spaces around the expression are very important!
- Same for switches (-e) and equals sign
#### Expression True if
- [[ $str ]] str is not empty
- [[ $str = “something” ]] str equals string “something”
- [[ $str=”something” ]] always returns true!
- [[ -e $!lename ]] !le $!lename exists
- [[ -d $dirname ]] $dirname is a directory

### Sending both error and output to a single file
- cmd > logfile 2>&1
- Don’t do this: cmd > logfile
- $# contains number of script arguments
- $? contains exit status for last command
- Input redirection: <
- Output redirection: >, >>
- Redirecting a specific stream: 2>
- Redirecting into another stream: 2>&1
#### demo: 1

```
#!/bin/bash

# get the date
declare -r date=$(date)

# get the topic
declare -r topic="$1"

# determine where to save our notes
declare notesdir="${HOME}"
[[ $NOTESDIR ]] && notesdir="${NOTESDIR}"

declare notesdir=${NOTESDIR:-$HOME}

if [[ ! -d $notesdir ]]; then
	mkdir "${notesdir}" 2> /dev/null || { echo "Cannot make directory ${notesdir}" 1>&2; exit 1; }
fi

# filename to write to
declare -r filename="${notesdir}/${topic}notes.txt"

# Does file exist? If not, create it
if [[ ! -f $filename ]]; then
	touch "${filename}" 2> /dev/null || { echo "Cannot create file ${filename}" 1>&2; exit 1; }
fi

# Is file writeable?
[[ -w $filename ]] || {	echo "${filename} is not writeable" 1>&2; exit 1; }

# Ask user for input
read -p "Your note: " note

if [[ $note ]]; then
	echo "$date: $note" >> "$filename"
	echo "Note '$note' saved to $filename" 1>&2
else
	echo "No input; note wasn't saved." 1>&2 
	exit 2
fi

exit 0
```
