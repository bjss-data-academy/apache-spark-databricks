# How to unit test transforms
At its most basic level, testing turns _should work_ into _did work_. But it is also a useful aid in requirements capture and software design, when test are written first (TDD).

## Prerequisites
This section assumes familiarity with
- [Python Unit Testing](https://github.com/bjss-data-academy/python-essentials/blob/main/08-unit-test.md) 
  
## How to write great Spark tests
As we write tests for our Spark analytics code, we will be applying these principles:

- Arrange, Act, Assert
- Write functions that work on DataFrames
- Keep tests F.I.R.S.T
- Test behaviour, not implementation
- Separate I/O from logic
- Test all behaviours we need to work

More detail is available in the prerequisites, but let's summarise.

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
> 
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
> 
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
>
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

## Write functions that work on DataFrames

## Keep tests F.I.R.S.T.

## Test behaviour, not implementation

## Separate I/O from logic

## Test all behaviours we need to work

# Further Reading
To improve TDD, unit test and design skills, check out our comprehensive guide:

- [Advanced TDD](https://github.com/bjssacademy/advanced-tdd) 

# Next
Resources to help study for certificates:

[Certification Resources](/certification.md)

[Back to Contents](/contents.md)
