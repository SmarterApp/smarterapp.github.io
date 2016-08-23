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
| Repository Location | [https://github.com/SmarterApp/TDS_ItemScoring/tree/master/sympy-scripts](https://github.com/SmarterApp/TDS_ItemScoring/tree/master/sympy-scripts){:target="_blank"} |
| Additional Documentation | [student-progman-config.txt](https://github.com/SmarterApp/TDS_Student/blob/master/Documents/Installation/student-progman-config.txt){:target="_blank"} |

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
* Download the `sympy` module from source control to the server:
  * `sudo wget https://github.com/SmarterApp/TDS_ItemScoring/blob/master/sympy-scripts/site-packages.zip?raw=true -O ~/air-sympy/site-packages.zip`
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
* Get the latest `EqScoringWebService.py` from the `TDS_ItemScoring`repository
  * `sudo wget https://raw.githubusercontent.com/SmarterApp/TDS_ItemScoring/master/sympy-scripts/EqScoringWebService.py -O /opt/sbtds/eqsvc/EqScoringWebService.py`

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