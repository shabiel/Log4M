# Log4M

The XTMLOG\* routines provide a Log4M capability similar to Log4J.  The 
logging commands can be embedded in the code and activated by initializing 
logging in one of several different ways.  Otherwise, the logging commands 
are checked and then ignored.

Logging is useful to verify behavior during development and to find out
problems in production. If it is turned off, it has no effect.

# Brief Guide
A more comprehensive guide is in the wiki.

Before you start logging, you must turn logging on. Normally, this is done by
creating an entry in the `LOG4M CONFIG (#8992.7)` file, but we will skip that for
the simple examples below. After it's turned on, it needs to be turned off;
however, if the process exits, it will turn off by itself.

To turn logging on, you can do this (C means print to the console. You can
also save to a global or stream to a socket, but that will be covered later).

```
  D INITEASY^XTMLOG("C")
```

To turn logging off, you can just do (but don't do it yet!):

```
  D ENDLOG^XTMLOG()
```

XTMLOG can be used to output a simple message: 

```
   D INFO^XTMLOG("ENTERED CHKWATCH")
```
   
will result in an output similar to

```
   20080207.145617 INFO CHKWATCH+3 XTDEBUG5 - ENTERED CHKWATCH
```
   
XTMLOG can be used to output a message along with values for one or more
comma separated variables

```
   D DEBUG^XTMLOG("DATA1","VALUE1,VALUE2")
   
   20080207.145617 DEBUG COMMANDS+11 XTDEBUG1 - DATA1 - VALUE1: 0
   20080207.145617 DEBUG COMMANDS+11 XTDEBUG1 - DATA1 - VALUE2: XVAL=XTDEBV(1)
```

   
XTMLOG can be used to output a message and values for variables including
array values for the variables if they exist.

```
   D DEBUG^XTMLOG("DATA2","VALUE1,VALUE2,VALUE3,^TMP($J,""DATA"")",1)

   20080207.145617 DEBUG COMMANDS+22 XTDEBUG1 - DATA2 - VALUE1: 3
   20080207.145617 DEBUG COMMANDS+22 XTDEBUG1 - DATA2 - VALUE2: XVAL=XTDEBV(1)
   20080207.145617 DEBUG COMMANDS+22 XTDEBUG1 - DATA2 - VALUE2("NEW"): 15
   20080207.145617 DEBUG COMMANDS+22 XTDEBUG1 - DATA2 - VALUE2("OLD",1): 
   20080207.145617 DEBUG COMMANDS+22 XTDEBUG1 - DATA2 - VALUE3: <UNDEFINED>
   20080207.145617 DEBUG COMMANDS+22 XTDEBUG1 - DATA2 - ^TMP($J,"DATA",1): LAST,FIRST M
   20080207.145617 DEBUG COMMANDS+22 XTDEBUG1 - DATA2 - ^TMP($J,"DATA",2): 04/01/2001
```

In addition to `INFO` and `DEBUG`, you also have `WARN`, `ERROR`,
and `FATAL^XTMLOG`. Each of them prints a different severity error. We discuss
the severities below.

The best way to control logging is through an entry in the `LOG4M CONFIG 
file (#8992.7)`.

A typical entry looks like this:

```
  NAME: MY-APPLICAITON                          ACTIVE: YES, EASY CONFIG
  EZ ENTRY: C;G,MYAPPLICATION                   EZ LEVEL: DEBUG
```

The NAME field (#.01) for an entry is used to control logging.  The NAME of 
an entry (e.g., XTEXAMPLE) can be referenced in the initialization process 
and if the file entry does not exist, the logging is not activated.  If the   
If the entry exists and the ACTIVE field (#.02) is set to NO, logging will not 
be activated.  Otherwise logging as instructed by the settings for this entry 
will be activated for the current job, until logging for the entry is 
terminated.  

  An XTMEXAMPLE entry could be created in the LOG4M CONFIG file.  It would 
then be activated in the code, processing performed, and the logging turned
off as in the example below.
  
```
  D FILEINIT^XTMLOG("XTMEXAMPLE")
  D PROCESSING
  D ENDLOG^XTMLOG("XTMEXAMPLE")
```

  If the XTMEXAMPLE entry does not exist in the LOG4M CONFIG file, logging will
not be turned on.  If the XTMEXAMPLE entry does exist, the value of the 
ACTIVE field (#.02) is checked, and if it is not specified or NO, logging 
will not be turned on.

  If the ACTIVE field of the XTMEXAMPLE entry is E for YES, EASY CONFIG, 
logging will be activated based on the values in the EASY ENTRY (#.03) and 
EASY LEVEL (#.04) fields. (The alternative to the E or YES, EASY CONFIG for
active logging is D or YES, DETAILED CONFIG which permits the user to specify
a text based configuration, similar to that used for Log4J and Java, in a
word processing field (DETAILED CONFIG field (#1)).  This alternative was 
originally based on the use of config files for Log4J, but it is recommended 
that the EASY CONFIG be used instead.)

The EASY ENTRY field (#.03) can be used to easily specify the type(s) of 
logging desired.  The value is a text string with semi-colon separated 
specifiers for logging modes.

```
   C - indicates logging to the user's console.  Logging messages are sent 
       to the console as they are generated.
   G - indicates logging to a global location and is followed by a comma 
       and an identifer for the global location under ^XTMP("XTMLOG", [e.g., 
       G,TEST4 would result in data being stored under the global location
       ^XTMP("TEST4",DUZ,yymmdd.hhmmss,$J, where DUZ is the internal entry
       number for the user in the NEW PERSON file (#200), and yymmdd.hhmmss is
       the date and time the logging was initialized, and $J is the job number
       of the user's process].  When logging is initialized for a subscript,
       such as TEST4, the lifetime for the ^XTMP("TEST4" global is set or 
       updated to a week from the current date.
   S - (E.g. S,127.0.0.1:60002) will stream logging data to a remote TCP/IP
       socket. This is useful when you are (for example) using CPRS and want
       to see messages dynamically as they are generated.
```
       
The EASY LEVEL field (#.04) can be used to specify the level of logging 
desired.  The highest level (with respect to usual urgency) is FATAL.  
The lowest level (again with respect to usual urgency) is DEBUG.  
Between these two extremes are (in decreasing order of usual urgency) ERROR, 
WARN, and INFO.  Choosing a specific level (e.g., INFO) will include all 
higher levels as well (e.g., for INFO, any logging calls with a level of 
FATAL, ERROR, WARN or INFO would be output).

The ROUTINE FILTER field (#.05) may be used to control the routines which 
logging is active in.  These controls permit the amount of 
data logged to the system to be maintained at a reasonable amount, even if 
a large number of users are actively using the code which is being logged.

If data has been entered in the USER FILTER field (#.06), logging will only 
be activated for users whose internal entry number in file 200 are included 
in the USER FILTER field.  

Specialized fields, which would not normally be required, but are available, 
are:

```
^DD(8992.7,.07,0)=OUTPUT ON CLOSE^S^N:NONE;P:PRINTER;M:MAIL MESSAGE;^0;7^Q
^DD(8992.7,.08,0)=OUTPUT SPECS^F^^0;8^K:$L(X)>25!($L(X)<1) X
^DD(8992.7,1,0)=DETAILED CONFIG^8992.71^^1;0
^DD(8992.7,2.01,0)=PRINT LAYOUT^F^^2;1^K:$L(X)>40!($L(X)<5) X
```

If you saved data into a global, there is an entry point which can display
it for you: `VIEW^XTMLOG1` as a colored output.

To delete data you saved into a global, the entry point ```CLEAR^XTMLOG1```
clears it for you. WARNING: This call now deletes the XTMP entry with the
same name as the .01 field entry in Log4M Config file. It *should* rather
try to figure out the name of the global from EZ Config or Detailed Config
or ^XTMP and delete that.

# Developer Notes
 
 * TODOs are listed at the top of ^XTMLOG.
 * Unit Tests are in ^XTMLT1, which is the driver for the other Unit
   Test routines. Coverage is about 35%. This obviously needs work.
