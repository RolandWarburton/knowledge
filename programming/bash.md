# Bash

Bash is... er, yeah.

## Functions

### Simple Function

```bash
function my_function() {
	# print hello
	echo "hello"
	# will also work
	echo hello
}

# call the function
my_function
```

You can also do this, buts its not as correct as above

```bash
my_function {

}

my_function
```

### Function with parameter

```bash
function my_function {
	echo $1
}

my_function hello
```

### Function with multiple parameters

```bash
function my_function {
	echo $1
	echo $2
}

my_function hello world
```

### Function with parameters inside if statement

Omit the square backet`[` utility from the if statement when calling a function.

```bash
function my_function {
	# convert the argument to an int
	declare -i var=$1

	# do not return 1 and 0 for true and false
	if [ $var -eq 1 ]; then
		true
	else
		false
	fi
}

if my_function 1; then
	echo "yeet"
else
	echo "aww"
fi
```

## Arrays

Length of array

```bash
echo "${#my_array[@]}"
```

## Flags

Flags are passed into a bash script.

In this example from [Job Alemedia](https://jonalmeida.com/posts/2013/05/26/different-ways-to-implement-flags-in-bash/)

* -b need to be entered by itself with no paired value.
* -f needs an accompanying value. Eg. `-f string`.

```bash
usage() {
	printf "%s\t%s\n" "-V" "USAGE: my_script -V"
	printf "%s\t%s\n" "-h, --help" "Show help"
	exit 0
}

while getopts "bf:" OPTION
do
	case $OPTION in
		b)
			echo "You set flag -b"
			exit
			;;
		f)
			echo "The value of -f is $OPTARG"
			MYOPTF=$OPTARG
			echo $MYOPTF
			exit
			;;
		\?)
			echo "Used for the help menu"
			usage
			exit
			;;
	esac
done
```
