Chapter 8. Pytest for DevOps
Continuous integration, continuous delivery, deployments, and any pipeline workflow in general with some thought put into it will be filled with validation. This validation can happen at every step of the way and when achieving important objectives.

For example, if in the middle of a long list of steps to produce a deployment, a curl command is called to get an all-important file, do you think the build should continue if it fails? Probably not! curl has a flag that can be used to produce a nonzero exit status (--fail) if an HTTP error happens. That simple flag usage is a form of validation: ensure that the request succeeded, otherwise fail the build step. The key word is to ensure that something succeeded, and that is at the core of this chapter: validation and testing strategies that can help you build better infrastructure.

Thinking about validation becomes all the more satisfying when Python gets in the mix, harnessing testing frameworks like pytest to handle the verification of systems.

This chapter reviews some of the basics associated with testing in Python using the phenomenal pytest framework, then dives into some advanced features of the framework, and finally goes into detail about the TestInfra project, a plug-in to pytest that can do system verification.

Testing Superpowers with pytest
We can’t say enough good things about the pytest framework. Created by Holger Krekel, it is now maintained by quite a few people that do an incredible job at producing a high-quality piece of software that is usually part of our everyday work. As a full-featured framework, it is tough to narrow down the scope enough to provide a useful introduction without repeating the project’s complete documentation.

TIP
The pytest project has lots of information, examples, and feature details in its documentation that are worth reviewing. There are always new things to learn as the project continues to provide new releases and different ways to improve testing.

When Alfredo was first introduced to the framework, he was struggling to write tests, and found it cumbersome to adhere to Python’s built-in way of testing with unittest (this chapter goes through the differences later). It took him a couple of minutes to get hooked into pytest’s magical reporting. It wasn’t forcing him to move away from how he had written his tests, and it worked right out of the box with no modifications! This flexibility shows throughout the project, and even when things might not be possible today, you can extend its functionality via plug-ins or its configuration files.

By understanding how to write more straightforward test cases, and by taking advantage of the command-line tool, reporting engine, plug-in extensibility, and framework utilities, you will want to write more tests that will undoubtedly be better all around.

Getting Started with pytest
In its simplest form, pytest is a command-line tool that discovers Python tests and executes them. It doesn’t force a user to understand its internals, which makes it easy to get started with. This section demonstrates some of the most basic features, from writing tests to laying out files (so that they get automatically discovered), and finally, looking at the main differences between it and Python’s built-in testing framework, unittest.

TIP
Most Integrated Development Environments (IDEs), such as PyCharm and Visual Studio Code, have built-in support for running pytest. If using a text editor like Vim, there is support via the pytest.vim plug-in. Using pytest from the editor saves time and makes debugging failures easier, but be aware that not every option or plug-in is supported.

Testing with pytest
Make sure that you have pytest installed and available in the command line:

$ python3 -m venv testing
$ source testing/bin/activate
Create a file called test_basic.py; it should look like this:

def test_simple():
    assert True

def test_fails():
    assert False
If pytest runs without any arguments, it should show a pass and a failure:

$ (testing) pytest
============================= test session starts =============================
platform linux -- Python 3.6.8, pytest-4.4.1, py-1.8.0, pluggy-0.9.0
rootdir: /home/alfredo/python/testing
collected 2 items

test_basic.py .F                                                        [100%]

================================== FAILURES ===================================
_________________________________ test_fails __________________________________

    def test_fails():
>       assert False
E       assert False

test_basic.py:6: AssertionError
===================== 1 failed, 1 passed in 0.02 seconds ======================
The output is beneficial from the start; it displays how many tests were collected, how many passed, and which one failed—including its line number.

TIP
The default output from pytest is handy, but it might be too verbose. You can control the amount of output with configuration, reducing it with the -q flag.

There was no need to create a class to include the tests; functions were discovered and ran correctly. A test suite can have a mix of both, and the framework works fine in such an environment.

Layouts and conventions
When testing in Python, there are a few conventions that pytest follows implicitly. Most of these conventions are about naming and structure. For example, try renaming the test_basic.py file to basic.py and run pytest to see what happens:

$ (testing) pytest -q

no tests ran in 0.00 seconds
No tests ran because of the convention of prefixing test files with test_. If you rename the file back to test_basic.py, it should be automatically discovered and tests should run.

NOTE
Layouts and conventions are helpful for automatic test discovery. It is possible to configure the framework to use other naming conventions or to directly test a file that has a unique name. However, it is useful to follow through with basic expectations to avoid confusion when tests don’t run.

These are conventions that will allow the tool to discover tests:

The testing directory needs to be named tests.

Test files need to be prefixed with test; for example, test_basic.py, or suffixed with test.py.

Test functions need to be prefixed with test_; for example, def testsimple():.

Test classes need to be prefixed with Test; for example, class TestSimple.

Test methods follow the same conventions as functions, prefixed with test_; for example, def test_method(self):.

Because prefixing with test_ is a requirement for automatic discovery and execution of tests, it allows introducing helper functions and other nontest code with different names, so that they get excluded automatically.

Differences with unittest
Python already comes with a set of utilities and helpers for testing, and they are part of the unittest module. It is useful to understand how pytest is different and why it is highly recommended.

The unittest module forces the use of classes and class inheritance. For an experienced developer who understands object-oriented programming and class inheritance, this shouldn’t be a problem, but for beginners, it is an obstacle. Using classes and inheritance shouldn’t be a requisite for writing basic tests!

Part of forcing users to inherit from unittest.TestCase is that you are required to understand (and remember) most of the assertion methods that are used to verify results. With pytest, there is a single assertion helper that can do it all: assert.

These are a few of the assert methods that can be used when writing tests with unittest. Some of them are easy to grasp, while others are very confusing:

self.assertEqual(a, b)

self.assertNotEqual(a, b)

self.assertTrue(x)

self.assertFalse(x)

self.assertIs(a, b)

self.assertIsNot(a, b)

self.assertIsNone(x)

self.assertIsNotNone(x)

self.assertIn(a, b)

self.assertNotIn(a, b)

self.assertIsInstance(a, b)

self.assertNotIsInstance(a, b)

self.assertRaises(exc, fun, *args, **kwds)

self.assertRaisesRegex(exc, r, fun, *args, **kwds)

self.assertWarns(warn, fun, *args, **kwds)

self.assertWarnsRegex(warn, r, fun, *args, **kwds)

self.assertLogs(logger, level)

self.assertMultiLineEqual(a, b)

self.assertSequenceEqual(a, b)

self.assertListEqual(a, b)

self.assertTupleEqual(a, b)

self.assertSetEqual(a, b)

self.assertDictEqual(a, b)

self.assertAlmostEqual(a, b)

self.assertNotAlmostEqual(a, b)

self.assertGreater(a, b)

self.assertGreaterEqual(a, b)

self.assertLess(a, b)

self.assertLessEqual(a, b)

self.assertRegex(s, r)

self.assertNotRegex(s, r)

self.assertCountEqual(a, b)

pytest allows you to use assert exclusively and does not force you to use any of the above. Moreover, it does allow you to write tests using unittest, and it even executes them. We strongly advise against doing that and suggest you concentrate on just using plain asserts.

Not only is it easier to use plain asserts, but pytest also provides a rich comparison engine on failures (more on this in the next section).

pytest Features
Aside from making it easier to write tests and execute them, the framework provides lots of extensible options, such as hooks. Hooks allow you to interact with the framework internals at different points in the runtime. If you want to alter the collection of tests, for example, a hook for the collection engine can be added. Another useful example is if you want to implement a nicer report when a test fails.

While developing an HTTP API, we found that sometimes the failures in the tests that used HTTP requests against the application weren’t beneficial: an assertion failure would be reported because the expected response (an HTTP 200) was an HTTP 500 error. We wanted to know more about the request: to what URL endpoint? If it was a POST request, did it have data? What did it look like? These are things that were already available in the HTTP response object, so we wrote a hook to poke inside this object and include all these items as part of the failure report.

Hooks are an advanced feature of pytest that you might not need at all, but it is useful to understand that the framework can be flexible enough to accommodate different requirements. The next sections cover how to extend the framework, why using assert is so valuable, how to parametrize tests to reduce repetition, how to make helpers with fixtures, and how to use the built-in ones.

conftest.py
Most software lets you extend functionality via plug-ins (web browsers call them extensions, for example); similarly, pytest has a rich API for developing plug-ins. The complete API is not covered here, but its simpler approach is: the conftest.py file. In this file, the tool can be extended just like a plug-in can. There is no need to fully understand how to create a separate plug-in, package it, and install it. If a conftest.py file is present, the framework will load it and consume any specific directives in it. It all happens automatically!

Usually, you will find that a conftest.py file is used to hold hooks, fixtures, and helpers for those fixtures. Those fixtures can then be used within tests if declared as arguments (that process is described later in the fixture section).

It makes sense to add fixtures and helpers to this file when more than one test module will use it. If there is only a single test file, or if only one file is going to make use of a fixture or hook, there is no need to create or use a conftest.py file. Fixtures and helpers can be defined within the same file as the test and behave the same.

The only condition for loading a conftest.py file is to be present in the tests directory and match the name correctly. Also, although this name is configurable, we advise against changing it and encourage you to follow the default naming conventions to avoid potential issues.

The Amazing assert
When we have to describe how great the pytest tooling is, we start by describing the important uses of the assert statement. Behind the scenes, the framework is inspecting objects and providing a rich comparison engine to better describe errors. This is usually met with resistance because a bare assert in Python is terrible at describing errors. Compare two long strings as an example:

>>> assert "using assert for errors" == "using asert for errors"
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError
Where is the difference? It is hard to tell without spending some time looking at those two long lines closely. This will cause people to recommend against it. A small test shows how pytest augments when reporting the failure:

$ (testing) pytest test_long_lines.py
============================= test session starts =============================
platform linux -- Python 3.6.8, pytest-4.4.1, py-1.8.0, pluggy-0.9.0
collected 1 item

test_long_lines.py F                                                    [100%]

================================== FAILURES ===================================
_______________________________ test_long_lines _______________________________

    def test_long_lines():
>      assert "using assert for errors" == "using asert for errors"
E      AssertionError: assert '...rt for errors' == '...rt for errors'
E        - using assert for errors
E        ?        -
E        + using asert for errors

test_long_lines.py:2: AssertionError
========================== 1 failed in 0.04 seconds ===========================
Can you tell where the error is? This is tremendously easier. Not only does it tell you it fails, but it points to exactly where the failure is. The example is a simple assert with a long string, but the framework handles other data structures like lists and dictionaries without a problem. Have you ever compared very long lists in tests? It is impossible to easily tell what items are different. Here is a small snippet with long lists:

    assert ['a', 'very', 'long', 'list', 'of', 'items'] == [
            'a', 'very', 'long', 'list', 'items']
E   AssertionError: assert [...'of', 'items'] == [...ist', 'items']
E     At index 4 diff: 'of' != 'items'
E     Left contains more items, first extra item: 'items'
E     Use -v to get the full diff
After informing the user that the test failed, it points exactly to the index number (index four or fifth item), and finally, it says that one list has one extra item. Without this level of introspection, debugging failures would take a very long time. The bonus in reporting is that, by default, it omits very long items when making comparisons, so that only the relevant portion shows in the output. After all, what you want is to know not only that the lists (or any other data structure) are different but exactly where they are different.

Parametrization
Parametrization is one of the features that can take a while to understand because it doesn’t exist in the unittest module and is a feature unique to the pytest framework. It can be clear once you find yourself writing very similar tests that had minor changes in the inputs but are testing the same thing. Take, for example, this class that is testing a function that returns True if a string is implying a truthful value. The string_to_bool is the function under test:

from my_module import string_to_bool

class TestStringToBool(object):

    def test_it_detects_lowercase_yes(self):
        assert string_to_bool('yes')

    def test_it_detects_odd_case_yes(self):
        assert string_to_bool('YeS')

    def test_it_detects_uppercase_yes(self):
        assert string_to_bool('YES')

    def test_it_detects_positive_str_integers(self):
        assert string_to_bool('1')

    def test_it_detects_true(self):
        assert string_to_bool('true')

    def test_it_detects_true_with_trailing_spaces(self):
        assert string_to_bool('true ')

    def test_it_detects_true_with_leading_spaces(self):
        assert string_to_bool(' true')
See how all these tests are evaluating the same result from similar inputs? This is where parametrization shines because it can group all these values and pass them to the test; it can effectively reduce them to a single test:

import pytest
from my_module import string_to_bool

true_values = ['yes', '1', 'Yes', 'TRUE', 'TruE', 'True', 'true']

class TestStrToBool(object):

    @pytest.mark.parametrize('value', true_values)
    def test_it_detects_truish_strings(self, value)
        assert string_to_bool(value)
There are a couple of things happening here. First pytest is imported (the framework) to use the pytest.mark.parametrize module, then true_values is defined as a (list) variable of all the values to use that should evaluate the same, and finally, it replaces all the test methods to a single one. The test method uses the parametrize decorator, which defines two arguments. The first is a string, value, and the second is the name of the list defined previously. This can look a bit odd, but it is telling the framework that value is the name to use for the argument in the test method. That is where the value argument comes from!

If the verbosity is increased when running, the output will show exactly what value was passed in. It almost looks like the single test got cloned into every single iteration passed in:

test_long_lines.py::TestLongLines::test_detects_truish_strings[yes] PASSED
test_long_lines.py::TestLongLines::test_detects_truish_strings[1] PASSED
test_long_lines.py::TestLongLines::test_detects_truish_strings[Yes] PASSED
test_long_lines.py::TestLongLines::test_detects_truish_strings[TRUE] PASSED
test_long_lines.py::TestLongLines::test_detects_truish_strings[TruE] PASSED
test_long_lines.py::TestLongLines::test_detects_truish_strings[True] PASSED
test_long_lines.py::TestLongLines::test_detects_truish_strings[true] PASSED
The output includes the values used in each iteration of the single test in brackets. It is reducing the very verbose test class into a single test method, thanks to parametrize. The next time you find yourself writing tests that seem very similar and that assert the same outcome with different inputs, you will know that you can make it simpler with the parametrize decorator.

Fixtures
We think of pytest fixtures like little helpers that can get injected into a test. Regardless of whether you are writing a single test function or a bunch of test methods, fixtures can be used in the same way. If they aren’t going to be shared among other test files, it is fine to define them in the same test file; otherwise they can go into the conftest.py file. Fixtures, just like helper functions, can be almost anything you need for a test, from simple data structures that get pre-created to more complex ones like setting a database for a web application.

These helpers can also have a defined scope. They can have specific code that cleans up for every test method, class, and module, or even allows setting them up once for the whole test session. By defining them in a test method (or test function), you are effectively getting the fixture injected at runtime. If this sounds a bit confusing, it will become clear through examples in the next few sections.

Getting Started
Fixtures are so easy to define and use that they are often abused. We know we’ve created a few that could have been simple helper methods! As we’ve mentioned already, there are many different use cases for fixtures—from simple data structures to more complex ones, such as setting up whole databases for a single test.

Recently, Alfredo had to test a small application that parses the contents of a particular file called a keyring file. It has some structure similar to an INI file, with some values that have to be unique and follow a specific format. The file structure can be very tedious to recreate on every test, so a fixture was created to help. This is how the keyring file looks:

[mon.]
    key = AQBvaBFZAAAAABAA9VHgwCg3rWn8fMaX8KL01A==
    caps mon = "allow *"
The fixture is a function that returns the contents of the keyring file. Let’s create a new file called test_keyring.py with the contents of the fixture, and a small test function that verifies the default key:

import pytest
import random

@pytest.fixture
def mon_keyring():
    def make_keyring(default=False):
        if default:
            key = "AQBvaBFZAAAAABAA9VHgwCg3rWn8fMaX8KL01A=="
        else:
            key = "%032x==" % random.getrandbits(128)

        return """
    [mon.]
        key = %s
            caps mon = "allow *"
        """ % key
    return make_keyring

def test_default_key(mon_keyring):
    contents = mon_keyring(default=True)
    assert "AQBvaBFZAAAAABAA9VHgwCg3rWn8fMaX8KL01A==" in contents
The fixture is using a nested function that does the heavy lifting, allows using a default key value, and returns the nested function in case the caller wants to have a randomized key. Inside the test, it receives the fixture by declaring it part of the argument of the test function (mon_keyring in this case), and is calling the fixture with default=True so that the default key is used, and then verifying it is generated as expected.

NOTE
In a real-world scenario, the generated contents would be passed to the parser, ensuring expected behavior after parsing and that no errors happen.

The production code that used this fixture eventually grew to do other kinds of testing, and at some point, the test wanted to verify that the parser could handle files in different conditions. The fixture was returning a string, so it needed extending. Existing tests already made use of the mon_keyring fixture, so to extend the functionality without altering the current fixture, a new one was created that used a feature from the framework. Fixtures can request other fixtures! You define the required fixture as an argument (like a test function or test method would), so the framework injects it when it gets executed.

This is how the new fixture that creates (and returns) the file looks:

@pytest.fixture
def keyring_file(mon_keyring, tmpdir):
    def generate_file(default=False):
        keyring = tmpdir.join('keyring')
        keyring.write_text(mon_keyring(default=default))
        return keyring.strpath
    return generate_file
Going line by line, the pytest.fixture decorator tells the framework that this function is a fixture, then the fixture is defined, asking for two fixtures as arguments: mon_keyring and tmpdir. The first is the one created previously in the test_keyring.py file earlier, and the second one is a built-in fixture from the framework (more on built-in fixtures in the next section). The tmpdir fixture allows you to use a temporary directory that gets removed after the test completes, then the keyring file is created, and the text generated by the mon_keyring fixture is written, passing the default argument. Finally, it returns the absolute path of the new file created so that the test can use it.

This is how the test function would use it:

def test_keyring_file_contents(keyring_file):
    keyring_path = keyring_file(default=True)
    with open(keyring_path) as fp:
        contents = fp.read()
    assert "AQBvaBFZAAAAABAA9VHgwCg3rWn8fMaX8KL01A==" in contents
You should now have a good idea of what fixtures are, where can you define them, and how to consume them in tests. The next section goes through a few of the most useful built-in fixtures that are part of the framework.

Built-in Fixtures
The previous section briefly touched on one of the many built-in fixtures that pytest has to offer: the tmpdir fixture. The framework provides a few more fixtures. To verify the full list of available fixtures, run the following command:

$ (testing) pytest  -q --fixtures
There are two fixtures that we use a lot: monkeypatch and capsys, and they are in the list produced when the above command is run. This is the brief description you will see in the terminal:

capsys
    enables capturing of writes to sys.stdout/sys.stderr and makes
    captured output available via ``capsys.readouterr()`` method calls
    which return a ``(out, err)`` tuple.
monkeypatch
    The returned ``monkeypatch`` funcarg provides these
    helper methods to modify objects, dictionaries or os.environ::

    monkeypatch.setattr(obj, name, value, raising=True)
    monkeypatch.delattr(obj, name, raising=True)
    monkeypatch.setitem(mapping, name, value)
    monkeypatch.delitem(obj, name, raising=True)
    monkeypatch.setenv(name, value, prepend=False)
    monkeypatch.delenv(name, value, raising=True)
    monkeypatch.syspath_prepend(path)
    monkeypatch.chdir(path)

    All modifications will be undone after the requesting
    test function has finished. The ``raising``
    parameter determines if a KeyError or AttributeError
    will be raised if the set/deletion operation has no target.
capsys captures any stdout or stderr produced in a test. Have you ever tried to verify some command output or logging in a unit test? It is challenging to get right and is something that requires a separate plug-in or library to patch Python’s internals and then inspect its contents.

These are two test functions that verify the output produced on stderr and stdout, respectively:

import sys

def stderr_logging():
    sys.stderr.write('stderr output being produced')

def stdout_logging():
    sys.stdout.write('stdout output being produced')

def test_verify_stderr(capsys):
    stderr_logging()
    out, err = capsys.readouterr()
    assert out == ''
    assert err == 'stderr output being produced'

def test_verify_stdout(capsys):
    stdout_logging()
    out, err = capsys.readouterr()
    assert out == 'stdout output being produced'
    assert err == ''
The capsys fixture handles all the patching, setup, and helpers to retrieve the stderr and stdout produced in the test. The content is reset for every test, which ensures that the variables populate with the correct output.

monkeypatch is probably the fixture that we use the most. When testing, there are situations where the code under test is out of our control, and patching needs to happen to override a module or function to have a specific behavior. There are quite a few patching and mocking libraries (mocks are helpers to set behavior on patched objects) available for Python, but monkeypatch is good enough that you might not need to install a separate library to help out.

The following function runs a system command to capture details from a device, then parses the output, and returns a property (the ID_PART_ENTRY_TYPE as reported by blkid):

import subprocess

def get_part_entry_type(device):
    """
    Parses the ``ID_PART_ENTRY_TYPE`` from the "low level" (bypasses the cache)
    output that uses the ``udev`` type of output.
    """
    stdout = subprocess.check_output(['blkid', '-p', '-o', 'udev', device])
    for line in stdout.split('\n'):
        if 'ID_PART_ENTRY_TYPE=' in line:
            return line.split('=')[-1].strip()
    return ''
To test it, set the desired behavior on the check_output attribute of the subprocess module. This is how the test function looks using the monkeypatch fixture:

def test_parses_id_entry_type(monkeypatch):
    monkeypatch.setattr(
        'subprocess.check_output',
        lambda cmd: '\nID_PART_ENTRY_TYPE=aaaaa')
    assert get_part_entry_type('/dev/sda') == 'aaaa'
The setattr call sets the attribute on the patched callable (check_output in this case). It patches it with a lambda function that returns the one interesting line. Since the subprocess.check_output function is not under our direct control, and the get_part_entry_type function doesn’t allow any other way to inject the values, patching is the only way.

We tend to favor using other techniques like injecting values (known as dependency injection) before attempting to patch, but sometimes there is no other way. Providing a library that can patch and handle all the cleanup on testing is one more reason pytest is a joy to work with.

Infrastructure Testing
This section explains how to do infrastructure testing and validation with the Testinfra project. It is a pytest plug-in for infrastructure that relies heavily on fixtures and allows you to write Python tests as if testing code.

The previous sections went into some detail on pytest usage and examples, and this chapter started with the idea of verification at a system level. The way we explain infrastructure testing is by asking a question: How can you tell that the deployment was successful? Most of the time, this means some manual checks, such as loading a website or looking at processes, which is insufficient; it is error-prone and can get tedious if the system is significant.

Although you can initially get introduced to pytest as a tool to write and run Python unit tests, it can be advantageous to repurpose it for infrastructure testing. A few years ago Alfredo was tasked to produce an installer that exposed its features over an HTTP API. This installer was to create a Ceph cluster, involving many machines. During the QA portion of launching the API, it was common to get reports where the cluster wouldn’t work as expected, so he would get the credentials to log in to these machines and inspect them. There is a multiplier effect once you have to debug a distributed system comprising several machines: multiple configuration files, different hard drives, network setups, anything and everything can be different even if they appear to be similar.

Every time Alfredo had to debug these systems, he had an ever-growing list of things to check. Is the configuration the same on all servers? Are the permissions as expected? Does a specific user exist? He would eventually forget something and spend time trying to figure out what he was missing. It was an unsustainable process. What if I could write simple test cases against the cluster? Alfredo wrote a few simple tests to verify the items on the list to execute them against the machines making up the cluster. Before he knew it, he had a good set of tests that took a few seconds to run that would identify all kinds of issues.

That was an incredible eye-opener for improving the delivery process. He could even execute these (functional) tests while developing the installer and catch things that weren’t quite right. If the QA team caught any issues, he could run the same tests against their setup. Sometimes tests caught environmental issues: a drive was dirty and caused the deployment to fail; a configuration file from a different cluster was left behind and caused issues. Automation, granular tests, and the ability to run them often made the work better and alleviated the amount of work the QA team had to put up with.

The TestInfra project has all kinds of fixtures to test a system efficiently, and it includes a complete set of backends to connect to servers; regardless of their deployment type: Ansible, Docker, SSH, and Kubernetes are some of the supported connections. By supporting many different connection backends, you can execute the same set of tests regardless of infrastructure changes.

The next sections go through different backends and get into examples of a production project.

What Is System Validation?
System validation can happen at different levels (with monitoring and alert systems) and at different stages in the life cycle of an application, such as during pre-deployment, at runtime, or during deployment. An application that Alfredo recently put into production needed to handle client connections gracefully without any disruption, even when restarted. To sustain traffic, the application is load balanced: when the system is under heavy loads, new connections get sent to other servers with a lighter load.

When a new release gets deployed, the application has to be restarted. Restarting means that clients experience an odd behavior at best, or a very broken experience at the worst. To avoid this, the restart process waits for all client connections to terminate, the system refuses new connections, allowing it to finish work from existing clients, and the rest of the system picks up the work. When no connections are active, the deployment continues and stops services to get the newer code in.

There is validation at every step of the way: before the deployment to tell the balancer to stop sending new clients and later, verifying that no new clients are active. If that workflow converts to a test, the title could be something like: make sure that no clients are currently running. Once the new code is in, another validation step checks whether the balancer has acknowledged that the server is ready to produce work once again. Another test here could be: balancer has server as active. Finally, it makes sure that the server is receiving new client connections—yet another test to write!

Throughout these steps, verification is in place, and tests can be written to verify this type of workflow.

System validation can also be tied to monitoring the overall health of a server (or servers in a clustered environment) or be part of the continuous integration while developing the application and testing functionally. The basics of validation apply to these situations and anything else that might benefit from status verification. It shouldn’t be used exclusively for testing, although that is a good start!

Introduction to Testinfra
Writing unit tests against infrastructure is a powerful concept, and having used Testinfra for over a year, we can say that it has improved the quality of production applications we’ve had to deliver. The following sections go into specifics, such as connecting to different nodes and executing validation tests, and explore what type of fixtures are available.

To create a new virtual environment, install pytest:

$ python3 -m venv validation
$ source testing/bin/activate
(validation) $ pip install pytest
Install testinfra, ensuring that version 2.1.0 is used:

(validation) $ pip install "testinfra==2.1.0"
NOTE
pytest fixtures provide all the test functionality offered by the Testinfra project. To take advantage of this section, you will need to know how they work.

Connecting to Remote Nodes
Because different backend connection types exist, when the connection is not specified directly, Testinfra defaults to certain ones. It is better to be explicit about the connection type and define it in the command line.

These are all the connection types that Testinfra supports:

local

Paramiko (an SSH implementation in Python)

Docker

SSH

Salt

Ansible

Kubernetes (via kubectl)

WinRM

LXC

A testinfra section appears in the help menu with some context on the flags that are provided. This is a neat feature from pytest and its integration with Testinfra. The help for both projects comes from the same command:

(validation) $ pytest --help
...

testinfra:
  --connection=CONNECTION
                        Remote connection backend (paramiko, ssh, safe-ssh,
                        salt, docker, ansible)
  --hosts=HOSTS         Hosts list (comma separated)
  --ssh-config=SSH_CONFIG
                        SSH config file
  --ssh-identity-file=SSH_IDENTITY_FILE
                        SSH identify file
  --sudo                Use sudo
  --sudo-user=SUDO_USER
                        sudo user
  --ansible-inventory=ANSIBLE_INVENTORY
                        Ansible inventory file
  --nagios              Nagios plugin
There are two servers up and running. To demonstrate the connection options, let’s check if they are running CentOS 7 by poking inside the /etc/os-release file. This is how the test function looks (saved as test_remote.py):

def test_release_file(host):
    release_file = host.file("/etc/os-release")
    assert release_file.contains('CentOS')
    assert release_file.contains('VERSION="7 (Core)"')
It is a single test function that accepts the host fixture, which runs against all the nodes specified.

The --hosts flag accepts a list of hosts with a connection scheme (SSH would use ssh://hostname for example), and some other variations using globbing are allowed. If testing against more than a couple of remote servers at a time, passing them on the command line becomes cumbersome. This is how it would look to test against two servers using SSH:

(validation) $ pytest -v --hosts='ssh://node1,ssh://node2' test_remote.py
============================= test session starts =============================
platform linux -- Python 3.6.8, pytest-4.4.1, py-1.8.0, pluggy-0.9.0
cachedir: .pytest_cache
rootdir: /home/alfredo/python/python-devops/samples/chapter16
plugins: testinfra-3.0.0, xdist-1.28.0, forked-1.0.2
collected 2 items

test_remote.py::test_release_file[ssh://node1] PASSED                   [ 50%]
test_remote.py::test_release_file[ssh://node2] PASSED                   [100%]

========================== 2 passed in 3.82 seconds ===========================
The increased verbosity (with the -v flag) shows that Testinfra is executing the one test function in the two remote servers specified in the invocation.

NOTE
When setting up the hosts, it is important to have a passwordless connection. There shouldn’t be any password prompts, and if using SSH, a key-based configuration should be used.

When automating these types of tests (as part of a job in a CI system, for example), you can benefit from generating the hosts, determining how they connect, and any other special directives. Testinfra can consume an SSH configuration file to determine what hosts to connect to. For the previous test run, Vagrant was used, which created these servers with special keys and connection settings. Vagrant can generate an ad-hoc SSH config file for the servers it has created:

(validation) $ vagrant ssh-config

Host node1
  HostName 127.0.0.1
  User vagrant
  Port 2200
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/alfredo/.vagrant.d/insecure_private_key
  IdentitiesOnly yes
  LogLevel FATAL

Host node2
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/alfredo/.vagrant.d/insecure_private_key
  IdentitiesOnly yes
  LogLevel FATAL
Exporting the contents of the output to a file and then passing that to Testinfra offers greater flexibility if using more than one host:

(validation) $ vagrant ssh-config > ssh-config
(validation) $ pytest --hosts=default --ssh-config=ssh-config test_remote.py
Using --hosts=default avoids having to specify them directly in the command line, and the engine feeds from the SSH configuration. Even without Vagrant, the SSH configuration tip is still useful if connecting to many hosts with specific directives.

Ansible is another option if the nodes are local, SSH, or Docker containers. The test setup can benefit from using an inventory of hosts (much like the SSH config), which can group the hosts into different sections. The host groups can also be specified so that you can single out hosts to test against, instead of executing against all.

For node1 and node2 used in the previous example, this is how the inventory file is defined (and saved as hosts):

[all]
node1
node2
If executing against all of them, the command changes to:

$ pytest --connection=ansible --ansible-inventory=hosts test_remote.py
If defining other hosts in the inventory that need an exclusion, a group can be specified as well. Assuming that both nodes are web servers and are in the nginx group, this command would run the tests on only that one group:

$ pytest --hosts='ansible://nginx' --connection=ansible \
  --ansible-inventory=hosts test_remote.py
TIP
A lot of system commands require superuser privileges. To allow escalation of privileges, Testinfra allows specifying --sudo or --sudo-user. The --sudo flag makes the engine use sudo when executing the commands, while the --sudo-user command allows running with higher privileges as a different user.The fixture can be used directly as well.

Features and Special Fixtures
So far, the host fixture is the only one used in examples to check for a file and its contents. However, this is deceptive. The host fixture is an all-included fixture; it contains all the other powerful fixtures that Testinfra provides. This means that the example has already used the host.file, which has lots of extras packed in it. It is also possible to use the fixture directly:

In [1]: import testinfra

In [2]: host = testinfra.get_host('local://')

In [3]: node_file = host.file('/tmp')

In [4]: node_file.is_directory
Out[4]: True

In [5]: node_file.user
Out[5]: 'root'
The all-in-one host fixture makes use of the extensive API from Testinfra, which loads everything for each host it connects to. The idea is to write a single test that gets executed against different nodes, all accessible from the same host fixture.

These are a couple dozen attributes available. These are some of the most used ones:

host.ansible
Provides full access to any of the Ansible properties at runtime, such as hosts, inventory, and vars

host.addr
Network utilities, like checks for IPV4 and IPV6, is host reachable, is host resolvable

host.docker
Proxy to the Docker API, allows interacting with containers, and checks if they are running

host.interface
Helpers for inspecting addresses from a given interface

host.iptables
Helpers for verifying firewall rules as seen by host.iptables

host.mount_point
Check mounts, filesystem types as they exist in paths, and mount options

host.package
Very useful to query if a package is installed and at what version

host.process
Check for running processes

host.sudo
Allows you to execute commands with host.sudo or as a different user

host.system_info
All kinds of system metadata, such as distribution version, release, and codename

host.check_output
Runs a system command, checks its output if runs successfully, and can be used in combination with host.sudo

host.run
Runs a command, allows you to check the return code, host.stderr, and host.stdout

host.run_expect
Verifies that the return code is as expected

Examples
A frictionless way to start developing system validation tests is to do so while creating the actual deployment. Somewhat similar to Test Driven Development (TDD), any progress warrants a new test. In this section, a web server needs to be installed and configured to run on port 80 to serve a static landing page. While making progress, tests will be added. Part of writing tests is understanding failures, so a few problems will be introduced to help us figure out what to fix.

With a vanilla Ubuntu server, start by installing the Nginx package:

$ apt install nginx
Create a new test file called test_webserver.py for adding new tests after making progress. After Nginx installs, let’s create another test:

def test_nginx_is_installed(host):
    assert host.package('nginx').is_installed
Reduce the verbosity in pytest output with the -q flag to concentrate on failures. The remote server is called node4 and SSH is used to connect to it. This is the command to run the first test:

(validate) $ pytest -q --hosts='ssh://node4' test_webserver.py
.
1 passed in 1.44 seconds
Progress! The web server needs to be up and running, so a new test is added to verify that behavior:

def test_nginx_is_running(host):
    assert host.service('nginx').is_running
Running again should work once again:

(validate) $ pytest -q --hosts='ssh://node4' test_webserver.py
.F
================================== FAILURES ===================================
_____________________ test_nginx_is_running[ssh://node4] ______________________

host = <testinfra.host.Host object at 0x7f629bf1d668>

    def test_nginx_is_running(host):
>       assert host.service('nginx').is_running
E       AssertionError: assert False
E        +  where False = <service nginx>.is_running
E        +    where <service nginx> = <class 'SystemdService'>('nginx')

test_webserver.py:7: AssertionError
1 failed, 1 passed in 2.45 seconds
Some Linux distributions do not allow packages to start the services when they get installed. Moreover, the test has caught the Nginx service not running, as reported by systemd (the default unit service). Starting Nginx manually and running the test should make everything pass once again:

(validate) $ systemctl start nginx
(validate) $ pytest -q --hosts='ssh://node4' test_webserver.py
..
2 passed in 2.38 seconds
As mentioned at the beginning of this section, the web server should be serving a static landing page on port 80. Adding another test (in test_webserver.py) to verify the port is the next step:

def test_nginx_listens_on_port_80(host):
    assert host.socket("tcp://0.0.0.0:80").is_listening
This test is more involved and needs attention to some details. It opts to check for TCP connections on port 80 on any IP in the server. While this is fine for this test, if the server has multiple interfaces and is configured to bind to a specific address, then a new test would have to be added. Adding another test that checks if port 80 is listening on a given address might seem like overkill, but if you think about the reporting, it helps explain what is going on:

Test nginx listens on port 80 : PASS

Test nginx listens on address 192.168.0.2 and port 80: FAIL

The above tells us that Nginx is binding to port 80, just not to the right interface. An extra test is an excellent way to provide granularity (at the expense of extra verbosity).

Run the newly added test again:

(validate) $ pytest -q --hosts='ssh://node4' test_webserver.py
..F
================================== FAILURES ===================================
_________________ test_nginx_listens_on_port_80[ssh://node4] __________________

host = <testinfra.host.Host object at 0x7fbaa64f26a0>

    def test_nginx_listens_on_port_80(host):
>       assert host.socket("tcp://0.0.0.0:80").is_listening
E       AssertionError: assert False
E        +  where False = <socket tcp://0.0.0.0:80>.is_listening
E        +    where <socket tcp://0.0.0.0:80> = <class 'LinuxSocketSS'>

test_webserver.py:11: AssertionError
1 failed, 2 passed in 2.98 seconds
No address has anything listening on port 80. Looking at the configuration for Nginx reveals that it is set to listen on port 8080 using a directive in the default site that configures the port:

(validate) $ grep "listen 8080" /etc/nginx/sites-available/default
    listen 8080 default_server;
After changing it back to port 80 and restarting the nginx service, the tests pass again:

(validate) $ grep "listen 80" /etc/nginx/sites-available/default
    listen 80 default_server;
(validate) $ systemctl restart nginx
(validate) $ pytest -q --hosts='ssh://node4' test_webserver.py
...
3 passed in 2.92 seconds
Since there isn’t a built-in fixture to handle HTTP requests to an address, the final test uses the wget utility to retrieve the contents of the running website and make assertions on the output to ensure that the static site renders:

def test_get_content_from_site(host):
    output = host.check_output('wget -qO- 0.0.0.0:80')
    assert 'Welcome to nginx' in output
Running test_webserver.py once more verifies that all our assumptions are correct:

(validate) $ pytest -q --hosts='ssh://node4' test_webserver.py
....
4 passed in 3.29 seconds
Understanding the concepts of testing in Python, and repurposing those for system validation, is incredibly powerful. Automating test runs while developing applications or even writing and running tests on existing infrastructure are both excellent ways to simplify day-to-day operations that can become error-prone. pytest and Testinfra are great projects that can help you get started, and make it easy when extending is needed. Testing is a level up on skills.

Testing Jupyter Notebooks with pytest
One easy way to introduce big problems into your company is to forget about applying software engineering best practices when it comes to data science and machine learning. One way to fix this is to use the nbval plug-in for pytest that allows you to test your notebooks. Take a look at this Makefile:

setup:
    python3 -m venv ~/.myrepo

install:
    pip install -r requirements.txt

test:
    python -m pytest -vv --cov=myrepolib tests/*.py
    python -m pytest --nbval notebook.ipynb

lint:
    pylint --disable=R,C myrepolib cli web

all: install lint test
The key item is the --nbval flag that also allows the notebook in the repo to be tested by the build server.

Exercises
Name at least three conventions needed so that pytest can discover a test.

What is the conftest.py file for?

Explain parametrization of tests.

What is a fixture and how can it be used in tests? Is it convenient? Why?

Explain how to use the monkeypatch fixture.

Case Study Question
Create a test module to use testinfra to connect to a remote server. Test that Nginx is installed, is running with systemd, and the server is binding to port 80. When all tests pass, try to make them fail by configuring Nginx to listen on a different port.