django-docker-test
==================

Run Django tests in parallel using Docker containers to isolate apps from each other.

Why?
----

The YunoJuno test suite (not including any external apps) currently weighs in at around 1,300 tests, many of which involve a lot of `setUp` work to wrangle our internal state machine into position. They currently take 20+ minutes to run on a 2013 MBA (8GB), which makes it impractical to run any kind of CI. We need better, faster, tests.

A secondary issue is that of branch changes mid-test run. Given that the tests take so long, we often find ourselves getting bored, switching to another branch (say to fix a live bug), only to find the tests now fail because the template directory has changed mid-run. Aargh. 

What?
----

The intention is to speed up the tests by running them in parallel, logically one app per container (we have ten separate apps, not in the pure, decoupled, Django sense - they are more like namespaces). In addition, each container would have the current branch context uploaded to it, and so would be isolated from any changes in the underlying file system.

We are proposing a framework around docker that would spin up X containers, each of which would run a single app's test suite, and then aggregate the output in some useful format. Containers would stop once their test suite is complete.

How?
----

Not sure. Something based on Fig perhaps, or something we roll ourselves, using the Docker API. We have a [Docker image](https://index.docker.io/u/yunojuno/dev/) in the public index that contains the baseline required to run a Django app, and [another](https://github.com/YunoJuno/docker-django) in Github that will prep a new application environment (load requirements etc). We'll look to build on those.

The big issue is around external services, in particular the test database. We don't advocate running tests against SQLite as the results often differ from what you would get when running them against Postgres (for example), however... for this particular project we may have to start out down that path, as it reduces the complexity significantly. Having multiple contains creating their own 'test_app' databases and running `setUp`, `tearDown` functions will be complicated enough.

Features
--------

Initial cut will probably do this:

* Build single test run Docker image
* Configure multiple test suites
* Run each test suite in a dedicated Docker container, built from base image
* Aggregate test output from all containers

It probably won't to this:

* Handle external services gracefully
* Use external container database
* Work seamlessly, all the time, every time


