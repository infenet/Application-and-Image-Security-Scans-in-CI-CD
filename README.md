# Application-and-Image-Security-Scans-in-CI-CD
# Base pipeline configuration with no security

Let’s start with a base pipeline configuration so that you can see a pipeline without a security
integration. I’ve been working a lot with Python applications these days, so below is an example
pipeline configuration for a Python app that doesn’t include any security actions.

This pipeline configuration accomplishes the following:

* Installs application dependencies defined in the requirements.txt manifest file

* Executes the application’s unit tests
Creates/builds a new Docker image for the application

* Publishes the newly created Docker image to the Docker Hub registry for later use

* The goal of this pipeline is to build, test, and deploy the code via a Docker image. At no point in this pipeline are there any security or vulnerability scans.

# Security enabled pipeline - the Snyk app scan

Now you’ve seen a glimpse of an “insecure” pipeline, and it should make your skin crawl. Next,
I’m going to show you an example of a security-enabled pipeline configuration.

The pipeline block below uses the​ ​ Snyk orb​ to easily integrate the Snyk tool into the pipeline.
This block is equivalent to an import or include statement in a scripting or programming
language. In this block, you’re also declaring the version of the Snyk orb you’d like to use.

```
version: 2.1
orbs:
snyk: snyk/snyk@0.0.8
```

The next pipeline block defines the Docker image used to run the build. It then does a checkout
or “git clone” of your source code into the container. Following that, the run: block will install thedependencies listed in the requirements.txt file. This file lists all of your application libraries and
dependencies which can be considered a​ ​ Software Bill of Materials (SBOM)​ specific to the
Python aspects of the project. It also feeds the list of software to Snyk, so that it knows what to
scan and test.


```
jobs:
build_test:
docker:
- image: circleci/python:3.7.4
steps:
- checkout
- run:
name: Install Python Dependencies
command: |
echo 'export PATH=~$PATH:~/.local/bin' >> $BASH_ENV && source $BASH_ENV
pip install --user -r requirements.txt
```

The next block is where we execute some DevSecOps action within the pipeline. - snyk/scan
calls the scan command from the Snyk orb. It will read the requirements.txt file, and then
compare that list of software against the Snyk vulnerability databases to look for any matches. If
there are any matches, Snyk will flag it and fail this segment of the pipeline. The goal here is to
alert teams to security issues as early as possible so that they can be quickly mitigated and the
CI/CD process can securely continue.

```
- snyk/scan
- run:
name: Run Unit Tests
command: |
pytest
```

The remainder of the example pipeline configuration deals with the Docker image build
segments.
After the Snyk application security scan is complete, your build will pass if there are no
vulnerabilities detected. If the scan detects a vulnerability, the build will fail. This is the Snyk
orb’s default behavior (see the​ ​ Snyk orb parameters​ for more details on these parameters).
Along with the pipeline failing after the scan, Snyk will provide a detailed report on why the build
failed.
