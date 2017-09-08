.. This work is licensed under a Creative Commons Attribution 4.0 International License.
.. SPDX-License-Identifier: CC-BY-4.0

***********************
Testing developer guide
***********************

.. toctree::
   :numbered:
   :maxdepth: 2


============
Introduction
============

The OPNFV testing ecosystem is wide.

The goal of this guide consists in providing some guidelines for new developers
involved in test areas.

For the description of the ecosystem, see `[1]`_.

=================
Developer journey
=================

There are several ways to join test projects as a developer. In fact you may:

 * Develop new test cases
 * Develop frameworks
 * Develop tooling (reporting, dashboards, graphs, middleware,...)

These different tasks may be done within a specific project or as a shared
resource accross the different projects.

If you develop new test cases, the best practice is to contribute upstream as
much as possible. You may contact the testing group to know which project - in
OPNFV or upstream - would be the best place to host the test cases. Such
contributions are usually directly connected to a specific project, more details
can be found in the user guides of the testing projects.

Each OPNFV testing project provides test cases and the framework to manage them.
As a developer, you can obviously contribute to them. The developer guide of
the testing projects shall indicate the procedure to follow.

Tooling may be specific to a project or generic to all the projects. For
specific tooling, please report to the test project user guide. The tooling used
by several test projects will be detailed in this document.

Be involved in the testing group
================================

The testing group is a self organized working group. The OPNFV projects dealing
with testing are invited to participate in order to elaborate and consolidate a
consistant test strategy (test case definition, scope of projects, resources for
long duration, documentation, ...) and align tooling or best practices.

A weekly meeting is organized, the agenda may be amended by any participant.
2 slots have been defined (US and APAC). Agendas and minutes are public.See
`[8]`_ for details.
The testing group IRC channel is #opnfv-testperf

Best practices
==============

All the test projects do not have the same maturity and/or number of
contributors. The nature of the test projects may be also different. The
following best practices may not be acurate for all the projects and are only
indicative. Contact the testing group for further details.


Repository structure
---------------------

Most of the projects have a similar structure, which can be defined as follows::

  `-- home
    |-- requirements.txt
    |-- setup.py
    |-- tox.ini
    |
    |-- <project>
    |       |-- <api>
    |       |-- <framework>
    |       `-- <test cases>
    |
    |-- docker
    |     |-- Dockerfile
    |     `-- Dockerfile.aarch64.patch
    |-- <unit tests>
    `- docs
       |-- release
       |   |-- release-notes
       |   `-- results
       `-- testing
           |-- developer
           |     `-- devguide
           |-- user
                 `-- userguide


API
---
Test projects are installing tools and triggering tests. When it is possible it
is recommended to implement an API in order to perform the different actions.

Each test project should be able to expose and consume APIs from other test
projects. This pseudo micro service approach should allow a flexible use of
the different projects and reduce the risk of overlapping. In fact if project A
provides an API to deploy a traffic generator, it is better to reuse it rather
than implementing a new way to deploy it. This approach has not been implemented
yet but the prerequisite consiting in exposing and API has already been done by
several test projects.


CLI
---
Most of the test projects provide a docker as deliverable. Once connected, it is
possible to prepare the environement and run tests through a CLI.


Dockerization
-------------
Dockerization has been introduced in Brahmaputra and adopted by the test
projects. Docker containers are pulled on the jumphost of OPNFV POD.




Unit tests
----------


Traffic generators
------------------


======================================
Testing group configuration parameters
======================================


Testing categories
==================

The testing group defined several categories also known as tiers. These
categories can be used to group test suites.

+----------------+-------------------------------------------------------------+
| Category       | Description                                                 |
+================+=============================================================+
| Healthcheck    | Simple and quick healthcheck tests case                     |
+----------------+-------------------------------------------------------------+
| Smoke          | Set of smoke test cases/suites to validate the release      |
+----------------+-------------------------------------------------------------+
| Features       | Test cases that validate a specific feature on top of OPNFV.|
|                | Those come from Feature projects and need a bit of support  |
|                | for integration                                             |
+----------------+-------------------------------------------------------------+
| Components     | Tests on a specific component (e.g. OpenStack, OVS, DPDK,..)|
|                | It may extend smoke tests                                   |
+----------------+-------------------------------------------------------------+
| Performance    | Performance qualification                                   |
+----------------+-------------------------------------------------------------+
| VNF            | Test cases related to deploy an open source VNF including   |
|                | an orchestrator                                             |
+----------------+-------------------------------------------------------------+
| Stress         | Stress and robustness tests                                 |
+----------------+-------------------------------------------------------------+
| In Service     | In service testing                                          |
+----------------+-------------------------------------------------------------+

Testing domains
===============

The domains deal with the technical scope of the tests. It shall correspond to
domains defined for the certification program:

 * compute
 * network
 * storage
 * hypervisor
 * container
 * vim
 * mano
 * vnf
 * ...

Testing coverage
=================
One of the goals of the testing working group is to identify the poorly covered
areas and avoid testing overlap.
Ideally based on the declaration of the test cases, through the tags, domains
and tier fields, it shall be possible to create heuristic maps.


=================
TestAPI framework
=================

The OPNFV testing group created a test collection database to collect
the test results from CI:


 http://testresults.opnfv.org/test/swagger/spec.html

Any test project running on any lab integrated in CI can push the
results to this database.
This database can be used to see the evolution of the tests and compare
the results versus the installers, the scenarios or the labs.
It is used to produce a dashboard with the current test status of the project.


Overall Architecture
====================
The Test result management can be summarized as follows::

  +-------------+    +-------------+    +-------------+
  |             |    |             |    |             |
  |   Test      |    |   Test      |    |   Test      |
  | Project #1  |    | Project #2  |    | Project #N  |
  |             |    |             |    |             |
  +-------------+    +-------------+    +-------------+
           |               |               |
           V               V               V
       +-----------------------------------------+
       |                                         |
       |         Test Rest API front end         |
       |                                         |
       +-----------------------------------------+
           |                |
           |                V
           |     +-------------------------+
           |     |                         |
           |     |    Test Results DB      |
           |     |         Mongo DB        |
           |     |                         |
           |     +-------------------------+
           |
           |
     +----------------------+
     |                      |
     |    test Dashboard    |
     |                      |
     +----------------------+

TestAPI description
===================
The TestAPI is used to declare pods, projects, test cases and test
results. Pods are the sets of bare metal or virtual servers and networking
equipments used to run the tests.

The results pushed in the database are related to pods, projects and test cases.
If you try to push results of test done on non referenced pod, the API will
return an error message.

An additional method dashboard has been added to post-process
the raw results in release Brahmaputra (deprecated in Colorado).

The data model is very basic, 5 objects are created:

  * Pods
  * Projects
  * Testcases
  * Results
  * Scenarios

The code of the API is hosted in the releng repository `[6]`_.
The static documentation of the API can be found at `[7]`_.
The TestAPI has been dockerized and may be installed locally in your
lab. See `[15]`_ for details.

The deployment of the TestAPI has been automated.
A jenkins job manages:

  * the unit tests of the TestAPI
  * the creation of a new docker file
  * the deployment of the new TestAPI
  * the archive of the old TestAPI
  * the backup of the Mongo DB

TestAPI Authorization
---------------------

PUT/DELETE/POST operations of the TestAPI now require token based authorization. The token needs
to be added in the request using a header 'X-Auth-Token' for access to the database.

e.g::
    headers['X-Auth-Token']

The value of the header i.e the token can be accessed in the jenkins environment variable
*TestApiToken*. The token value is added as a masked password.

.. code-block:: python

    headers['X-Auth-Token'] = os.environ.get('TestApiToken')

The above example is in Python. Token based authentication has been added so that only ci pods
jenkins job can have access to the database.

Please note that currently token authorization is implemented but is not yet enabled.

===============================
Feedback from the testing group
================================

Test case catalog
===================

A test case catalog has been realized. Roll over the project then click to get
the list of test cases, click on the case to get more details.

.. raw:: html
   :url: http://testresults.opnfv.org/reporting2/reporting/index.html#!/select/visual

Reporting
=========

An automatic reporting page has been created in order to provide a
consistent view of the scenarios.

In this page, each scenario is evaluated according to test criteria.
The code for the automatic reporting is available at `[8]`_.

The results are collected from the centralized database every day and,
per scenario. A score is calculated based on the results from the last
10 days.

Dashboard
=========

Dashboard is used to provide a consistent view of the results collected in CI.
The results showed on the dashboard are post processed from the Database,
which only contains raw results.

It can be used in addition of the reporting page (high level view) to allow
the creation of specific graphs according to what the test owner wants to show.

In Brahmaputra, a basic home made dashboard was created in Functest.
In Colorado, Yardstick adopted Grafana (time based graphs) and ELK (complex
graphs).
Since Danube, the testing community decided to adopt ELK framework and to rely
on bitergia. It was not implemented for Danube but it is planned for Euphrates.

Bitergia already provides a dashboard for code and infrastructure.
A new Test tab will be added. The dataset will be built by consuming
the TestAPI.

See `[3]`_ for details.


=======
How TOs
=======

Where can I find information on the different test projects?
===========================================================


How can I contribute to a test project?
=======================================


Where can I find hardware resources?
====================================


How do I integrate my tests in CI?
==================================


How to declare my tests in the test Database?
=============================================


How to push your results into the Test Database?
================================================

The test database is used to collect test results. By default it is
enabled only for CI tests from Production CI pods.

Please note that it is possible to create your own local database.

A dedicated database is for instance created for each plugfest.

The architecture and associated API is described in previous chapter.
If you want to push your results from CI, you just have to call the API
at the end of your script.

You can also reuse a python function defined in functest_utils.py `[5]`_


Where can I find the documentation on the test API?
===================================================

http://artifacts.opnfv.org/releng/docs/testapi.html



I have tests, to which category should I declare them?
======================================================



The main ambiguity could be between features and VNF.
In fact sometimes you have to spawn VMs to demonstrate the capabilities of the
feature you introduced.
We recommend to declare your test in the feature category.

VNF category is really dedicated to test including:

 * creation of resources
 * deployement of an orchestrator/VNFM
 * deployment of the VNF
 * test of the VNFM
 * free resources

The goal is not to study a particular feature on the infrastructure but to have
a whole end to end test of a VNF automatically deployed in CI.
Moreover VNF are run in weekly jobs (one a week), feature tests are in daily
jobs and use to get a scenario score.

Where are the logs of CI runs?
==============================

Logs and configuration files can be pushed to artifact server from the CI under
http://artifacts.opnfv.org/<project name>


==========
References
==========

_`[1]`: http://docs.opnfv.org/en/stable-danube/testing/ecosystem/overview.html

_`[2]`: http://www.opnfv.org

_`[3]`: https://wiki.opnfv.org/display/testing/Result+alignment+for+ELK+post-processing

_`[4]`: https://wiki.opnfv.org/display/INF/CI+Scenario+Naming

_`[5]`: https://git.opnfv.org/functest/tree/functest/utils/functest_utils.py#176

_`[6]`: https://git.opnfv.org/functest/tree/releng

_`[7]`: http://artifacts.opnfv.org/releng/docs/testapi.html

_`[8]`: https://wiki.opnfv.org/display/meetings/Test+Working+Group+Weekly+Meeting


IRC support chan: #opnfv-testperf