Table of Contents
-----------------

*   [Table of Contents](#InternalEngineeringBack-endUnitTests-TableofContents)
*   [Why](#InternalEngineeringBack-endUnitTests-Why)
*   [What They Are](#InternalEngineeringBack-endUnitTests-WhatTheyAre)
*   [What They Are Not](#InternalEngineeringBack-endUnitTests-WhatTheyAreNot)
*   [How](#InternalEngineeringBack-endUnitTests-How)
    *   [How to Install](#InternalEngineeringBack-endUnitTests-HowtoInstall)
    *   [What to Test](#InternalEngineeringBack-endUnitTests-WhattoTest)
    *   [How to Write](#InternalEngineeringBack-endUnitTests-HowtoWrite)
        *   [Testing Tools](#InternalEngineeringBack-endUnitTests-TestingTools)
            *   [Asserts](#InternalEngineeringBack-endUnitTests-Asserts)
            *   [Fixtures](#InternalEngineeringBack-endUnitTests-Fixtures)
            *   [Parametrization](#InternalEngineeringBack-endUnitTests-Parametrization)
    *   [How to Run](#InternalEngineeringBack-endUnitTests-HowtoRun)
*   [Adding to Tests](#InternalEngineeringBack-endUnitTests-AddingtoTests)

Why
---

1.  Catch bugs.
    
2.  Increase confidence.
    

What They Are
-------------

Unit tests are automated tests of _parts_ of the program.

They run in isolation from the whole program.

They are typically written in the form of functions that validate the behavior of various functionalities within a software program. 

What They Are Not
-----------------

Unit tests are not:

*   End-to-end tests. They aren’t checking to see if the entire program works as expected from start to finish.
    
*   Smoke tests. They don’t attempt to handle every conceivable input, totaling to thousands of inputs.
    
*   External systems tests. They don’t deal with environment setup or with database input; they deal with the codebase itself and how data is handled internally.
    

These tests have their place and add value, but they are not unit tests.

How
---

At LGCY Power, we use Python’s `pytest` library to perform unit tests.

We like `pytest` for its out-of-the-box capability to test with low amounts of code.

Unit tests allow us to think in terms of preconditions and post conditions. Another way of thinking about tests is, “Arrange, Act, Assert”. Both methods are describing the same thing, just with different vocabulary.

### How to Install

In a local virtual environment, run:

```java
pip install pytest --upgrade
```

### What to Test

Think of unit testing as “scenario testing”.

First, test the common case(s). Identify [preconditions and post conditions](https://medium.com/@mlbors/preconditions-and-postconditions-5913fc0fcdaf) (arrange code, execute code, then assert code). Think, “I expect the data to look like this when the function begins and like this when the function ends,” or, “I expect this thing to happen when this method runs.” Test to ensure that your conditions are met.

Second, test one or more edge cases. Throw something at the code that is likely to happen, but is not the highest expectation or priority.

### How to Write

Create a folder called `tests` that contains copies of all files used in the program.

Rename the files with the word “test\_” prepended on the name. This lets `pytest` know which files to test. Then append what is being tested along with the expected result to the end of the function name.

Add an empty `__init__.py` file to the `tests` folder.

Write functions for the conditions you want to test. Prepend all function names with “test\_”. This again lets `pytest` know which functions to test.

#### Testing Tools

Asserts, Fixtures, and Parameters are tools we use to easily test preconditions and post conditions.

##### Asserts

Asserts are sanity checks. Typical use cases are to ensure that data exists as expected or that data is transformed as expected.

Here is an example. Say I have this function:

```py
def perform_division(number1, number2):
  try:
  # divide first number by second
    final_number = number1 / number2
  except ZeroDivisionError:
  # if there is an error, return 0
    final_number = 0
  
  return final_number
```

Then my tests will handle three probable outcomes: the try outcomes of float and int, and the except. My tests could look like the below example.

Notice that the test functions call their original counterparts. This allows me to make changes to the original functions only and the tests will still work.

Notice further that tests are a good way to uncover stinky code. Division by zero is impossible but still an acceptable input in the `number2` argument, so my code should be able to handle that. Inability to handle the ZeroDivisionError error should indicate to me that my code needs to be more robust.

```py
from my_module import perform_division
import pytest

# include what I am testing in the function name
def test_perform_division_two_ints_expect_float():
    # this tests that the function can handle normal input
    number3 = perform_division(number1=1, number2=2)
    # ensure that post conditions are met
    assert number3 == 0.5
    
def test_perform_division_two_ints_expect_int():
    # this tests that the function can handle normal input
    number3 = perform_division(number1=4, number2=2)
    # ensure that post conditions are met
    assert number3 == 2

# include what I am testing in the function name
def test_perform_division_int_and_zero_expect_zero(number1=1, number2=0):
    # this tests that the function can handle a string input
    number_zero = perform_division(number1=1, number2=2)
    # ensure that post conditions are met
    assert number_zero == 0
```

Here is another, more relevant, example:

```py
import mysql.connector
import os

def query_database(query):
    # establish connection to database
    conn = mysql.connector.connect(
        user=os.getenv("USER"),
        password=os.getenv("PASSWORD"),
        host=os.getenv("HOST"),
        database=os.getenv("NAME")
    )
    # create cursor
    cursor = conn.cursor(dictionary=True)
    # query database
    cursor.execute(query)
    
    # get results and close out connection
    query_results = cursor.fetchall()
    cursor.close()
    conn.close()
    
    return query_results
```

Then my test function could look like this:

```py
import pytest
import mysql.connector
import os

def test_query_database(query="SELECT * FROM my_db"):
    # ensure precondition is met
    # -----------------------------------
    assert type(query) == str
    
    # establish connection to database
    conn = mysql.connector.connect(
        user=os.getenv("USER"),
        password=os.getenv("PASSWORD"),
        host=os.getenv("HOST"),
        database=os.getenv("NAME")
    )
    
    # ensure post condition is met
    # -----------------------------------
    # did the code connect to the db?
    assert conn
```

##### Fixtures

Now say that I want to **test multiple functions with a single input**. This is where fixtures come in handy. A fixture allows you to use a function to define the variable that will be used. Back to our star wars examples:

```py
def perform_division(number1, number2):
  try:
  # divide first number by second
    final_number = number1 / number2
  except ZeroDivisionError:
  # if there is an error, return 0
    final_number = 0
  
  return final_number

def perform_multiplication(number1, number2):
  # multiply first number by second
  final_number = number1 * number2
  
  return final_number
```

If I need to keep using the same input, I can use the fixture decorator to define that input:

```py
from my_module import perform_division, perform_multiplication
import pytest

# fixture decorator to indentify function with my variable
@pytest.fixture
def generate_test_number1():
    # This function could perform an operation to arrive at a value to be re-used
    # For simplicity, we'll just return a variable we'll re-use
    return 70
    
@pytest.fixture
def generate_test_number2():
    return 35

def test_perform_division_two_ints_expect_int(generate_test_number1, generate_test_number2):
    # this tests that the function can handle normal input
    number3 = perform_division(generate_test_number1, generate_test_number2)
    # ensure that post conditions are met
    assert number3 == 2
```

##### Parametrization

Finally, say that I want to **test a single function but with various inputs and expected outputs**. This is where parameters come in handy.

Here is an example function:

```py
def divide_by_two(number1):
  try:
  # divide first number by second
    final_number = number1 / 2
  except ZeroDivisionError:
  # if there is an error, return 0
    final_number = 0
  
  return final_number
```

If the program relies on an unchanging response, my test might look like this.

```py
import pytest
from my_module import divide_by_two

# Use pytest's parametrize decorator
@pytest.mark.parametrize("test_input, expected_output", [
    (4, 2),
    (16, 8)
])
# ensure the results come in as expected
def test_divide_by_two_expect_whole_numbers(test_input, expected_output):
    assert divide_by_two(test_input) == expected_output
```

### How to Run

Running tests with `pytest` is easy. In Terminal/Command Line, navigate to the newly created `tests` folder. Next, type the command `pytest` and run.

`pytest` automatically displays results, including any failures that need your attention. Keep testing until all tests have passed.

Adding to Tests
---------------

If you receive a bug in Jira, take these steps:

1.  Fix the bug.
    
2.  Add its preconditions and post conditions as a new unit test.
    
3.  Create a PR to deploy the update.
