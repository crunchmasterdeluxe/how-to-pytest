# Python Unit Tests with `pytest`
An explanation of how to run unit tests using Python's pytest library

## Table of Contents
- [Why](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#why)
- [What They Are](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#what-they-are)
- [What They Are Not](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#what-they-are-not)
- [How](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#how)
  - [How to Install](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#how-to-install)
  - [What to Test](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#what-to-test)
  - [How to Write](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#how-to-write)
  - [Testing Tools](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#testing-tools)
   - [Asserts](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#asserts)
   - [Fixtures](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#fixtures)
   - [Parametrization](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#parametrization)
  - [How to Run](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#how-to-run)
- [Adding to Tests](https://github.com/crunchmasterdeluxe/how-to-pytest/blob/main/README.md#adding-to-tests)

## Why

1. Catch bugs.

2. Increase confidence.

## What They Are

Unit tests are automated tests of parts of the program. 

They run in isolation from the whole program.

They are typically written in the form of functions that validate the behavior of various functionalities within a software program. 

## What They Are Not

Unit tests are not:

End-to-end tests. They aren’t checking to see if the entire program works as expected from start to finish.

Smoke tests. They don’t attempt to handle every conceivable input, totaling to thousands of inputs.

External systems tests. They don’t deal with environment setup or with database input; they deal with the codebase itself and how data is handled internally.

These tests have their place and add value, but they are not unit tests.

## How

`pytest` is nice for its out-of-the-box capability to test with low amounts of code.

Unit tests allow us to think in terms of preconditions and post conditions. Another way of thinking about tests is, “Arrange, Act, Assert”. Both methods are describing the same thing, just with different vocabulary.

### How to Install

In a local virtual environment, run:

    pip install pytest --upgrade

### What to Test

Think of unit testing as “scenario testing”.

First, test the common case(s). Identify preconditions and post conditions (arrange code, execute code, then assert code). Think, “I expect the data to look like this when the function begins and like this when the function ends,” or, “I expect this thing to happen when this method runs.” Test to ensure that your conditions are met.

Second, test one or more edge cases. Throw something at the code that is likely to happen, but is not the highest expectation or priority. 

### How to Write

Create a folder called tests that contains copies of all files used in the program. 

Rename the files with the word “test_” prepended on the name. This lets pytest know which files to test.

Write functions for the conditions you want to test. Prepend all function names with “test_”. This again lets pytest know which functions to test.

### Testing Tools

Asserts, Fixtures, and Parameters are tools we use to easily test preconditions and post conditions.

#### Asserts

Asserts are sanity checks. Typical use cases are to ensure that data exists as expected or that data is transformed as expected. 

Here is an example. Say I have this function:

    import requests

    def make_api_call(identifier):
        # compose the url
        star_wars_url = "https://swapi.dev/api/people/{}/".format(identifier)
        # make api request
        star_wars_response = requests.get(url=star_wars_url)
        # format response
        person = star_wars_response.json()

        return person

Then my tests will handle three probable conditions: int input, string input, and api error handling. My tests could look like the below example. 

Notice that the test functions call their original counterparts. This allows me to make changes to the original functions only and the tests will still work.

    from my_module import make_api_call
    import pytest
    import requests

    def test_make_api_call(identifier = 1):
        # this tests that the function can handle an int input
        person = make_api_call(identifier)
        # ensure that post conditions are met
        # -----------------------------------
        # ensure that data exists as expected
        assert person["id"] == 1

    def test_make_api_call(identifier = "1"):
        # this tests that the function can handle a string input
        person = make_api_call(identifier)
        # ensure that post conditions are met
        # -----------------------------------
        # ensure that data exists as expected
        assert person["id"] == 1

    def test_make_api_call_error():
        # this test would change depending on how you want 
        # to handle the request returning an error code
        # but say for example we wanted to raise a custom exception, 
        # we would just test that the exception was raised
        try:
            person = make_api_call(identifier='one')
        # ensure that post conditions are met
        # -----------------------------------
        # Bad input data, so exception should be raised
        except CustomHTTPException:
            assert True
        assert False

#### Fixtures

Now say that I want to test multiple functions with a single input. This is where fixtures come in handy. A fixture allows you to use a function to define the variable that will be used. Back to our star wars examples:

    import requests

    def make_person_api_call(identifier):
        # compose the url
        star_wars_url = "https://swapi.dev/api/people/{}/".format(identifier)
        # make api request
        star_wars_response = requests.get(url=star_wars_url)
        # format response
        person = star_wars_response.json()

        return person['name']

    def make_species_api_call(identifier):
        # compose the url
        star_wars_url = "https://swapi.dev/api/species/{}/".format(identifier)
        # make api request
        star_wars_response = requests.get(url=star_wars_url)
        # format response
        species = star_wars_response.json()

        return species['name']

If I need to keep using the same input, I can use the fixture decorator to define that input:

    from my_module import make_person_api_call, make_species_api_call
    import pytest
    import requests

    # fixture decorator to indentify function with my variable
    @pytest.fixture
    def generate_test_input():
        # This function could perform an operation to arrive at a value to be re-used
        # For simplicity, we'll just return a variable we'll re-use
        return 1

    # ensure post conditions are met
    # -----------------------------------
    person_name = test_make_person_api_call(generate_test_input)
    assert person_name

    species_name = test_make_species_api_call(generate_test_input)
    assert species_name

#### Parametrization

Finally, say that I want to test a single function but with various inputs and expected outputs. This is where parameters come in handy. 

Here is an example function:

    import requests

    def make_api_call(identifier):
        # compose the url
        star_wars_url = "https://swapi.dev/api/people/{}/".format(identifier)
        # make api request
        star_wars_response = requests.get(url=star_wars_url)
        # format response
        person = star_wars_response.json()

        return person['name']

If the program relies on an unchanging response, my test might look like this.

    import pytest
    import requests
    from my_module import make_api_call

    # ensure pre- and post conditions are met
    # -----------------------------------
    # Use pytest's parametrize decorator
    @pytest.mark.parametrize("test_identifier, expected_output", [
        (1, "Luke Skywalker"),
        (2, "C-3PO")
    ])
    # ensure the results come in as expected
    def test_make_api_call(test_identifier, expected_output):
        assert make_api_call(test_input) == expected_output


### How to Run

Running tests with pytest is easy. In Terminal/Command Line, navigate to the newly created tests folder. Next, type the command pytest and run.

pytest automatically displays results, including any failures that need your attention. Keep testing until all tests have passed.

## Adding to Tests

Add tests to your code as bugs come up in the future by taking these steps:

1. Fix the bug.

2. Add its preconditions and post conditions as a new unit test.
