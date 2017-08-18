This library is a tool to be used to automate parsing of the options/parameters
of a bash script and automate the generation of the documentation.

The main motivations that drove to write this library
=====================================================
  * writing consistent APIs in script is not easy
  * writing again and again documentation that is the same for some common parameters
    is boring
  * Formatting the documentation of a script is not the most passionating thing to do
  * Writing the parser is as boring as formatting the documentation AND it is not
    good for the readibility of the script.
  * Maintenance of the documentation is time consuming.
  * Writing the parser is time consuming and does not any value
  * Adding a parameter forces to add new documentation and new parsing.

How the library solves those issues
===================================
  * centalization of the parameters/options : there is a file that contains the list
    of the options your scripts will support
  * formalization of the declaration of the options and documentation.

How is the parameters file built ?
==================================
There is a strict naming convention that makes possible to guess a lot of things.
A block of a parameter description is the following :
```
DOC_INPUT_FILE="The absolute path to the input file."
OPTION_INPUT_FILE="--input-file"
P_INPUT_FILE=""
```
Let's explain the structure : 
The first line is the generic documentation for the parameter. This might be used
in more than just one script. We will see later that you can override the documentation
of the parameter in your script.
The variable is always named DOC_ and then the parameter name. This name will be
repeated for the 3 lines of the description. The next line is the option you will
propose for your API. I had previously a short option attached (SHORT_OPTION_)
but I never use this. The problem is that when your list of options grows, the short
options are limited to 26 and the letters are no more relevant.
So in this evolution of the library I simply removed the short option thing.

At last, you find the value for the parameter. The formalization is P_
which stands for parameter (soooo exotic...). You can set there a default value
for the parameter.

This formalization forces the parameter to have a value (the value will be separated
by a space on the command line, for instance --input-file /tmp/file-to-handle.json).
But some parameters require no value. Specifically because they are flags.
So there is another category of parameter/option that is a flag.
Here is an example :
``` bash
DOC_FLAG_AUTO_DELETION="If this flag is set, then the auto deletion of the archive is enabled."
OPTION_FLAG_AUTO_DELETION="--auto-deletion"
P_FLAG_AUTO_DELETION=false
```

This naming has also an advantage : it gives a strong knowledge about what the variable is.
First, you know it is a parameter (since prefixed by P_). Second, in case of flag,
you know it is a boolean. You should avoid the double negation like :
```
DOC_FLAG_DO_NOT_DELETE="Requires to not delete the temporary folder."
OPTION_FLAG_DO_NOT_DELETE=--do-not-delete
P_FLAG_DO_NOT_DELETE=false
```
since they force the reader/maintener to think with double negation which (according to
me) quite confusing. The flags are automatically set to the opposite value of the default
value.


How to use the library
======================
Here is how to use the library. First create a lib folder next
to your script. Copy the parsing library to the folder and also copy the script named
script-parameters.sh in the lib folder.

``` bash
#!/bin/bash

SELF=$(readlinf -f $0)
SELF_FOLDER=$(dirname ${SELF})

if [ -e "${SELF_FOLDER}/lib/cli-parser.sh" ]; then
    source "${SELF_FOLDER}/lib/cli-parser.sh"
fi

GBL_SUPPORTED_PARAMETERS=(
	INPUT_FILE
)

GBL_MANDATORY_PARAMETERS=(
	INPUT_FILE
)

# Sometimes, you have scripts that will accept a set of arguments that are not
# fitting in a parameter that you can describe this way. You can then use a
# local parsing to handle this :

# ###############################################################################
# Local parsing of the script. This parsing implements reject of options that
# are not in the list of supported options.
# ###############################################################################
parsing_of_script() {
    echo "Parameter or value '$1' is not supported."
    usage
    exit 1
}

parseParameters "${SELF_FOLDER}/lib" "$@"

# After this, the P_ variables that you have declared above are automatically assigned.
```

To check a full example, see the sample.sh script.

Defining a set of default options for your script
=================================================
It is usually useful to be able to declare some options that are common to all of your
scripts and you don't want to repeat them in each of your script. For instance :
  * the -? of --help option that will display the usage of your script
  * a debug flag that enable debug traces
  * any option that you like to see
The advantage of the common variables is that they are centralized in the library
so disabling one options is straightforward.

To define a set of default variables, check the cli-parser.sh . You have a variable
GBL_DEFAULT_OPTIONS where you can add or remove options.
