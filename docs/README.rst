Evently
=======

Essentially it does two things:
One is it allows you to write complex jQuery code in a
declarative fashion. This makes code reuse easier, by avoiding the
tangled web of dependencies you often see in deeply nested jQuery code.
The other is that Evently knows a bunch of CouchDB tricks, so you can
get it to run a view query and hand you the results, without having to
write nested callbacks in the Ajax style.

As a coincidence, Evently's declarative structures happen to map onto
JSON objects nicely. It also happens that ``couchapp push`` maps
filesystem structures to JSON code as well. It was only after I'd
written lots of Evently code all in one file, that I realized I could
nest the JSON structures into a tree of folders and JavaScript files.

People think that Evently and the deeply-nested folders and files things
must go together, but it is just one way of doing things.


`Evently primer <evently-primer.rst>`_


Other libs
----------

Pathbinder
++++++++++

`Pathbinder <https://github.com/jchris/couchapp-docs/blob/master/vendor/couchapp/_attachments/jquery.pathbinder.js>`_
makes it so you can assign events to be triggered when the hash part of
the browser URL changes. Evently knows how to use it, so you can
declaratively link paths and events. `Pathbinder docs are
here. <http://couchapp.iriscouch.com/docs/_design/docs/index.html#/topic/pathbinder>`_
