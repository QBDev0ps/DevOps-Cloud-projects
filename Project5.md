## DOCUMENTATION FOR SHELL SCRIPTING (HANDS ON) PROJECT

This project shows to write shell scripts. It covers essential aspects of Shell Scripting from basic commands to advanced automation and it enables us learn how to streamline tasks and boost productivity.

### <br>Introduction to Shell Scripting and User Inputs<br/>

A [Shell Script](https://en.wikipedia.org/wiki/Shell_script) is a collection of commands and instructions that are executed sequentially in a shell. Essentially, shell scripting helps us automate repetitive tasks. For instance, rather than type the **`git clone`** command 1000 times to clone 1000 repositories, we can simply write a script that does this job and then we can call the script whenever we want to perform the task. Shell scripts are created by saving a series of commands in a text file with **.sh** extension. These scripts can then be executed directly from the command line or called from another script.

### <br>Shell Scripting Syntax Elements<br/>

1. **Variables:** Bash allows a user to define and work with variables. Variables can store various data types such as numbers strings and arrays. Values can be assigned to variables using the **=** operator. And a variable's value can be accessed by preceeding the variable name with a **$** sign.

For example, to assign value to a variable we wnter the command below:

**`name="Moyosore"`**

To retrieve the value from a variable:

**`echo $name`**

![echo name](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/bf1b48ed-113a-4fc9-89a0-ffc77d51dfc0)

2. **Control Flow:** Bash enables us to contol the flow of execution in our scripts by implementing the use of control flow statements such as **if-else**, **else-if (elsif)**, **for loops**, **while loops** and **case** statements. All these statements allow us make decisions and execute different commands based on different conditions.

For Example, to use _**if-else**_ to execute a script based on conditions, we enter the following:

```
#!/bin/bash

# Example script to check if a number is positive, negative, or zero

read -p "Enter a number: " num

if [ $num -gt 0 ]; then
    echo "The number is positive."
elif [ $num -lt 0 ]; then
    echo "The number is negative."
else
    echo "The number is zero."
fi
```

As shown in the output image below, the above piece of code prompts us to type a number and prints a statement stating if the number is positive or negative:

![if-else statement](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/845b2327-6a62-4e08-8d4e-9802fe6742fa)

To illustrate another example, we can use a _**for loop**_ to iterate through a list with the following commnad:

```
#!/bin/bash

# Example script to print numbers from 1 to 5 using a for loop

for (( i=1; i<=5; i++ ))
do
    echo $i
done
```

The output of the command is shown below:

![For Loop](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/602a7fe1-8258-4c66-a4db-919b21b4c05b)

3. **Command Substitution:** This enables us to capture the output of a command and use it as a value within our script. we can use the bactick **`** or the **$()** syntax for command substitution.

For example, to use bactick for command substitution, we enter the following:

```
current_date=`date +%Y-%m-%d`
```

To use $() syntax for command substitution, we enter the following:

```
current_date=$(date +%Y-%m-%d)
```

4. **Input and Output:** Bash allows users a number of ways to handle input and output. We can use the **`read`** command to accept user input and we can use the **`echo`** command to output text to the CLI. In addition, we can also redirect input and output using < (input from a file), > (output to a file), and | (use the output of one command as input to another.

An example of code accepting user input:

```
echo "Enter your name:"
read name
```

An example of code to output text to the terminal:

**`echo "Hello World"`**

An example showing how to output the result of a command into a file:

**`echo "hello world" > index.txt`**

Example showing how to pass the content of a file as input into a command:

**`grep "pattern" < input.txt`**


Example for passing the content of a command as input into another command:

**`echo "hello world" | grep "pattern"`**

5. **Functions:** Bash enables us to define and use functions so that we can group and execute related commands together. functions provide a way to modularize code and make it more reusable. Functions can be defined by the use of the function keyword or by declaring the function name followed by parentheses.

For example we define a function we call **greet** below:

```
#!/bin/bash

# Define a function to greet the user
greet() {
    echo "Hello, $1! Nice to meet you."
}

# Call the greet function and pass the name as an argument
greet "John"
```

As seen in the last line of the code above, we call the greet function and set the name "John" as the value and this results in the ouput shown below when we run this script:

![greet function](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/481f1320-2d3e-47c7-8376-cf10f416f6dc)

### We can write our first Shell Script for this project by following the steps below:

**Step 1:** To store all the scripts we will be creating in this project, we create a folder called _**shell-scripting**_ by entering the following command in our terminal:

**`mkdir shell-scripting`**

**Step 1:**
