---
title: "Structure and Organization"
teaching: 15
exercises: 0
questions:
- "How can code be best organized to make it easier to understand and maintain it?"
objectives:
- "At the end of this module you should have a better understanding of how the different parts of code can be connected in a way so your code is better maintainable."
keypoints:
- ""
---
# Layer 2: Structure and Organization

## The Dependency Hell

<div style="float:right; padding-left: 10px; ">
<a href="https://xkcd.com/2347/" target="_blank">
<img src="/assets/images/dependency_2x.png" alt="XKCD - Dependency" width="280px">
</a>
</div>

Dependency hell refers to the situation in which your code depends on a number of other packages (dependencies) that themselves depend on other packages that depend on other packages, and so on. Some of your dependencies might use the same packages as some other dependencies but a different version of them. Or maybe the dependencies you are using are only available for an outdated version of the programming language you are using and that keeps you from upgrading your code. The more dependencies your code has, the more complex and harder to update it becomes. Therefore, you should carefully choose any package your code *depends* on.

When you choose to use another package, first check who maintains that package and when it was last updated. Is it actively maintained and has a community that reports bugs and makes pull requests? Is it potentially backed by a company? How long has it been since the last release and the last commit? Does the maintainer respond to issues created? By answering these questions, you will get a better sense of how actively maintained the package is. If there hasn’t been much activity in the last few weeks or months, it might be an indicator that the package has been abandoned or will be soon. In that case, you want to be careful about depending on it in your code. If there is not much activity but you still want to use a certain package, then you should be prepared to maintain a fork of the package yourself should it become necessary.

<div style="float:clear"></div>

## Code Coupling

In software development, “coupling” refers to how two pieces of code are connected with each other. Coupling happens on a scale from tight to loosely. The tighter two parts of a system are coupled, the more they need to know about each other.

Cleancommit.io defines loose coupling like this:

> “In a loosely coupled system, the components are independent of each other. Each component has its own well-defined interface and communicates with other components through standardized protocols. Changes to one component do not require changes to other components, making the system more flexible and easier to maintain.”
>
> Source: <a href="https://cleancommit.io/blog/whats-the-difference-between-tight-and-loose-coupling/">cleancommit.io</a>

Tight coupling is the opposite of loose coupling, which means that when you change one component you need to change the other one as well to adapt it to the changes of the first component.

Coupling happens everywhere. Between modules, between classes, between layers, between components, and between whole systems. In a well-designed system, things that logically belong together will be kept together (*cohesion*) and the different components don’t know much about the inner workings of each other but communicate over clearly defined interfaces (*coupling*).


As an example, look at the following code and think about why this is tightly coupled. How could it be more loosely coupled? Then check the discussion below.

~~~
def addition():
    num_1 = float(raw_input("Enter Number One"))
    num_2 = float(raw_input("Enter Number Two"))
    addition = num_1 + num_2
    print addition
~~~
{: .language-python }

> ## Discussion
> The function first asks the user to input two numbers. It then adds the two numbers and prints the result. The means of retrieving the two numbers and then add them are tightly coupled as we would have to change this function if wanted the numbers to come from somewhere (e.g. a different calculation). This function can only be used in one scenario, when the user is supposed to enter two numbers that are then added. The following code, achieves the same but decouples the input retrieval from the adding step.
> ~~~
> def addition(num_1, num_2):
>    addition = num_1 + num_2
>    print addition
>
> num_1 = float(raw_input("Enter Number One"))
> num_2 = float(raw_input("Enter Number Two"))
> 
> addition(num_1, num_2)
> ~~~
> {: .language-python }
> 
> In this code, the function `addition` can be called with any two numbers independent from where those numbers are coming from. If we want to change the code so that the numbers are read from a file, all we need to change are the two lines in which `num_1` and `num_2` are defined.
> 
{: .solution}

## How to know if something is tightly coupled?

You might be wondering how you can know if something is tightly coupled. In theory, it is fairly easy to find out, just ask the following question: If I want to replace component A, do I need to change component B a lot?

If the answer is yes, then your two components are tightly coupled. If the answer is no, then they are loosely coupled. There are degrees of tight coupling, some components are more tightly coupled than others. Similarly, some components are more loosely coupled than others. You won’t always achieve the loosest coupling possible, nor is it always recommendable. This is because, in many cases, coupling two components more loosely comes with the cost of more complexity. And the more complexity there is, the harder it can be to maintain your code.

For instance, look at the following piece of code, does it look familiar?

~~~
def addition(numbers):
    total = 0
    for i in numbers:
        total = total + i
    return total

def get_number(question):
    response = raw_input(question)
    num = 0
    try:
        num = float(response)
    except:
        num = 0
    return num

num_numbers = get_number("How many values would you like to add? ")
numbers = []
for i in range(0, num_numbers):
    numbers.append(get_number("Please enter a number: "))

answer = addition(numbers)
print "The answer to the addition is: %d" % (answer)
~~~
{: .language-python}

This is the same code from before that adds two numbers. However, it is as loosely coupled as it can be. The different pieces of code are not only loosely coupled to each other, but the code is also loosely coupled to its purpose. It doesn’t have to be two numbers anymore that are being added but can be however many numbers need to be summed up. However, the code got considerably longer and it is a lot harder to understand its purpose. Therefore, you should always ask yourself, do I need my code to be more loosely coupled or more generalized or is this the right balance between loose coupling and understandability?

## Means of Decoupling

The goal of many software engineering best practices is to loosely couple different parts of a system. The following is a (non-exclusive) list of techniques that aim to decouple code (make it more loosely coupled).

- **Information Hiding**
- **Abstraction**
- **Single Responsibility Principle**
- Modularization
- Software Design Patterns
- Well-defined APIs
- Configuration Management
- Dependency Injection
- Microservices

In the following, we will talk about the first three in a little bit more detail.

## Information Hiding 

*Information hiding*, or sometimes also referred to as *encapsulation*, describes the practice of “hiding” the inner workings of a piece of code from whoever calls it. The goal of this technique is to avoid that one has to change the caller when the callee is changed. If calling function A requires knowledge about how A is implemented, it not only tightly couples function A with the code that calls it, it also makes it more difficult to understand the code as looking at the function name and its parameters does not provide all the details needed to use function A.

Some programming languages (like Java or C#) provide means of hiding information that are part of the standard curriculum when the language is taught. Other languages (like Python) offer no or only limited support. Especially in these cases, it is important to learn the standard coding conventions of the language and follow them.

As an example for information hiding, let’s look at the following piece of code. There are two files, one that defines the function `say_something` and one that uses it. If you want to change the animal that makes the sound, you need to know that there is a global variable in `sound.py` that you need to change. 

~~~
# sound.py
animal_type = "cat"

def say_something(sound):
    print("The {} says {}.".format(animal_type, sound))


# script.py
import sound

sound.animal_type = "dog" # violates information hiding principle
sound.say_something("bark")
~~~
{: .language-python}

The following piece of code hides the information about the global variable. Instead, a function parameter is added that you can use to set the animal. If the implementation of `say_something_animal` changes to for example set a global variable to the passed in value, then the call in `script.py` does not have to change at all. The internal workings of `say_something_animal` stay completely hidden.

~~~
# sound.py
def say_something_animal(sound, animal_type="cat"):
    print("The {} says {}.".format(animal_type, sound))


# script.py
import sound

sound.say_something("bark", animal_type="dog")
~~~
{: .language-python }