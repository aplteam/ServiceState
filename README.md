# Interface between the Windows Service Control Manager and APL


`ServiceState` is a member of the APLTree library. The library is a collection of classes etc. that aim to support the Dyalog APL programmer. Search GitHub for "apltree" and you will find solutions to many every-day problems Dyalog APL programmers might have to solve.


## Overview 

`ServiceState` is a namespace script that offers an interface between the Windows SCM (Service Control Manager) and a Dyalog APL application running as a service.
 
In you application workspace you need to have a function on `⎕LX` similar to this:

```
      Start;ps
[1]   ps←#.ServiceState.CreateParmSpace
[2]   ps.ride←1
[3]   #.ServiceState.Init ps
[4]   :If #.ServiceState.IsRunningAsService
[5]      {#.TestService.Run ⍵}&⍬
[6]      ⎕DQ'.'
[7]     #.TestService.Off
[8]  :Else
[9]     #.TestService.Run ⍬
[10] :EndIf
```

Note that this example focuses on getting the application running as a service. For the purpose of demonstrating how to overwrite defaults (no RIDE) it shows how to create a parameter space with default settings on line [1] and then overwrite a default on line [2].

If you are happy with the defaults anyway (which you can check by executing `#.ServiceState.CreateParmSpace.∆List`) then you can specify `⍬` as the right argument of `#.ServiceState.Init` in line [3].

Remarks:

**[4]**: Checks whether the application is actually running as a Windows Service. This allows to execute the application in its own thread in case its running as a service but not otherwise - that makes debugging the application significantly easier.

**[5]**: Here we start the real application. Note that this **must** run in its own thread because otherwise the application might well become unresponsive.

**[6]**: We need to execute `⎕DQ '.'` in order to make sure that the interpreter processes all events.

**[7]**: At this stage we need to execute `⎕OFF`; that's what `#.TestService.Off` is doing.


## The application 

The application must make sure in its main loop that it checks for state changes, and act accordingly. This is all done by the Operator `#.ServiceState.CheckServiceMessages` which takes the name of a logging function as operand:

```
  MainLoop dummmy;buffer;data;event;obj;rc;r;msg;cs
[1]   :Repeat
[2]       ⎕DL 1                     ⍝ What is reasonable here depends on the application
[3]       :If (Log #.ServiceState.CheckServiceMessages)#.ServiceState.IsRunningAsService
[4]           Log'The service is in the process of shutting down...'
[5]           :Leave
[6]       :EndIf
[7]   :Until 0
``` 

Remarks:

Line **[3]** checks for any changes signalled from the Service Control Manager (SCM). If a "Pause" is detected, then the function loops until either a "Continue" or a "Stop" is detected. The explicit result is 1 in case a "Stop" is detected and 0 otherwise.