# How to unit test transforms
At its most basic level, testing turns _should work_ into _did work_. But it is also a useful aid in requirements capture and software design, when test are written first (TDD).

## Prerequisites
This section assumes familiarity with
- [Python Unit Testing](https://github.com/bjss-data-academy/python-essentials/blob/main/08-unit-test.md) 
  
## How to write great Spark tests
As we write tests for our Spark analytics code, we will be applying these principles:

- Arrange, Act, Assert
- Write functions that work on DataFrames
- Use a separate test notebook
- Keep tests F.I.R.S.T
- Test behaviour, not implementation
- Separate I/O from logic
- Test all behaviours we need to work

More detail is available in the prerequisites/further reading, but let's summarise.

## Arrange, Act, Assert
A unit test has three sections to it, called the Arrange, Act and Assert sections.

Here's a unit test:

```python
from sum import calculate_sum

def test_sums_three_numbers():
        # Arrange
    input = [1, 2, 3]

    # Act
    actual = calculate_sum( input )

    # Assert
    expected = 1 + 2 + 3
    assert actual == expected
```

### Arrange section
The first part of a test is the _Arrange_ section.

This code arranges the _system under test (SUT)_ to be ready to use. This will include 
- Creating test data
- Creating object instances
- Wiring up dependencies
   
In our example, we only need to arrange one thing: create some test data values in variable `input`:

```python
        # Arrange
    input = [1, 2, 3]
```

> _Test Smell: Messy Arrange_
> If the Arrange code looks messy, our system under test is coupled to too many things.
>
> Fix: Reduce coupling by splitting the design

### Act section
Code in the Act section causes the SUT to take the action we want to test. It also captures the programming interface we have designed for that action: the code here is how the rest of the codebase will use our code.

In the example, we have just the one function call:

```python
actual = calculate_su( input )
```

Using the name _actual_ is a convention for the actual result of our action. 

> _Test Smell: Messy Act_
> If the Act code is hard to follow, then our code is hard to use.
> 
> Fix: Refactor. Consider _Replace Parameters With Parameter Object_ and _Extract Method_ to reduce multi-stage method calls

### Assert section
This is where we compare the `actual` value we got with the _expected_value if everything worked.

In the example, we want to verify that we got the sum of our three input numbers:

```python
    expected = 1 + 2 + 3
    assert actual == expected
```

Note we used `expected` as an _explaining variable_ so future readers know that is the value we are expecting. We also left the computer to add up the numbers, to avoid mistakes.

> _Test Smell: Messy Arrange_
> If the Assert code is a mess, our _output mechanism_ is clunky. The way we communicate results of our action is difficult to use.
>
> Fix: Consider replacing with a simpler mechanism
> 

### Test naming
We name the tests according to _what should have happened when the test passes_. 

In our example, the name was:

```python
def test_sums_three_numbers():
```

Which concisely describes what behaviour we are testing in this test: _does it sum three numbers?_

Another way of looking at this is _requirements capture_. We name the test according to what needs to be done.

> Pytest _requires_ test names to start with **test_** so the framework can find them

## Test behaviour, not implementation
A unit test of the SUT should not know _anything_ about how the SUT is implemented.

Another way to look at this is the SUT can have its internal working completely replaced by some other way of achieveing the same goal, and the test should pass.

If the SUT is a function, the test should depend only on the inputs and outputs of that function. Not anything implemented nor assumed inside that function.

> Testing behaviour not implementation is __really important__ if you want to avoid massive pain as the code is changed

## Write functions that work on DataFrames
Our data transforms are best expressed as a function that takes one or more dataframes as input, and returns a dataframe as output:

```python
def findEmailsOfTopScorers(scores_df:DataFrame, contacts_df:DataFrame) -> DataFrame:
   # implementation goes here - unimportant detail from test perspective
   return result_df
```

These are ideally suited to unit testing behaviour, not implementation.

## Use a separate test notebook
Test code is not part of the production application.

One reasonable way to organise tests is to keep them in separate notebooks/files from the production code.

## Keep tests F.I.R.S.T.
Unit tests should be FIRST:

- Fast: run very quickly, wo we can afford to run them often
- Isolated: Tests must be able to run in any order or individually and not depend on prior test runs
- Repeatable: The test result is consistent. No flaky tests. No dependency on anything outside the test setup
- Self-checking: The test has an assertion to automatically check the results. No human inspection
- Timely: We write tests at the same time as we write production code. It is not a spearate project phase

For our Spark code we achieve these ideas by:
- Putting logic into functions working on DataFrames (see above)
- Corollary: keeping table and file read/writes out of the analytics logic
- Avoiding global variables

That will result in code which supports FIRST test goals.
 
## Separate I/O from logic
The general principle of separation of concerns, mentioned above.

Instead of one function to read a table, transform, write a table - split into three functions:
- read table into dataframe
- transform dataframe into a result dataframe
- write dataframe to table

That allows us to rapidly test the transform logic in isolation.

The transform code is the bulk of what we are responsible for inventing.

## Test all behaviours we need to work
How many tests do we need?

_One test per everything we care about working_.

Don't care if something works or gives a wrong result? Don't test it!
Go one step further: if we don't care about the answer, _delete that code_. No answer is as good as a wrong answer, and much less code to maintain.

If our users _do_ care about the results being correct, __test__. Put your money where your mouth is.

Don't make the users test your code for you.

> Tests: Turn _it should work_ into _it did work_

## Running Pytest in a notebook
Documentation on doing this is thin on the ground, and the AI was no help whatsoever.

So with thanks to this [medium.com post](https://medium.com/@ssharma31/integrating-pytest-with-databricks-a9e47afecd85), here is how to run pytest inside Notebooks.

### Installing pytest
Step 1: Create a new notebook.

Step 2: Create a code cell and enter this code:

```python
%sh pip install pytest
```

Step 3: Run the cell to install the pytest library into this notebook.


Step 4: Create another code cell. Enter this code:

```python
import pytest
import os
import sys
import test_sum

sys.dont_write_bytecode = True
os.chdir("/Workspace/Users/alan.mellor@bjss.com/")
```

Step 5: Run that code. Nothing exciting seems to happen, but check thhat it executes successfully.

That's the installation magic done. Next up: writing a test to run.

### Creating a test file
We create a new _file_ to add our test code into.

- Click _Workspace_ on the left-side menu bar
- Click the _Create_ button over on the right hand top
- Select _File_
- Enter the file name *test_sum.py*

![Menu selection screenshot](/images/workspace-create-file.png)

> The filename for a test file __must__ start with **test_** when using Pytest

Inside our new file, enter the following test code:

```python
import pytest
from sum import *

def test_sums_two_numbers():
  # Arrange
  numbers = [1, 2]

  # Act
  actual = calculate_sum(numbers)

  # Assert
  expected = 1 + 2
  assert actual == expected
```

This will be the single unit test we will run. 

We've done this test-first style. We've decided to have a function `calculate_sum` that will take an array of numbers. It should add them up and return a single number as the result.

Next job - write the code (or get your favourite AI code grunt to do it, of course):

__Create a new file__ in the workspace (as we did above), and name it _sum.py_.

Enter this code:

```python
def calculate_sum(numbers):
    return sum(numbers)
```

We're ready to run our tests against this file. For that, we need to go back to our notebook and add a cell.

### Running all tests from the Notebook
Go back to our Notebook, and add the following code cell:

```python
pytest.main(["-v"])
```

Run that cell (assuming the two cells above it have run). It will find all our *test_* files and run all the tests in them.

We see:

![Unit test run in notebook output](/images/pytest-output-notebook.png)

All is well!

### Confirm test can fail
It is best to follow the [Red, Green, Refactor (RGR)](https://github.com/bjssacademy/advanced-tdd/blob/main/chapter05/chapter05.md) cycle of testing. Here, we make sure a test _fails_ before we write code that makes it pass. This builds confidence that the test is _actually_ testing what we want it to test.

Above, we pasted in some code that made the test pass.

Let's deliberatly break that production code, just to check the test can fail.

We modify _sum.py_ to look like this:

```python
def calculate_sum(numbers):
    return -999 # sum(numbers)
```

Run the test again in our notebook, and see:

![Test failure displayed in notebook](/images/pytest-failed-notebook.png)

Put the _sum.py_ code back to where it worked, run the test again to double check. 

> __WARNING__
> The tests _do not fail_
>
> This is due to some Python caching in Dtabaricks presenting the "first code we entered" to the test.
>
> Massive catastrophe for testing.
>
> Have yet to find a solution

## Example using DataFrame

Add the following two files into the workspace (as above):

__calculate_average.py__:

```python
```

__test_average.py__:

```python
```

Go to the notebook and run the cells. See the test output.

# Further Reading
To improve TDD, unit test and design skills, check out our comprehensive guide:
- [Advanced TDD](https://github.com/bjssacademy/advanced-tdd)

Using tests inside Databricks notebooks:
- [Test Databricks notebooks](https://learn.microsoft.com/en-us/azure/databricks/notebooks/test-notebooks)

# Next
Resources to help study for certificates:

[Certification Resources](/certification.md)

[Back to Contents](/contents.md)
