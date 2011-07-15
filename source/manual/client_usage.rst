.. index:: client, adapter

=====================
S4 Client Usage Guide
=====================

In this guide, we will look at some examples of using the client API to
communicate with the S4 cluster.

Getting Set Up
-----------------

To run the examples, you need a usable S4 image that contains the Speech02 sample application. See :ref:`here <building_and_running_speech02>`.

Set environment variables:

.. code-block:: bash

	export PYTHONPATH=${S4_IMAGE}/s4-driver/lib/python
	export PERLLIB=${S4_IMAGE}/s4-driver/lib/perl

Injecting Events with a Perl Client
-----------------------------------

1. Start the S4 cluster, specifying the name of the client adapter cluster
   (``-r``)

.. code-block:: bash

   $S4_IMAGE/scripts/start-s4.sh -r client-adapter &

2. Start the client adapter

.. code-block:: bash

   $S4_IMAGE/scripts/run-client-adapter.sh -s client-adapter \
   -g s4 -d $S4_IMAGE/s4-core/conf/default/client-stub-conf.xml &

3. Inject events

.. code-block:: bash

	perl  $S4_IMAGE/s4-driver/scripts/inject.pl RawSpeech \
	io.s4.example.speech01.Speech < $S4_IMAGE/s4-example-testinput/speech.in 

	perl  $S4_IMAGE/s4-driver/scripts/inject.pl RawSentence \
	io.s4.example.speech01.Sentence < $S4_IMAGE/s4-example-testinput/sentence.in


4. As in the ``speech02`` tutorial, observe that messages are printed by the
   event catcher PE.

Injecting Events with a Java Client
-----------------------------------

1. Start the S4 cluster, specifying the name of the client adapter cluster
   (``-r``)

.. code-block:: bash

   $S4_IMAGE/scripts/start-s4.sh -r client-adapter &

2. Start the client adapter

.. code-block:: bash

   $S4_IMAGE/scripts/run-client-adapter.sh -s client-adapter \
   -g s4 -d $S4_IMAGE/s4-core/conf/default/client-stub-conf.xml &

3. Inject events

.. code-block:: bash

	$S4_IMAGE/s4-driver/scripts/inject.sh localhost 2334 RawSpeech \
	io.s4.example.speech01.Speech < $S4_IMAGE/s4-example-testinput/speech.in 

	$S4_IMAGE/s4-driver/scripts/inject.sh localhost 2334 RawSentence \
	io.s4.example.speech01.Sentence < $S4_IMAGE/s4-example-testinput/sentence.in

4. As in the ``speech02`` tutorial, observe that messages are printed by the
   event catcher PE.

Receiving Events
----------------

1. Follow same steps as above to start S4 server and client adapter.
2. Start a reader client:

.. code-block:: bash

   perl $S4_IMAGE/s4-driver/scripts/read.pl  \
     '{ 
         readMode => "select",
         readInclude => ["SentenceJoined"]
       }'

   This client should connect and print a message something like this:

.. code-block:: perl

      --------------------------------------------------------------------------------
      Initialized: $VAR1 = bless( {
                       'protocol' => {
                                       'versionMinor' => 0,
                                       'versionMajor' => 1,
                                       'name' => 'generic-json'
                                     },
                       'uuid' => '4df124b0-c103-4193-90b1-10ef372c1c0a',
                       'port' => 2334,
                       'host' => 'localhost'
                     }, 'IO::S4::Client' );
      --------------------------------------------------------------------------------
      $VAR1 = undef;
      $VAR2 = undef;
      $VAR3 = {
                'readMode' => 'select',
                'readInclude' => [
                                   'SentenceJoined'
                                 ]
              };

2. In a different window, inject messages like in the previous section.

.. code-block:: bash

	perl  $S4_IMAGE/s4-driver/scripts/inject.pl RawSpeech \
	io.s4.example.speech01.Speech < $S4_IMAGE/s4-example-testinput/speech.in 

	perl  $S4_IMAGE/s4-driver/scripts/inject.pl RawSentence \
	io.s4.example.speech01.Sentence < $S4_IMAGE/s4-example-testinput/sentence.in

3. The reader client prints joined sentence events.

.. code-block:: perl

      $VAR1 = {
                'object' => '{"id":12000001,"speechId":12000000,"text":"Four score and
      seven years ago our fathers brought forth on this continent a new nation,
      conceived in liberty and dedicated to the proposition that all men are created
      equal.","time":1242799205000,"location":"gettysburg, pa, us"}',
                'stream' => 'SentenceJoined',
                'class' => 'io.s4.example.speech01.Sentence'
              };

      $VAR1 = {
                'object' => '{"id":12000002,"speechId":12000000,"text":"Now we are
      engaged in a great civil war, testing whether that nation or any nation so
      conceived and so dedicated can long
      endure.","time":1242799220000,"location":"gettysburg, pa, us"}',
                'stream' => 'SentenceJoined',
                'class' => 'io.s4.example.speech01.Sentence'
              };

      $VAR1 = {
                'object' => '{"id":12000003,"speechId":12000000,"text":"We are met on
      a great battlefield of that war.","time":1242799232000,"location":"gettysburg,
      pa, us"}',
                'stream' => 'SentenceJoined',
                'class' => 'io.s4.example.speech01.Sentence'
              };


Request-Response
----------------

It is possible to send requests into the S4 cluster and receive responses in
return. In general, one request can result in zero, one, or more responses. The
client application is expected to use a timed batch receive method, or some
emulation of it.

There are currently two classes of requests: *Prototype Requests* which target
the prototypes of a particular PE type; and *Single PE Requests* which target a
particular instance of a PE.

All requests have the following attributes

====================   ===============================
Attribute              Definition
====================   ===============================
*Target*               The element (PE or prototype)
                       to which the request must be
                       sent

*Queries*              The content of the request. A
                       list of strings.
                       The result of a query may be
                       either (a) a result, or (b) an
                       exception. These are typically
                       returned to the originating
                       client as an
                       io.s4.message.Response event.

*Return Information*   Information which can be used
                       return the response to the
                       originating PE.
====================   ===============================

Prototype Request
^^^^^^^^^^^^^^^^^

This type of request is made by sending an event of type
``io.s4.message.PrototypeRequest``. These are targeted at the prototype of PEs
of a particular type. One copy of the request is sent to every S4 node in the
cluster, and each node typically responds with one response message. So if the
S4 cluster has ``N`` nodes, the caller should expect up to ``N`` response
events. However, due to the non-guaranteed nature of UDP, not all messages may
be delivered.

Targeting
"""""""""

A Prototype Request is targeted at a particular prototype using the *bean id* of
the PE prefixed with the ``#`` character as the stream name for the request
event.

Return Information
""""""""""""""""""

A query identifier (``long``) may be specified by the client. This can be used
to associate responses with requests.

Query
"""""

Currently, only one operator is supported.

=================  =================================
Query              Meaning
=================  =================================
``count``          Number of PEs cloned from this
                   prototype.
=================  =================================


Single PE Request
^^^^^^^^^^^^^^^^^

This type of request is encoded as an ``io.s4.message.SinglePERequest`` event.
It is targeted at a single PE and can be used to access propertes of the PE with
a public getter method. One request results in at most one response.

Targeting
"""""""""

The PE is targeted by specifying information on two dimensions: the *type* and
the *key value*. The type is specified as using the bean id of the PE prototype
as the stream name of the request event (like prototype requests). The key value
is specified in the Request object's ``target`` field.

Return Information
""""""""""""""""""

A query identifier (``long``) may be specified by the client. This can be used
to associate responses with requests.

Query
"""""

To access a property ``p`` with a public getter method named ``"get" + p``, the
corresponding query is the string ``p``.


Response
^^^^^^^^

The response for a request, consisting of a list of queries, is an object with
the following properties:

:``results``:
    a mapping from queries to corresponding results, for all queries whose
    evaluation did not result in an exception being thrown.
:``exceptions``:
    a mapping from queries to string representations of exceptions that were
    caught, for all queries whose evaluation results in an exception being
    thrown.
:``request``:
    Request object to which the result corresponds.


Example 1
^^^^^^^^^

In this example, we will query the prototype of the joiner (``SentenceJoinPE``)
in the ``speech02`` application.

The JSON representation of the corresponding prototype request is:

.. code-block:: javascript

  {
    "query": ["count"],
    "rinfo": {"id":123}
  }

The ``driver`` repository contains a script to send requests and receive
responses at ``$S4_IMAGE/s4-driver/scripts/request.py``::

  import io.s4.client.driver
  import pprint;
  import sys;

  mode = {'readMode': 'private', 'writeMode': 'enabled'};

  stream = sys.argv[1];
  clazz = sys.argv[2];

  d = io.s4.client.driver.Driver('localhost', 2334)

  #Enable debug messages
  d.setDebug(True);

  d.initialize();
  d.connect(mode);

  print "Sending all requests..."

  for req in sys.stdin.readlines():
      d.send(stream, clazz, req);

  print "Waiting 5 sec to collect all responses..."
  responses = d.recvAll(5);
  print "\n"*4

  print "Done. Results:"
  print pprint.pformat(responses, indent=4);

  d.disconnect();

Use this as follows:

.. code-block:: bash

   python $S4_IMAGE/s4-driver/scripts/request.py '#sentenceJoinPE' \
   'io.s4.message.PrototypeRequest' < $S4_IMAGE/s4-example-testinput/proto-query 

The resulting session starts with something like the following::

  <<[0]
  >>[117]{"protocol":{"name":"generic-json","versionMajor":1,"versionMinor":0},"uuid":"c7c9df6e-754a-41c5-aa8b-a373dcf2b4a6"}

  Initialized. uuid: c7c9df6e-754a-41c5-aa8b-a373dcf2b4a6
  <<[95]{"writeMode": "enabled", "readMode": "private", "uuid":
  "c7c9df6e-754a-41c5-aa8b-a373dcf2b4a6"}
  >>[15]{"status":"ok"}
  Connected
  Sending all requests...


See the protocol in action. In particular, the response object (pretty
formatted) is::

  {
    "result": {"count":11},
    "exception":{},
    "request":{
                "query":["count"],
                "rinfo":{
                          "requesterUUID":"c7c9df6e-754a-41c5-aa8b-a373dcf2b4a6",
                          "id":123,
                          "stream":"@client-adapter",
                          "partition":0
                        }
              }
  }

The result indicates that there are 11 joiner PEs. These correspond to the 11
sentences in the test input file.

.. code-block:: bash

  $ wc -l ${SOURCE_BASE}/examples/testinput/speech.in
  11  ...

Also notice that there is infomation in the ``rinfo`` field, which was not
present in the request that we sent. These are added by the adapter.

Example 2
^^^^^^^^^

Example request to a single PE from
``$S4_IMAGE/s4-example-testinput/proto-query``

.. code-block:: javascript

  {"target":["16000000"],"query":["$outputClassName"],"rinfo":{"id":0,"stream":"@client"}}
  {"target":["*"],"query":["$outputClassName"],"rinfo":{"id":1,"stream":"@client"}}

Send this to the S4 cluster using:

.. code-block:: bash

   python $S4_IMAGE/s4-driver/scripts/request.py '#sentenceJoinPE' \
   'io.s4.message.SinglePERequest' < $S4_IMAGE/s4-example-testinput/pe-query


The input contains two queries. The two corresponding responses
(pretty-formatted and truncated) are::

  {
    "result":{"$outputClassName":"io.s4.example.speech01.Sentence"},
    "exception":{},
    "request":{
                "target":["16000000"],
                "query":["$outputClassName"],
                "rinfo":{
                          "requesterUUID":"cf2d1726-2919-4eb1-85fc-7e420908587e",
                          "id":0,
                          "stream":"@client-adapter",
                          "partition":0
                        }
              }
  }

  {
    "result":{},
    "exception":{"$outputClassName":"java.lang.Exception: Null Target"},
    "request":{
                "target":["*"],
                "query":["$outputClassName"],
                "rinfo":{
                          "requesterUUID":"cf2d1726-2919-4eb1-85fc-7e420908587e",
                          "id":1,
                          "stream":"@client-adapter",
                          "partition":0
                        }
              }
  }

The first response (request ``id`` 0) was sent to a PE, and that PE
responded with its output class name as requested. The second query (request
``id`` 1) was targeted at a joiner corresponding to the key value ``"*"``. No
such PE exists, so an exception was thrown.


