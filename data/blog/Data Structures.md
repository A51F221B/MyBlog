---
title: Data Structures in python
date: '2023-08-02'
tags: ['python', 'Programming', 'Datastructures','python3','python programming']
draft: false
summary: Python like other programming languages has a lot of data structures that can be used to store data in different ways.Each data structure has its own advantages and disadvantages.

---

# Python Data Structures
---

## Lists
To create a list in python we have to use `[]` brackets.Anything in between these brackets will be the data of that particular list.

```python
courses=['Maths','programming','science']
print(courses)

# output :
┌─[root@parrot]─[/home/python]
└──╼ #python3 practice.py 
['maths', 'programming', 'science']
```

When we use the `print(courses)` it prints the entire list.But mostly we are working with individual items from the list.So to access individual item we use the index number of the item we want to access.List index starts from zero (in this case 'Maths' is on zero 0 index).

```python
courses=['maths','programming','science']
print(courses[0]) # will print maths
print(courses[1]) # will print programming
print(courses[2]) # will print science
```

Few characteristics of lists are 
- Lists are in ordered form
- They can be accessed by index
- They can contain any arbitrary object
- List are dynamic
- A list can be nested inside another list

### List Nesting
We can put a list inside another list! Cool,Right? 
```python

courses=['maths','physics','chemistry',['programming fundamentals','Data structures','Web'],['Ethical hacking','databases',['data science','machine learning']]]

print(courses[0])
print(courses[3])
print(courses[3][1])
print(courses[4])
print(courses[4][2][1])

```

Output of this code is

```bash
┌─[✗]─[root@parrot]─[/home/asif221b/Documents/python]
└──╼ #python3 practice.py 
maths
['programming fundamentals', 'Data structures', 'Web']
Data structures
['Ethical hacking', 'databases', ['data science', 'machine learning']]
machine learning
```

# Lambda Functions

Learn how to define a Python function in one line!

A _function_ is an object that is able to accept some sort of input, possibly modify it, and return some sort of output. In Python, a _lambda function_ is a one-line shorthand for function. A simple lambda function might look like this:

```
add_two = lambda my_input: my_input + 2
```

So this code:

```
print(add_two(3))print(add_two(100))print(add_two(-2))
```

would print:

```
>>> 5>>> 102>>> 0
```

Let’s break this syntax down:

1.  The function is stored in a variable called `add_two`
2.  `lambda` declares that this is a lambda function (if you are familiar with normal Python functions, this is similar to how we use `def` to declare a function)
3.  `my_input` is what we call the input we are passing into `add_two`
4.  We are returning `my_input` plus 2 (with normal Python functions, we use the keyword `return`)

Let’s write a lambda function that checks if a string is a substring of the string `“This is the master string”`.

```
is_substring = lambda my_string: my_string in "This is the master string"
```

So, the code:

```python
print(is_substring('I'))print(is_substring('am'))print(is_substring('the'))print(is_substring('master'))
```

would print:

```
>>> False>>> False>>> True>>> True
```

We might want a function that will perform differently based on different inputs. Let’s say that we have a function `check_if_A_grade` that outputs `'Got an A!'` if a grade is at least 90, and otherwise says you `'Did not get an A…'`. So, the code:

```
print(check_if_A_grade(91))print(check_if_A_grade(70))print(check_if_A_grade(20))
```

would print:

```
>>> 'Got an A!'>>> 'Did not get an A...'>>> 'Did not get an A...'
```

We can do this using an _if statement_ in our lambda function, with syntax that looks like:

```
<WHAT TO RETURN IF STATEMENT IS TRUE> if <IF STATEMENT> else <WHAT TO RETURN IF STATEMENT IS FALSE>
```

So this is what our `check_if_A_grade` function might look like:

```
check_if_A_grade = lambda grade: 'Got an A!' if grade >= 90 else 'Did not get an A...'
```

This is what this line of code does:

1.  Declare lambda function with an input called `grade` (`lambda grade:`)
2.  Return `'Got an A!'` if this statement is true:
    
    ```
    grade >= 90
    ```
    
3.  Otherwise, return `'Did not get an A...'` if this statement is not true:
    
    ```
    grade >= 90
    ```
    

Lambda functions only work if we’re just doing a one line command. If we wanted to something longer, we’d need a more complex function. Lambda functions are great when you need to use a function once. Because you aren’t defining a function, the reusability aspect functions is not present with lambda functions. By saving the work of defining a function, a lambda function allows us to efficiently run an expression and produce an output for a specific task, such as defining a column in a table, or populating information in a dictionary.

Now you can make simple Python functions in one line!