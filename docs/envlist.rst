=============
Env Detection
=============

Env detection is the primary feature of Tox-Travis.
Based on the matrix created in ``.travis.yml``,
it decides which Tox envs
need to be run for each Travis job.


Usage
=====

Configure the Python versions to test with in ``.travis.yml``:

.. code-block:: yaml

    language: python
    python:
      - "2.7"
      - "3.4"
    install: pip install tox-travis
    script: tox

And it will run the appropriate testenvs,
which by default are any declared env with
``py27`` or ``py34`` as factors of the name.
If no environments match a given factor,
the ``py27`` or ``py34`` envs are used as a fallback.


Advanced Configuration
======================

To customize what environments tox will run on Travis,
add a section to ``tox.ini`` telling it what environments
to run under which versions of Python:

.. code-block:: ini

    [tox]
    envlist = py{27,34}-django{17,18}, docs

    [travis]
    python =
      2.7: py27
      3.4: py34, docs

This would run the Python 2.7 variants under 2.7,
and the Python 3.4 variants and the ``docs`` env under 3.4.

Note that Travis won't run all the envs simultaneously,
because its build matrix is only aware of the Python versions.
Only one Travis build will be run per Python version,
unless other settings are specified in the Travis build matrix.

If you are using multiple Travis factors,
then you can use those factors to decide what will run.
For example, see the following ``.travis.yml`` and ``tox.ini``:

.. code-block:: yaml

    language: python
    python:
      - "2.7"
      - "3.4"
    env:
      - DJANGO="1.7"
      - DJANGO="1.8"
    matrix:
      include:
        - os: osx
          language: generic
    install: pip install tox-travis
    script: tox

.. code-block:: ini

    [tox]
    envlist = py{27,34}-django{17,18}, docs

    [travis]
    os =
      linux: py{27,34}-django{17,18}, docs
      osx: py{27,34}-django{17,18}
    python =
      3.4: py34, docs

    [travis:env]
    DJANGO =
      1.7: django17
      1.8: django18, docs

Travis will run 5 different jobs,
which will each run jobs as specified by the factors given.

* os: linux (default), language: python, python: 2.7, env: DJANGO=1.7

  This will run the env ``py27-django17``,
  because ``py27`` is the default,
  and ``django17`` is specified.

* os: linux (default), language: python, python: 3.4, env: DJANGO=1.7

  This will run the env ``py34-django17``,
  but not ``docs``,
  because ``docs`` is not included in the DJANGO 1.7 configuration.

* os: linux (default), language: python, python: 2.7, env: DJANGO=1.8

  This will run the env ``py27-django18``,
  because ``py27`` is the default.
  ``docs`` is not run,
  because Python 2.7 doesn't include ``docs``
  in the defaults that are not overridden.

* os: linux (default), language: python, python: 3.4, env: DJANGO=1.8

  This will run the envs ``py34-django18`` and ``docs``,
  because all specified factors match,
  and ``docs`` is present in all related factors.

* os: osx, language: generic

  This will run envs ``py27-django17``, ``py34-django17``,
  ``py27-django18``, and ``py34-django18``,
  because the ``os`` factor is present,
  and limits it to just those envs.


Unignore Outcomes
=================

By default, when using ``ignore_outcome`` in your Tox configuration,
any build errors will show as successful on Travis. This might not
be desired, as you might want to control allowed failures inside your
``.travis.yml``. To cater this need, you can set ``unignore_outcomes``
to ``True``. This will override ``ignore_outcome`` by setting it to
``False`` for all environments.

Configure the allowed failures in the build matrix in your ``.travis.yml``:

.. code-block:: yaml

    matrix:
      allow_failures:
      - python: 3.6
        env: DJANGO=master

And in your ``tox.ini``:

.. code-block:: ini

    [travis]
    unignore_outcomes = True

