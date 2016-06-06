---
title: Equation Scoring Service Installation Checklist
permalink: "deployment/checklist/eqscoring.html"
layout: "document"
categories: ["deployment", "checklist", "tds"]
---

# Overview

| Item | Description |
|:-----|:------------|
| Purpose | Python service for scoring some math equations |
| Communicates With | Student |
| Repository Location | [https://bitbucket.org/sbacoss/itemscoring_release/sympy-scripts](https://bitbucket.org/sbacoss/itemscoring_release/src/50919ffcdb29d6f5bb42b65433de90971a0b8c7f/sympy-scripts/?at=default) |
| Additional Documentation | [student-progman-config.txt](https://bitbucket.org/sbacoss/student_release/src/a5de012d932d58a2cf1e29c06fd8047fcbce1a00/Documents/Installation/student-progman-config.txt?at=default) |

***NOTE:***{: style="color: #f00"} *The Equation Scoring Service is typically deployed to the same application server that hosts the Student and Proctor applications.  That is, a single AWS instance will host the Student and Proctor applications in addition to the Equation Scoring Service.*{: style="color: #04384e"}

# Instructions - Deploy Student Report Processor to AWS Instance That Already Hosts Student and Proctor

## Deploy the Equation Scoring Service

### Create a Directory For the Equation Scoring Service Script
* Create a directory where the Equation Scoring Service script will be deployed:
  * Example:
    * `sudo mkdir -p `<span class="placeholder-example">/opt/sbtds/eqsvc/logs</span>
    * `sudo chown -R ubuntu:ubuntu `<span class="placeholder-example">/opt/sbtds/eqsvc</span>

### Install Python Libraries
* Verify that `python` is installed and at least version 2.7.x:
  * `python -V`
  * Example output:

~~~~
ubuntu@tds-web-deploy:~$ python -V
Python 2.7.6
~~~~

* Install `pip` and `bottle`:
  * `sudo apt-get install -y python-pip python-bottle`

#### Install Sympy
* Use `pip` to install `sympy`:
  * `sudo pip install sympy`
* Create a directory for the `sympy` library customized by AIR:
  * `mkdir ~/air-sympy`
* Copy the custom `sympy` module from source control to the server:
  * **TODO**
  * Example:
    * `scp -i `<span class="placeholder-example">~/.ssh/tds/ssh-dev.pem</span>` `<span class="placeholder-example">/Users/jjohnson/dev/ucla/sbac/sbrepo/repositories/itemscoring_release/sympy-scripts/</span>`site-packages.zip ubuntu@`<span class="placeholder-example">54.186.182.136</span>`:`<span class="placeholder-example">~/air-sympy</span>
* Extract the contents of the custom `sympy` module:
  * Example:
    * `unzip `<span class="placeholder-example">~/air-sympy</span>`/site-packages.zip -d `<span class="placeholder-example">~/air-sympy</span>
* Copy the contents of the extracted `site-packages.zip` to where the python packages are deployed:
  * `sudo cp -r `<span class="placeholder-example">~/air-sympy/sympy</span>` /usr/local/lib/python2.7/dist-packages/sympy`
  * `sudo cp `<span class="placeholder-example">~/air-sympy/</span>`sympy-0.7.1-py2.7.egg-info /usr/local/lib/python2.7/dist-packages/`

#### Install CherryPy 3.6
  * Download the source for `CherryPy 3.6.0`:
    * `sudo wget https://pypi.python.org/packages/source/C/CherryPy/CherryPy-3.6.0.tar.gz#md5=9772dbee426d656f01a13881e2b139d8 -O `<span class="placeholder-example">~</span>`/CherryPy-3.6.0.tar.gz`
  * Extract contents of `CherryPy-3.6.0.tar.gz`:
    * `sudo tar -xvzf CherryPy-3.6.0.tar.gz`
  * Install the CherryPy module:
    * `cd CherryPy-3.6.0 && sudo python setup.py install`

### Deploy the Equation Scoring Service
* Clone the `itemscoring_release` repository:
  * `hg clone https://`[*BitBucket user or team name*{: style="color: #f00;"}]`/fwsbac/itemscoring_release`
  * Example:
    * `hg clone https://`<span class="placeholder-example">jjohnson-fwtech@bitbucket.org</span>`/fwsbac/itemscoring_release`
* Copy the [*/path/to/itemscoring/repo*{: style="color: #f00;"}]`/sympy-scripts/EqScoringWebService.py` to the desired deployment directory (created earlier)
  * Example (assuming the `hg clone` was run on a machine other than the AWS instance):
    * `scp -i `<span class="placeholder-example">~/.ssh/tds/ssh-dev.pem</span>` `<span class="placeholder-example">~/dev/ucla/sbac/sbrepo/repositories</span>`/itemscoring_release/sympy-scripts/EqScoringWebService.py ubuntu@`<span class="placeholder-example">54.186.182.136</span>`:`<span class="placeholder-example">/opt/sbtds/eqsvc</span>

### Start the Equation Scoring Service
* Run the following command to start the Equation Scoring Service as a background process:
  * `nohup python `[*/path/to/**EqScoringWebService.py***{: style="color: #f00;"}]` 2>&1 > `[*/path/to/log/file*{: style="color: #f00;"}]` &`
  * Example:
    * `nohup python `<span class="placeholder-example">/opt/sbtds/eqsvc</span>`/EqScoringWebService.py 2>&1 > `<span class="placeholder-example">/opt/sbtds/eqsvc/logs/EqScoringWebService.py.log</span>` &`

## Verification
* Verify the `EqScoringWebService.py` is running:
  * `ps -ef | grep EqScoring`
* Output should appear similar to as follows:

<div class="highlighter-rouge">
<pre class="highlight">
<code>ubuntu@tds-web-deploy:/opt/sbtds/eqsvc/logs$ ps -ef | grep EqScoring
ubuntu    5571  5270  0 20:58 pts/0    00:00:00 python <span class="placeholder-example">/opt/sbtds/eqsvc</span>/EqScoringWebService.py
ubuntu    5593  5270  0 21:00 pts/0    00:00:00 grep --color=auto EqScoring</code>
</pre>
</div>


[back to Deployment Checklists](index.html)