Chapter 2. Automating Files and the Filesystem
One of Python’s most powerful features is its ability to manipulate text and files. In the DevOps world, you are continually parsing, searching, and changing the text in files, whether you’re searching application logs or propagating configuration files. Files are a means of persisting the state of your data, code, and configuration; they are how you look back at what happened in logs and how you control what happens with configuration. With Python, you can create, read, and change files and text in the code that you can use repeatedly. Automating these tasks is indeed one aspect of modern DevOps that separates it from traditional system administration. Rather than keeping a set of instructions that you have to follow manually, you can write code. This diminishes your chances of missing steps or doing them out of order. If you are confident that your system uses the same steps every time you run it, you can have greater understanding and confidence in the process.

Reading and Writing Files
You can use the open function to create a file object that can read and write files. It takes two arguments, the path of the file and the mode (mode optionally defaults to reading). You use the mode to indicate, among other things, if you want to read or write a file and if it is text or binary data. You can open a text file using the mode r to read its contents. The file object has a read method that returns the contents of the file as a string:

In [1]: file_path = 'bookofdreams.txt'
In [2]: open_file = open(file_path, 'r')
In [3]: text = open_file.read()
In [4]: len(text)
Out[4]: 476909

In [5]: text[56]
Out[5]: 's'

In [6]: open_file
Out[6]: <_io.TextIOWrapper name='bookofdreams.txt' mode='r' encoding='UTF-8'>

In [7]: open_file.close()
NOTE
It is a good practice to close a file when you finish with it. Python closes a file when it is out of scope, but until then the file consumes resources and may prevent other processes from opening it.

You can also read a file using the readlines method. This method reads the file and splits its contents on newline characters. It returns a list of strings. Each string is one line of the original text:

In [8]: open_file = open(file_path, 'r')
In [9]: text = open_file.readlines()
In [10]: len(text)
Out[10]: 8796

In [11]: text[100]
Out[11]: 'science, when it admits the possibility of occasional hallucinations\n'

In [12]: open_file.close()
A handy way of opening files is to use with statements. You do not need to close a file explicitly in this case. Python closes it and releases the file resource at the end of the indented block:

In [13]: with open(file_path, 'r') as open_file:
    ...:     text = open_file.readlines()
    ...:

In [14]: text[101]
Out[14]: 'in the sane and healthy, also admits, of course, the existence of\n'

In [15]: open_file.closed
Out[15]: True
Different operating systems use different escaped characters to represent line endings. Unix systems use \n and Windows systems use \r\n. Python converts these to \n when you open a file as text. If you are opening a binary file, such as a .jpeg image, you are likely to corrupt the data by this conversion if you open it as text. You can, however, read binary files by appending a b to mode:

In [15]: file_path = 'bookofdreamsghos00lang.pdf'
In [16]: with open(file_path, 'rb') as open_file:
    ...:     btext = open_file.read()
    ...:

In [17]: btext[0]
Out[17]: 37

In [18]: btext[:25]
Out[18]: b'%PDF-1.5\n%\xec\xf5\xf2\xe1\xe4\xef\xe3\xf5\xed\xe5\xee\xf4\n18'
Adding this opens the file without any line-ending conversion.

To write to a file, use the write mode, represented as the argument w. The tool direnv is used to automatically set up some development environments. You can define environment variables and application runtimes in a file named .envrc; direnv uses it to set these things up when you enter the directory with the file. You can set the environment variable STAGE to PROD and TABLE_ID to token-storage-1234 in such a file in Python by using open with the write flag:

In [19]: text = '''export STAGE=PROD
    ...: export TABLE_ID=token-storage-1234'''

In [20]: with open('.envrc', 'w') as opened_file:
    ...:     opened_file.write(text)
    ...:

In [21]: !cat .envrc
export STAGE=PROD
export TABLE_ID=token-storage-1234
WARNING
Be warned that pathlib’s write method will overwrite a file if it already exists.

The open function creates a file if it does not already exist and overwrites if it does. If you want to keep existing contents and only append the file, use the append flag a. This flag appends new text to the end of the file while keeping the original content. If you are writing nontext content, such as the contents of a .jpeg file, you are likely to corrupt it if you use either the w or a flag. This corruption is likely as Python converts line endings to platform-specific ones when it writes text data. To write binary data, you can safely use wb or ab.

Chapter 3 covers pathlib in depth. Two useful features are convenience functions for reading and writing files. pathlib handles the file object behind the scenes. The following allows you to read text from a file:

In [35]: import pathlib

In [36]: path = pathlib.Path(
           "/Users/kbehrman/projects/autoscaler/check_pending.py")

In [37]: path.read_text()
To read binary data, use the path.read_bytes method.

When you want to overwrite a file or write a new file, there are methods for writing text and for writing binary data:

In [38]: path = pathlib.Path("/Users/kbehrman/sp.config")

In [39]: path.write_text("LOG:DEBUG")
Out[39]: 9

In [40]: path = pathlib.Path("/Users/kbehrman/sp")
Out[41]: 8
Reading and writing using the file object’s read and write functions is usually adequate for unstructured text, but what if you are dealing with more complex data? The Javascript Object Notation (JSON) format is widely used to store simple structured data in modern web services. It uses two data structures: a mapping of key-value pairs similar to a Python dict and a list of items somewhat similar to a Python list. It defines data types for numbers, strings, booleans (which hold true/false values), and nulls (empty values). The AWS Identity and Access Management (IAM) web service allows you to control access to AWS resources. It uses JSON files to define access policies, as in this sample file:

{
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Allow",
        "Action": "service-prefix:action-name",
        "Resource": "*",
        "Condition": {
            "DateGreaterThan": {"aws:CurrentTime": "2017-07-01T00:00:00Z"},
            "DateLessThan": {"aws:CurrentTime": "2017-12-31T23:59:59Z"}
        }
    }
}
You could use the standard file object read or readlines methods to get the data from such a file:

In [8]: with open('service-policy.json', 'r') as opened_file:
   ...:     policy = opened_file.readlines()
   ...:
   ...:
The result would not be immediately usable, as it would be a single string or list of strings, depending on your chosen read method:

In [9]: print(policy)
['{\n',
 '    "Version": "2012-10-17",
\n',
 '    "Statement": {\n',
 '        "Effect": "Allow",
\n',
 '        "Action": "service-prefix:action-name",
\n',
 '        "Resource": "*",
\n',
 '        "Condition": {\n',
 '            "DateGreaterThan": {"aws:CurrentTime": "2017-07-01T00:00:00Z"},
\n',
 '            "DateLessThan": {"aws:CurrentTime": "2017-12-31T23:59:59Z"}\n',
 '        }\n',
 '    }\n',
 '}\n']
You would then need to parse this string (or strings) into data structures and types that match the original, which may be a great deal of work. A far better way is to use the json module:

In [10]: import json

In [11]: with open('service-policy.json', 'r') as opened_file:
    ...:     policy = json.load(opened_file)
    ...:
    ...:
    ...:
This module parses the JSON format for you, returning the data in appropriate Python data structures:

In [13]: from pprint import pprint

In [14]: pprint(policy)
{'Statement': {'Action': 'service-prefix:action-name',
               'Condition': {'DateGreaterThan':
                                  {'aws:CurrentTime': '2017-07-01T00:00:00Z'},
                             'DateLessThan':
                                  {'aws:CurrentTime': '2017-12-31T23:59:59Z'}},
               'Effect': 'Allow',
               'Resource': '*'},
 'Version': '2012-10-17'}
NOTE
The pprint module automatically formats Python objects for printing. Its output is often more easily read and is a handy way of looking at nested data structures.

Now you can use the data with the original file structure. For example, here is how you would change the resource whose access this policy controls to S3:

In [15]: policy['Statement']['Resource'] = 'S3'

In [16]: pprint(policy)
{'Statement': {'Action': 'service-prefix:action-name',
               'Condition': {'DateGreaterThan':
                                {'aws:CurrentTime': '2017-07-01T00:00:00Z'},
                             'DateLessThan':
                                {'aws:CurrentTime': '2017-12-31T23:59:59Z'}},
               'Effect': 'Allow',
               'Resource': 'S3'},
 'Version': '2012-10-17'}
You can write a Python dictionary as a JSON file by using the json.dump method. This is how you would update the policy file you just modified:

In [17]: with open('service-policy.json', 'w') as opened_file:
    ...:     policy = json.dump(policy, opened_file)
    ...:
    ...:
    ...:
Another language commonly used in configuration files is YAML (“YAML Ain’t Markup Language”). It is a superset of JSON, but has a more compact format, using whitespace similar to how Python uses it.

Ansible is a tool used to automate software configuration, management, and deployment. Ansible uses files, referred to as playbooks, to define actions you want to automate. These playbooks use the YAML format:

---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum:
      name: httpd
      state: latest
  ...
The most commonly used library for parsing YAML files in Python is PyYAML. It is not in the Python Standard Library, but you can install it using pip:

$ pip install PyYAML
Once installed, you can use PyYAML to import and export YAML data much as you did with JSON:

In [18]: import yaml

In [19]: with open('verify-apache.yml', 'r') as opened_file:
    ...:     verify_apache = yaml.safe_load(opened_file)
    ...:
The data loads as familiar Python data structures (a list containing a dict):

In [20]: pprint(verify_apache)
[{'handlers': [{'name': 'restart apache',
                'service': {'name': 'httpd', 'state': 'restarted'}}],
  'hosts': 'webservers',
  'remote_user': 'root',
  'tasks': [{'name': 'ensure apache is at the latest version',
             'yum': {'name': 'httpd', 'state': 'latest'}},
            {'name': 'write the apache config file',
             'notify': ['restart apache'],
             'template': {'dest': '/etc/httpd.conf', 'src': '/srv/httpd.j2'}},
            {'name': 'ensure apache is running',
             'service': {'name': 'httpd', 'state': 'started'}}],
  'vars': {'http_port': 80, 'max_clients': 200}}]
You can also save Python data to a file in YAML format:

In [22]: with open('verify-apache.yml', 'w') as opened_file:
    ...:     yaml.dump(verify_apache, opened_file)
    ...:
    ...:
    ...:
Another language widely used for representing structured data is Extensible Markup Language (XML). It consists of hierarchical documents of tagged elements. Historically, many web systems used XML to transport data. One such use is for Real Simple Syndication (RSS) feeds. RSS feeds are used to track and notify users of updates to websites and have been used to track the publication of articles from various sources. RSS feeds use XML-formatted pages. Python offers the xml library for dealing with XML documents. It maps the XML documents’ hierarchical structure to a tree-like data structure. The nodes of the tree are elements, and a parent-child relationship is used to model the hierarchy. The top parent node is referred to as the root element. To parse an RSS XML document and get its root:

In [1]: import xml.etree.ElementTree as ET
In [2]: tree = ET.parse('http_feeds.feedburner.com_oreilly_radar_atom.xml')

In [3]: root = tree.getroot()

In [4]: root
Out[4]: <Element '{http://www.w3.org/2005/Atom}feed' at 0x11292c958>
You can walk down the tree by iterating over the child nodes:

In [5]: for child in root:
   ...:     print(child.tag, child.attrib)
   ...:
{http://www.w3.org/2005/Atom}title {}
{http://www.w3.org/2005/Atom}id {}
{http://www.w3.org/2005/Atom}updated {}
{http://www.w3.org/2005/Atom}subtitle {}
{http://www.w3.org/2005/Atom}link {'href': 'https://www.oreilly.com'}
{http://www.w3.org/2005/Atom}link {'rel': 'hub',
                                   'href': 'http://pubsubhubbub.appspot.com/'}
{http://www.w3.org/2003/01/geo/wgs84_pos#}long {}
{http://rssnamespace.org/feedburner/ext/1.0}emailServiceId {}
...
XML allows for namespacing (using tags to group data). XML prepends tags with namespaces enclosed in brackets. If you know the structure of the hierarchy, you can search for elements by using their paths. You can supply a dictionary that defines namespaces as a convenience:

In [108]: ns = {'default':'http://www.w3.org/2005/Atom'}
In [106]: authors = root.findall("default:entry/default:author/default:name", ns)

In [107]: for author in authors:
     ...:     print(author.text)
     ...:
Nat Torkington
VM Brasseur
Adam Jacob
Roger Magoulas
Pete Skomoroch
Adrian Cockcroft
Ben Lorica
Nat Torkington
Alison McCauley
Tiffani Bell
Arun Gupta
You may find yourself dealing with data stored as comma-separated values (CSV). This format is common for spreadsheet data. You can use the Python csv module to read these easily:

In [16]: import csv
In [17]: file_path = '/Users/kbehrman/Downloads/registered_user_count_ytd.csv'

In [18]: with open(file_path, newline='') as csv_file:
    ...:     off_reader = csv.reader(csv_file, delimiter=',')
    ...:     for _ in range(5):
    ...:         print(next(off_reader))
    ...:
['Date', 'PreviousUserCount', 'UserCountTotal', 'UserCountDay']
['2014-01-02', '61', '5336', '5275']
['2014-01-03', '42', '5378', '5336']
['2014-01-04', '26', '5404', '5378']
['2014-01-05', '65', '5469', '5404']
The csv reader object iterates through the .csv file one line at a time, allowing you to process the data one row at a time. Processing a file this way is especially useful for large .csv files that you do not want to read into memory all at once. Of course, if you need to do multiple row calculations across columns and the file is not overly large, you should load it all at once.

The Pandas package is a mainstay in the data science world. It includes a data structure, the pandas.DataFrame, which acts like a data table, similar to a very powerful spreadsheet. If you have table-like data on which you want to do statistical analysis or that you want to manipulate by rows and columns, DataFrames is the tool for you. It is a third-party library, so you need to install it with pip. You can use a variety of methods to load data into the DataFrames; one of the most common is from a .csv file:

In [54]: import pandas as pd

In [55]: df = pd.read_csv('sample-data.csv')

In [56]: type(df)
Out[56]: pandas.core.frame.DataFrame
You can take a look at the top rows of your DataFrame using the head method:

In [57]: df.head(3)
Out[57]:
   Attributes     open       high        low      close     volume
0     Symbols        F          F          F          F          F
1        date      NaN        NaN        NaN        NaN        NaN
2  2018-01-02  11.3007    11.4271    11.2827    11.4271   20773320
You can get a statistical insight using the describe method:

In [58]: df.describe()
Out[58]:
        Attributes    open      high    low     close     volume
count          357     356       356    356       356        356
unique         357     290       288    297       288        356
top     2018-10-18  10.402    8.3363   10.2    9.8111   36298597
freq             1       5         4      3         4          1
Alternatively, you can view a single column of data by using its name in square brackets:

In [59]: df['close']
Out[59]:
0            F
1          NaN
2      11.4271
3      11.5174
4      11.7159
        ...
352       9.83
353       9.78
354       9.71
355       9.74
356       9.52
Name: close, Length: 357, dtype: object
Pandas has many more methods for analyzing and manipulating table-like data, and there are many books on its use. It is a tool you should be aware of if you have the need to do data analysis.

Using Regular Expressions to Search Text
The Apache HTTP server is an open source web server widely used to serve web content. The web server can be configured to save log files in different formats. One widely used format is the Common Log Format (CLF). A variety of log analysis tools can understand this format. Below is the layout of this format:

<IP Address> <Client Id> <User Id> <Time> <Request> <Status> <Size>
What follows is an example line from a log in this format:

127.0.0.1 - swills [13/Nov/2019:14:43:30 -0800] "GET /assets/234 HTTP/1.0" 200 2326
Chapter 1 introduced you to regular expressions and the Python re module, so let’s use it to pull information from a log in the common log format. One trick to constructing regular expressions is to do it in sections. Doing so enables you to get each subexpression working without the complication of debugging the whole expression. You can create a regular expression using named groups to pull out the IP address from a line:

In[1]: line = '127.0.0.1 - rj [13/Nov/2019:14:34:30 -0000] "GET HTTP/1.0" 200'

In [2]: re.search(r'(?P<IP>\d+\.\d+\.\d+\.\d+)', line)
Out[2]: <re.Match object; span=(0, 9), match='127.0.0.1'>

In [3]: m = re.search(r'(?P<IP>\d+\.\d+\.\d+\.\d+)', line)

In [4]: m.group('IP')
Out[4]: '127.0.0.1'
You can also create a regular expression to get the time:

In [5]: r = r'\[(?P<Time>\d\d/\w{3}/\d{4}:\d{2}:\d{2}:\d{2})\]'

In [6]: m = re.search(r, line)

In [7]: m.group('Time')
Out[7]: '13/Nov/2019:14:43:30'
You can grab multiple elements, as has been done here: the IP, user, time, and request:

In [8]:  r = r'(?P<IP>\d+\.\d+\.\d+\.\d+)'

In [9]: r += r' - (?P<User>\w+) '

In [10]: r += r'\[(?P<Time>\d\d/\w{3}/\d{4}:\d{2}:\d{2}:\d{2})\]'

In [11]: r += r' (?P<Request>".+")'

In [12]:  m = re.search(r, line)

In [13]: m.group('IP')
Out[13]: '127.0.0.1'

In [14]: m.group('User')
Out[14]: 'rj'

In [15]: m.group('Time')
Out[15]: '13/Nov/2019:14:43:30'

In [16]: m.group('Request')
Out[16]: '"GET HTTP/1.0"'
Parsing a single line of a log is interesting but not terribly useful. However, you can use this regular expression as a basis for designing one to pull information from the whole log. Let’s say you want to pull all of the IP addresses for GET requests that happened on November 8, 2019. Using the preceding expression, you make modifications based on the specifics of your request:

In [62]: r = r'(?P<IP>\d+\.\d+\.\d+\.\d+)'
In [63]: r += r'- (?P<User>\w+)'
In [64]: r += r'\[(?P<Time>08/Nov/\d{4}:\d{2}:\d{2}:\d{2} [-+]\d{4})\]'
In [65]: r += r' (?P<Request>"GET .+")'
Use the finditer method to process the log, printing the IP addresses of the matching lines:

In [66]: matched = re.finditer(r, access_log)

In [67]: for m in matched:
    ...:     print(m.group('IP'))
    ...:
127.0.0.1
342.3.2.33
There is a lot that you can do with regular expressions and texts of all sorts. If they do not daunt you, you will find them one of the most powerful tools in dealing with text.

Dealing with Large Files
There are times that you need to process very large files. If the files contain data that can be processed one line at a time, the task is easy with Python. Rather than loading the whole file into memory as you have done up until now, you can read one line at a time, process the line, and then move to the next. The lines are removed from memory automatically by Python’s garbage collector, freeing up memory.

NOTE
Python automatically allocates and frees memory. Garbage collection is one means of doing this. The Python garbage collector can be controlled using the gc package, though this is rarely needed.

The fact that operating systems use alternate line endings can be a hassle when reading a file created on a different OS. Windows-created files have \r characters in addition to \n. These show up as part of the text on a Linux-based system. If you have a large file and you want to correct the line endings to fit your current OS, you can open the file, read one line at a time, and save it to a new file. Python handles the line-ending translation for you:

In [23]: with open('big-data.txt', 'r') as source_file:
    ...:     with open('big-data-corrected.txt', 'w') as target_file:
    ...:         for line in source_file:
    ...:             target_file.write(line)
    ...:
Notice that you can nest the with statements to open two files at once and loop through the source file object one line at a time. You can define a generator function to handle this, especially if you need to parse multiple files a single line at a time:

In [46]: def line_reader(file_path):
    ...:     with open(file_path, 'r') as source_file:
    ...:         for line in source_file:
    ...:             yield line
    ...:

In [47]: reader = line_reader('big-data.txt')

In [48]: with open('big-data-corrected.txt', 'w') as target_file:
    ...:     for line in reader:
    ...:         target_file.write(line)
    ...:
If you do not or cannot use line endings as a means of breaking up your data, as in the case of a large binary file, you can read your data in chunks. You pass the number of bytes read in each chunk to the file objects read method. When there is nothing left to read, the expression returns an empty string:

In [27]: with open('bb141548a754113e.jpg', 'rb') as source_file:
    ...:     while True:
    ...:         chunk = source_file.read(1024)
    ...:         if chunk:
    ...:             process_data(chunk)
    ...:         else:
    ...:             break
    ...:
Encrypting Text
There are many times you need to encrypt text to ensure security. In addition to Python’s built-in package hashlib, there is a widely used third-party package called cryptography. Let’s take a look at both.

Hashing with Hashlib
To be secure, user passwords must be stored encrypted. A common way to handle this is to use a one-way function to encrypt the password into a bit string, which is very hard to reverse engineer. Functions that do this are called hash functions. In addition to obscuring passwords, hash functions ensure that documents sent over the web are unchanged during transmission. You run the hash function on the document and send the result along with the document. The recipient can then confirm that the value is the same when they hash the document. The hashlib includes secure algorithms for doing this, including SHA1, SHA224, SHA384, SHA512, and RSA’s MD5. This is how you would hash a password using the MD5 algorithm:

In [62]: import hashlib

In [63]: secret = "This is the password or document text"

In [64]: bsecret = secret.encode()

In [65]: m = hashlib.md5()

In [66]: m.update(bsecret)

In [67]: m.digest()
Out[67]: b' \xf5\x06\xe6\xfc\x1c\xbe\x86\xddj\x96C\x10\x0f5E'
Notice that if your password or document is a string, you need to turn it into a binary string by using the encode method.

Encryption with Cryptography
The cryptography library is a popular choice for handling encryption problems in Python. It is a third-party package, so you must install it with pip. Symmetric key encryption is a group of encryption algorithms based on shared keys. These algorithms include Advanced Encryption Algorithm (AES), Blowfish, Data Encryption Standard (DES), Serpent, and Twofish. A shared key is similar to a password that is used to both encrypt and decrypt text. The fact that both the creator and the reader of an encrypted file need to share the key is a drawback when compared to asymmetric key encryption, which we will touch on later. However, symmetric key encryption is faster and more straightforward, and so is appropriate for encrypting large files. Fernet is an implementation of the popular AES algorithm. You first need to generate a key:

In [1]: from cryptography.fernet import Fernet

In [2]: key = Fernet.generate_key()

In [3]: key
Out[3]: b'q-fEOs2JIRINDR8toMG7zhQvVhvf5BRPx3mj5Atk5B8='
You need to store this key securely, as you need it to decrypt. Keep in mind that anyone who has access to it is also able to decrypt your files. If you choose to save the key to a file, use the binary data type. The next step is to encrypt the data using the Fernet object:

In [4]: f = Fernet(key)

In [5]: message = b"Secrets go here"


In [6]: encrypted = f.encrypt(message)

In [7]: encrypted
Out[7]: b'gAAAAABdPyg4 ... plhkpVkC8ezOHaOLIA=='
You can decrypt the data using a Fernet object created with the same key:

In [1]: f = Fernet(key)

In [2]: f.decrypt(encrypted)
Out[2]: b'Secrets go here'
Asymmetric key encryption uses a pair of keys, one public and one private. The public key is designed to be widely shared, while a single user holds the private one. The only way you can decrypt messages that have been encrypted using your public key is by using your private key. This style of encryption is widely used to pass information confidentially both on local networks and across the internet. One very popular asymmetric key algorithm is Rivest-Shamir-Adleman (RSA), which is widely used for communication across networks. The cryptography library offers the ability to create public/private key pairs:

In [1]: from cryptography.hazmat.backends import default_backend

In [2]: from cryptography.hazmat.primitives.asymmetric import rsa

In [3]: private_key = rsa.generate_private_key(public_exponent=65537,
                                               key_size=4096,
                                               backend=default_backend())

In [4]: private_key
Out[4]: <cryptography.hazmat.backends.openssl.rsa._RSAPrivateKey at 0x10d377c18>

In [5]: public_key = private_key.public_key

In [6]: public_key = private_key.public_key()

In [7]: public_key
Out[7]: <cryptography.hazmat.backends.openssl.rsa._RSAPublicKey at 0x10da642b0>
You can then use the public key to encrypt:

In [8]: message = b"More secrets go here"

In [9]: from cryptography.hazmat.primitives.asymmetric import padding
In [11]: from cryptography.hazmat.primitives import hashes

In [12]: encrypted = public_key.encrypt(message,
    ...:    padding.OAEP(mgf=padding.MGF1(algorithm=hashes.SHA256()),
    ...:    algorithm=hashes.SHA256(),
    ...:    label=None))
You can use the private key to decrypt messages:

In [13]: decrypted = private_key.decrypt(encrypted,
    ...:    padding.OAEP(mgf=padding.MGF1(algorithm=hashes.SHA256()),
    ...:    algorithm=hashes.SHA256(),
    ...:    label=None))

In [14]: decrypted
Out[14]: b'More secrets go here'
The os Module
The os module is one of the most used modules in Python. This module handles many low-level operating system calls and attempts to offer a consistent interface across multiple operating systems, which is important if you think your application might run on both Windows and Unix-based systems. It does offer some operating-specific features (os.O_TEXT for Windows and os.O_CLOEXEC on Linux) that are not available across platforms. Use these only if you are confident that your application does not need to be portable across operating systems. Example 2-1 shows some of the most useful additional methods of the os module.

Example 2-1. More os methods
In [1]: os.listdir('.') 
Out[1]: ['__init__.py', 'os_path_example.py']

In [2]: os.rename('_crud_handler', 'crud_handler') 

In [3]: os.chmod('my_script.py', 0o777) 

In [4]: os.mkdir('/tmp/holding') 

In [5]: os.makedirs('/Users/kbehrman/tmp/scripts/devops') 

In [6]: os.remove('my_script.py') 

In [7]: os.rmdir('/tmp/holding') 

In [8]: os.removedirs('/Users/kbehrman/tmp/scripts/devops') 

In [9]: os.stat('crud_handler') 
Out[9]: os.stat_result(st_mode=16877,
                       st_ino=4359290300,
                       st_dev=16777220,
                       st_nlink=18,
                       st_uid=501,
                       st_gid=20,
                       st_size=576,
                       st_atime=1544115987,
                       st_mtime=1541955837,
                       st_ctime=1567266289)

List the contents of a directory.


Rename a file or directory.


Change the permission settings of a file or directory.


Create a directory.


Recursively create a directory path.


Delete a file.


Delete a single directory.


Delete a tree of directories, starting with the leaf directory and working up the tree. The operation stops with the first nonempty directory.


Get stats about the file or directory. These stats include st_mode, the file type and permissions, and st_atime, the time the item was last accessed.

Managing Files and Directories Using os.path
In Python, you can use strings (binary or otherwise) to represent paths. The os.path module offers a plethora of path-related methods for creating and manipulating paths as strings. As previously mentioned, the os module tries to offer cross-platform behaviors, and the os.path submodule is no exception. This module interprets paths based on the current operating system, using forward slashes to separate directories in Unix-like systems and backward slashes in Windows. Your program can construct paths on the fly that work on the current system, whichever it is. The ability to easily split and join paths is probably the most used functionality of os.path. The three methods used to split paths are split, basename, and dirname:

In [1]: import os

In [2]: cur_dir = os.getcwd() 

In [3]: cur_dir
Out[3]: '/Users/kbehrman/Google-Drive/projects/python-devops/samples/chapter4'

In [4]: os.path.split(cur_dir) 
Out[4]: ('/Users/kbehrman/Google-Drive/projects/python-devops/samples',
         'chapter4')

In [5]: os.path.dirname(cur_dir) 
Out[5]: '/Users/kbehrman/Google-Drive/projects/python-devops/samples'

In [6]: os.path.basename(cur_dir) 
Out[6]: 'chapter4'

Get the current working directory.


os.path.split splits the leaf level of the path from the parent path.


os.path.dirname returns the parent path.


os.path.basename returns the leaf name.

You can easily use os.path.dirname to walk up a directory tree:

In [7]: while os.path.basename(cur_dir):
   ...:     cur_dir = os.path.dirname(cur_dir)
   ...:     print(cur_dir)
   ...:
/Users/kbehrman/projects/python-devops/samples
/Users/kbehrman/projects/python-devops
/Users/kbehrman/projects
/Users/kbehrman
/Users
/
Using files to configure an application at runtime is a common practice; files in Unix-like systems are named by convention as dotfiles ending with rc. Vim’s .vimrc file and the Bash shell’s .bashrc are two common examples. You can store these files in different locations. Often programs will define a hierarchy of locations to check. For example, your tool might look first for an environment variable that defines which rc file to use, and in its absence, check the working directory, and then the user’s home directory. In Example 2-2 we try to locate an rc file in these locations. We use the file variable that Python automatically sets when Python code runs from a file. This variable is populated with a path relative to the current working directory, not an absolute or full path. Python does not automatically expand paths, as is common in Unix-like systems, so we must expand this path before we use it to construct the path to check our rc file. Similarly, Python does not automatically expand environment variables in paths, so we must expand these explicitly.

Example 2-2. find_rc method
def find_rc(rc_name=".examplerc"):

    # Check for Env variable
    var_name = "EXAMPLERC_DIR"
    if var_name in os.environ: 
        var_path = os.path.join(f"${var_name}", rc_name) 
        config_path = os.path.expandvars(var_path) 
        print(f"Checking {config_path}")
        if os.path.exists(config_path): 
            return config_path

    # Check the current working directory
    config_path = os.path.join(os.getcwd(), rc_name)  
    print(f"Checking {config_path}")
    if os.path.exists(config_path):
        return config_path

    # Check user home directory
    home_dir = os.path.expanduser("~/")  
    config_path = os.path.join(home_dir, rc_name)
    print(f"Checking {config_path}")
    if os.path.exists(config_path):
        return config_path

    # Check Directory of This File
    file_path = os.path.abspath(__file__) 
    parent_path = os.path.dirname(file_path) 
    config_path = os.path.join(parent_path, rc_name)
    print(f"Checking {config_path}")
    if os.path.exists(config_path):
        return config_path

    print(f"File {rc_name} has not been found")

Check whether the environment variable exists in the current environment.


Use join to construct a path with the environment variable name. This will look something like $EXAMPLERC_DIR/.examplerc.


Expand the environment variable to insert its value into the path.


Check to see if the file exists.


Construct a path using the current working directory.


Use the expanduser function to get the path to the user’s home directory.


Expand the relative path stored in file to an absolute path.


Use dirname to get the path to the directory holding the current file.

The path submodule also offers ways to interrogate stats about a path. You can determine if a path is a file, a directory, a link, or a mount. You can get stats such as it’s size or time of last access or modification. In Example 2-3 we use path to walk down a directory tree and report on the size and last access time of all files therein.

Example 2-3. os_path_walk.py
#!/usr/bin/env python

import fire
import os

def walk_path(parent_path):
    print(f"Checking: {parent_path}")
    childs = os.listdir(parent_path) 

    for child in childs:
        child_path = os.path.join(parent_path, child) 
        if os.path.isfile(child_path): 
            last_access = os.path.getatime(child_path) 
            size = os.path.getsize(child_path) 
            print(f"File: {child_path}")
            print(f"\tlast accessed: {last_access}")
            print(f"\tsize: {size}")
        elif os.path.isdir(child_path): 
            walk_path(child_path) 

if __name__ == '__main__':
    fire.Fire()

os.listdir returns the contents of a directory.


Construct the full path of an item in the parent directory.


Check to see if the path represents a file.


Get the last time the file was accessed.


Get the size of the file.


Check if the path represents a directory.


Check the tree from this directory down.

You could use a script like this to identify large files or files that have not been accessed and then report, move, or delete them.

Walking Directory Trees Using os.walk
The os module offers a convenience function for walking directory trees called os.walk. This function returns a generator that in turn returns a tuple for each iteration. The tuple consists of the current path, a list of directories, and a list of files. In Example 2-4 we rewrite our walk_path function from Example 2-3 to use os.walk. As you can see in this example, with os.walk you don’t need to test which paths are files or recall the function with every subdirectory.

Example 2-4. Rewrite walk_path
def walk_path(parent_path):
    for parent_path, directories, files in os.walk(parent_path):
        print(f"Checking: {parent_path}")
        for file_name in files:
            file_path = os.path.join(parent_path, file_name)
            last_access = os.path.getatime(file_path)
            size = os.path.getsize(file_path)
            print(f"File: {file_path}")
            print(f"\tlast accessed: {last_access}")
            print(f"\tsize: {size}")
Paths as Objects with Pathlib
The pathlib library represents paths as objects rather than strings. In Example 2-5 we rewrite Example 2-2 using pathlib rather than os.path.

Example 2-5. rewrite find_rc
def find_rc(rc_name=".examplerc"):

    # Check for Env variable
    var_name = "EXAMPLERC_DIR"
    example_dir = os.environ.get(var_name) 
    if example_dir:
        dir_path = pathlib.Path(example_dir) 
        config_path = dir_path / rc_name 
        print(f"Checking {config_path}")
        if config_path.exists(): 
            return config_path.as_postix() 

    # Check the current working directory
    config_path = pathlib.Path.cwd() / rc_name 
    print(f"Checking {config_path}")
    if config_path.exists():
        return config_path.as_postix()

    # Check user home directory
    config_path = pathlib.Path.home() / rc_name 
    print(f"Checking {config_path}")
    if config_path.exists():
        return config_path.as_postix()

    # Check Directory of This File
    file_path = pathlib.Path(__file__).resolve() 
    parent_path = file_path.parent 
    config_path = parent_path / rc_name
    print(f"Checking {config_path}")
    if config_path.exists():
        return config_path.as_postix()

    print(f"File {rc_name} has not been found")

As of this writing, pathlib does not expand environment variables. Instead you grab the value of the variable from os.environ.


This creates a pathlib.Path object appropriate for the currently running operating system.


You can construct new pathlib.Path objects by following a parent path with forward slashes and strings.


The pathlib.Path object itself has an exists method.


Call as_postix to return the path as a string. Depending on your use case, you can return the pathlib.Path object itself.


The class method pathlib.Path.cwd returns a pathlib.Path object for the current working directory. This object is used immediately here to create the config_path by joining it with the string rc_name.


The class method pathlib.Path.home returns a pathlib.Path object for the current user’s home directory.


Create a pathlib.Path object using the relative path stored in file and then call its resolve method to get the absolute path.


This returns a parent pathlib.Path object directly from the object itself.