---
layout: post
title:  "Creating a Python Application Accessible via Command Line"
date:   2019-06-11 
categories: update
comments: false
---


# Creating a Python Application Accessible via Command Line

Once we wrote a Python Script there are many ways to execute it. The most obvious way is to execute the file containing the python script or the file containing \_\_main__  method (myfile.py) with python command as follows.

```
python calc.py
```

What is we need to create our own entry point in command line, like python in the above example? We can achive this easily using [setuptools](https://pypi.org/project/setuptools/). I assume you have already installed pip. 

First install setuptools 

```
pip install setuptools
```

Let's begin with our code now. Following is my simple program that I need to access via a command line entry point. That is in calc.py. I would like to print an introduction phrase as one access my program in comman line. So the code goes as follows. 


```
#calc.py:
def intro():
  print('This is Calculator program')
```
In order to install our program in a way accessible via command line let's make a setup.py file.


```
#setup.py:
from setuptools import setup

setup(
    name="calculator",
    version='0.1',
    entry_points='''
        [console_scripts]
        mycalc=calc:intro
    ''',
)
```

So in our setup.py the line mycalc=calc:intro denotes that the entry point for the program will be a command 'mycalc' that user would type in the command line which will trigger the intro() function in calc.py.

Now that we have the setup.py, let's install our program by following command.

```
pip install --editable .
```
The `--editable' says that the program is in development mode. 

## Adding Parameters to the Command

So far we know how to call an entry point to our application from the command line. Let's add some functionalities to the calculator application and call them from the command line. For example in order to add two numbers, say 134 and 27, I hope to call an add function with two parameter arguments 134 and 27 as in, 'mycalc add 134 27'.

```
#calc.py:

import sys

def intro():
  print('This is Calculator program')
  run_command()

def add(a,b):
    print(a+b)

def sub(a,b):
    print(a-b)

def mul(a,b):
    print(a*b)

def div(a,b):
    print(a/b)

def print_help():
    print('calc <command> [options]\n')
    print('Commands:')
    print('add a b\t\t Perform addition over the numbers a and b')
    print('sub a b\t\t Perform subtraction over the numbers a and b')
    print('mul a b\t\t Perform multiplication over the numbers a and b')
    print('div a b\t\t Perform division over the numbers a and b')
    

def run_command():
	args = sys.argv # get command line arguments
	
	if len(args) == 1:
		print_help()
	elif len(args) == 4:
            command = args[1] 
            a,b = int(args[2]),int(args[3])
            if command == 'add':
                    add(a,b)
            elif command == 'sub':
                    sub(a,b)
            elif command == 'mul':
                    mul(a,b)
            elif command == 'div':
                    div(a,b)
	else:
            print('Unrecognised argument.')

```

The run_options() function mediate the command line commands. sys.argv returns the line of command entered by the user in the command line, as a list of words separated by spaces. In this application if only "mycalc" is entered in the command line, it will display a guide to the available commands. If the command is in expected length, the second word is considered as the mathematical operation to perform and next two arguments are considered as the operands. Obviously, this is not a perfect application and there are lot of error handling to be done, such as handling casting errors.

Now when I saved mycalc.py file and run 'mycalc add 134 27' it returns the following output.

```
$mycalc add 134 27
>>>This is Calculator program
>>>161
```

## Using tools for the command line interface

Well, now we know how to write a command line interface to our program. However, this is too much of work when we have a significantly large real application to develop. To make our life easy, there are many tools available which can handle the command line interface for us. In this tutorial I use [click](https://palletsprojects.com/p/click/). 

'Click' provides a well explained [documentation](https://click.palletsprojects.com/en/7.x/), you can have a look to this for a well structured understanding. Here I will simply convert the calculator application into code with click. With click we can omit entire `run_option()` method in the previous calc.py file. The steps to convert our calc.py and setup.py are mentioned below.        

#####Modifications on calc.py

```
#calc.py

import click

@click.group()
def intro():
  """This is Calculator program"""

@intro.command()
@click.argument('a', type=int)
@click.argument('b', type=int)
@click.option("-e", "--explain",is_flag=True, help='get the answer with explanation') 
def add(a,b,explain):
    """ Perform addition over two numbers """
    if(explain):
      click.echo('%d + %d = %d ' %(a, b, (a+b)))
    else:
      click.echo(a+b)

@intro.command()
@click.argument('a', type=int)
@click.argument('b', type=int)
def sub(a,b):
    """ Perform subtraction over two numbers """   
    click.echo(a-b)


@intro.command()
@click.argument('a', type=int)
@click.argument('b', type=int)
def mul(a,b):
    """ Perform multiplication over two numbers """   
    click.echo(a*b)

@intro.command()
@click.argument('a', type=int)
@click.argument('b', type=int)
def div(a,b):
    """ Perform division over two numbers """   
    click.echo(a/b)
```

1. import click
2. **Creating commands** : Add `@click.command()` decorator to the functions that you need to run when called in command line. In our application those will be add(a,b), sub(a,b), mul(a,b) and div(a,b) functions. 
3. **Command groups** : In addition to defining commands we need to access them in the command line via an entry point, like we called `run_option()` via `intro()` entry point previously. Therefore in order to make the `intro()` function the entry point I define `intro()` as a group of commands, where other commands should be called after calling `intro()` in a hierarchical manner, just as we did before. So that I add `@click.group()` decorator to `intro()` and make other functions its sub commands by adding `**@intro**.command` instead of `@click.command`. Another way to get the same result is using `@click.command()` decorative and later adding required commands to the group via `@intro.add_command(<calling function>)`.

4. **Command arguments** : In order to add parameter arguments add `@click.argument()` decorative. You can pass the name of the argument and type of the argument as decorative parameters. (There are more available, you can always check the user friendly [documentation](https://click.palletsprojects.com/en/7.x/) of click)
5. **Command options** : You can easily add command line [options using click](https://click.palletsprojects.com/en/7.x/options/). An option is a way to change the behaviour of the calling function. For example, in `pip install --editable .` command that we executed before, `--editable` is an option to say that installation should happen in development mode. As an example to the option, I added '--explain' option to the add(a,b,) method. When add command is executed with `--explain` option (`mycalc add 1 2 --explain`), that will return the answer with the operation ( `1 + 2 = 3`). The first string parameter of the option decoration takes command line string to call the option. 'is_Flag=True' says that the option is denoting a Boolean value. You can add the options help menu by `help=''"` argument.
6. **Help menu**: With click we do not have to worry about generating a help menu. The comments you add after the function definition will be taken as the help description, and will be executed with the --help option or with no arguments as expected. For command arguments and options that you define, you can add help menu with `help=""` parameter as mentioned above.


#####Modifications on setup.py

It is not mandatory to use a setup.py with click. But in order to defining the entry point (call `mycalc' instead of `python calc.py`) we can use setuptools. 

```
#setup.py

from setuptools import setup

setup(
    name="calculator",
    version='0.2',
    install_requires=[
        'Click',
    ],
    entry_points='''
        [console_scripts]
        mycalc=calc:intro
    ''',
)
```

In addition to the previous code, here I have added `install_requires=['Click',]` line to install click library in case it is not already installed in the system.
 
Well, still this description is only a basic example and definitely not enough to build a good application with command line interface. Please check the documentation for more details.          




##### Resources
https://www.makeuseof.com/tag/python-command-line-programs-click/
https://pymbook.readthedocs.io/en/latest/click.html
https://setuptools.readthedocs.io/en/latest/
https://click.palletsprojects.com/en/7.x/quickstart/





















