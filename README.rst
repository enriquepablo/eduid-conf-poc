
Proof of concept for git governed configuration
+++++++++++++++++++++++++++++++++++++++++++++++

The aim is to have project wide configuration managed and applied by git. The
idea is that the files versioned by git only keep the names of the
configuration settings, in the places where their values should be. Git
also knows the default values for those config settings. When the project is
checked out by git, all the config setting names are replaced by their
(default, or, possibly, locally customized,) values; and when changes are
commited to the repository, the config setting values are replaced by their
names.

This is achieved by using a filter like those explained in the documentation
for `git attributes <https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes#_keyword_expansion>`_.

The possible advantages of these scheme would be two fold:

 * All configurable files can be kept under git; all environments can be
   managed by files kept in git. Therefore configuration at this level can be
   used to rule configuration at all other levels. Of course this is an
   hyperbole, but it conveys my meaning; for a counter-example, configuring the
   actual git with ths scheme would not be possible.
 * There is no danger of accidentally commiting local settings during
   development; In this sense, this idea can be seen as a very fine grained
   gitignore scheme.

Testing the POC
---------------

The first thing we need is this code::

   $ git clone https://github.com/enriquepablo/eduid-conf-poc.git /path/to/eduid-conf-poc

make a git repo::

    $ mkdir test-poc
    $ cd test-poc
    $ git init

We also need to activate a virtualenv with the diff_match_patch library::

    $ virtualenv venv
    $ source venv/bin/activate
    $ pip install diff_match_patch
    


We now copy a few files from ``eduid-conf-poc`` over to our newly created repo.
First, we copy the ``.gitattributes`` file. In it, we indicate that files that
match ``*.test`` will use the filter eduid-config::

    $ cp /path/to/eduid-conf-poc/.gitattributes .

Now we copy over ``eduidconf-gitconfig``, where the filter is defined, with
references to the scripts that will be used to check the code in and out::

    $ cp /path/to/eduid-conf-poc/eduidconf-gitconfig .

We get those scripts::

    $ cp /path/to/eduid-conf-poc/eduidconf-gitfilter-in .
    $ cp /path/to/eduid-conf-poc/eduidconf-gitfilter-out .

Finally we get the file with the default settings, used by the scripts::

    $ cp /path/to/eduid-conf-poc/eduidconf_defaults.py .

In this file there is already a single setting, to wit::

    EDUIDCONF_TEST="testing"

All settings for this POC must start with ``EDUIDCONF``.

We now need to activate the git configuration in ``eduidconf-gitconfig``. This
can be achieved with this command::

    $ git config --local include.path ../eduidconf-gitconfig

We can now start playing with this setup. Let's first create and add a file
that uses our setting::

    $ echo "This is a test with a EDUIDCONF_TEST setting" >> one.test
    $ git add one.test
    $ git commit -m 'testing settings'

We now have the file with the same contents that git keeps internally::

    $ cat one.test
    This is a test with a EDUIDCONF_TEST setting

To use this POC, we need to force the file to be checked out. Of course, if all
this were to be implemented as git commands, this would be automatic::

    $ git checkout -b test-branch
    $ echo "more text" >> one.test
    $ git add one.test
    $ git commit -m 'testing settings'

We can now check out our master branch, and see that the name of our config
setting has been replaced by its value, but what has been committed is the
name, and ``git status`` does not see it as an addable change::

    $ git checkout -
    $ cat one.test
    This is a test with a testing setting
    $ git diff HEAD~1
    diff --git a/one.test b/one.test
    new file mode 100644
    index 0000000..8f6f4ef
    --- /dev/null
    +++ b/one.test
    @@ -0,0 +1,2 @@
    +This is a test with a EDUIDCONF_TEST setting
    +
    $ git status

If we modify the file again, and we commit the changes, only the new changes
are commited, with our local copy showing the value of the setting, but the
copy held by git having the name of the setting::

    $ echo "and a bit more text" >> one.test
    $ git add one.test
    $ git commit -m 'testing settings'
    $ cat one.test
    This is a test with a testing setting
    and a bit more text

If we check the last commit, we can see that git is indeed holding the setting
name::

    $ git diff HEAD~1
    diff --git a/one.test b/one.test
    index 8f6f4ef..e96f4bf 100644
    --- a/one.test
    +++ b/one.test
    @@ -1,2 +1,3 @@
     This is a test with a EDUIDCONF_TEST setting
    +and a bit more text

To customize the settings, we have to use a ``eduidconf_custom.py`` module, with
the same contents as in ``eduidconf_defaults.py``, but customizing the values
therein::

    $ cp eduidconf_defaults.py eduidconf_custom.py
    $ vim eduidconf_custom.py
    $ git checkout -
    $ git checkout -
    $ cat one.test
    This is a test with a testong setting
    and a bit more text

Note that ``testing`` has become ``testong``.
