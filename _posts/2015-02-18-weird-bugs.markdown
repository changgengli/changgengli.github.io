---
layout: post
title: Wired Bugs - A SocketTimeoutException before request is being sent
---

Troubleshooting is fun, espcially for those bugs hard to reproduce.

This issue was first detected by our integration test framework. Occasionlly
some of the test failed after we introduce the [async
client](https://hc.apache.org/httpcomponents-asyncclient-dev/). 

What is interesting with this one is we cannot find the request in server side
access.log. 

The client is logging a SocketTimeoutException in the stack trace.

    java.util.concurrent.ExecutionException: java.net.SocketTimeoutException
        at org.apache.http.concurrent.BasicFuture.getResult(BasicFuture.java:68)
        at org.apache.http.concurrent.BasicFuture.get(BasicFuture.java:94)
        at org.apache.http.AsyncClientTest.testTimeout(AsyncClientTest.java:36)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
    ....
    Caused by: java.net.SocketTimeoutException
        at org.apache.http.nio.protocol.HttpAsyncRequestExecutor.timeout(HttpAsyncRequestExecutor.java:286)
        at org.apache.http.impl.nio.client.LoggingAsyncRequestExecutor.timeout(LoggingAsyncRequestExecutor.java:121)
        at org.apache.http.impl.nio.client.InternalIODispatch.onTimeout(InternalIODispatch.java:85)
        at org.apache.http.impl.nio.client.InternalIODispatch.onTimeout(InternalIODispatch.java:37)
        at org.apache.http.impl.nio.reactor.AbstractIODispatch.timeout(AbstractIODispatch.java:172)
        at org.apache.http.impl.nio.reactor.BaseIOReactor.sessionTimedOut(BaseIOReactor.java:255)
        at org.apache.http.impl.nio.reactor.AbstractIOReactor.timeoutCheck(AbstractIOReactor.java:493)
        at org.apache.http.impl.nio.reactor.BaseIOReactor.validate(BaseIOReactor.java:205)
        at org.apache.http.impl.nio.reactor.AbstractIOReactor.execute(AbstractIOReactor.java:281)
        at org.apache.http.impl.nio.reactor.BaseIOReactor.execute(BaseIOReactor.java:105)
        at org.apache.http.impl.nio.reactor.AbstractMultiworkerIOReactor$Worker.run(AbstractMultiworkerIOReactor.java:584)
        at java.lang.Thread.run(Thread.java:662) 


After going through the code where the original SocketTimeOutException is
thrown, we found:

* The connection will be put back into a connection pool after it being used

* To send the next request, a connection will be fetch from the pool

* There is a timeout check before the request is being sent, and the timeout
   check is based on the lastReadTime and lastWriteTime

* After the connection is put back in pool, the lastReadTime and lastWriteTime
   won't be updated, so if the connection stays in the pool long enough, there
   is a chance it gets timeout.

We believe the timeout should count from the time a connection is fetched from
the pool and this is a bug in http async client libary.

Finally this [jira ticket](https://issues.apache.org/jira/browse/HTTPASYNC-88)
is created.

