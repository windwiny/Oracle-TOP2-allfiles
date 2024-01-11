

Chapter 3: Analysis of Reproducible Problems
    Tracing Database Calls
        SQL Trace
            Enabling SQL Trace with ALTER SESSION
            Enabling SQL Trace with DBMS_MONITOR
                Session Level
                Client Level
                Component Level
                Database Level
            Enabling SQL Trace with DBMS_SESSION
            Triggering SQL Trace
            Timing Information in Trace Files
            Limiting the Size of Trace Files
            Finding Trace Files
        Structure of the Trace Files
        Using TRCSESS
        Profilers
        Using TKPROF
            TKPROF Arguments
            Interpreting TKPROF Output
        Using TVD$XTAT
            Why Is TKPROF Not Enough?
            Installation
            TVD$XTAT Arguments
            Interpreting TVD$XTAT Output
    Profiling PL/SQL Code
        Using DBMS_HPROF
            Installing the Output Tables
            Gathering the Profiling Data
            Manually Reporting the Profiling Data
            Using PLSHPROF
            Using a GUI
        Using DBMS_PROFILER
            Installing the Output Tables
            Gathering the Profiling Data
            Manually Reporting the Profiling Data
            Using a GUI
        Triggering the Profilers
    On to Chapter 4


# Tracing Database Calls

## SQL Trace

### ALTER SESSION

ALTER SESSION SET sql_trace = TRUE ;   

 which yields a level 1 trace
 
ALTER SESSION SET events '10046 trace name context forever, level 12' ;

ALTER SESSION SET events '10046 trace name context off' ;

   
### DBMS_MONITOR

#### Session Level

dbms_monitor.session_trace_enable(session_id => 127,
 serial_num => 29,
 waits => TRUE,
 binds => FALSE,
 plan_stat => 'first_execution')

SELECT sql_trace, sql_trace_waits, sql_trace_binds, sql_trace_plan_stats
  FROM v$session
  WHERE sid = 127;
 
#### Client Level

dbms_monitor.client_id_trace_enable(client_id => 'helicon.antognini.ch',
 waits => TRUE,
 binds => TRUE,
 plan_stat => 'first_execution');

SELECT primary_id AS client_id, waits, binds, plan_stats
  FROM dba_enabled_traces
  WHERE trace_type = 'CLIENT_ID';

#### Component Level

dbms_monitor.serv_mod_act_trace_enable(service_name => 'DBM11203.antognini.ch',
 module_name => 'mymodule',
 action_name => 'myaction',
 waits => TRUE,
 binds => TRUE,
 instance_name => NULL,
 plan_stat => 'all_executions');

SELECT primary_id AS service_name, qualifier_id1 AS module_name,
  qualifier_id2 AS action_name, waits, binds, plan_stats
  FROM dba_enabled_traces
  WHERE trace_type IN ('SERVICE', 'SERVICE_MODULE', 'SERVICE_MODULE_ACTION');

#### Database Level

dbms_monitor.database_trace_enable(waits => TRUE,
 binds => TRUE,
 instance_name => 'DBM11203',
 plan_stat => 'first_execution');


SELECT instance_name, waits, binds, plan_stats
  FROM dba_enabled_traces
  WHERE trace_type = 'DATABASE';
  


### DBMS_SESSION

BEGIN
 dbms_session.session_trace_enable(waits => TRUE,
 binds => TRUE,
 plan_stat => 'all_executions');
END;
/

SELECT sql_trace, sql_trace_waits, sql_trace_binds, sql_trace_plan_stats
  FROM v$session
  WHERE sid = sys_context('userenv','sid');
  
  
### Triggering

CREATE ROLE sql_trace;
CREATE OR REPLACE TRIGGER enable_sql_trace AFTER LOGON ON DATABASE
 BEGIN
 IF (dbms_session.is_role_enabled('SQL_TRACE'))
 THEN
  EXECUTE IMMEDIATE 'ALTER SESSION SET timed_statistics = TRUE';
  EXECUTE IMMEDIATE 'ALTER SESSION SET max_dump_file_size = unlimited';
  dbms_session.session_trace_enable;
 END IF;
END;


### Timing Info in Trace Files

ALTER SESSION SET timed_statistics = TRUE ;

### Limiting the size of Trace Files

ALTER SESSION SET max_dump_file_size = 'unlimited' ;


### Finding Trace Files

SELECT value FROM v$parameter WHERE name = 'diagnostic_dest';
SELECT value FROM v$diag_info WHERE name = 'Diag Trace';

{instance name}_{process name}_{process id}.trc

SELECT value FROM v$diag_info WHERE name = 'Default Trace File';
SELECT p.tracefile FROM v$process p, v$session s   WHERE p.addr = s.paddr
  AND s.sid = sys_context('userenv','sid');

{instance name}_{process name}_{process id}_{tracefile identifier}.trc


## Structure of the Trace Files

## Using TRCSESS

trcsess [output=<output file name >] [session=<session ID>] [clientid=<clientid>]
        [service=<service name>] [action=<action name>] [module=<module name>]
        <trace file names>

  output=<output file name> output destination default being standard output.
  session=<session Id> session to be traced.
    Session id is a combination of session Index & session serial number e.g. 8.13.
  clientid=<clientid> clientid to be traced.
  service=<service name> service to be traced.
  action=<action name> action to be traced.
  module=<module name> module to be traced.
  <trace_file_names> Space separated list of trace files with wild card '*' supported.

## Profiles


## Using TKPROF

tkprof DBM11106_ora_6334.trc DBM11106_ora_6334.txt


## Using  TVD$XTAT

Usage: tkprof tracefile outputfile [explain= ] [table= ]
              [print= ] [insert= ] [sys= ] [sort= ]
  table=schema.tablename Use 'schema.tablename' with 'explain=' option.
  explain=user/password Connect to ORACLE and issue EXPLAIN PLAN.
  print=integer List only the first 'integer' SQL statements.
  aggregate=yes|no
  insert=filename List SQL statements and data inside INSERT statements.
  sys=no TKPROF does not list SQL statements run as user SYS.
  record=filename Record non-recursive statements found in the trace file.
  waits=yes|no Record summary for any wait events found in the trace file.
  sort=option Set of zero or more of the following sort options:
    prscnt number of times parse was called
    prscpu cpu time parsing
    prsela elapsed time parsing
    prsdsk number of disk reads during parse
    prsqry number of buffers for consistent read during parse
    prscu number of buffers for current read during parse
    prsmis number of misses in library cache during parse
    execnt number of execute was called
    execpu cpu time spent executing
    exeela elapsed time executing
    exedsk number of disk reads during execute
    exeqry number of buffers for consistent read during execute
    execu number of buffers for current read during execute
    exerow number of rows processed during execute
    exemis number of library cache misses during execute
    fchcnt number of times fetch was called
    fchcpu cpu time spent fetching
    fchela elapsed time fetching
    fchdsk number of disk reads during fetch
    fchqry number of buffers for consistent read during fetch
    fchcu number of buffers for current read during fetch
    fchrow number of rows fetched
    userid userid of user that parsed the cursor








# Profiling PL/SQL Code


## Using DBMS_HPROF

Usage: plshprof [<option>...] <tracefile1> [<tracefile2>]
Options:
-trace <symbol> (no default) specify function name of tree root
-skip <count> (default=0) skip first <count> invokations
-collect <count> (default=1) collect info for <count> invokations
-output <filename> (default=<symbol>.html or <tracefile1>.html)
-summary print time only


## Using DBMS_PROFILER

SELECT dbms_profiler.start_profiler AS status
  FROM dual;

execute perfect_triangles(1000)
SQL> SELECT dbms_profiler.stop_profiler AS status,
  plsql_profiler_runnumber.currval AS runid
  FROM dual;
  


SELECT s.line,
  round(ratio_to_report(p.total_time) OVER ()*100,1) AS time,
  total_occur,
  s.text
  FROM all_source s,
  (SELECT u.unit_owner, u.unit_name, u.unit_type,
  d.line#, d.total_time, d.total_occur
  FROM plsql_profiler_units u, plsql_profiler_data d
  WHERE u.runid = 1
  AND d.runid = u.runid
  AND d.unit_number = u.unit_number) p
  WHERE s.owner = p.unit_owner (+)
  AND s.name = p.unit_name (+)
  AND s.type = p.unit_type (+)
  AND s.line = p.line# (+)
  AND s.owner = user
  AND s.name = 'PERFECT_TRIANGLES'
  AND s.type IN ('PROCEDURE', 'PACKAGE BODY', 'TYPE BODY')
  ORDER BY s.line;



## Triggering the Profiles

CREATE TRIGGER start_hprof_profiler AFTER LOGON ON DATABASE
BEGIN
IF (dbms_session.is_role_enabled('HPROF_PROFILE'))
THEN
dbms_hprof.start_profiling(
location => 'PLSHPROF_DIR',
filename => 'dbms_hprof_'||sys_context('userenv','sessionid')||'.trc'
);
END IF;
END;
/
CREATE TRIGGER stop_hprof_profiler BEFORE LOGOFF ON DATABASE
BEGIN
IF (dbms_session.is_role_enabled('HPROF_PROFILE'))
THEN
dbms_hprof.stop_profiling();
END IF;
END;
/


