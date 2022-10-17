---
title: Defensive Programming
teaching: 30
exercises: 10
questions:
- "How can I make my programs more reliable?"
objectives:
- "Signal errors by raising exceptions."
- "Use try-except blocks to catch and handle exceptions."
- "Add assertions that check the program's state is correct."
- "Explain what test-driven development is, and use it when creating new functions."
- "Use a logging framework to report on program activity."
keypoints:
- "Program defensively, i.e., assume that errors are going to arise,
   and write code to detect them when they do."
- "Raise exceptions in your own code."
- "Put try-except blocks in programs to catch and handle exceptions."
- "Put assertions in programs to check their state as they run,
   and to help readers understand how those programs are supposed to work."
- "Write tests before writing code in order to help determine exactly
   what that code is supposed to do."
- "Use a logging framework instead of `print` statements to report program activity."
---

Our previous lessons have introduced the basic tools of programming:
variables and lists,
file I/O,
loops,
conditionals,
and functions.
What they *haven't* done is show us how to tell
whether a program is getting the right answer,
and how to tell if it's *still* getting the right answer
as we make changes to it.

To achieve that,
there are a number of tools and approaches at our disposal.
We can:

- Raise and handle errors to check and respond to program inputs.
- Use assertions to make sure nothing crazy or unexpected has happened.
- Write unit tests to make sure each component of our program produces expected outputs.
- Use a logging framework to report on program activity.

The good news is,
doing these things will speed up our programming,
not slow it down.
As in real carpentry --- the kind done with lumber --- the time saved
by measuring carefully before cutting a piece of wood
is much greater than the time that measuring takes.

## Raising errors

One of the most commonly used tools in the defensive programming toolkit is the `raise` keyword,
which can be used to trigger an error (otherwise known as "raising an exception").

~~~
problem = True
if problem:
    raise Exception('There has been a problem')
~~~
{: .language-python}

~~~
Exception                                 Traceback (most recent call last)
<ipython-input-2-44b57a133722> in <module>
      1 if problem:
----> 2     raise Exception('There has been a problem')
      3 

Exception: There has been a problem
~~~
{: .error}

In this case we've chosen to raise a generic `Exception`,
but we could pick any of the [builtin exception types](https://docs.python.org/3/library/exceptions.html#exception-hierarchy).
It's also possible to define your own [custom exception class](https://towardsdatascience.com/how-to-define-custom-exception-classes-in-python-bfa346629bca),
but that's beyond the scope of this lesson.

A common use case for raising exceptions is to catch invalid user input.
For example,
some of the exercises in the previous lesson involved writing a rescaling function.

~~~
def rescale(input_array, low_val=0.0, high_val=1.0):
    """rescales input array values to lie between low_val and high_val"""
    L = numpy.min(input_array)
    H = numpy.max(input_array)
    intermed_array = (input_array - L) / (H - L)
    output_array = intermed_array * (high_val - low_val) + low_val
    return output_array
    
data = [1, 2, 3, 4, 5]
print(rescale(data, low_val=0, high_val=10))
~~~
{: .language-python}

~~~
[ 0.   2.5  5.   7.5 10. ]
~~~
{: .output}

We can see that the function works fine for predictable input
(in this case a scaling between 0 and 10),
but if the user mixes up the high and low value the function
happily produces a non-sensical scaling.

~~~
print(rescale(test, low_val=10, high_val=0))
~~~
{: .language-python}

~~~
[10.   7.5  5.   2.5  0. ]
~~~
{: .output}

In order to catch a high/low mix up,
we could have the function raise a `ValueError` if the high value
isn't greater than the low value.

~~~
def rescale(input_array, low_val=0.0, high_val=1.0):
    """rescales input array values to lie between low_val and high_val"""
    if low_val >= high_val:
        raise ValueError('The low_val is >= the high_val')
    L = numpy.min(input_array)
    H = numpy.max(input_array)
    intermed_array = (input_array - L) / (H - L)
    output_array = intermed_array * (high_val - low_val) + low_val
    return output_array
    
print(rescale(data, low_val=10, high_val=0))
~~~
{: .language-python}

~~~
ValueError                                Traceback (most recent call last)
<ipython-input-8-a1053c5ce6f8> in <module>
----> 1 print(rescale(data, low_val=10, high_val=0))

<ipython-input-7-ca802222cd28> in rescale(input_array, low_val, high_val)
      2     """rescales input array values to lie between low_val and high_val"""
      3     if low_val >= high_val:
----> 4         raise ValueError('The low_val is >= the high_val')
      5     L = numpy.min(input_array)
      6     H = numpy.max(input_array)

ValueError: The low_val is >= the high_val
~~~
{: .error}

## Handling errors

As we've seen in the examples above and in the previous lesson,
if exceptions aren't dealt with the program crashes.
The error message upon crashing is sometimes easy to understand
(particularly if you wrote the `raise` statement yourself)
but can often be cryptic.

If we'd rather the program didn't crash when a particular exception occurs,
we can use a try-except block to catch and handle the exception.
The syntax of the try-except block is:

~~~
try:
    <do something>
except Exception:
    <handle the error>
~~~
{: .language-python} 

The code in the except block is only executed if an exception occurred in the try block.
The except block is required with a try block, even if it contains only the `pass` statement
(i.e. ignore the exception and carry on).

For example, let's say we want to calculate the availability of medical professionals
to treat arthritis patients in American cities. 
If a given city has zero rheumatologists,
we'd be left in a situation where dividing by zero raises a `ZeroDivisionError`.

~~~
n_patients = 457
n_rheumatologists = 0
rheumatologists_per_patient = n_patients / n_rheumatologists
~~~
{: .language-python}

~~~
ZeroDivisionError                         Traceback (most recent call last)
<ipython-input-3-cad92f580d24> in <module>
----> 1 rheumatologists_per_patient = n_patients / n_rheumatologists

ZeroDivisionError: division by zero
~~~
{: .error}

If we'd prefer our program carried on by setting `rheumatologists_per_patient` to NaN
if `n_rheumatologists` is zero,
we could catch and handle the `ZeroDivisionError` as follows.

~~~
n_patients = 457
n_rheumatologists = 0
try:
    rheumatologists_per_patient = n_patients / n_rheumatologists
except ZeroDivisionError:
    rheumatologists_per_patient = numpy.nan
print(rheumatologists_per_patient)
~~~
{: .language-python}

~~~
nan
~~~
{: .output}

## Assertions

Unexpected behaviour in a program can sometimes propagate a long way
before triggering an exception or producing a perplexing result.
For instance,
if a data file contains an unrealistic inflammation value (e.g. a negative value)
that value could be used in various downstream calculations.
The final plot of average inflammation across all patients (for instance) might look wrong
(or not, which would be even worse)
to the researcher who wrote and executed the code,
but it wouldn't be immediately obvious that the inflammation data
was the source of the problem.

In order to avoid propagation,
it's best to nip unexpected behaviour in the bud right when it occurs.
One way to do this is to add [assertions]({{ page.root }}/reference.html#assertion) to your code.
An assertion is simply a statement that something must be true at a certain point in a program.
When Python sees one,
it evaluates the assertion's condition.
If it's true,
Python does nothing,
but if it's false,
Python halts the program immediately
and raises an `AssertionError` with a custom error message if one is provided.
For example,
this piece of code halts as soon as the loop encounters a value that isn't positive:

~~~
numbers = [1.5, 2.3, 0.7, -0.001, 4.4]
total = 0.0
for num in numbers:
    assert num > 0.0, 'Data should only contain positive values'
    total += num
print('total is:', total)
~~~
{: .language-python}

~~~
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
<ipython-input-19-33d87ea29ae4> in <module>()
      2 total = 0.0
      3 for num in numbers:
----> 4     assert num > 0.0, 'Data should only contain positive values'
      5     total += num
      6 print('total is:', total)

AssertionError: Data should only contain positive values
~~~
{: .error}

Assertions aren't just about catching errors:
they also help people understand programs.
Each assertion gives the person reading the program
a chance to check (consciously or otherwise)
that their understanding matches what the code is doing.

## Unit Testing and Test-Driven Development

An assertion checks that something is true at a particular point in the program.
The next step is to check the overall behavior of a piece of code,
i.e.,
to make sure that it produces the right output when it's given a particular input.
For example,
suppose we need to find where two or more time series overlap.
The range of each time series is represented as a pair of numbers,
which are the time the interval started and ended.
The output is the largest range that they all include:

![Graph showing three number lines and, at the bottom,
the interval that they overlap.](../fig/python-overlapping-ranges.svg)

Most novice programmers would solve this problem like this:

1.  Write a function `range_overlap`.
2.  Call it interactively on two or three different inputs.
3.  If it produces the wrong answer, fix the function and re-run that test.

This clearly works --- after all, thousands of scientists are doing it right now --- but
there's a better way:

1.  Write a short function for each test.
2.  Write a `range_overlap` function that should pass those tests.
3.  If `range_overlap` produces any wrong answers, fix it and re-run the test functions.

Writing the tests *before* writing the function they exercise
is called [test-driven development]({{ page.root }}/reference.html#test-driven-development) (TDD).
Its advocates believe it produces better code faster because:

1.  If people write tests after writing the thing to be tested,
    they are subject to confirmation bias,
    i.e.,
    they subconsciously write tests to show that their code is correct,
    rather than to find errors.
2.  Writing tests helps programmers figure out what the function is actually supposed to do.

Here are three test functions for `range_overlap`:

~~~
assert range_overlap([ (0.0, 1.0) ]) == (0.0, 1.0)
assert range_overlap([ (2.0, 3.0), (2.0, 4.0) ]) == (2.0, 3.0)
assert range_overlap([ (0.0, 1.0), (0.0, 2.0), (-1.0, 1.0) ]) == (0.0, 1.0)
~~~
{: .language-python}

~~~
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
<ipython-input-25-d8be150fbef6> in <module>()
----> 1 assert range_overlap([ (0.0, 1.0) ]) == (0.0, 1.0)
      2 assert range_overlap([ (2.0, 3.0), (2.0, 4.0) ]) == (2.0, 3.0)
      3 assert range_overlap([ (0.0, 1.0), (0.0, 2.0), (-1.0, 1.0) ]) == (0.0, 1.0)

AssertionError:
~~~
{: .error}

The error is actually reassuring:
we haven't written `range_overlap` yet,
so if the tests passed,
it would be a sign that someone else had
and that we were accidentally using their function.

And as a bonus of writing these tests,
we've implicitly defined what our input and output look like:
we expect a list of pairs as input,
and produce a single pair as output.

Something important is missing, though.
We don't have any tests for the case where the ranges don't overlap at all:

~~~
assert range_overlap([ (0.0, 1.0), (5.0, 6.0) ]) == ???
~~~
{: .language-python}

What should `range_overlap` do in this case:
fail with an error message,
produce a special value like `(0.0, 0.0)` to signal that there's no overlap,
or something else?
Any actual implementation of the function will do one of these things;
writing the tests first helps us figure out which is best
*before* we're emotionally invested in whatever we happened to write
before we realized there was an issue.

And what about this case?

~~~
assert range_overlap([ (0.0, 1.0), (1.0, 2.0) ]) == ???
~~~
{: .language-python}

Do two segments that touch at their endpoints overlap or not?
Mathematicians usually say "yes",
but engineers usually say "no".
The best answer is "whatever is most useful in the rest of our program",
but again,
any actual implementation of `range_overlap` is going to do *something*,
and whatever it is ought to be consistent with what it does when there's no overlap at all.

Since we're planning to use the range this function returns
as the X axis in a time series chart,
we decide that:

1.  every overlap has to have non-zero width, and
2.  we will return the special value `None` when there's no overlap.

`None` is built into Python,
and means "nothing here".
(Other languages often call the equivalent value `null` or `nil`).
With that decision made,
we can finish writing our last two tests:

~~~
assert range_overlap([ (0.0, 1.0), (5.0, 6.0) ]) == None
assert range_overlap([ (0.0, 1.0), (1.0, 2.0) ]) == None
~~~
{: .language-python}

~~~
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
<ipython-input-26-d877ef460ba2> in <module>()
----> 1 assert range_overlap([ (0.0, 1.0), (5.0, 6.0) ]) == None
      2 assert range_overlap([ (0.0, 1.0), (1.0, 2.0) ]) == None

AssertionError:
~~~
{: .error}

Again,
we get an error because we haven't written our function,
but we're now ready to do so:

~~~
def range_overlap(ranges):
    """Return common overlap among a set of [left, right] ranges."""
    max_left = 0.0
    min_right = 1.0
    for (left, right) in ranges:
        max_left = max(max_left, left)
        min_right = min(min_right, right)
    return (max_left, min_right)
~~~
{: .language-python}

Take a moment to think about why we calculate the left endpoint of the overlap as
the maximum of the input left endpoints, and the overlap right endpoint as the minimum
of the input right endpoints.
We'd now like to re-run our tests,
but they're scattered across three different cells.
To make running them easier,
let's put them all in a function:

~~~
def test_range_overlap():
    assert range_overlap([ (0.0, 1.0) ]) == (0.0, 1.0)
    assert range_overlap([ (2.0, 3.0), (2.0, 4.0) ]) == (2.0, 3.0)
    assert range_overlap([ (0.0, 1.0), (0.0, 2.0), (-1.0, 1.0) ]) == (0.0, 1.0)
    assert range_overlap([ (0.0, 1.0), (5.0, 6.0) ]) == None
    assert range_overlap([ (0.0, 1.0), (1.0, 2.0) ]) == None
~~~
{: .language-python}

We can now test `range_overlap` with a single function call:

~~~
test_range_overlap()
~~~
{: .language-python}

~~~
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
<ipython-input-29-cf9215c96457> in <module>()
----> 1 test_range_overlap()

<ipython-input-28-5d4cd6fd41d9> in test_range_overlap()
      1 def test_range_overlap():
      2     assert range_overlap([ (0.0, 1.0) ]) == (0.0, 1.0)
----> 3     assert range_overlap([ (2.0, 3.0), (2.0, 4.0) ]) == (2.0, 3.0)
      4     assert range_overlap([ (0.0, 1.0), (0.0, 2.0), (-1.0, 1.0) ]) == (0.0, 1.0)
      5     assert range_overlap([ (0.0, 1.0), (5.0, 6.0) ]) == None

AssertionError:
~~~
{: .error}

The second test fails,
so we know something is wrong with our function.
We *don't* know whether the other tests passed or failed
because Python halted the program as soon as it spotted the first error.
Still,
some information is better than none,
and if we trace the behavior of the function with that input,
we realize that we're initializing `max_left` and `min_right` to 0.0 and 1.0 respectively,
regardless of the input values.
In the case of the failed test (input ranges `[ (2.0, 3.0), (2.0, 4.0) ]`)
`min_right` starts at 1.0 and is never updated,
despite the fact that it doesn't lie within any of the input ranges.
We can update our function so that it initialises `max_left` and `min_right`
from the input data itself.

~~~
def range_overlap(ranges):
    """Return common overlap among a set of [left, right] ranges."""
    max_left, min_right = ranges[0]
    for (left, right) in ranges[1:]:
        max_left = max(max_left, left)
        min_right = min(min_right, right)
    return (max_left, min_right)
~~~
{: .language-python}

Re-running our tests we see that we now pass the second test,
but fail one of the non-overlapping cases.

~~~
test_range_overlap()
~~~
{: .language-python}

~~~
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
<ipython-input-29-cf9215c96457> in <module>()
----> 1 test_range_overlap()

<ipython-input-28-5d4cd6fd41d9> in test_range_overlap()
      3 assert range_overlap([ (2.0, 3.0), (2.0, 4.0) ]) == (2.0, 3.0)
      4 assert range_overlap([ (0.0, 1.0), (0.0, 2.0), (-1.0, 1.0) ]) == (0.0, 1.0)
----> 5 assert range_overlap([ (0.0, 1.0), (5.0, 6.0) ]) == None
      6 assert range_overlap([ (0.0, 1.0), (1.0, 2.0) ]) == None

AssertionError:
~~~
{: .error}

It turns out that for our current implementation of the `range_overlap` function,
`max_left` is either equal to or greater than `min_right` for non-overlapping cases.

~~~
range_overlap([ (0.0, 1.0), (5.0, 6.0) ])
~~~
{: .language-python}

~~~
(5.0, 1.0)
~~~
{: .output}

We can edit our function one last time to catch this behaviour and return `None`.

~~~
def range_overlap(ranges):
    """Return common overlap among a set of [left, right] ranges."""
    max_left, min_right = ranges[0]
    for (left, right) in ranges[1:]:
        max_left = max(max_left, left)
        min_right = min(min_right, right)
    if max_left >= min_right:
        overlap = None
    else:
        overlap = (max_left, min_right)
    return overlap
~~~
{: .language-python}

~~~
test_range_overlap()
~~~
{: .language-python}

We get no output from `test_range_overlap()`,
which means all the tests passed.

> ## Test frameworks
>
> We've seen that `test_range_overlap` will halt as soon as one of the assertions fails.
> Ideally, we'd like it to continue and run all the assertions
> so we can find out if there are other points of failure.
> This is where a test framework (also called a test runner)
> such as [pytest](https://docs.pytest.org/en/7.1.x/) can be very useful.
> Test frameworks are beyond the scope of this lesson,
> but as you start to incorporate unit testing into your workflows
> they are an important tool for coordinating the process.
{: .callout}


## Logging

So far we've considered how to make our programs halt or handle the situation when things go wrong,
and how to write unit tests to check for correct behaviour. 
One final option in our defensive programming toolkit is to have our programs report their own activity.
We saw this earlier when we used `print` statements to report potential problems with our data.

~~~
def detect_problems(filename):

    data = numpy.loadtxt(fname=filename, delimiter=',')

    if numpy.max(data, axis=0)[0] == 0 and numpy.max(data, axis=0)[20] == 20:
        print('Suspicious looking maxima!')
    elif numpy.sum(numpy.min(data, axis=0)) == 0:
        print('Minima add up to zero!')
    else:
        print('Seems OK!') 
~~~
{: .language-python}

The problem with this approach is that information printed to the screen is lost
once we close our session.
Constantly adding, removing or commenting out `print` statements from code
is also tedious and error-prone.

A better approach is to use a logging framework,
such as Python’s `logging` library.
This lets us leave debugging statements in our code and turn them on or off at will.
Let's start by replacing our `print` statements with `logging` commands.

In order of increasing severity, the available logging levels are:

- `debug`: very detailed information used for localizing errors.
- `info`: confirmation that things are working as expected.
- `warning`: something unexpected happened, but the program will keep going.
- `error`: something has gone badly wrong, but the program hasn’t hurt anything.
- `critcal`: potential loss of data, security breach, etc.

~~~
import logging

def detect_problems(filename):

    data = numpy.loadtxt(fname=filename, delimiter=',')

    if numpy.max(data, axis=0)[0] == 0 and numpy.max(data, axis=0)[20] == 20:
        logging.warning(f'Suspicious looking maxima in {filename}')
    elif numpy.sum(numpy.min(data, axis=0)) == 0:
        logging.warning(f'Minima add up to zero in {filename}')
    else:
        logging.info(f'{filename} seems OK') 
~~~
{: .language-python}

By default, only information from logging levels "warning" and more severe is reported.

~~~
detect_problems('inflammation-03.csv')
~~~
{: .language-python}

~~~
WARNING:root:Minima add up to zero in inflammation-03.csv
~~~
{: .output}

If we want to see the output from less severe levels (i.e. turn our information statement on),
we'd need to change the minimum level in the logging configuration.
We can also provide the name of a file to write the logging information to,
so that it isn't lost when we finish our session.

~~~    
logging.basicConfig(level=logging.INFO, filename='log.txt')

detect_problems('inflammation-05.csv')
~~~
{: .language-python}

Notice that we've used capital INFO to set the logging level.
We used the function `logging.info` earlier to define our logging message,
whereas `logging.INFO` is an integer value used to define the logging level.

~~~
print(logging.INFO)
~~~
{: .language-python}

~~~
20
~~~
{: .output}

By setting the logging level to "info",
our output "log.txt" file will now capture all logging information
with a flag of "info" or higher - that is,
all logging outputs will be written to our log file.

~~~
$ cat log.txt
~~~
{: .language-bash}

~~~
INFO:root:inflammation-05.csv seems ok
~~~
{: .output}

> ## Logging configuration in notebooks
> 
> If you're working in a Jupyter notebook,
> you'll need to execute the following loop before
> running `logging.BasicConfig`.
>
> ~~~
> for handler in logging.root.handlers[:]:
>     logging.root.removeHandler(handler)
> ~~~
> {: .language-python}
{: .callout}

> ## Rounding data
>
> The following function takes an array and rounds the data up, down
> or to the nearest integer. 
>
> ~~~
> def round_to_integer(data, method):
>     if method == 'up':
>         rounded_data = numpy.ceil(data)
>     elif method == 'nearest':
>         rounded_data = numpy.round(data)
>     else:
>         rounded_data = numpy.floor(data)
>
>     return rounded_data
> ~~~
> {: .language-python}
>
> The problem is that if someone using the function provides any `method` besides
> `up` or `nearest` (e.g. they might try `closest` in instead of `nearest`)
> the function will round down.
>
> Edit `round_to_integer` so it raises a `ValueError` if any `method` besides
> `up`, `nearest` or `down` is used.
>
> > ## Solution
> > ~~~
> > def round_to_integer(data, method):
> >     if method == 'up':
> >         rounded_data = numpy.ceil(data)
> >     elif method == 'nearest':
> >         rounded_data = numpy.round(data)
> >     elif method == 'down':
> >         rounded_data = numpy.floor(data)
> >     else:
> >         raise ValueError("""Method argument must be 'up', 'down' or 'nearest'""")
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}

> ## Ignore missing files
>
> In an earlier lesson we were looping over a list of inflammation files
> in order to visualise the data and detect problems.
>
> ~~~
> for filename in filenames:
>     print(filename)
>     visualize(filename)
>     detect_problems(filename)
> ~~~
> {: .language-python}
>
> As it stands, if there's a problem reading in any of the data files in `filenames`
> (e.g. if the file doesn't exist)
> the `numpy.loadtxt` function in `visualize` will trigger an `OSError`
> and the loop will halt. 
> For example:
>
> ~~~
> numpy.loadtxt(fname='inflammation-20.csv', delimiter=',')
> ~~~
> {: .language-python}
>
> ~~~
> OSError: inflammation-20.csv not found.
> ~~~  
> {: .error}
>
> Add a try-except block to the loop so that if a given `filename`
> triggers an `OSError` the loop simply continues to the next file in the list.
>
> > ## Solution
> > ~~~
> > for filename in filenames:
> >    try:
> >        print(filename)
> >        visualize(filename)
> >        detect_problems(filename)
> >    except OSError:
> >        pass
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}

> ## Absolute zero
>
> In an earlier lesson we defined a function for converting temperature values
> from Celsius to Kelvin.
> 
> ~~~
> def celsius_to_kelvin(temp_c):
>     return temp_c + 273.15
> ~~~
> {: .language-python}
>
> The Kelvin temperature scale is defined such that values less than zero are
> physically impossible (0 K is often referred to as absolute zero).
> Add an assertion to `celsius_to_kelvin` to check that the calculated
> temperatures are not less than zero.
>
> > ## Solution
> > ~~~
> > def celsius_to_kelvin(temp_c):
> >     temp_k = temp_c + 273.15
> >     assert numpy.min(temp_k) >= 0, f"There is a temperature/s less than absolute zero"
> >     return temp_k
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}

{% include links.md %}
