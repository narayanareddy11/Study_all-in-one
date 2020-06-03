### Bash shell scripts

- Execute bash commands from a file
- Automate sequences of shell commands

#### Shebang line:
- Species which interpreter should run the code

#### variable
- PATH=”$PATH:~/bin”
- x=10
- If x already existed, it is assigned the new value
- filenames=”notes.txt picture.jpg movie.mov”
- Values containing spaces: use quotes
- echo “${foo}bar”
- Use $HOME instead of ~
- read -p “Type your name: “ name
- Use -x option in hashbang line
- use “set -x” to enable and “set +x” to disable
 - Storing a value
- x[0]=”some”
- x[1]=”word”
- reguler expresion 
#### Retrieving a value
- echo ${x[0]}   # read value 

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
- Uses IFS variable for delimiters ex: IFS=: READ  a b , 2:5,    echo $a echo $b
- cmd > logfile 2>&1
- Don’t do this: cmd > logfile
- $# contains number of script arguments
- $? contains exit status for last command
- Input redirection: <
- Output redirection: >, >>
- Redirecting a specific stream: 2>
- Redirecting into another stream: 2>&1

#### export var

- 0: Standard Input (stdin)
/dev/stdin
- 1: Standard Output (stdout)
/dev/stdout
- 2: Standard Error (stderr)
- export var=”value”
- declare -x var
- declare -x var=”value”   # global varible and different files 

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
#### demo :
```
#!/bin/bash

# compare length of two strings
# check number of arguments
if [[ $# -ne 2 ]]; then
	echo "Need exactly two arguments"
	exit 1
fi

length_1=${#1}
length_2=${#2}

# Which one has most files?
if [[ $length_1 -gt $length_2 ]]; then
	echo "$1 is longest"
elif [[ $length_1 -eq $length_2 ]]; then
	echo "length is equal"
else 
	echo "$2 is longest"
fi

exit 0


```
#### demo if :
```
#!/bin/bash

# compare file count of two directories

# check number of arguments
if [[ $# -ne 2 ]]; then
	echo "Need exactly two arguments"
	exit 1
fi

# Both arguments should be directories
if [[ ! -d $1  ]]; then
	echo "'$1' is not a directory"
	exit 1
fi
if [[ ! -d $2  ]]; then
	echo "'$2' is not a directory"
	exit 1
fi

dir1="$1"
dir2="$2"
# Get number of files in directories
count_1=$(ls -A1 "$dir1" | wc -l)
count_2=$(ls -A1 "$dir2" | wc -l)

# Which one has most files?
if [[ $count_1 -gt $count_2 ]]; then
	echo "${dir1} has most files"
elif [[ $count_1 -eq $count_2 ]]; then
	echo "number of files is equal"
else 
	echo "${dir2} has most files"
fi

exit 0

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
