Chapter 3. Working with the Command Line
The command line is where the rubber hits the road. Although there are many powerful tools with graphical interfaces, the command line is still home for DevOps work. Interacting with your shell environment from within Python and creating Python command-line tools are both necessary when using Python for DevOps.

Working with the Shell
Python offers tools for interacting with systems and shells. You should become familiar with the sys, os, and subprocess modules, as all are essential tools.

Talking to the Interpreter with the sys Module
The sys module offers access to variables and methods closely tied to the Python interpreter.

NOTE
There are two dominant ways to interpret bytes during reading. The first, little endian, interprets each subsequent byte as having higher significance (representing a larger digit). The other, big endian, assumes the first byte has the greatest significance and moves down from there.

You can use the sys.byteorder attribute to see the byte order of your current architecture:

In [1]: import sys

In [2]: sys.byteorder
Out[2]: 'little'
You can use sys.getsizeof to see the size of Python objects. This is useful if you are dealing with limited memory:

In [3]: sys.getsizeof(1)
Out[3]: 28
If you want to perform different behaviors, depending on the underlying operating system, you can use sys.platform to check:

In [5]: sys.platform
Out[5]: 'darwin'
A more common situation is that you want to use a language feature or module that is only available in specific versions of Python. You can use the sys.version_info to control behavior based on the running Python interpreter. Here we print different messages for Python 3.7, a Python version 3 that is below 3.7, and Python versions lower than 3:

if sys.version_info.major < 3:
    print("You need to update your Python version")
elif sys.version_info.minor < 7:
    print("You are not running the latest version of Python")
else:
    print("All is good.")
We cover more sys usage later in this chapter when we write command-line tools.

Dealing with the Operating System Using the os Module
You have seen the os module used in Chapter 2 for dealing with the filesystem. It also has a grab bag of various attributes and functions related to dealing with the operating system. In Example 3-1 we demonstrate some of them.

Example 3-1. os module examples
In [1]: import os

In [2]: os.getcwd() 
Out[2]: '/Users/kbehrman/Google-Drive/projects/python-devops'

In [3]: os.chdir('/tmp') 

In [4]: os.getcwd()
Out[4]: '/private/tmp'

In [5]: os.environ.get('LOGLEVEL') 

In [6]: os.environ['LOGLEVEL'] = 'DEBUG' 

In [7]: os.environ.get('LOGLEVEL')
Out[7]: 'DEBUG'

In [8]: os.getlogin() 
Out[8]: 'kbehrman'

Get the current working directory.


Change the current working directory.


The os.environ holds the environment variables that were set when the os module was loaded.


This is the setting and environment variable. This setting exists for subprocesses spawned from this code.


This is the login of the user in the terminal that spawned this process.

The most common usage of the os module is to get settings from environment variables. These could be the level to set your logging, or secrets such as API keys.

Spawn Processes with the subprocess Module
There are many instances when you need to run applications outside of Python from within your Python code. This could be built-in shell commands, Bash scripts, or any other command-line application. To do this, you spawn a new process (instance of the application). The subprocess module is the right choice when you want to spawn a process and run commands within it. With subprocess, you can run your favorite shell command or other command-line software and collect its output from within Python. For the majority of use cases, you should use the subprocess.run function to spawn processes:

In [1]: cp = subprocess.run(['ls','-l'],
                            capture_output=True,
                            universal_newlines=True)

In [2]: cp.stdout
Out[2]: 'total 96
         -rw-r--r--  1 kbehrman  staff     0 Apr 12 08:48 __init__.py
         drwxr-xr-x  5 kbehrman  staff   160 Aug 18 15:47 __pycache__
         -rw-r--r--  1 kbehrman  staff   123 Aug 13 12:13 always_say_it.py
         -rwxr-xr-x  1 kbehrman  staff  1409 Aug  8 15:36 argparse_example.py
         -rwxr-xr-x  1 kbehrman  staff   734 Aug 12 09:36 click_example.py
         -rwxr-xr-x  1 kbehrman  staff   538 Aug 13 10:41 fire_example.py
         -rw-r--r--  1 kbehrman  staff    41 Aug 18 15:17 foo_plugin_a.py
         -rw-r--r--  1 kbehrman  staff    41 Aug 18 15:47 foo_plugin_b.py
         -rwxr-xr-x  1 kbehrman  staff   335 Aug 10 12:36 simple_click.py
         -rwxr-xr-x  1 kbehrman  staff   256 Aug 13 09:21 simple_fire.py
         -rwxr-xr-x  1 kbehrman  staff   509 Aug  8 10:27 simple_parse.py
         -rwxr-xr-x  1 kbehrman  staff   502 Aug 18 15:11 simple_plugins.py
         -rwxr-xr-x  1 kbehrman  staff   850 Aug  6 14:44 sys_argv.py
         -rw-r--r--  1 kbehrman  staff   182 Aug 18 16:24 sys_example.py
'
The subprocess.run function returns a CompletedProcess instance once the process completes. In this case, we run the shell command ls with the argument -l to see the contents of the current directory. We set it to capture stdout and stderr with the capture_output parameter. We then access the results using cp.stdout. If we run our ls command on a nonexistent directory, causing it to return an error, we can see the output in cp.stderr:

In [3]: cp = subprocess.run(['ls','/doesnotexist'],
                            capture_output=True,
                            universal_newlines=True)

In [3]: cp.stderr
Out[3]: 'ls: /doesnotexist: No such file or directory\n'
You can better integrate the handling of errors by using the check parameter. This raises an exception if the subprocess reports an error:

In [23]: cp = subprocess.run(['ls', '/doesnotexist'],
                             capture_output=True,
                             universal_newlines=True,
                             check=True)
---------------------------------------------------------------------------
CalledProcessError                        Traceback (most recent call last)
<ipython-input-23-c0ac49c40fee> in <module>
----> 1 cp = subprocess.run(['ls', '/doesnotexist'],
                            capture_output=True,
                            universal_newlines=True,
                            check=True)

~/.pyenv/versions/3.7.0/lib/python3.7/subprocess.py ...
    466         if check and retcode:
    467             raise CalledProcessError(retcode, process.args,
--> 468                                      output=stdout, stderr=stderr)
    469     return CompletedProcess(process.args, retcode, stdout, stderr)
    470

CalledProcessError: Command '['ls', '/doesnotexist']' returned non-zero exit
In this way, you don’t have to check stderr for failures. You can treat errors from your subprocess much as you would other Python exceptions.

Creating Command-Line Tools
The simplest way to invoke a Python script on the command line is to invoke it using Python. When you construct a Python script, any statements at the top level (not nested in code blocks) run whenever the script is invoked or imported. If you have a function you want to run whenever your code is loaded, you can invoke it at the top level:

def say_it():
    greeting = 'Hello'
    target = 'Joe'
    message = f'{greeting} {target}'
    print(message)

say_it()
This function runs whenever the script runs on the command line:

$ python always_say_it.py

Hello Joe
Also, when the file is imported:

In [1]: import always_say_it
Hello Joe
This should only be done with the most straightforward scripts, however. A significant downside to this approach is that if you want to import your module into other Python modules, the code runs during import instead of waiting to be invoked by the calling module. Someone who is importing your module usually wants control over when its contents are invoked. You can add functionality that only happens when called from the command line by using the global name variable. You have seen that this variable reports the name of the module during import. If the module is called directly on the command line, this sets it to the string main. The convention for modules running on the command line is to end with a block testing for this and run command-line specific code from this block. To modify the script to run a function automatically only when invoked on the command line, but not during import, put the function invocation into the block after the test:

def say_it():
    greeting = 'Hello'
    target = 'Joe'
    message = f'{greeting} {target}'
    print(message)

if __name__ == '__main__':
    say_it()
When you import this function, this block does not run, as the __name__ variable reflects the module path as imported. It runs when the module is run directly, however:

$ python say_it.py

Hello Joe
MAKING YOUR SHELL SCRIPT EXECUTABLE
To eliminate the need to explicitly call type python on the command line when you run your script, you can add the line #!/usr/bin/env python to the top of your file:

#!/usr/bin/env python

def say_it():
    greeting = 'Hello'
    target = 'Joe'
    message = f'{greeting} {target}'
    print(message)

if __name__ == '__main__':
    say_it()
Then make the file executable using chmod (a command-line tool for setting permissions):

chmod +x say_it.py`
You can then call it in a shell without directly invoking Python:

$ ./say_it.py

Hello Joe
The first step in creating command-line tools is separating code that should only run when invoked on the command line. The next step is to accept command-line arguments. Unless your tool only does one thing, you need to accept commands to know what to do. Also, command-line tools that do more than the simplest tasks accept optional flags to configure their workings. Remember that these commands and flags are the user interface (UI) for anyone using your tools. You need to consider how easy they are to use and understand. Providing documentation is an essential part of making your code understandable.

Using sys.argv
The simplest and most basic way to process arguments from the command line is to use the argv attribute of the sys module. This attribute is a list of arguments passed to a Python script at runtime. If the script runs on the command line, the first argument is the name of the script. The rest of the items in the list are any remaining command-line arguments, represented as strings:

#!/usr/bin/env python
"""
Simple command-line tool using sys.argv
"""
import sys

if __name__ == '__main__':
    print(f"The first argument:  '{sys.argv[0]}'")
    print(f"The second argument: '{sys.argv[1]}'")
    print(f"The third argument:  '{sys.argv[2]}'")
    print(f"The fourth argument: '{sys.argv[3]}'")
Run it on the command line and see the arguments:

$ ./sys_argv.py --a-flag some-value 13

The first argument:  './sys_argv.py'
The second argument: '--a-flag'
The third argument:  'some-value'
The fourth argument: '13'
You can use these arguments to write your own argument parser. To see what this might look like, check out Example 3-2.

Example 3-2. Parsing with sys.argv
#!/usr/bin/env python
"""
Simple command-line tool using sys.argv
"""
import sys

def say_it(greeting, target):
    message = f'{greeting} {target}'
    print(message)

if __name__ == '__main__': 
    greeting = 'Hello'  
    target = 'Joe'

    if '--help' in sys.argv:  
        help_message = f"Usage: {sys.argv[0]} --name <NAME> --greeting <GREETING>"
        print(help_message)
        sys.exit()  

    if '--name' in sys.argv:
        # Get position after name flag
        name_index = sys.argv.index('--name') + 1 
        if name_index < len(sys.argv): 
            name = sys.argv[name_index]

    if '--greeting' in sys.argv:
        # Get position after greeting flag
        greeting_index = sys.argv.index('--greeting') + 1
        if greeting_index < len(sys.argv):
            greeting = sys.argv[greeting_index]

    say_it(greeting, name) 

Here we test to see if we are running from the command line.


Default values are set in these two lines.


Check if the string --help is in the list of arguments.


Exit the program after printing the help message.


We need the position of the value after the flag, which should be the associated value.


Test that the arguments list is long enough. It will not be if the flag was provided without a value.


Call the function with the values as modified by the arguments.

Example 3-2 goes far enough to print out a simple help message and accept arguments to the function:

$ ./sys_argv.py --help
Usage: ./sys_argv.py --name <NAME> --greeting <GREETING>

$ ./sys_argv.py --name Sally --greeting Bonjour
Bonjour Sally
This approach is fraught with complication and potential bugs. Example 3-2 fails to handle many situations. If a user misspells or miscapitalizes a flag, the flag is ignored with no useful feedback. If they use commands that are not supported or try to use more than one value with a flag, once again the error is ignored. You should be aware of the argv parsing approach, but do not use it for any production code unless you specifically set out to write an argument parser. Luckily there are modules and packages designed for the creation of command-line tools. These packages provide frameworks to design the user interface for your module when running in a shell. Three popular solutions are argparse, click, and python-fire. All three include ways to design required arguments, optional flags, and means to display help documentation. The first, argparse, is part of the Python standard library, and the other two are third-party packages that need to be installed separately (using pip).

Using argparse
argparse abstracts away many of the details of parsing arguments. With it, you design your command-line user interface in detail, defining commands and flags along with their help messages. It uses the idea of parser objects, to which you attach commands and flags. The parser then parses the arguments, and you use the results to call your code. You construct your interface using ArgumentParser objects that parse user input for you:

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Maritime control')
You add position-based commands or optional flags to the parser using the add_argument method (see Example 3-3). The first argument to this method is the name of the new argument (command or flag). If the name begins with a dash, it is treated as an optional flag argument; otherwise it is treated as a position-dependent command. The parser creates a parsed-arguments object, with the arguments as attributes that you can then use to access input. Example 3-3 is a simple program that echoes a users input and shows the basics of how argparse works.

Example 3-3. simple_parse.py
#!/usr/bin/env python
"""
Command-line tool using argparse
"""
import argparse


if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Echo your input') 
    parser.add_argument('message',               
                        help='Message to echo')

    parser.add_argument('--twice', '-t',         
                        help='Do it twice',
                        action='store_true')     

    args = parser.parse_args()  

    print(args.message)    
    if args.twice:
        print(args.message)

Create the parser object, with its documentation message.


Add a position-based command with its help message.


Add an optional argument.


Store the optional argument as a boolean value.


Use the parser to parse the arguments.


Access the argument values by name. The optional argument’s name has the -- removed.

When you run it with the --twice flag, the input message prints twice:

$ ./simple_parse.py hello --twice
hello
hello
argparse automatically sets up help and usage messages based on the help and description text you supply:

$ ./simple_parse.py  --help
usage: simple_parse.py [-h] [--twice] message

Echo your input

positional arguments:
  message      Message to echo

optional arguments:
  -h, --help   show this help message and exit
  --twice, -t  Do it twice
Many command-line tools use nested levels of commands to group command areas of control. Think of git. It has top-level commands, such as git stash, which have separate commands under them, such as git stash pop. With argparse, you create subcommands by creating subparsers under your main parser. You can create a hierarchy of commands using subparsers. In Example 3-4, we implement a maritime application that has commands for ships and sailors. Two subparsers are added to the main parser; each subparser has its own commands.

Example 3-4. argparse_example.py
#!/usr/bin/env python
"""
Command-line tool using argparse
"""
import argparse

def sail():
    ship_name = 'Your ship'
    print(f"{ship_name} is setting sail")

def list_ships():
    ships = ['John B', 'Yankee Clipper', 'Pequod']
    print(f"Ships: {','.join(ships)}")

def greet(greeting, name):
    message = f'{greeting} {name}'
    print(message)

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Maritime control') 

    parser.add_argument('--twice', '-t',   
                        help='Do it twice',
                        action='store_true')

    subparsers = parser.add_subparsers(dest='func') 

    ship_parser =  subparsers.add_parser('ships',  
                                         help='Ship related commands')
    ship_parser.add_argument('command', 
                             choices=['list', 'sail'])

    sailor_parser = subparsers.add_parser('sailors', 
                                          help='Talk to a sailor')
    sailor_parser.add_argument('name', 
                               help='Sailors name')
    sailor_parser.add_argument('--greeting', '-g',
                               help='Greeting',
                               default='Ahoy there')

    args = parser.parse_args()
    if args.func == 'sailors': 
        greet(args.greeting, args.name)
    elif args.command == 'list':
        list_ships()
    else:
        sail()

Create the top-level parser.


Add a top-level argument that can be used along with any command under this parser’s hierarchy.


Create a subparser object to hold the subparsers. The dest is the name of the attribute used to choose a subparser.


Add a subparser for ships.


Add a command to the ships subparser. The choices parameter gives a list of possible choices for the command.


Add a subparser for sailors.


Add a required positional argument to the sailors subparser.


Check which subparser is used by checking the func value.

Example 3-4 has one top-level optional argument (twice) and two subparsers. Each subparser has its own commands and flags. argparse automatically creates a hierarchy of help messages and displays them with the --help flag. The top-level help commands, including the subparsers and the top-level twice argument, are documented:

$ ./argparse_example.py --help
usage: argparse_example.py [-h] [--twice] {ships,sailors} ...

Maritime control

positional arguments:
  {ships,sailors}
    ships          Ship related commands
    sailors        Talk to a sailor

optional arguments:
  -h, --help       show this help message and exit
  --twice, -t      Do it twice
You can dig into the subcommands (subparsers) by using the help flag after the command:

$ ./argparse_example.py ships --help
usage: argparse_example.py ships [-h] {list,sail}

positional arguments:
  {list,sail}

optional arguments:
  -h, --help   show this help message and exit
As you can see, argparse gives you a lot of control over your command-line interface. You can design a multilayered interface with built-in documentation with many options to fine-tune your design. Doing so takes a lot of work on your part, however, so let’s look at some easier options.

Using click
The click package was first developed to work with web framework flask. It uses Python function decorators to bind the command-line interface directly with your functions. Unlike argparse, click interweaves your interface decisions directly with the rest of your code.

FUNCTION DECORATORS
Python decorators are a special syntax for functions which take other functions as arguments. Python functions are objects, so any function can take a function as an argument. The decorator syntax provides a clean and easy way to do this. The basic format of a decorator is:

In [2]: def some_decorator(wrapped_function):
   ...:     def wrapper():
   ...:         print('Do something before calling wrapped function')
   ...:         wrapped_function()
   ...:         print('Do something after calling wrapped function')
   ...:     return wrapper
   ...:
You can define a function and pass it as an argument to this function:

In [3]: def foobat():
   ...:     print('foobat')
   ...:

In [4]: f = some_decorator(foobat)

In [5]: f()
Do something before calling wrapped function
foobat
Do something after calling wrapped function
The decorator syntax simplifies this by indicating which function should be wrapped by decorating it with @decorator_name. Here is an example using the decorator syntax with our some_decorator function:

In [6]: @some_decorator
   ...: def batfoo():
   ...:     print('batfoo')
   ...:

In [7]: batfoo()
Do something before calling wrapped function
batfoo
Do something after calling wrapped function
Now you call your wrapped function using its name rather than the decorator name. Pre-built functions intended as decorators are offered both as part of the Python Standard Library (staticMethod, classMethod) and as part of third-party packages, such as Flask and Click.

This means that you tie your flags and options directly to the parameters of the functions that they expose. You can create a simple command-line tool from your functions using click’s command and option functions as decorators before your function:

#!/usr/bin/env python
"""
Simple Click example
"""
import click

@click.command()
@click.option('--greeting', default='Hiya', help='How do you want to greet?')
@click.option('--name', default='Tammy', help='Who do you want to greet?')
def greet(greeting, name):
    print(f"{greeting} {name}")

if __name__ == '__main__':
    greet()
click.command indicates that a function should be exposed to command-line access. click.option adds an argument to the command-line, automatically linking it to the function parameter of the same name (--greeting to greet and --name to name). click does some work behind the scenes so that we can call our greet method in our main block without parameters that are covered by the options decorators.

These decorators handle parsing command-line arguments and automatically produce help messages:

$ ./simple_click.py --greeting Privet --name Peggy
Privet Peggy

$ ./simple_click.py --help
Usage: simple_click.py [OPTIONS]

Options:
  --greeting TEXT  How do you want to greet?
  --name TEXT      Who do you want to greet?
  --help           Show this message and exit.
You can see that with click you can expose your functions for command-line use with much less code than argparse. You can concentrate on the business logic of your code rather than designing the interface.

Now let’s look at a more complicated example with nested commands. Commands are nested by using click.group creating functions that represent the groups. In Example 3-5 we nest commands with argparse, using an interface that is very similar to the one from Example 3-4.

Example 3-5. click_example.py
#!/usr/bin/env python
"""
Command-line tool using click
"""
import click

@click.group() 
def cli(): 
    pass

@click.group(help='Ship related commands') 
def ships():
    pass

cli.add_command(ships) 

@ships.command(help='Sail a ship') 
def sail():
    ship_name = 'Your ship'
    print(f"{ship_name} is setting sail")

@ships.command(help='List all of the ships')
def list_ships():
    ships = ['John B', 'Yankee Clipper', 'Pequod']
    print(f"Ships: {','.join(ships)}")

@cli.command(help='Talk to a sailor')  
@click.option('--greeting', default='Ahoy there', help='Greeting for sailor')
@click.argument('name')
def sailors(greeting, name):
    message = f'{greeting} {name}'
    print(message)

if __name__ == '__main__':
    cli()  

Create a top-level group under which other groups and commands will reside.


Create a function to act as the top-level group. The click.group method transforms the function into a group.


Create a group to hold the ships commands.


Add the ships group as a command to the top-level group. Note that the cli function is now a group with an add_command method.


Add a command to the ships group. Notice that ships.command is used instead of click.command.


Add a command to the cli group.


Call the top-level group.

The top-level help messages generated by click look like this:

./click_example.py --help
Usage: click_example.py [OPTIONS] COMMAND [ARGS]...

Options:
  --help  Show this message and exit.

Commands:
  sailors  Talk to a sailor
  ships    Ship related commands
You can dig into the help for a subgroup like this:

$ ./click_example.py ships --help
Usage: click_example.py ships [OPTIONS] COMMAND [ARGS]...

  Ship related commands

Options:
  --help  Show this message and exit.

Commands:
  list-ships  List all of the ships
  sail        Sail a ship
If you compare Example 3-4 and Example 3-5, you will see some of the differences between using argparse and click. The click approach certainly requires less code, almost half in these examples. The user interface (UI) code is interspersed throughout the whole program; it is especially important when creating functions that solely act as groups. If you have a complex program, with a complex interface, you should try as best as possible to isolate different functionality. By doing so, you make individual pieces easier to test and debug. In such a case, you might choose argparse to keep your interface code separate.

DEFINING CLASSES
A class definition starts with the keyword class followed by the class name and parentheses:

In [1]: class MyClass():
Attributes and method definitions follow in the indented code block. All methods of a class recieve as their first parameter a copy of the instantiated class object. By convention this is refered to as self:

In [1]: class MyClass():
   ...:     def some_method(self):
   ...:         print(f"Say hi to {self}")
   ...:

In [2]: myObject = MyClass()

In [3]: myObject.some_method()
Say hi to <__main__.MyClass object at 0x1056f4160>
Every class has an init method. When the class is instantiated, this method is called. If you do not define this method, it gets a default one, inherited from the Python base object class:

In [4]: MyClass.__init__
Out[4]: <slot wrapper '__init__' of 'object' objects>
Generally you define an object’s attributes in the init method:

In [5]: class MyOtherClass():
   ...:     def __init__(self, name):
   ...:         self.name = name
   ...:

In [6]: myOtherObject = MyOtherClass('Sammy')

In [7]: myOtherObject.name
Out[7]: 'Sammy'
fire
Now, let’s take a step farther down the road of making a command-line tool with minimal UI code. The fire package uses introspection of your code to create interfaces automatically. If you have a simple function you want to expose, you call fire.Fire with it as an argument:

#!/usr/bin/env python
"""
Simple fire example
"""
import fire

def greet(greeting='Hiya', name='Tammy'):
    print(f"{greeting} {name}")

if __name__ == '__main__':
    fire.Fire(greet)
fire then creates the UI based on the method’s name and arguments:

$ ./simple_fire.py --help

NAME
    simple_fire.py

SYNOPSIS
    simple_fire.py <flags>

FLAGS
    --greeting=GREETING
    --name=NAME
In simple cases, you can expose multiple methods automatically by invoking fire with no arguments:

#!/usr/bin/env python
"""
Simple fire example
"""
import fire

def greet(greeting='Hiya', name='Tammy'):
    print(f"{greeting} {name}")

def goodbye(goodbye='Bye', name='Tammy'):
    print(f"{goodbye} {name}")

if __name__ == '__main__':
    fire.Fire()
fire creates a command from each function and documents automatically:

$ ./simple_fire.py --help
INFO: Showing help with the command 'simple_fire.py -- --help'.

NAME
    simple_fire.py

SYNOPSIS
    simple_fire.py GROUP | COMMAND

GROUPS
    GROUP is one of the following:

     fire
       The Python fire module.

COMMANDS
    COMMAND is one of the following:

     greet

     goodbye
(END)
This is really convenient if you are trying to understand someone else’s code or debug your own. With one line of additional code, you can interact with all of a module’s functions from the command-line. That is powerful. Because fire uses the structure of your program itself to determine the interface, it is even more tied to your non-interface code than argparse or click. To mimic our nest command interface, you need to define classes with the structure of the interface you want to expose. To see an approach to this, check out Example 3-6.

Example 3-6. fire_example.py
#!/usr/bin/env python
"""
Command-line tool using fire
"""
import fire

class Ships(): 
    def sail(self):
        ship_name = 'Your ship'
        print(f"{ship_name} is setting sail")

    def list(self):
        ships = ['John B', 'Yankee Clipper', 'Pequod']
        print(f"Ships: {','.join(ships)}")

def sailors(greeting, name): 
    message = f'{greeting} {name}'
    print(message)

class Cli(): 

    def __init__(self):
        self.sailors = sailors
        self.ships = Ships()

if __name__ == '__main__':
    fire.Fire(Cli) 

Define a class for the ships commands.


sailors has no subcommands, so it can be defined as a function.


Define a class to act as the top group. Add the sailors function and the Ships as attributes of the class.


Call fire.Fire on the class acting as the top-level group.

The automatically generated documentation at the top level represents the Ships class as a group, and the sailors command as a command:

$ ./fire_example.py

NAME
    fire_example.py

SYNOPSIS
    fire_example.py GROUP | COMMAND

GROUPS
    GROUP is one of the following:

     ships

COMMANDS
    COMMAND is one of the following:

     sailors
(END)
The documentation for the ships group shows the commands representing the methods attached to the Ships class:

$ ./fire_example.py ships --help
INFO: Showing help with the command 'fire_example.py ships -- --help'.

NAME
    fire_example.py ships

SYNOPSIS
    fire_example.py ships COMMAND

COMMANDS
    COMMAND is one of the following:

     list

     sail
(END)
The parameters for the sailors function are turned into positional arguments:

$ ./fire_example.py sailors --help
INFO: Showing help with the command 'fire_example.py sailors -- --help'.

NAME
    fire_example.py sailors

SYNOPSIS
    fire_example.py sailors GREETING NAME

POSITIONAL ARGUMENTS
    GREETING
    NAME

NOTES
    You can also use flags syntax for POSITIONAL ARGUMENTS
(END)
You can call the commands and subcommands as expected:

$ ./fire_example.py ships sail
Your ship is setting sail
$ ./fire_example.py ships list
Ships: John B,Yankee Clipper,Pequod
$ ./fire_example.py sailors Hiya Karl
Hiya Karl
An exciting feature of fire is the ability to enter an interactive mode easily. By using the --interactive flag, fire opens an IPython shell with the object and functions of your script available:

$ ./fire_example.py sailors Hiya Karl -- --interactive
Hiya Karl
Fire is starting a Python REPL with the following objects:
Modules: fire
Objects: Cli, Ships, component, fire_example.py, result, sailors, self, trace

Python 3.7.0 (default, Sep 23 2018, 09:47:03)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.5.0 -- An enhanced Interactive Python. Type '?' for help.
 ---------------------------------------------------------------------------
In [1]: sailors
Out[1]: <function __main__.sailors(greeting, name)>

In [2]: sailors('hello', 'fred')
hello fred
Here we run the maritime program’s sailors command in interactive mode. An IPython shell opens, and you have access to the sailors function. This interactive mode, in combination with the ease of exposing objects with fire, makes it the right tool both for debugging and introducing yourself to new code.

You have now run the gamut in command-line tool building libraries, from the very hands-on argparse, to the less verbose click, and lastly to the minimal fire. So which one should you use? We recommend click for most use cases. It balances ease and control. In the case of complex interfaces where you want to separate the UI code from business logic, argparse is the way to go. Moreover, if you need to access code that does not have a command-line interface quickly, fire is right for you.

Implementing Plug-ins
Once you’ve implemented your application’s command-line user interface, you might want to consider a plug-in system. Plug-ins are pieces of code supplied by the user of your program to extend functionality. Plug-in systems are used in all sorts of applications, from large applications like Autodesk’s Maya to minimal web frameworks like Flask. You could write a tool that handles walking a filesystem and allows a user to provide plug-ins to operate on its contents. A key part of any plug-in system is plug-in discover. Your program needs to know what plug-ins are available to load and run. In Example 3-7, we write a simple application that discovers and runs plug-ins. It uses a user-supplied prefix to search for, load, and run plug-ins.

Example 3-7. simple_plugins.py
#!/usr/bin/env python
import fire
import pkgutil
import importlib

def find_and_run_plugins(plugin_prefix):
    plugins = {}

    # Discover and Load Plugins
    print(f"Discovering plugins with prefix: {plugin_prefix}")
    for _, name, _ in  pkgutil.iter_modules(): 
        if name.startswith(plugin_prefix): 
            module = importlib.import_module(name) 
            plugins[name] = module

    # Run Plugins
    for name, module in plugins.items():
        print(f"Running plugin {name}")
        module.run()  

if __name__ == '__main__':
    fire.Fire()

pkgutil.iter_modules returns all modules available in the current sys.path.


Check if the module uses our plug-in prefix.


Use importlib to load the module, saving it in a dict for later use.


Call the run method on the plug-in.

Writing supplying plug-ins to Example 3-7 is as simple as supplying modules whose names use a shared prefix and whose functionality is accessed using a method named run. If you write two files using the prefix foo_plugin with individual run methods:

def run():
    print("Running plugin A")
def run():
    print("Running plugin B")
You can discover and run them with our plugin application:

$ ./simple_plugins.py find_and_run_plugins foo_plugin
Running plugin foo_plugin_a
Running plugin A
Running plugin foo_plugin_b
Running plugin B
You can easily extend this simple example to create plug-in systems for your applications.

Case Study: Turbocharging Python with Command-Line Tools
It’s as good a time as ever to be writing code these days; a little bit of code goes a long way. Just a single function is capable of performing incredible things. Thanks to GPUs, machine learning, the cloud, and Python, it’s easy to create “turbocharged” command-line tools. Think of it as upgrading your code from using a basic internal combustion engine to a jet engine. What’s the basic recipe for the upgrade? One function, a sprinkle of powerful logic, and, finally, a decorator to route it to the command line.

Writing and maintaining traditional GUI applications—web or desktop—is a Sisyphean task at best. It all starts with the best of intentions, but can quickly turn into a soul crushing, time-consuming ordeal where you end up asking yourself why you thought becoming a programmer was a good idea in the first place. Why did you run that web framework setup utility that essentially automated a 1970s technology—the relational database—into series of Python files? The old Ford Pinto with the exploding rear gas tank has newer technology than your web framework. There has got to be a better way to make a living.

The answer is simple: stop writing web applications and start writing jet-powered command-line tools instead. The turbocharged command-line tools discussed in the following sections are focused on fast results vis-à-vis minimal lines of code. They can do things like learn from data (machine learning), make your code run two thousand times faster, and best of all, generate colored terminal output.

Here are the raw ingredients that will be used to make several solutions:

Click framework

Python CUDA framework

Numba framework

Scikit-learn machine learning framework

Using the Numba Just-in-Time (JIT) Compiler
Python has a reputation for slow performance because it’s fundamentally a scripting language. One way to get around this problem is to use the Numba Just-in-Time (JIT) compiler. Let’s take a look at what that code looks like. You can also access the full example in GitHub.

First, use a timing decorator to get a grasp on the runtime of your functions:

def timing(f):
    @wraps(f)
    def wrap(*args, **kwargs):
        ts = time()
        result = f(*args, **kwargs)
        te = time()
        print(f"fun: {f.__name__}, args: [{args}, {kwargs}] took: {te-ts} sec")
        return result
    return wrap
Next, add a numba.jit decorator with the nopython keyword argument and set it to True. This will ensure that the code will be run by JIT instead of regular Python.

@timing
@numba.jit(nopython=True)
def expmean_jit(rea):
    """Perform multiple mean calculations"""

    val = rea.mean() ** 2
    return val
When you run it, you can see both a jit as well as a regular version being run via the command-line tool:

$ python nuclearcli.py jit-test
Running NO JIT
func:'expmean' args:[(array([[1.0000e+00, 4.2080e+05, 2350e+05, ...,
                                  1.0543e+06, 1.0485e+06, 1.0444e+06],
       [2.0000e+00, 5.4240e+05, 5.4670e+05, ...,
              1.5158e+06, 1.5199e+06, 1.5253e+06],
       [3.0000e+00, 7.0900e+04, 7.1200e+04, ...,
              1.1380e+05, 1.1350e+05, 1.1330e+05],
       ...,
       [1.5277e+04, 9.8900e+04, 9.8100e+04, ...,
              2.1980e+05, 2.2000e+05, 2.2040e+05],
       [1.5280e+04, 8.6700e+04, 8.7500e+04, ...,
              1.9070e+05, 1.9230e+05, 1.9360e+05],
       [1.5281e+04, 2.5350e+05, 2.5400e+05, ..., 7.8360e+05, 7.7950e+05,
        7.7420e+05]], dtype=float32),), {}] took: 0.0007 sec
$ python nuclearcli.py jit-test --jit
Running with JIT
func:'expmean_jit' args:[(array([[1.0000e+00, 4.2080e+05, 4.2350e+05, ...,
                                     0543e+06, 1.0485e+06, 1.0444e+06],
       [2.0000e+00, 5.4240e+05, 5.4670e+05, ..., 1.5158e+06, 1.5199e+06,
        1.5253e+06],
       [3.0000e+00, 7.0900e+04, 7.1200e+04, ..., 1.1380e+05, 1.1350e+05,
        1.1330e+05],
       ...,
       [1.5277e+04, 9.8900e+04, 9.8100e+04, ..., 2.1980e+05, 2.2000e+05,
        2.2040e+05],
       [1.5280e+04, 8.6700e+04, 8.7500e+04, ..., 1.9070e+05, 1.9230e+05,
        1.9360e+05],
       [1.5281e+04, 2.5350e+05, 2.5400e+05, ..., 7.8360e+05, 7.7950e+05,
@click.option('--jit/--no-jit', default=False)
        7.7420e+05]], dtype=float32),), {}] took: 0.2180 sec
How does that work? Just a few lines of code allow for this simple toggle:

@cli.command()
def jit_test(jit):
    rea = real_estate_array()
    if jit:
        click.echo(click.style('Running with JIT', fg='green'))
        expmean_jit(rea)
    else:
        click.echo(click.style('Running NO JIT', fg='red'))
        expmean(rea)
In some cases, a JIT version could make code run thousands of times faster, but benchmarking is key. Another item to point out is this line:

click.echo(click.style('Running with JIT', fg='green'))
This script allows for colored terminal output, which can be very helpful when creating sophisticated tools.

Using the GPU with CUDA Python
Another way to turbocharge your code is to run it straight on a GPU. This example requires you run it on a machine with a CUDA enabled. Here’s what that code looks like:

@cli.command()
def cuda_operation():
    """Performs Vectorized Operations on GPU"""

    x = real_estate_array()
    y = real_estate_array()

    print("Moving calculations to GPU memory")
    x_device = cuda.to_device(x)
    y_device = cuda.to_device(y)
    out_device = cuda.device_array(
        shape=(x_device.shape[0],x_device.shape[1]), dtype=np.float32)
    print(x_device)
    print(x_device.shape)
    print(x_device.dtype)

    print("Calculating on GPU")
    add_ufunc(x_device,y_device, out=out_device)

    out_host = out_device.copy_to_host()
    print(f"Calculations from GPU {out_host}")
It’s useful to point out that if the Numpy array is first moved to the GPU, then a vectorized function does the work on the GPU. After that work is completed, the data is moved from the GPU. By using a GPU, there could be a monumental improvement to the code, depending on what it’s running. The output from the command-line tool is shown here:

$ python nuclearcli.py cuda-operation
Moving calculations to GPU memory
<numba.cuda.cudadrv.devicearray.DeviceNDArray object at 0x7f01bf6ccac8>
(10015, 259)
float32
Calculating on GPU
Calculcations from GPU [
 [2.0000e+00 8.4160e+05 8.4700e+05 ... 2.1086e+06 2.0970e+06 2.0888e+06]
 [4.0000e+00 1.0848e+06 1.0934e+06 ... 3.0316e+06 3.0398e+06 3.0506e+06]
 [6.0000e+00 1.4180e+05 1.4240e+05 ... 2.2760e+05 2.2700e+05 2.2660e+05]
 ...
 [3.0554e+04 1.9780e+05 1.9620e+05 ... 4.3960e+05 4.4000e+05 4.4080e+05]
 [3.0560e+04 1.7340e+05 1.7500e+05 ... 3.8140e+05 3.8460e+05 3.8720e+05]
 [3.0562e+04 5.0700e+05 5.0800e+05 ... 1.5672e+06 1.5590e+06 1.5484e+06]
]
Running True Multicore Multithreaded Python Using Numba
One common performance problem with Python is the lack of true, multithreaded performance. This also can be fixed with Numba. Here’s an example of some basic operations:

@timing
@numba.jit(parallel=True)
def add_sum_threaded(rea):
    """Use all the cores"""

    x,_ = rea.shape
    total = 0
    for _ in numba.prange(x):
        total += rea.sum()
        print(total)

@timing
def add_sum(rea):
    """traditional for loop"""

    x,_ = rea.shape
    total = 0
    for _ in numba.prange(x):
        total += rea.sum()
        print(total)

@cli.command()
@click.option('--threads/--no-jit', default=False)
def thread_test(threads):
    rea = real_estate_array()
    if threads:
        click.echo(click.style('Running with multicore threads', fg='green'))
        add_sum_threaded(rea)
    else:
        click.echo(click.style('Running NO THREADS', fg='red'))
        add_sum(rea)
Note that the key difference between the parallel version is that it uses @numba.jit(parallel=True) and numba.prange to spawn threads for iteration. As you can see in Figure 3-1, all of the CPUs are maxed out on the machine, but when almost the exact same code is run without the parallelization, it only uses a core.


Figure 3-1. Using all of the cores
$ python nuclearcli.py thread-test
$ python nuclearcli.py thread-test --threads
KMeans Clustering
Another powerful thing that can be accomplished with a command-line tool is machine learning. In the example below, a KMeans clustering function is created with just a few lines of code. This clusters a Pandas DataFrame into a default of three clusters:

def kmeans_cluster_housing(clusters=3):
    """Kmeans cluster a dataframe"""
    url = "https://raw.githubusercontent.com/noahgift/\
           socialpowernba/master/data/nba_2017_att_val_elo_win_housing.csv"
    val_housing_win_df =pd.read_csv(url)
    numerical_df =(
        val_housing_win_df.loc[:,["TOTAL_ATTENDANCE_MILLIONS", "ELO",
        "VALUE_MILLIONS", "MEDIAN_HOME_PRICE_COUNTY_MILLIONS"]]
    )
    #scale data
    scaler = MinMaxScaler()
    scaler.fit(numerical_df)
    scaler.transform(numerical_df)
    #cluster data
    k_means = KMeans(n_clusters=clusters)
    kmeans = k_means.fit(scaler.transform(numerical_df))
    val_housing_win_df['cluster'] = kmeans.labels_
    return val_housing_win_df
The cluster number can be changed by passing in another number (as shown below) using click:

@cli.command()
@click.option("--num", default=3, help="number of clusters")
def cluster(num):
    df = kmeans_cluster_housing(clusters=num)
    click.echo("Clustered DataFrame")
    click.echo(df.head())
Finally, the output of the Pandas DataFrame with the cluster assignment is shown next. Note that it now has cluster assignment as a column:

$ python -W nuclearcli.py cluster
Clustered DataFrame
               TEAM  GMS    ...         COUNTY   cluster
0     Chicago Bulls   41    ...           Cook         0
1  Dallas Mavericks   41    ...         Dallas         0
2  Sacramento Kings   41    ...     Sacremento         1
3        Miami Heat   41    ...     Miami-Dade         0
4   Toronto Raptors   41    ...    York-County         0

[5 rows x 12 columns]
$ python -W nuclearcli.py cluster --num 2
Clustered DataFrame
               TEAM  GMS     ...         COUNTY   cluster
0     Chicago Bulls   41     ...           Cook         1
1  Dallas Mavericks   41     ...         Dallas         1
2  Sacramento Kings   41     ...     Sacremento         0
3        Miami Heat   41     ...     Miami-Dade         1
4   Toronto Raptors   41     ...    York-County         1

[5 rows x 12 columns]
Exercises
Use sys to write a script that prints command line only when run from the command line.

Use click to create a command-line tool that takes a name as an argument and prints it if it does not begin with a p.

Use fire to access methods in an existing Python script from the command line.