.. title: Learning Twisted #1 reactor basics
.. slug: learning-twisted-1-reactor-basics
.. date: 2010/08/19 01:43:49
.. tags: python,twisted
.. link: 
.. description: 
.. type: text

Learning Twisted #1 reactor basics
=============================================

`Twisted framework`_ is an event based networking framework in Python. 
This description suffices for initial impression about this framework.

These and subsequent posts will serve as my notes for learning Twisted.

At the core of `event based programming`_ is reactor loop that run endlessly unless asked to stop. 

While in the loop, reactor listens to events and dispatches these events to event handlers for processing of events. 
The example below is the simplest way of starting up the reactor.

.. code-block:: python

	print("To stop this example, press ctrl-c")
	from twisted.internet import reactor
	reactor.run()

reactor.run() should be the last line of code that gets executed immediately. 
After this, the control is handed to reactor loop, which listens for events endlessly. 
Some of the methods of this class also allows you to register for events and event handlers to process something real.

There are many reactors available, which needs to be installed. 
Default reactor is select based reactor - "`SelectReactor`_"

Twisted framework uses it's own logging mechanism. 
At the moment, we will see a simple use of logging, till I understand logging better.

.. code-block:: python

	import sys
	from twisted.internet import reactor
	from twisted.python import log
	# Log to standard output
	log.startLogging(sys.stdout)
	print "\nTo stop this example, press ctrl-c\n"
	reactor.run()


There are default `system signal`_ handlers. When ever a signal is recieved, reactor dispatches them to these handlers. Some of them are : sigInt, sigTerm, sigBreak. In the above example, you can see the log message from sigInt. The above code is a dull piece of code, since there are no interesting events and event handlers registered with the reactor.

The capabilities in Twisted are defined at twisted.internet.interface.py. twisted.internet.base.py implements the base capabilities as defined in class IReactorCore.  This provides a mechanism to register an handler for an event which is fired when reactor starts running in listen loop - callWhenRunning.

.. code-block:: python

	import sys
	from twisted.internet import reactor
	from twisted.python import log
	log.startLogging(sys.stdout)
	 
	def callfunc(x):
	    log.msg("In callfunc as event handler")
	    log.msg(str(x))
	    log.msg("Will shut down")
	    reactor.stop()
	    
	reactor.callWhenRunning(callfunc, "This was passed to callfunc")
	reactor.run()

Core reactor functionalities as implemented in twisted.internet.base.py (`base`_) ::

	+----------------+
	|  IReactorCore  |    +--------------+    XXXXXXXXXXXXXXXXXX
	|                |..... IReactorTime |... X                X
	|                |    +--------------+    X  IDelayedCall  X
	+----------------+                        X                X
	                                          XXXXXXXXXXXXXXXXXX

IReactorTime provides a way to register a function to be called after some delay.

.. code-block:: python

	import sys, time
	from twisted.internet import reactor
	from twisted.python import log
	log.startLogging(sys.stdout)
	 
	def callfunc(x):
	    log.msg(str(x))
	    now = time.localtime(time.time())
	    log.msg(str(time.strftime("%y/%m/%d %H:%M:%S",now)))
	    log.msg("Will stop")
	    reactor.stop() # try calling reactor.sigInt() or crash() etc
	    
	now = time.localtime(time.time())
	log.msg("str(time.strftime("%y/%m/%d %H:%M:%S",now)))
	_DelayedCallObj = reactor.callLater(4, callfunc, "callfunc called after 4 sec")
	reactor.run()

callLater function returns a DelayedCall object. You can cancel , delay or reset it's timer.

So far, we have seen some event handlers and events available when reactor gets started. You could add your own custome events and eventHandlers, use the recator to fire the events and let it dispatch it to your custome event handler.

.. code-block:: python
	
	import sys, time
	from twisted.internet import reactor
	from twisted.python import log
	log.startLogging(sys.stdout)
	 
	def callfunc(x):
	    log.msg(str(x))
	    now = time.localtime(time.time())
	    log.msg("str(time.strftime("%y/%m/%d %H:%M:%S",now)))
	    log.msg("Fire Event1 after 4 seconds")
	    reactor.callLater(4, reactor.fireSystemEvent, 'Event1')
	        
	def Event1_CustomEventHandler():
	    now = time.localtime(time.time())
	    log.msg("str(time.strftime("%y/%m/%d %H:%M:%S",now)))
	    log.msg("In Event1_CustomEventHandler .., shutting down")
	    reactor.stop()
	        
	now = time.localtime(time.time())
	log.msg(str(time.strftime("%y/%m/%d %H:%M:%S",now)))
	_DelayedCallObj = reactor.callLater(4, callfunc, "callfunc called after 4 sec")
	reactor.addSystemEventTrigger('during', 'Event1', Event1_CustomEventHandler)
	reactor.run()

Like the above example, you could register system signal event handlers too. This way, you can trap certain signal and do your own custom shutdown.

.. code-block:: python

	import sys
	from twisted.internet import reactor
	from twisted.python import log
	log.startLogging(sys.stdout)
	import signal
	 
	def SIGINT_CustomEventHandler(num, frame):
	    k={1:"SIGHUP", 2:"SIGINT"}
	    log.msg("Recieved signal - " + k[num])
	    if frame is not None:
	        log.msg("SIGINT at %s:%s"%(frame.f_code.co_name, frame.f_lineno))
	    log.msg("In SIGINT_CustomEventHandler")
	    if num == 2:
	        log.msg("shutting down ....")
	        reactor.stop()
	 
	signal.signal(signal.SIGINT, SIGINT_CustomEventHandler)
	signal.signal(signal.SIGHUP, SIGINT_CustomEventHandler)
	reactor.run()

This should suffice to show some of the basic aspects of a reactor, events and event handlers. 


.. _Twisted Framework: http://twistedmatrix.com/trac/wiki/Documentation

.. _event based programming: http://en.wikipedia.org/wiki/Event-driven_programming

.. _SelectReactor: http://twistedmatrix.com/trac/browser/trunk/twisted/internet/selectreactor.py

.. _system signal: http://en.wikipedia.org/wiki/Signal_(computing)

.. _base: http://twistedmatrix.com/trac/browser/trunk/twisted/internet/interfaces.py

