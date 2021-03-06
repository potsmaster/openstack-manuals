=================
Analyze log files
=================

Use the swift command-line client to analyze log files.

The swift client is simple to use, scalable, and flexible.

Use the swift client ``-o`` or ``-output`` option to get short answers
to questions about logs.

You can use the ``-o`` or ``--output`` option with a single object
download to redirect the command output to a specific file or to STDOUT
(``-``). The ability to redirect the output to STDOUT enables you to
pipe (``|``) data without saving it to disk first.

Upload and analyze log files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. This example assumes that ``logtest`` directory contains the
   following log files::

       2010-11-16-21_access.log
       2010-11-16-22_access.log
       2010-11-15-21_access.log
       2010-11-15-22_access.log


   Each file uses the following line format::

       Nov 15 21:53:52 lucid64 proxy-server - 127.0.0.1 15/Nov/2010/22/53/52 DELETE /v1/AUTH_cd4f57824deb4248a533f2c28bf156d3/2eefc05599d44df38a7f18b0b42ffedd HTTP/1.0 204 - \
       - test%3Atester%2CAUTH_tkcdab3c6296e249d7b7e2454ee57266ff - - - txaba5984c-aac7-460e-b04b-afc43f0c6571 - 0.0432


#. Change into the ``logtest`` directory::

       $ cd logtest

#. Upload the log files into the ``logtest`` container::

       $ swift -A http://swift-auth.com:11000/v1.0 -U test:tester -K testing upload logtest *.log

   .. code::

       2010-11-16-21_access.log
       2010-11-16-22_access.log
       2010-11-15-21_access.log
       2010-11-15-22_access.log

#. Get statistics for the account::

       $ swift -A http://swift-auth.com:11000/v1.0 -U test:tester -K testing \
       -q stat

   .. code::

       Account: AUTH_cd4f57824deb4248a533f2c28bf156d3
       Containers: 1
       Objects: 4
       Bytes: 5888268

#. Get statistics for the logtest container::

       $ swift -A http://swift-auth.com:11000/v1.0 -U test:tester -K testing \
       stat logtest

   .. code::

       Account: AUTH_cd4f57824deb4248a533f2c28bf156d3
       Container: logtest
       Objects: 4
       Bytes: 5864468
       Read ACL:
       Write ACL:

#. List all objects in the logtest container::

       $ swift -A http:///swift-auth.com:11000/v1.0 -U test:tester -K testing \
       list logtest

   .. code::

       2010-11-15-21_access.log
       2010-11-15-22_access.log
       2010-11-16-21_access.log
       2010-11-16-22_access.log

Download and analyze an object
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This example uses the ``-o`` option and a hyphen (``-``) to get
information about an object.

Use the swift ``download`` command to download the object. On this
command, stream the output to ``awk`` to break down requests by return
code and the date ``2200 on November 16th, 2010``.

Using the log line format, find the request type in column 9 and the
return code in column 12.

After ``awk`` processes the output, it pipes it to ``sort`` and ``uniq
-c`` to sum up the number of occurrences for each request type and
return code combination.

#. Download an object::

       $ swift -A http://swift-auth.com:11000/v1.0 -U test:tester -K testing \
            download -o - logtest 2010-11-16-22_access.log | awk '{ print \
            $9"-"$12}' | sort | uniq -c

   .. code::

       805 DELETE-204
       12 DELETE-404
       2 DELETE-409
       723 GET-200
       142 GET-204
       74 GET-206
       80 GET-304
       34 GET-401
       5 GET-403
       18 GET-404
       166 GET-412
       2 GET-416
       50 HEAD-200
       17 HEAD-204
       20 HEAD-401
       8 HEAD-404
       30 POST-202
       25 POST-204
       22 POST-400
       6 POST-404
       842 PUT-201
       2 PUT-202
       32 PUT-400
       4 PUT-403
       4 PUT-404
       2 PUT-411
       6 PUT-412
       6 PUT-413
       2 PUT-422
       8 PUT-499

#. Discover how many PUT requests are in each log file.

   Use a bash for loop with awk and swift with the ``-o`` or
   ``--output`` option and a hyphen (``-``) to discover how many PUT
   requests are in each log file.

   Run the swift ``list`` command to list objects in the logtest
   container. Then, for each item in the list, run the swift ``download
   -o -`` command. Pipe the output into grep to filter the PUT requests.
   Finally, pipe into ``wc -l`` to count the lines.

   .. code::

       $ for f in `swift -A http://swift-auth.com:11000/v1.0 -U test:tester \
       -K testing list logtest` ; \
               do  echo -ne "PUTS - " ; swift -A \
               http://swift-auth.com:11000/v1.0 -U test:tester \
               -K testing download -o -  logtest $f | grep PUT | wc -l ; \
           done

   .. code::

       2010-11-15-21_access.log - PUTS - 402
       2010-11-15-22_access.log - PUTS - 1091
       2010-11-16-21_access.log - PUTS - 892
       2010-11-16-22_access.log - PUTS - 910

#. List the object names that begin with a specified string.

#. Run the swift ``list -p 2010-11-15`` command to list objects in the
   logtest container that begin with the ``2010-11-15`` string.

#. For each item in the list, run the swift **download -o -** command.

#. Pipe the output to **grep** and **wc**. Use the **echo** command to
   display the object name::

       $ for f in `swift -A http://swift-auth.com:11000/v1.0 -U test:tester \
       -K testing list -p 2010-11-15 logtest` ; \
               do  echo -ne "$f - PUTS - " ; swift -A \
               http://127.0.0.1:11000/v1.0 -U test:tester \
               -K testing download -o - logtest $f | grep PUT | wc -l ; \
             done

   .. code::

       2010-11-15-21_access.log - PUTS - 402
       2010-11-15-22_access.log - PUTS - 910

