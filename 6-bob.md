Chapter 6. Continuous Integration and Continuous Deployment
Author: Noah

The practices of continuous integration (CI) and continuous deployment (CD) are essential to a modern software development life cycle process. A CI system clones the codebase for the software under consideration from a source control system such as GitHub, builds the software into an artifact that can be a binary, a tar archive, or a Docker image, and, very importantly, also runs unit and/or integration tests for the software. A CD system deploys the artifacts built by the CI system to a target environment. This deployment can be automated for nonproduction environments, but usually includes a manual approval step for production. A more advanced type of such systems is a continuous delivery platform, which automates the deployment step to production and is capable of rolling back the deployment based on metrics obtained from monitoring and logging platforms.

Real-World Case Study: Converting a Poorly Maintained WordPress Site to Hugo
A while back, a friend asked for a favor fixing their company website. The company sold very expensive, used scientific equipment and its inventory was served via a WordPress site that was frequently hacked, performed horribly, or was down for days. Typically I try to avoid getting sucked into projects like this, but since it was a friend, I decided to help. You can reference the code for the conversion project at this Git repository.

Each step of the conversion process is covered in the GitHub repo. The steps include:

Backup

Convert

Upgrade

Deploy

NOTE
The story had a funny ending. After creating a bulletproof, “tank” of a website that had incredible performance, security, auto-deployment, and incredible SEO, it ran for years with zero vulnerabilities or downtime. Long after I had forgotten about the project, I got a text from my friend. It had been a couple years since I had last talked with him. He said the website was down and he needed my help.

I texted back to him to ask how this was possible. It is running off Amazon S3 which has 99.999999999% uptime. He texted back that he had recently converted it back to WordPress because it was “easier” to make changes. I laughed and told him I wasn’t a good fit for his project. As they say, no good deed is left unpunished.

Some of the requirements I considered were:

It needed to be continuously deployed.

It needed to be fast to run and develop against!

It should be a static site hosted from a cloud provider.

There should be a reasonable workflow for converting from WordPress.

It should be possible to create a reasonable search interface using Python.

In the end I decided to use Hugo, AWS, and Algolia. The general architecture looked like Figure 6-1.

# Rasm1

Figure 6-1. Continuous deployment with Hugo
Setting Up Hugo
Getting started with Hugo is very straightforward (see the getting started Hugo guide). First, install the software. On my OS X machine I did it this way:

brew install hugo
If you already installed Hugo, you may need to upgrade:

Error: hugo 0.40.3 is already installed
To upgrade to 0.57.2, run brew upgrade hugo.
If you are on another platform, you can follow instructions here. To verify things are working, run hugo version:

(.python-devops) ➜  ~ hugo version
Hugo Static Site Generator v0.57.2/extended darwin/amd64 BuildDate: unknown
The only thing left to do is to initialize a skeleton Hugo app and install a theme:

hugo new site quickstart
This creates a new site called quickstart. You can build this site again, VERY QUICKLY, by running hugo. This compiles the markdown files to HTML and CSS.

Converting WordPress to Hugo Posts
Next, I converted the WordPress database to JSON via a raw dump. Then I wrote a Python script to convert this data into Hugo posts in markdown format. Here is that code:

"""Conversion code of old database fields into markdown example.

If you did a database dump of WordPress and then converted it to JSON, you could
tweak this."""

import os
import shutil
from category import CAT
from new_picture_products import PICTURES

def check_all_category():
  ares = {}
  REC = []
  for pic in PICTURES:
    res  = check_category(pic)
    if not res:
      pic["categories"] = "Other"
      REC.append(pic)
      continue

    title,key = res
    if key:
      print("FOUND MATCH: TITLE--[%s], CATEGORY--[%s]" %\
        (title, key))
      ares[title]= key
      pic["categories"] = key
      REC.append(pic)
  return ares, REC

def check_category(rec):

  title = str(rec['title'])
  for key, values in CAT.items():
    print("KEY: %s, VALUE: %s" % (key, values))
    if title in key:
      return title,key
    for val in values:
      if title in val:
        return title,key

def move_image(val):
  """Creates a new copy of the uploaded images to img dir"""

  source_picture = "static/uploads/%s" % val["picture"]
  destination_dir = "static/img/"
  shutil.copy(source_picture, destination_dir)

def new_image_metadata(vals):
  new_paths = []
  for val in vals:
    pic = val['picture'].split("/")[-1:].pop()
    destination_dir = "static/img/%s" % pic
    val['picture'] = destination_dir
    new_paths.append(val)
  return new_paths

CAT_LOOKUP = {'2100': 'Foo',
 'a': 'Biz',
 'b': 'Bam',
 'c': 'Bar',
 '1': 'Foobar',
 '2': 'bizbar',
 '3': 'bam'}

def write_post(val):

    tags = val["tags"]
    date = val["date"]
    title = val["title"]
    picture = val["picture"]
    categories = val["categories"]
    out = """
+++
tags = ["%s"]
categories = ["%s"]
date = "%s"
title = "%s"
banner = "%s"
+++
[![%s](%s)](%s)
 **Product Name**: %s""" %\
 (tags, categories, date, title, picture.lstrip("/"),
   title, picture, picture, title)

    filename = "../content/blog/%s.md" % title
    if os.path.exists(filename):
        print("Removing: %s" % filename)
        os.unlink(filename)

    with open(filename, 'a') as the_file:
        the_file.write(out)

if __name__ == '__main__':
    from new_pic_category import PRODUCT
    for product in PRODUCT:
        write_post(product)
Creating an Algolia Index and Updating It
With the database products converted to markdown posts, the next step is to write some Python code that creates an Algolia index and syncs it. Algolia is a great tool to use because it quickly solves the search engine problem and has nice Python support as well.

This script crawls through all of the markdown files and generates a search index that can be uploaded to Algolia:

"""
Creates a very simple JSON index for Hugo to import into Algolia. Easy to extend.

#might be useful to run this on content directory to remove spaces
for f in *\ *; do mv "$f" "${f// /_}"; done

"""
import os
import json

CONTENT_ROOT = "../content/products"
CONFIG = "../config.toml"
INDEX_PATH = "../index.json"

def get_base_url():
    for line in open(CONFIG):
        if line.startswith("baseurl"):
            url = line.split("=")[-1].strip().strip('""')
            return url

def build_url(base_url, title):

    url = "<a href='%sproducts/%s'>%s</a>" %\
         (base_url.strip(), title.lower(), title)
    return url

def clean_title(title):
    title_one = title.replace("_", " ")
    title_two = title_one.replace("-", " ")
    title_three = title_two.capitalize()
    return title_three

def build_index():
    baseurl = get_base_url()
    index =[]
    posts = os.listdir(CONTENT_ROOT)
    for line in posts:
        print("FILE NAME: %s" % line)
        record = {}
        title = line.strip(".md")
        record['url'] = build_url(baseurl, title)
        record['title'] = clean_title(title)
        print("INDEX RECORD: %s" % record)
        index.append(record)
    return index

def write_index():
    index = build_index()
    with open(INDEX_PATH, 'w') as outfile:
        json.dump(index,outfile)

if __name__ == '__main__':
    write_index()
Finally, the index can be sent to Algolia with this snippet:

import json
from algoliasearch import algoliasearch

def update_index():
    """Deletes index, then updates it"""
    print("Starting Updating Index")
    client = algoliasearch.Client("YOUR_KEY", "YOUR_VALUE")
    index = client.init_index("your_INDEX")
    print("Clearing index")
    index.clear_index()
    print("Loading index")
    batch = json.load(open('../index.json'))
    index.add_objects(batch)

if __name__ == '__main__':
    update_index()
Orchestrating with a Makefile
Using a Makefile allows you to replicate the steps your deployment process will use later. I typically set up a Makefile to orchestrate this locally. Here is what the entire build and deploy process looks like:

build:
  rm -rf public
  hugo

watch: clean
  hugo server -w

create-index:
  cd algolia;python make_algolia_index.py;cd ..

update-index:
  cd algolia;python sync_algolia_index.py;cd ..

make-index: create-index update-index

clean:
  -rm -rf public

sync:
  aws s3 --profile <yourawsprofile> sync --acl \
    "public-read" public/ s3://example.com

build-deploy-local: build sync

all: build-deploy-local
Deploying with AWS CodePipeline
Amazon Web Services (AWS) is a common deployment target for hosting a static website via Amazon S3, Amazon Route 53, and Amazon CloudFront. AWS CodePipeline, their build server service, works very well as the deployment mechanism for these sites. You can log into AWS CodePipeline, set up a new build project, and tell it to use a buildspec.yml file. The code can be customized and the portions that are templated out can be replaced with actual values.

As soon as GitHub gets a change event, CodePipeline runs the install in a container. First it grabs the specific version of Hugo specified. Next it builds the Hugo pages. Thousands of Hugo pages can be rendered subsecond because of the speed of Go.

Finally, the HTML pages are synced to Amazon S3. Because this is running inside of AWS and is synced, it is also extremely fast. The final step is that CloudFront is invalidated:

version: 0.1

environment_variables:
  plaintext:
    HUGO_VERSION: "0.42"

phases:
  install:
    commands:
      - cd /tmp
      - wget https://github.com/gohugoio/hugo/releases/\
      download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_Linux-64bit.tar.gz
      - tar -xzf hugo_${HUGO_VERSION}_Linux-64bit.tar.gz
      - mv hugo /usr/bin/hugo
      - cd -
      - rm -rf /tmp/*
  build:
    commands:
      - rm -rf public
      - hugo
  post_build:
    commands:
      - aws s3 sync public/ s3://<yourwebsite>.com/ --region us-west-2 --delete
      - aws s3 cp s3://<yourwebsite>.com/\
      s3://<yourwebsite>.com/ --metadata-directive REPLACE \
        --cache-control 'max-age=604800' --recursive
      - aws cloudfront create-invalidation --distribution-id=<YOURID> --paths '/*'
      - echo Build completed on `date`
Real-World Case Study: Deploying a Python App Engine Application with Google Cloud Build
Back in 2008 I wrote the very first article on using Google App Engine. You have to use the Wayback Machine to get it from the O’Reilly blog.

Here is a reboot for the modern era. This is another version of Google App Engine, but this time it uses Google Cloud Build. The Google Cloud Platform (GCP) Cloud Build works a lot like AWS CodePipeline. Here is a config file that is checked into a GitHub repo. The config file is named cloudbuild.yaml. You can see all of the source code for this project in this Git repository:

steps:
- name: python:3.7
  id: INSTALL
  entrypoint: python3
  args:
  - '-m'
  - 'pip'
  - 'install'
  - '-t'
  - '.'
  - '-r'
  - 'requirements.txt'
- name: python:3.7
  entrypoint: ./pylint_runner
  id: LINT
  waitFor:
  - INSTALL
- name: "gcr.io/cloud-builders/gcloud"
  args: ["app", "deploy"]
timeout: "1600s"
images: ['gcr.io/$PROJECT_ID/pylint']
Note that the cloudbuild.yaml file installs the packages seen here in the requirements.txt file and also runs gcloud app deploy, which deploys the App Engine application on check-in to GitHub:

Flask==1.0.2
gunicorn==19.9.0
pylint==2.3.1
Here is a walk-through of how to set up this entire project:

Create the project.

Activate the cloud shell.

Refer to the hello world docs for the Python 3 App Engine.

Run describe:

verify project is working
```bash
gcloud projects describe $GOOGLE_CLOUD_PROJECT
```
output of command:
```bash
createTime: '2019-05-29T21:21:10.187Z'
lifecycleState: ACTIVE
name: helloml
projectId: helloml-xxxxx
projectNumber: '881692383648'
```
You may want to verify that you have the correct project. If not, do this to switch:

gcloud config set project $GOOGLE_CLOUD_PROJECT
Create the App Engine app:

gcloud app create
This will ask for the region. Go ahead and pick us-central [12].

Creating App Engine application in project [helloml-xxx]
and region [us-central]....done.
Success! The app is now created.
Please use `gcloud app deploy` to deploy your first app.
Clone the hello world sample app repo:

git clone https://github.com/GoogleCloudPlatform/python-docs-samples
cd into the repo:

cd python-docs-samples/appengine/standard_python37/hello_world
Update the Cloudshell image (note that this is optional):

git clone https://github.com/noahgift/gcp-hello-ml.git
# Update .cloudshellcustomimagerepo.json with project and image name
# TIP: enable "Boost Mode" in in Cloudshell
cloudshell env build-local
cloudshell env push
cloudshell env update-default-image
# Restart Cloudshell VM
Create and source the virtual environment:

virtualenv --python $(which python) venv
source venv/bin/activate
Double-check that it works:

which python
/home/noah_gift/python-docs-samples/appengine/\
  standard_python37/hello_world/venv/bin/python
Activate the cloud shell editor.

Install the packages:

pip install -r requirements.txt
This should install Flask:

Flask==1.0.2
Run Flask locally. This runs Flask locally in the GCP shell:

python main.py
Use the web preview (see Figure 6-2).

# Rasm2

Figure 6-2. Web preview
Update main.py:

from flask import Flask
from flask import jsonify

app = Flask(__name__)

@app.route('/')
def hello():
    """Return a friendly HTTP greeting."""
    return 'Hello I like to make AI Apps'

@app.route('/name/<value>')
def name(value):
    val = {"value": value}
    return jsonify(val)

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=8080, debug=True)
Test out passing in parameters to exercise this function:

@app.route('/name/<value>')
def name(value):
    val = {"value": value}
    return jsonify(val)
For example, calling this route will take the word lion and pass it into the name function in Flask:

https://8080-dot-3104625-dot-devshell.appspot.com/name/lion
Returns a value in the web browser:

{
value: "lion"
}
Now deploy the app:

gcloud app deploy
Be warned! The first deploy could take about 10 minutes. You might also need to enable the cloud build API.

Do you want to continue (Y/n)?  y
Beginning deployment of service [default]...
╔════════════════════════════════════════════════════════════╗
╠═ Uploading 934 files to Google Cloud Storage              ═╣
Now stream the log files:

gcloud app logs tail -s default
The production app is deployed and should like this:

Setting traffic split for service [default]...done.
Deployed service [default] to [https://helloml-xxx.appspot.com]
You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

  $ gcloud app browse
(venv) noah_gift@cloudshell:~/python-docs-samples/appengine/\
  standard_python37/hello_world (helloml-242121)$ gcloud app
 logs tail -s default
Waiting for new log entries...
2019-05-29 22:45:02 default[2019]  [2019-05-29 22:45:02 +0000] [8]
2019-05-29 22:45:02 default[2019]  [2019-05-29 22:45:02 +0000] [8]
 (8)
2019-05-29 22:45:02 default[2019]  [2019-05-29 22:45:02 +0000] [8]
2019-05-29 22:45:02 default[2019]  [2019-05-29 22:45:02 +0000] [25]
2019-05-29 22:45:02 default[2019]  [2019-05-29 22:45:02 +0000] [27]
2019-05-29 22:45:04 default[2019]  "GET /favicon.ico HTTP/1.1" 404
2019-05-29 22:46:25 default[2019]  "GET /name/usf HTTP/1.1" 200
Add a new route and test it out:

@app.route('/html')
def html():
    """Returns some custom HTML"""
    return """
    <title>This is a Hello World World Page</title>
    <p>Hello</p>
    <p><b>World</b></p>
    """
Install Pandas and return JSON results. At this point, you may want to consider creating a Makefile and doing this:

touch Makefile
#this goes inside that file
install:
  pip install -r requirements.txt
You also may want to set up lint:

pylint --disable=R,C main.py
------------------------------------
Your code has been rated at 10.00/10
The web route syntax looks like the following block. Add Pandas import at the top:

import pandas as pd

@app.route('/pandas')
def pandas_sugar():
    df = pd.read_csv(
      "https://raw.githubusercontent.com/noahgift/sugar/\
      master/data/education_sugar_cdc_2003.csv")
    return jsonify(df.to_dict())
When you call the route https://<yourapp>.appspot.com/pandas, you should get something like Figure 6-3.

# Rasm3

Figure 6-3. Example of JSON out
Add this Wikipedia route:

import wikipedia
@app.route('/wikipedia/<company>')
def wikipedia_route(company):
    result = wikipedia.summary(company, sentences=10)
    return result
Add NLP to the app:

Run IPython Notebook.

Enable the Cloud Natural Language API.

Run pip install google-cloud-language:

In [1]: from google.cloud import language
   ...: from google.cloud.language import enums
   ...:
   ...: from google.cloud.language import types
In [2]:
In [2]: text = "LeBron James plays for the Cleveland Cavaliers."
   ...: client = language.LanguageServiceClient()
   ...: document = types.Document(
   ...:         content=text,
   ...:         type=enums.Document.Type.PLAIN_TEXT)
   ...: entities = client.analyze_entities(document).entities
In [3]: entities
Here is an end-to-end AI API example:

from flask import Flask
from flask import jsonify
import pandas as pd
import wikipedia

app = Flask(__name__)

@app.route('/')
def hello():
    """Return a friendly HTTP greeting."""
    return 'Hello I like to make AI Apps'

@app.route('/name/<value>')
def name(value):
    val = {"value": value}
    return jsonify(val)

@app.route('/html')
def html():
    """Returns some custom HTML"""
    return """
    <title>This is a Hello World World Page</title>
    <p>Hello</p>
    <p><b>World</b></p>
    """
@app.route('/pandas')
def pandas_sugar():
    df = pd.read_csv(
      "https://raw.githubusercontent.com/noahgift/sugar/\
      master/data/education_sugar_cdc_2003.csv")
    return jsonify(df.to_dict())

@app.route('/wikipedia/<company>')
def wikipedia_route(company):

    # Imports the Google Cloud client library
    from google.cloud import language
    from google.cloud.language import enums
    from google.cloud.language import types
    result = wikipedia.summary(company, sentences=10)

    client = language.LanguageServiceClient()
    document = types.Document(
        content=result,
        type=enums.Document.Type.PLAIN_TEXT)
    entities = client.analyze_entities(document).entities
    return str(entities)


if __name__ == '__main__':
    app.run(host='127.0.0.1', port=8080, debug=True)
This section has shown how to both set up an App Engine application from scratch in the Google Cloud Shell, as well as how to do continuous delivery using GCP Cloud Build.

Real-World Case Study: NFSOPS
NFOPS is an operational technique that uses NFS (Network File System) mount points to manage clusters of computers. It sounds like it is a new thing, but it has been around since Unix has been around. Noah used NFS mount points on Caltech’s Unix systems back in 2000 to manage and maintain software. What is old is new again.

As a part-time consultant at a virtual reality startup in San Francisco, one problem I faced was how to build a jobs framework, quickly, that would dispatch work to thousands of AWS Spot Instances.

The solution that ultimately worked was to use NFSOPS (Figure 6-4) to deploy Python code to thousands of Computer Vision Spot Instances subsecond.

# rasm4
Figure 6-4. NFSOPS
NFSOPS works by using a build server, in this case Jenkins, to mount several Amazon Elastic File System (EFS) mount points (DEV, STAGE, PROD). When a continuous integration build is performed, the final step is an rsync to the respective mount point:

#Jenkins deploy build step
rsync -az --delete * /dev-efs/code/
The “deploy” is then subsecond to the mount point. When spot instances are launched by the thousands, they are preconfigured to mount EFS (the NFS mount points) and use the source code. This is a handy deployment pattern that optimizes for simplicity and speed. It can also work quite well in tandom with IAC, Amazon Machine Image (AMI), or Ansible.
