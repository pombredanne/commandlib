CommandLib
==========

CommandLib is a pythonic wrapper around subprocess that lets you pass around command objects
and daisy-chain:

* Arguments
* Paths
* Other environment variables
* Runtime directory
* Other runtime properties (run in shell, conceal stdout/stderr, ignore error codes, etc.)

It is somewhat inspired by amoffat's sh, Kenneth Reitz's requests, jaraco's path.py
and SQLAlchemy.


Example::

    >>> from commandlib import Command
    >>> ls = Command("ls")
    >>> ls("-t").in_dir("/").with_shell().run()
    sys  tmp  run  dev  proc  etc  boot  sbin  root  vmlinuz  initrd.img  bin  lib  opt  vmlinuz.old  initrd.img.old  media  home  cdrom  lost+found  var  srv  usr  mnt


CommandPath example::

    >>> from commandlib import CommandPath
    >>> bin = CommandPath("/bin")
    >>> bin.ls("-t").in_dir("/").run()
    sys  tmp  run  dev  proc  etc  boot  sbin  root  vmlinuz  initrd.img  bin  lib  opt  vmlinuz.old  initrd.img.old  media  home  cdrom  lost+found  var  srv  usr  mnt


Install
-------

    pip install commandlib


API
---

    >>> from commandlib import Command, run
    >>> py = Command("/usr/bin/python")
    >>> py = py.with_env(PYTHONPATH="/home/user/pythondirectory")    # Run with *additional* environment variable PYTHONPATH (*added* to global environment when command is run)
    >>> py = py.with_path("/home/user/bin")                          # Run with additional path (appended to existing PATH environment variable when command is run)
    >>> py = py.in_dir("/home/user/mydir")                           # Run in specified directory.
    >>> py = py.with_shell()                                         # Run with shell
    >>> py = py.only_errors()                                        # Suppress stderr
    >>> py = py.silently()                                           # Suppress stdout and stderr
    >>> run(py)                                                      # Run command explicitly with all of the above


Why?
----

Commandlib is a library to make it easier to pass around command objects between different
modules and classes and incrementally modify the command's behavior in a readable way
- adding environment variables, paths, etc.

* call, check_call and Popen do not have the friendliest of APIs and code that uses them a lot can get ugly fast.
* sh does a similar thing but has a lot of magic (e.g. overriding import).
* envoy and sarge are more focused on chaining commands rather than arguments, environment variables, etc.

Advanced API
------------

Add trailing arguments:

    >>> from commandlib import Command, run
    >>> manage = Command(["/usr/bin/python", "manage.py"]).with_trailing_arguments("--settings", "local_settings.py").in_dir("projectdir")
    >>> run(manage("runserver"))
    [ Runs "/usr/bin/python manage.py runserver --settings local_settings.py" inside projectdir ]

Dynamically generate command bundles from directories with executables in them:

    >>> from commandlib import CommandPath, Command, run
    >>> postgres94 = CommandPath("/usr/lib/postgresql/9.4/bin/")
    >>> run(postgres94.postgres)
    [ Runs postgres ]

    >>> run(postgres94.createdb)
    [ Runs createdb ]

Use with path.py (or any other library where str(object) resolves to a string:

    >>> from path import Path
    >>> postgres94 = Commands(Path("/usr/lib/postgresql/9.4/bin/"))
    >>> run(postgres94.postgres)


Hacking
-------

If you want to hack, you can TDD with::

    curl https://raw.githubusercontent.com/mitsuhiko/pipsi/master/get-pipsi.py | python
    pipsi install hitch
    cd tests
    hitch init
    hitch test . --settings tdd.settings
