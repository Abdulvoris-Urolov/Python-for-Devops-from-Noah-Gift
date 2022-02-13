Chapter 5. Package Management
Often, small scripts grow in usefulness and importance, which creates a need to share and distribute their contents. Python libraries, as well as other code projects, require packaging. Without packaging, distributing code becomes problematic and brittle.

Once past the proof of concept stage, it is helpful to keep track of changes, advertise the type of change (for example, when introducing a backward-incompatible update), and provide a way for users to depend on a specific version. Even in the most straightforward use cases, it is beneficial to follow a few (packaging) guidelines. This, at the very least, should mean keeping track of a changelog and determining a version.

There are several strategies to follow for package management, and knowing a few of the ones most commonly used allows you to adopt the best option to solve a problem. For example, it might be easier to distribute a Python library through the Python Package Index (PyPI) instead of making it a system package like Debian and RPM. If a Python script needs to run at specific intervals or if it is a long-running process, then system packaging working together with systemd might work better.

Although systemd is not a packaging tool, it does play well on systems that depend on it to manage processes and the server startup sequence. Learning how to handle processes with a few systemd configuration settings and some packaging is a great way to increase the capabilities of a Python project further.

The native Python packaging tools have a public hosting instance for packages (PyPI). However, for Debian and RPM packages, it requires some effort to provide a local repository. This chapter covers a few tools that make it easier to create and manage a package repository, including a local alternative to PyPI.

Having a good understanding of the different packaging strategies, and healthy practices like proper versioning and keeping a changelog, provide a stable, consistent experience when distributing software.

Why Is Packaging Important?
Several factors make packaging software an essential feature of a project (regardless of size!). Keeping track of versions and changes (via a changelog) is an excellent way to provide some insight into new features and bug fixes. Versioning allows others to determine better what might work within a project.

When trying to identify issues and bugs, a changelog with an accurate description of changes is an invaluable tool to help identify potential causes of system breakage.

It takes discipline and hard work to version a project, describe changes in a changelog, and provide a way for others to install and use a project. However, the benefits when distributing, debugging, upgrading, or even uninstalling are significant.

When Packaging Might Not Be Needed
Sometimes you don’t need to distribute a project to other systems at all. Ansible playbooks are usually run from one server to manage other systems in the network. In cases like Ansible, it might be enough to follow versioning and keep a changelog.

Version control systems like Git make this easy with the use of tags. Tagging in Git would still be useful if a project does need to get packaged, since most tooling can consume the tag (as long as a tag represents a version) to produce a package.

Recently, there was a long debug session to determine why an installer for a large software project had stopped working. Suddenly, all the functional tests for a small Python tool that depended on the installer completing its deployment were failing. The installer did have versions and kept those versions synchronized with version control, but there was no changelog whatsoever that would have explained that recent changes were going to break an existing API. To find the issue, we had to go through all the recent commits to determine what could be the problem.

Going through a few commits shouldn’t be difficult, but try doing so on a project with more than four thousand commits! After finding the cause, two tickets were opened: one that explained the bug and another one to ask for a changelog.

Packaging Guidelines
Before packaging, a few things are worth considering so that the process is as smooth as possible. Even when you don’t plan to package a product, these guidelines help to improve a project overall.

NOTE
Projects in version control are always ready to be packaged.

Descriptive Versioning
There are many ways to version software, but it is a good idea to follow a well-known schema. The Python developer’s guide has a clear definition for acceptable forms of versioning.

The versioning schema is meant to be extremely flexible, while keeping an eye on consistency so that installer tooling can make sense and prioritize accordingly (for example, a stable version over a beta one). In its purest form, and most commonly in Python packages, the following two variants are used: major.minor or major.minor.micro.

Valid versions would then look like:

0.0.1

1.0

2.1.1

NOTE
Although there are many variations described by the excellent Python developer’s guide, concentrate on the simpler forms (listed above). They are good enough to produce packages, while adhering to most guidelines for both system and native Python packages.

A commonly accepted format for releases is major.minor.micro (and also used by the Semantic Versioning scheme):

major for backward-incompatible changes

minor adds features that are also backward compatible

micro adds backward-compatible bug fixes

Following the listed versions above, you can deduce that a dependency on an application with version 1.0.0 might break with version 2.0.0.

Once the decision for a release occurs, then it is easy to determine the version number. Assuming the current released version of the project under development is 1.0.0, it means the following outcomes are possible:

If the release has backward-incompatible changes, the version is: 2.0.0

If the release has added features that do not break compatibility, the version is 1.1.0

If the release is to fix issues that also do not break compatibility, the version is 1.0.1

Once a schema is being followed, then a release process is immediately descriptive. Although it would be nice for all software to follow a similar pattern, some projects have a completely different schema of their own. For example, the Ceph project uses the following: major.[0|1|2].minor

major indicates a major release, while not necessarily breaking backward-compatibility.

0, 1, or 2 mean (in order) development release, release candidate, or stable version.

minor is used only for bug fixes and never for features.

That schema would mean that 14.0.0 is a development release, while 14.2.1 is a bug fix release for the stable version of the major release (14 in this case).

The changelog
As we have mentioned already, it is important to keep track of releases and what they mean in the context of a version number. Keeping a changelog is not that difficult, once a versioning schema is chosen. Although it can be a single file, large projects tend to break it down into smaller files in a directory. Best practice is using a simple format that is descriptive and easy to maintain.

The following example is an actual portion of a changelog file in a production Python tool:

1.1.3
^^^^^
22-Mar-2019

* No code changes - adding packaging files for Debian

1.1.2
^^^^^
13-Mar-2019

* Try a few different executables (not only ``python``) to check for a working
  one, in order of preference, starting with ``python3`` and ultimately falling
  back to the connection interpreter
The example provides four essential pieces of information:

The latest version number released

Whether the latest release is backward compatible

The release date of the last version

Changes included in the release

The file doesn’t need to be of a specific format, as long as it is consistent and informative. A proper changelog can provide several pieces of information with little effort. It is tempting to try and automate the task of writing a changelog with every release, but we would advise against a fully automated process: nothing beats a well-written, thoughtful entry about a bug fix or a feature added.

A poorly automated changelog is one that uses all the version control commits included in the release. This is not a good practice, since you can get the same information by listing the commits.

Choosing a Strategy
Understanding the type of distribution needed and what infrastructure services are available helps determine what type of packaging to use. Pure Python libraries that extend functionality for other Python projects are suitable as a native Python package, hosted on the Python Package Index (PyPI) or a local index.

Standalone scripts and long-running processes are good candidates for system packages like RPM or Debian, but ultimately, it depends on what type of systems are available and if it is at all possible to host (and manage) a repository. In the case of long-running processes, the packaging can have rules to configure a systemd unit that makes it available as a controllable process. systemd allows for the graceful handling of start, stop, or restart operations. These are things that aren’t possible with native Python packaging.

In general, the more a script or process needs to interact with a system, the better it is suited to a system package or a container. When writing a Python-only script, conventional Python packaging is the right choice.

NOTE
There aren’t hard requirements on what strategy to choose. It depends! Pick the best environment available for distribution (RPM if the servers are CentOS, for example). Different types of packaging are not mutually exclusive; one project can offer multiple packaging formats at the same time.

Packaging Solutions
In this section, the details on how to create a package and host it are covered.

To simplify the code examples, assume a small Python project called hello-world with the following structure:

hello-world
└── hello_world
    ├── __init__.py
    └── main.py

1 directory, 2 files
The project has a top-level directory called hello-world and a subdirectory (hello_world) with two files in it. Depending on the packaging choice, different files are needed to create a package.

Native Python Packaging
By far the simplest solution is using the native Python packaging tooling and hosting (via PyPI). Like the rest of the other packaging strategies, the project requires some files used by setuptools.

TIP
One easy way to source a virtual environment is to create a bash or zsh alias that both cd’s into the directory and sources the environment, like this: alias sugar="source ~/.sugar/bin/activate && cd ~/src/sugar"

To continue, create a new virtual environment and then activate:

$ python3 -m venv /tmp/packaging
$ source /tmp/packaging/bin/activate
NOTE
setuptools is a requirement to produce a native Python package. It is a collection of tools and helpers to create and distribute Python packages.

Once the virtual environment is active, the following dependencies exist:

setuptools
A set of utilities for packaging

twine
A tool for registering and uploading packages

Install them by running the following command:

$ pip install setuptools twine
NOTE
A very easy way to figure out what is installed is to use IPython and this snippet to list all of the Python packages as a JSON data structure:

In [1]: !pip list --format=json

[{"name": "appnope", "version": "0.1.0"},
 {"name": "astroid", "version": "2.2.5"},
 {"name": "atomicwrites", "version": "1.3.0"},
 {"name": "attrs", "version": "19.1.0"}]
Package files
To produce the native Python package, we have to add a few files. To keep things simple, focus on the minimum amount of files needed to produce the package. The file that describes the package to setuptools is named setup.py. It exists at the top-level directory. For the example project, this is how that file looks:

from setuptools import setup, find_packages

setup(
    name="hello-world",
    version="0.0.1",
    author="Example Author",
    author_email="author@example.com",
    url="example.com",
    description="A hello-world example package",
    packages=find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
)
The setup.py file will import two helpers from the setuptools module: setup and find_packages. The setup function is what requires the rich description about the package. The find_packages function is a utility to automatically detect where the Python files are. Additionally, the file imports classifiers that describe certain aspects of the package, such as the license, operating systems supported, and Python versions. These classifiers are called trove classifiers, and the Python Package Index has a detailed description of other classifiers available. Detailed descriptions make a package get discovered when uploaded to PyPI.

With just the addition of this one file, we can already produce a package, in this case, a source distribution package. Without a README file, a warning appears when running the commands. To prevent this, add an empty one in the top-level directory with the command: touch README.

The contents of the project directory should look like this:

hello-world
├── hello_world
│   ├── __init__.py
│   └── main.py
└── README
└── setup.py

1 directory, 4 files
To produce the source distribution from it, run the following command:

python3 setup sdist
The output should look similar to the following:

$ python3 setup.py sdist
running sdist
running egg_info
writing hello_world.egg-info/PKG-INFO
writing top-level names to hello_world.egg-info/top_level.txt
writing dependency_links to hello_world.egg-info/dependency_links.txt
reading manifest file 'hello_world.egg-info/SOURCES.txt'
writing manifest file 'hello_world.egg-info/SOURCES.txt'
running check
creating hello-world-0.0.1
creating hello-world-0.0.1/hello_world
creating hello-world-0.0.1/hello_world.egg-info
copying files to hello-world-0.0.1...
copying README -> hello-world-0.0.1
copying setup.py -> hello-world-0.0.1
copying hello_world/__init__.py -> hello-world-0.0.1/hello_world
copying hello_world/main.py -> hello-world-0.0.1/hello_world
Writing hello-world-0.0.1/setup.cfg
Creating tar archive
removing 'hello-world-0.0.1' (and everything under it)
At the top-level directory of the project, a new directory called dist is there; it contains the source distribution: a file hello-world-0.0.1.tar.gz. If we check the contents of the directory, it has changed once again:

hello-world
├── dist
│   └── hello-world-0.0.1.tar.gz
├── hello_world
│   ├── __init__.py
│   └── main.py
├── hello_world.egg-info
│   ├── dependency_links.txt
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   └── top_level.txt
├── README
└── setup.py

3 directories, 9 files
The newly created tar.gz file is an installable package! This package can now be uploaded to PyPI for others to install directly from it. By following the version schema, it allows installers to ask for a specific version (0.0.1 in this case), and the extra metadata passed into the setup() function enables other tools to discover it and show information about it, such as the author, description, and version.

The Python installer tool pip can be used to install the tar.gz file directly. To try it out, use the path to the file as an argument:

$ pip install dist/hello-world-0.0.1.tar.gz
Processing ./dist/hello-world-0.0.1.tar.gz
Building wheels for collected packages: hello-world
  Building wheel for hello-world (setup.py) ... done
Successfully built hello-world
Installing collected packages: hello-world
Successfully installed hello-world-0.0.1
The Python Package Index
The Python Package Index (PyPI) is a repository of Python software that allows users to host Python packages and also install from it. It is maintained by and for the community with the help of sponsors and donations as part of the Python Software Foundation.

NOTE
This section requires registration for the test instance of PyPI. Make sure you have an account already or register online. You need your username and password for the account to upload packages.

In the sample setup.py file, an example email address contains a placeholder. If the package is going to get published to the index, this needs to be updated to reflect the same email address that owns the project at PyPI. Update any other fields, like the author, url, and description, to more accurately reflect the project being built.

To make sure things work correctly, and to avoid pushing to production, the package is tested by uploading it to the test instance of PyPI. This test instance behaves the same as production and verifies that a package works correctly.

The setuptools and the setup.py file is the traditional method of uploading a package to PyPI. A new approach, called twine, can simplify things.

At the beginning of this section, twine got installed in the virtual environment. Next, it can be used to upload the package to the test instance of PyPI. The following command uploads the tar.gz file and prompts for the username and password:

$ twine upload --repository-url https://test.pypi.org/legacy/ \
  dist/hello-world-0.0.1.tar.gz
Uploading distributions to https://test.pypi.org/legacy/
Enter your username:
Enter your password:
To test out whether the package made it, we can try and install it with pip:

$ python3 -m pip install --index-url https://test.pypi.org/simple/ hello-world
The command looks like it has a space in the PyPI URL, but the index URL ends in /simple/, and hello-world is another argument that indicates the name of the Python package to be installed.

For an actual production release, an account would need to exist or be created. The same steps that are taken to upload to the test instance, including the validation, would also work for the real PyPI.

Older Python packaging guides may reference commands such as:

 $ python setup.py register
 $ python setup.py upload
These may still work and are part of the setuptools set of utilities to package and upload projects to a package index. However, twine offers secure authentication over HTTPS, and allows signing with gpg. Twine works regardless if python setup.py upload doesn’t, and finally, it provides a way to test a package before uploading to the index.

A final item to point out is that it may be helpful to create a Makefile and put a make command in it that automatically deploys your project and builds the documentation for you. Here is an example of how that could work:

deploy-pypi:
  pandoc --from=markdown --to=rst README.md -o README.rst
  python setup.py check --restructuredtext --strict --metadata
  rm -rf dist
  python setup.py sdist
  twine upload dist/*
  rm -f README.rst
Hosting an internal package index
In some situations, it might be preferable to host an internal PyPI.

A company where Alfredo used to work had private libraries that were not supposed to be public at all, so it was a requirement to host an instance of PyPI. Hosting has its caveats, though. All dependencies and versions of those dependencies have to exist in the instance; otherwise, installs can fail. An installer can’t fetch dependencies from different sources at the same time! On more than one occasion, a new version had a missing component, so that package had to be uploaded for the install to complete correctly.

If package A is hosted internally and has requirements on packages B and C, all three need to exist (along with their required versions) in the same instance.

An internal PyPI makes installations go faster, can keep packages private, and at its core, isn’t challenging to accomplish.

NOTE
A highly recommended full-featured tool for hosting an internal PyPI is devpi. It has features like mirroring, staging, replication, and Jenkins integration. The project documentation has great examples and detailed information.

First, create a new directory called pypi so that you can create a proper structure for hosting packages, and then create a subdirectory with the name of our example package (hello-world). The names of subdirectories are the names of the packages themselves:

$ mkdir -p pypi/hello-world
$ tree pypi
pypi
└── hello-world

1 directory, 0 files
Now copy the tar.gz file into the hello-world directory. The final version of this directory structure should look like this:

$ tree pypi
pypi
└── hello-world
    └── hello-world-0.0.1.tar.gz

1 directory, 1 file
The next step is to create a web server with auto indexing enabled. Python comes with a built-in web server that is good enough to try this out, and it even has the auto indexing enabled by default! Change directories to the pypi directory containing the hello-world package and start the built-in web server:

$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
In a new terminal session, create a temporary virtual environment to try out installing the hello-world package from the local PyPI instance. Activate it, and finally, try installing it by pointing pip to the custom local URL:

$ python3 -m venv /tmp/local-pypi
$ source /tmp/local-pypi/bin/activate
(local-pypi) $ pip install -i http://localhost:8000/ hello-world
Looking in indexes: http://localhost:8000/
Collecting hello-world
  Downloading http://localhost:8000/hello-world/hello-world-0.0.1.tar.gz
Building wheels for collected packages: hello-world
  Building wheel for hello-world (setup.py) ... done
Successfully built hello-world
Installing collected packages: hello-world
Successfully installed hello-world-0.0.1
In the session where the http.server module is running, there should be some logs demonstrating all the requests the installer made to retrieve the hello-world package:

Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
127.0.0.1 [09:58:37] "GET / HTTP/1.1" 200 -
127.0.0.1 [09:59:39] "GET /hello-world/ HTTP/1.1" 200 -
127.0.0.1 [09:59:39] "GET /hello-world/hello-world-0.0.1.tar.gz HTTP/1.1" 200
A production environment needs a better-performing web server. The http.server module is used in this example for simplicity, but it isn’t meant to handle simultaneous multiple requests or scaling out.

TIP
When building a local index without a tool like devpi, there is a defined specification that includes descriptions of normalized names for the directory structure. This specification can be found in PEP 503.

Debian Packaging
If targetting Debian (or a Debian-based distribution such as Ubuntu) for distributing a project, additional files are required. Understanding what these files are and how the Debian packaging tools use them improves the process of producing an installable .deb package and troubleshooting issues.

Some of these plain-text files require a very strict formatting, and if the format is even slightly incorrect, the packaging cannot install.

NOTE
This section assumes the packaging is in a Debian or Debian-based distro, so that it is easier to install and use the required packaging tools.

Package files
Debian packaging needs a debian directory with a few files in it. To narrow the scope of what is needed to produce a package, most of the available options are skipped, such as running a test suite before completing a build or declaring multiple Python versions.

Create the debian directory where all required files exist. In the end, the hello-world project structure should look like this:

$ tree
.
├── debian
├── hello_world
│   ├── __init__.py
│   └── main.py
├── README
└── setup.py

2 directories, 4 files
NOTE
Note that the directory includes the setup.py and README file from the native Python packaging section. It is required because Debian tooling uses these to produce the .deb package.

The changelog file
This file can be complicated to get right if done by hand. The errors produced when this file is not formatted correctly are not easy to debug. Most Debian packaging workflows rely on the dch tool to enhance debuggability.

I’ve ignored my advice before and have tried to manually create this file. In the end I wasted time because error reporting is not very good, and spotting issues is very difficult. Below is an example of an entry in the changelog file that caused a problem:

--Alfredo Deza <alfredo@example.com> Sat, 11 May 2013 2:12:00 -0800
That entry produced the following error:

parsechangelog/debian: warning: debian/changelog(l7): found start of entry where
  expected more change data or trailer
Can you spot the fix?

-- Alfredo Deza <alfredo@example.com> Sat, 11 May 2013 2:12:00 -0800
A space between the dashes and my name was the cause. Save yourself the heartache and use dch. The tool is part of the devscripts package:

$ sudo apt-get install devscripts
The dch command-line tool has many options, and it is useful to go through its documentation (the main page is comprehensive). We are going to run it to create the changelog for the first time (this requires the one-time use of the --create flag). Before running it, export your full name and email so that they get into the generated file:

$ export DEBEMAIL="alfredo@example.com"
$ export DEBFULLNAME="Alfredo Deza"
Now run dch to produce the changelog:

$ dch --package "hello-world" --create -v "0.0.1" \
  -D stable "New upstream release"
The newly created file should look similar to this:

hello-world (0.0.1) stable; urgency=medium

  * New upstream release

 -- Alfredo Deza <alfredo@example.com>  Thu, 11 Apr 2019 20:28:08 -0400
NOTE
The Debian changelog is specific to Debian packaging. It is fine to have a separate changelog for the project when the format doesn’t fit or if other information needs updating. Lots of projects keep the Debian changelog file as a separate Debian-only file.

The control file
This is the file that defines the package name, its description, and any dependencies needed for building and running the project. It also has a strict format, but it doesn’t need to change much (unlike the changelog). The file ensures that Python 3 is required and that it follows Debian’s Python naming guidelines.

NOTE
In the transition from Python 2 to Python 3, most distributions settled on using the following schema for Python 3 packages: python3-{package name}.

After adding the dependencies, naming conventions, and a short description, this is how the file should look:

Source: hello-world
Maintainer: Alfredo Deza <alfredo@example.com>
Section: python
Priority: optional
Build-Depends:
 debhelper (>= 11~),
 dh-python,
 python3-all
 python3-setuptools
Standards-Version: 4.3.0

Package: python3-hello-world
Architecture: all
Depends: ${misc:Depends}, ${python3:Depends}
Description: An example hello-world package built with Python 3
Other required files
There are a few other files needed to produce a Debian package. Most of them are just a couple of lines long and change infrequently.

The rules file is an executable file that tells Debian what to run to produce the package; in this case it should look like the following:

#!/usr/bin/make -f

export DH_VERBOSE=1

export PYBUILD_NAME=remoto

%:
    dh $@ --with python3 --buildsystem=pybuild
The compat file sets the corresponding debhelper (another packaging tool) compatibility, recommended to be set to 10 here. You might check to see whether a higher value is required if an error message complains about it:

$ cat compat
10
Without a license, the build process might not work, and it is a good idea to state the license explicitly. This particular example uses the MIT license, and this is how it should look in debian/copyright:

Format: http://www.debian.org/doc/packaging-manuals/copyright-format/1.0
Upstream-Name: hello-world
Source: https://example.com/hello-world

Files: *
Copyright: 2019 Alfredo Deza
License: Expat

License: Expat
  Permission is hereby granted, free of charge, to any person obtaining a
  copy of this software and associated documentation files (the "Software"),
  to deal in the Software without restriction, including without limitation
  the rights to use, copy, modify, merge, publish, distribute, sublicense,
  and/or sell copies of the Software, and to permit persons to whom the
  Software is furnished to do so, subject to the following conditions:
  .
  The above copyright notice and this permission notice shall be included in
  all copies or substantial portions of the Software.
  .
  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL
  THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
  FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
  DEALINGS IN THE SOFTWARE.
Finally, after adding all these new files to the debian directory, the hello-world project looks like this:

.
├── debian
│   ├── changelog
│   ├── compat
│   ├── control
│   ├── copyright
│   └── rules
├── hello_world
│   ├── __init__.py
│   └── main.py
├── README
└── setup.py

2 directories, 9 files
Producing the binary
To produce the binary, use the debuild command-line tool. For this example project, the package remains unsigned (the signing process requires a GPG key), and the debuild documentation uses an example that allows skipping the signing. The script is run from inside the source tree to build only the binary package. This command works for the hello-world project (truncated version shown here):

$ debuild -i -us -uc -b
...
dpkg-deb: building package 'python3-hello-world'
in '../python3-hello-world_0.0.1_all.deb'.
...
 dpkg-genbuildinfo --build=binary
 dpkg-genchanges --build=binary >../hello-world_0.0.1_amd64.changes
dpkg-genchanges: info: binary-only upload (no source code included)
 dpkg-source -i --after-build hello-world-debian
dpkg-buildpackage: info: binary-only upload (no source included)
Now running lintian hello-world_0.0.1_amd64.changes ...
E: hello-world changes: bad-distribution-in-changes-file stable
Finished running lintian.
A python3-hello-world_0.0.1_all.deb should now exist in the upper directory. The lintian call (a Debian packaging linter) complains at the very end that the changelog file has an invalid distribution, which is fine because we aren’t targeting a single distribution in particular (for example, Debian Buster). Rather, we are building a package that will most likely install in any Debian-base distro that complies with the dependencies (only Python 3, in this case).

Debian repositories
There are many tools to automate Debian repositories, but it is useful to understand how to go about creating one (Alfredo even helped develop one for both RPM and Debian!). To continue, ensure that the binary package created previously is available at a known location:

$ mkdir /opt/binaries
$ cp python3-hello-world_0.0.1_all.deb /opt/binaries/
For this section, the reprepro tool needs to be installed:

$ sudo apt-get install reprepro
Create a new directory somewhere in the system to hold packages. This example uses /opt/repo. The basic configuration for a repository needs a file, called distributions, that describes the contents and looks like this:

Codename: sid
Origin: example.com
Label: example.com
Architectures: amd64 source
DscIndices: Sources Release .gz .bz2
DebIndices: Packages Release . .gz .bz2
Components: main
Suite: stable
Description: example repo for hello-world package
Contents: .gz .bz2
Save this file at /opt/repo/conf/distributions. Create another directory to hold the actual repo:

$ mkdir /opt/repo/debian/sid
To create the repository, instruct reprepro to use the distributions file created, and that the base directory is /opt/repo/debian/sid. Finally, add the binary previously created as a target for the Debian sid distribution:

$ reprepro --confdir /opt/repo/conf/distributions -b /opt/repo/debian/sid \
  -C main includedeb sid /opt/binaries/python3-hello-world_0.0.1_all.deb
Exporting indices...
This command creates the repo for the Debian sid distribution! This command can be adapted for a different Debian-based distribution such as Ubuntu Bionic, for example. To do so would only require replacing sid with bionic.

Now that the repo exists, the next step is to ensure that it works as expected. For a production environment, a robust web server like Apache or Nginx would be a good choice, but to test this, use Python’s http.server module. Change directories to the directory containing the repository, and start the server:

$ cd /opt/repo/debian/sid
$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
Aptitude (or apt, the Debian package manager) needs some configuration to be aware of this new location for packages. This configuration is a simple file with a single line in it pointing to the URL and components of our repo. Create a file at /etc/apt/sources.lists.d/hello-world.list. It should look like this:

$ cat /etc/apt/sources.list.d/hello-world.list
deb [trusted=yes] http://localhost:8000/ sid main
The [trusted=yes] configuration tells apt not to enforce signed packages. On repositories that are properly signed, this step is not necessary.

After adding the file, update apt so that it recognizes the new location, and look for (and install) the hello-world package:

$ sudo apt-get update
Ign:1 http://localhost:8000 sid InRelease
Get:2 http://localhost:8000 sid Release [2,699 B]
Ign:3 http://localhost:8000 sid Release.gpg
Get:4 http://localhost:8000 sid/main amd64 Packages [381 B]
Get:5 http://localhost:8000 sid/main amd64 Contents (deb) [265 B]
Fetched 3,345 B in 1s (6,382 B/s)
Reading package lists... Done
Searching for the python3-hello-world package provides the description added in the distributions file when configuring reprepro:

$ apt-cache search python3-hello-world
python3-hello-world - An example hello-world package built with Python 3
Installing and removing the package should work without a problem:

$ sudo apt-get install python3-hello-world
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  python3-hello-world
0 upgraded, 1 newly installed, 0 to remove and 48 not upgraded.
Need to get 2,796 B of archives.
Fetched 2,796 B in 0s (129 kB/s)
Selecting previously unselected package python3-hello-world.
(Reading database ... 242590 files and directories currently installed.)
Preparing to unpack .../python3-hello-world_0.0.1_all.deb ...
Unpacking python3-hello-world (0.0.1) ...
Setting up python3-hello-world (0.0.1) ...
$ sudo apt-get remove --purge python3-hello-world
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be REMOVED:
  python3-hello-world*
0 upgraded, 0 newly installed, 1 to remove and 48 not upgraded.
After this operation, 19.5 kB disk space will be freed.
Do you want to continue? [Y/n] Y
(Reading database ... 242599 files and directories currently installed.)
Removing python3-hello-world (0.0.1) ...
RPM Packaging
Just as with Debian packaging, when working in RPM, it is necessary to have the native Python packaging done already. It should be possible to produce a Python package with a setup.py file. However, very unlike Debian, in which many files are needed, RPM packaging can work with just one: the spec file. If targeting a distribution like CentOS or Fedora, the RPM Package Manager (formerly known as Red Hat Package Manager) is the way to go.

The spec file
In its simplest form, the spec file (named hello-world.spec for this example) is not difficult to understand, and most sections are self-explanatory. It can even be generated by using setuptools:

$ python3 setup.py bdist_rpm --spec-only
running bdist_rpm
running egg_info
writing hello_world.egg-info/PKG-INFO
writing dependency_links to hello_world.egg-info/dependency_links.txt
writing top-level names to hello_world.egg-info/top_level.txt
reading manifest file 'hello_world.egg-info/SOURCES.txt'
writing manifest file 'hello_world.egg-info/SOURCES.txt'
writing 'dist/hello-world.spec'
The output file in dist/hello-world.spec should look similar to this:

%define name hello-world
%define version 0.0.1
%define unmangled_version 0.0.1
%define release 1

Summary: A hello-world example pacakge
Name: %{name}
Version: %{version}
Release: %{release}
Source0: %{name}-%{unmangled_version}.tar.gz
License: MIT
Group: Development/Libraries
BuildRoot: %{_tmppath}/%{name}-%{version}-%{release}-buildroot
Prefix: %{_prefix}
BuildArch: noarch
Vendor: Example Author <author@example.com>
Url: example.com

%description
A Python3 hello-world package

%prep
%setup -n %{name}-%{unmangled_version} -n %{name}-%{unmangled_version}

%build
python3 setup.py build

%install
python3 setup.py install --single-version-externally-managed -O1 \
--root=$RPM_BUILD_ROOT --record=INSTALLED_FILES

%clean
rm -rf $RPM_BUILD_ROOT

%files -f INSTALLED_FILES
%defattr(-,root,root)
Although it looks simple, it is already creating a potential issue: the version is input and requires updating every time. This process is similar to Debian’s changelog file, which needs to have the version bumped on each release.

The setuptools integration is advantageous, allows further modification to this file if needed, and copies to the root directory of the project for a permanent location. Some projects use a base template that gets populated to generate the spec file as part of the build process. This process is useful if following a rigorous release workflow. In the case of the Ceph project, the release is tagged via version control (Git), and the release scripts use that tag to apply it to the template via a Makefile. It is worth noting that additional methods exist to automate this process further.

TIP
Generating a spec file is not always useful, because certain sections might need to be hardcoded to follow some distribution rule or a specific dependency that is not part of the generated file. In such cases, it is best to generate it once and configure it further to finally save it and make the spec file a formal part of the project.

Producing the binary
There are a few different tools to produce RPM binaries; one in particular is the rpmbuild command-line tool:

$ sudo yum install rpm-build
NOTE
The command-line tool is rpmbuild but the package is called rpm-build, so make sure that rpmbuild (the command-line tool) is available in the terminal.

A directory structure is required by rpmbuild to create the binary. After the directories are created, the source file (the tar.gz file generated by setuptools) needs to be present in the SOURCES directory. This is how the structure should be created and how it will look once it is done:

$ mkdir -p /opt/repo/centos/{SOURCES,SRPMS,SPECS,RPMS,BUILD}
$ cp dist/hello-world-0.0.1.tar.gz /opt/repo/centos/SOURCES/
$ tree /opt/repo/centos
/opt/repo/centos
├── BUILD
├── BUILDROOT
├── RPMS
├── SOURCES
│   └── hello-world-0.0.1.tar.gz
├── SPECS
└── SRPMS

6 directories, 1 file
The directory structure is always needed, and by default, rpmbuild requires it in the home directory. To keep things isolated, a different location (in /opt/repo/centos) is used. This process means configure rpmbuild uses this directory instead. This process produces both a binary and a source package with the -ba flag (output is abbreviated):

$ rpmbuild -ba --define "_topdir /opt/repo/centos"  dist/hello-world.spec
...
Executing(%build): /bin/sh -e /var/tmp/rpm-tmp.CmGOdp
running build
running build_py
creating build
creating build/lib
creating build/lib/hello_world
copying hello_world/main.py -> build/lib/hello_world
copying hello_world/__init__.py -> build/lib/hello_world
Executing(%install): /bin/sh -e /var/tmp/rpm-tmp.CQgOKD
+ python3 setup.py install --single-version-externally-managed \
-O1 --root=/opt/repo/centos/BUILDROOT/hello-world-0.0.1-1.x86_64
running install
writing hello_world.egg-info/PKG-INFO
writing dependency_links to hello_world.egg-info/dependency_links.txt
writing top-level names to hello_world.egg-info/top_level.txt
reading manifest file 'hello_world.egg-info/SOURCES.txt'
writing manifest file 'hello_world.egg-info/SOURCES.txt'
running install_scripts
writing list of installed files to 'INSTALLED_FILES'
Processing files: hello-world-0.0.1-1.noarch
Provides: hello-world = 0.0.1-1
Wrote: /opt/repo/centos/SRPMS/hello-world-0.0.1-1.src.rpm
Wrote: /opt/repo/centos/RPMS/noarch/hello-world-0.0.1-1.noarch.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.gcIJgT
+ umask 022
+ cd /opt/repo/centos//BUILD
+ cd hello-world-0.0.1
+ rm -rf /opt/repo/centos/BUILDROOT/hello-world-0.0.1-1.x86_64
+ exit 0
The directory structure at /opt/repo/centos will have lots of new files, but we are only interested in the one that has the noarch RPM:

$ tree /opt/repo/centos/RPMS
/opt/repo/centos/RPMS
└── noarch
    └── hello-world-0.0.1-1.noarch.rpm

1 directory, 1 file
The noarch RPM is an installable RPM package! The tool produced other useful packages that can be published as well (look at /opt/repo/centos/SRPMS, for example).

RPM repositories
To create an RPM repository, use the createrepo command-line tool. It handles the creation of the repository metadata (XML-based RPM metadata) from the binaries it finds in a given directory. In this section, create (and host) the noarch binary:

$ sudo yum install createrepo
You can create the repository in the same location used to produce the noarch package, or use a new (clean) directory. Create new binaries if needed. Once that is completed, the package copies:

$ mkdir -p /var/www/repos/centos
$ cp -r /opt/repo/centos/RPMS/noarch /var/www/repos/centos
To create the metadata, run the createrepo tool:

$ createrepo -v /var/www/repos/centos/noarch
Spawning worker 0 with 1 pkgs
Worker 0: reading hello-world-0.0.1-1.noarch.rpm
Workers Finished
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Starting other db creation: Thu Apr 18 09:13:35 2019
Ending other db creation: Thu Apr 18 09:13:35 2019
Starting filelists db creation: Thu Apr 18 09:13:35 2019
Ending filelists db creation: Thu Apr 18 09:13:35 2019
Starting primary db creation: Thu Apr 18 09:13:35 2019
Ending primary db creation: Thu Apr 18 09:13:35 2019
Sqlite DBs complete
Although an x86_64 package does not exist, repeat the createrepo call for this new directory so that yum doesn’t complain about it later:

$ mkdir /var/www/repos/centos/x86_64
$ createrepo -v /var/www/repos/centos/x86_64
We are going to use the http.server module to serve this directory over HTTP:

$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
To access this repository, yum needs to be configured with a repo file. Create one at /etc/yum.repos.d/hello-world.repo. It should look like this:

[hello-world]
name=hello-world example repo for noarch packages
baseurl=http://0.0.0.0:8000/$basearch
enabled=1
gpgcheck=0
type=rpm-md
priority=1

[hello-world-noarch]
name=hello-world example repo for noarch packages
baseurl=http://0.0.0.0:8000/noarch
enabled=1
gpgcheck=0
type=rpm-md
priority=1
Note how the gpgcheck value is 0. This means we haven’t signed any packages and yum should not try to verify a signature, preventing a failure in this example. Searching for the package should now be possible, giving us the description as part of the output:

$ yum --enablerepo=hello-world search hello-world
Loaded plugins: fastestmirror, priorities
Loading mirror speeds from cached hostfile
 * base: reflector.westga.edu
 * epel: mirror.vcu.edu
 * extras: mirror.steadfastnet.com
 * updates: mirror.mobap.edu
base                                                                   | 3.6 kB
extras                                                                 | 3.4 kB
hello- world                                                           | 2.9 kB
hello-world-noarch                                                     | 2.9 kB
updates                                                                | 3.4 kB
8 packages excluded due to repository priority protections
===============================================================================
matched: hello-world
===============================================================================
hello-world.noarch : A hello-world example pacakge
The search function works correctly; installing the package should work as well:

$ yum --enablerepo=hello-world install hello-world
Loaded plugins: fastestmirror, priorities
Loading mirror speeds from cached hostfile
 * base: reflector.westga.edu
 * epel: mirror.vcu.edu
 * extras: mirror.steadfastnet.com
 * updates: mirror.mobap.edu
8 packages excluded due to repository priority protections
Resolving Dependencies
--> Running transaction check
---> Package hello-world.noarch 0:0.0.1-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved
Installing:
 hello-world          noarch          0.0.1-1            hello-world-noarch

Transaction Summary
Install  1 Package

Total download size: 8.1 k
Installed size: 1.3 k
Downloading packages:
hello-world-0.0.1-1.noarch.rpm                                         | 8.1 kB
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : hello-world-0.0.1-1.noarch
  Verifying  : hello-world-0.0.1-1.noarch

Installed:
  hello-world.noarch 0:0.0.1-1

Complete!
Removing has to work as well:

$ yum remove hello-world
Loaded plugins: fastestmirror, priorities
Resolving Dependencies
--> Running transaction check
---> Package hello-world.noarch 0:0.0.1-1 will be erased
--> Finished Dependency Resolution

Dependencies Resolved
Removing:
 hello-world          noarch          0.0.1-1           @hello-world-noarch

Transaction Summary
Remove  1 Package

Installed size: 1.3 k
Is this ok [y/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Erasing    : hello-world-0.0.1-1.noarch
  Verifying  : hello-world-0.0.1-1.noarch
Removed:
  hello-world.noarch 0:0.0.1-1
Complete!
The http.server module should display some activity, demonstrating that yum was reaching out to get the hello-world package:

[18/Apr/2019 03:37:24] "GET /x86_64/repodata/repomd.xml HTTP/1.1"
[18/Apr/2019 03:37:24] "GET /noarch/repodata/repomd.xml HTTP/1.1"
[18/Apr/2019 03:37:25] "GET /x86_64/repodata/primary.sqlite.bz2 HTTP/1.1"
[18/Apr/2019 03:37:25] "GET /noarch/repodata/primary.sqlite.bz2 HTTP/1.1"
[18/Apr/2019 03:56:49] "GET /noarch/hello-world-0.0.1-1.noarch.rpm HTTP/1.1"
Management with systemd
systemd is a system and service manager for Linux (also known as init system). It is the default init system for many distributions, such as Debian and Red Hat. Here are some of the many features systemd provides:

Easy parallelization

Hooks and triggers for on-demand behavior

Logging integration

Ability to depend on other units for orchestrating complicated startups

There are plenty of other exciting aspects of systemd, such as network, DNS, and even mounting for devices. The idea of handling processes with ease in Python has always been challenging; at one point there were a few init-like projects in Python to choose from, all with their configuration and handling APIs. Using systemd allows portability and makes it easy to collaborate with others since it is widely available.

TIP
Two well-known process handlers in Python are supervisord and circus.

Not long ago, Alfredo wrote a small Python HTTP API that needed to go into production. The project had transitioned from supervisord to circus, and things were working fine. Unfortunately, production constraints meant the integration of systemd with the OS. The transition was rough because systemd was reasonably new, but once things were in place, we benefited from having the same production-like handling for development and catching integration issues earlier in the development cycle. When the API went into the release, we already felt comfortable with systemd to troubleshoot problems and even fine-tune the configuration to cope with external issues. (Have you ever seen an init script fail because the network was not operational?)

In this section we build a small HTTP service that needs to be available when the system boots and can restart at any moment. The unit configuration handles logging and ensures that specific system resources are available before attempting to start.

Long-Running Processes
Processes that are meant to be running all the time are excellent candidates to be handled with systemd. Consider how a DNS or mail server works; these are always on programs, and they need some handling to capture logging or restart when configuration changes.

We are going to use a small HTTP API server, based on the Pecan web framework.

NOTE
There is nothing specific in this section as to how Pecan works, so that the examples can be used for other frameworks or long-running services.

Setting It Up
Pick a permanent location for the project to create a directory at /opt/http, and then create a new virtual environment and install the Pecan framework:

$ mkdir -p /opt/http
$ cd /opt/http
$ python3 -m venv .
$ source bin/activate
(http) $ pip install "pecan==1.3.3"
Pecan has some built-in helpers that can create the necessary files and directories for an example project. Pecan can be used to create a basic “vanilla” HTTP API project that hooks up to systemd. Version 1.3.3 has two options: the base and the rest-api flavors:

$ pecan create api rest-api
Creating /opt/http/api
Recursing into +package+
  Creating /opt/http/api/api
...
Copying scaffolds/rest-api/config.py_tmpl to /opt/http/api/config.py
Copying scaffolds/rest-api/setup.cfg_tmpl to /opt/http/api/setup.cfg
Copying scaffolds/rest-api/setup.py_tmpl to /opt/http/api/setup.py
TIP
It is important to use a consistent path, because it is used later when configuring the service with systemd.

By including the project scaffolding, we now have a fully functional project with no effort. It even has a setup.py file with everything in it, ready to become a native Python package! Let’s install the project so that we can run it:

(http) $ python setup.py install
running install
running bdist_egg
running egg_info
creating api.egg-info
...
creating dist
creating 'dist/api-0.1-py3.6.egg' and adding 'build/bdist.linux-x86_64/egg'
removing 'build/bdist.linux-x86_64/egg' (and everything under it)
Processing api-0.1-py3.6.egg
creating /opt/http/lib/python3.6/site-packages/api-0.1-py3.6.egg
Extracting api-0.1-py3.6.egg to /opt/http/lib/python3.6/site-packages
...
Installed /opt/http/lib/python3.6/site-packages/api-0.1-py3.6.egg
Processing dependencies for api==0.1
Finished processing dependencies for api==0.1
The pecan command-line tool requires a configuration file. The configuration file has already been created for you by the scaffolding, and it lives in the top directory. Start the server with the config.py file:

(http) $ pecan serve config.py
Starting server in PID 17517
serving on 0.0.0.0:8080, view at http://127.0.0.1:8080
Testing it out on the browser should produce a plain-text message. This is how it shows with the curl command:

(http) $ curl localhost:8080
Hello, World!
A long-running process starts with pecan serve config.py. The only way to stop this process is to send a KeyboardInterrupt with Control-C. Starting it again requires the virtual environment to be activated, and the same pecan serve command runs again.

The systemd Unit File
Unlike older init systems that work with executable scripts, systemd works with plain text files. The final version of the unit file looks like this:

[Unit]
Description=hello world pecan service
After=network.target

[Service]
Type=simple
ExecStart=/opt/http/bin/pecan serve /opt/http/api/config.py
WorkingDirectory=/opt/http/api
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
Save this file as hello-world.service. It will be copied into its final destination later in this section.

It is essential to get all the section names and the configuration directives correct, as all are case-sensitive. If names don’t match exactly, things won’t work. Let’s go into detail for each section of the HTTP service:

Unit
Provides a description and includes an After directive that tells systemd that this service unit needs to have an operational network environment before being started. Other units may have more complex requirements, not only to start the service but even after it starts! Condition and Wants are other directives that are very useful.

Service
This section is only needed when configuring a service unit. It defaults to Type=simple. Services of this type should not fork—they have to stay in the foreground so that systemd can handle their operation. The ExecStart line explains what the command should run to start the service. It is crucial to use absolute paths to avoid problems finding the right files.

Although not required, I’ve included the WorkingDirectory directive to ensure that the process is in the same directory where the application lives. If anything updates later, it might benefit from already being in a position relative to the application.

Both the StandardOutput and StandardError directives are great to work with, and show how much systemd has to offer here. It will handle all the logging emitted via stdout and stderr through systemd machinery. We will demonstrate this further when explaining how to interact with the service.

Install
The WantedBy directive explains how this unit handles once it is enabled. The multi-user.target is equivalent to runlevel 3 (the normal run level for a server that boots into a terminal). This type of configuration allows the system to determine how it behaves once enabled. Once enabled, a symlink is created in the multi-user.target.wants directory.

Installing the Unit
The configuration file itself has to go to a specific location so that systemd can pick it up and load it. Various locations are supported, but /etc/systemd/system is for units that are created or managed by an administrator.

TIP
It is useful to make sure that the ExecStart directive works with those paths. Using absolute paths increases the chance of introducing a typo. To verify, run the whole line in the terminal and look for output similar to this:

$ /opt/http/bin/pecan serve /opt/http/api/config.py
Starting server in PID 20621
serving on 0.0.0.0:8080, view at http://127.0.0.1:8080
After verifying that the command works, copy the unit file into this directory using hello-world.service as the name:

$ cp hello-world.service /etc/systemd/system/
Once in place, systemd needs to be reloaded to make it aware of this new unit:

$ systemctl daemon-reload
The service is now fully functional and can be started and stopped. This process is verified by using the status subcommand. Let’s go through the different commands you can use to interact with the service. First, let’s see if systemd recognizes it. This is how it should behave and what the output looks like:

$ systemctl status hello-world
● hello-world.service - hello world pecan service
   Loaded: loaded (/etc/systemd/system/hello-world.service; disabled; )
   Active: inactive (dead)
Since the service is not running, it is not surprising to see it reported as dead. Start the service next and check the status once again (curl should report nothing is running on port 8080):

$ curl localhost:8080
curl: (7) Failed to connect to localhost port 8080: Connection refused
$ systemctl start hello-world
$ systemctl status hello-world
● hello-world.service - hello world pecan service
   Loaded: loaded (/etc/systemd/system/hello-world.service; disabled; )
   Active: active (running) since Tue 2019-04-23 13:44:20 EDT; 5s ago
 Main PID: 23980 (pecan)
    Tasks: 1 (limit: 4915)
   Memory: 20.1M
   CGroup: /system.slice/hello-world.service
           └─23980 /opt/http/bin/python /opt/http/bin/pecan serve config.py

Apr 23 13:44:20 huando systemd[1]: Started hello world pecan service.
The service is running and fully operational. Verify it once again on port 8080 to make sure that the framework is up and running and responding to requests:

$ curl localhost:8080
Hello, World!
If you stop the service with systemctl stop hello-world, the curl command will report a connection failure once again.

So far, we have created and installed the unit, verified it works by starting and stopping the service, and checked if the Pecan framework is responding to requests on its default port. You want this service up and running if the server reboots at any time, and this is where the Install section helps. Let’s enable the service:

$ systemctl enable hello-world
Created symlink hello-world.service → /etc/systemd/system/hello-world.service.
When the server restarts, the small HTTP API service is up and running.

Log Handling
Since this is a configured service with logging configuration (all stdout and stderr is going directly into systemd), the handling works for free. No need to configure file-based logging, rotation, or even expiration. There are a few interesting and very nice features provided by systemd that allow you to interact with logs, such as limiting the time range and filtering by unit or process ID.

NOTE
The command to interact with logs from a unit is done through the journalctl command-line tool. This process might be a surprise if expecting another subcommand from systemd to provide the logging helpers.

Since we started the service and sent some requests to it via curl in the previous section, let’s see what the logs say:

$ journalctl -u hello-world
-- Logs begin at Mon 2019-04-15 09:05:11 EDT, end at Tue 2019-04-23
Apr 23 13:44:20 srv1 systemd[1]: Started hello world pecan service.
Apr 23 13:44:44 srv1 pecan[23980] [INFO    ] [pecan.commands.serve] GET / 200
Apr 23 13:44:55 srv1 systemd[1]: Stopping hello world pecan service...
Apr 23 13:44:55 srv1 systemd[1]: hello-world.service: Main process exited
Apr 23 13:44:55 srv1 systemd[1]: hello-world.service: Succeeded.
Apr 23 13:44:55 srv1 systemd[1]: Stopped hello world pecan service.
The -u flag specifies the unit, which in this case is hello-world, but you can also use a pattern or even specify multiple units.

A common way to follow a log as it produces entries is to use the tail command. Specifically, this looks like:

$ tail -f pecan-access.log
The command to accomplish the same thing with journalctl looks slightly different, but it works in the same way:

$ journalctl -fu hello-world
Apr 23 13:44:44 srv1 pecan[23980][INFO][pecan.commands.serve] GET / 200
Apr 23 13:44:44 srv1 pecan[23980][INFO][pecan.commands.serve] GET / 200
Apr 23 13:44:44 srv1 pecan[23980][INFO][pecan.commands.serve] GET / 200
TIP
If the systemd package is available with the pcre2 engine, it allows you to use --grep. This further filters out log entries based on a pattern.

The -f flag means to follow the log, and it starts from the most recent entries and continues to show the entries as they happen, just like tail -f would. In production, the number of logs may be too many, and errors might have been showing up today. In those cases, you can use a combination of --since and --until. Both these flags accept a few different types of parameters:

today

yesterday

"3 hours ago"

-1h

-15min

-1h35min

In our small example, journalctl is unable to find anything for the last 15 minutes. At the beginning of the output, it informs us of the range and produces the entries, if any:

$  journalctl -u hello-world --since "-15min"
-- Logs begin at Mon 2019-04-15 09:05:11 EDT, end at Tue 2019-04-23
-- No entries --
Exercises
Use three different commands to get log output from systemd using journalctl.

Explain what the WorkinDirectory configuration option is for systemd units.

Why is a changelog important?

What is a setup.py file for?

Name three differences between Debian and RPM packages.

Case Study Question
Create a local instance of PyPI using devpi, upload a Python package, and then try to install that Python package from the local devpi instance.