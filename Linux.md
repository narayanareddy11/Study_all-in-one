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

#### export var

- export var=”value”
- declare -x var
- declare -x var=”value”
#### all:
- $@: All
	-  Equivalent to $1 $2 $3 ... $N
	-  But when double quoted: “$1” “$2” “$3” ... “$N”
	- So parameters containing multiple words stay intact
- $*
	- Equivalent to $1 $2 $3 ... $N
	- But when double quoted: “$1 $2 $3 ... $N”
	- Don’t use this; use $@ instead!
- $#
	- Holds the number of arguments passed to the script
	
#### Removing part of a string

- ${var#pattern} Removes shortest match from begin of string
- ${var##pattern} Removes longest match from begin of string
- ${var%pattern} Removes shortest match from end of string
- ${var%%pattern} Removes longest match from end of string

#### At
	- Will execute your script at a speci!c time
	- at -f myscript noon tomorrow
- Cron
	-  Will execute your script according to a schedule

#### demo1: 

```
printf %s 'Enter a number: ' >&2
read -r number
if ((number == 1234)); then
    echo 'Good guess'
else
    echo 'Haha... :-P'
fi
```
#### demo3:
```
function printSum {
    typeset -A args
    typeset name
    for name in first second; do
        [[ -t 0 ]] && printf 'Enter %s positive integer: ' "$name" >&2
        read -r ${BASH_VERSION+-e} "args[$name]"
        [[ ${args[$name]} == +([[:digit:]]) ]] || return 1 # Validation is extremely important whenever user input is used in arithmetic.
    done
    printf 'The sum is %d.' $((${args[first]} + ${args[second]}))
}
```
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
#### Case:
```
#/bin/bash

# This script prints a range of numbers
# Usage: count [-r] [-b n] [-s n] stop
# -b gives the number to begin with (default: 0)
# -r reverses the count
# -s sets step size (default: 1)
# counting will stop at stop.

declare reverse=""
declare -i begin=0
declare -i step=1

while getopts ":b:s:r" opt; do
	case $opt in
		r)
			reverse="yes"			
			;;
		b) 
			[[ ${OPTARG} =~ ^[0-9]+$ ]] || { echo "${OPTARG} is not a number" >&2; exit 1; }
			start="${OPTARG}"
			;;
		s)
			[[ ${OPTARG} =~ ^[0-9]+$ ]] || { echo "${OPTARG} is not a number" >&2; exit 1; }
			step="${OPTARG}"
			;;
		:) 
			echo "Option -${OPTARG} is missing an argument" >&2
			exit 1
			;;
		\?)
			echo "Unknown option: -${OPTARG}" >&2
			exit 1
			;;
	esac
done

shift $(( OPTIND -1 ))

[[ $1 ]] || { echo "missing an argument" >&2; exit 1; }
[[ $1 =~ ^[0-9]+$ ]] || { echo "$1 is not a number" >&2; exit 1; }
declare end="$1"

if [[ ! $reverse ]]; then
	for (( i=start; i <= end; i+=step )); do
		echo $i
	done
else
	for (( i=end; i >= start; i-=step )); do
		echo $i
	done
fi

exit 0

```
#### Demo : 
```
#!/bin/bash

# Is there an argument?
if [[ ! $1 ]]; then
	echo "Missing argument"
	exit 1
fi

scriptname="$1"
bindir="${HOME}/bin"
filename="${bindir}/${scriptname}"

if [[ -e $filename ]]; then
	echo "File ${filename} already exists"
	exit 1
fi

if type "$scriptname" > /dev/null 2>&1; then
	echo "There is already a command with name ${scriptname}"
	exit 1
fi

# Check bin directory exists
if [[ ! -d $bindir ]]; then
	# if not: create bin directory
	if mkdir "$bindir"; then
		echo "created ${bindir}"
	else
		echo "Could not create ${bindir}."
		exit 1
	fi
fi

# Create a script with a single line
echo '#!/bin/bash' > "$filename"
# Add executable permission
chmod u+x "$filename"
# Open with editor
if [[ $EDITOR ]]; then 
	$EDITOR "$filename"
else
	echo "Script created; not starting editor because \$EDITOR is not set."
fi

```
