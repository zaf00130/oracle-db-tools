`alter session set nls_date_format = 'DD-MON-YYYY HH24:MI:SS';`

```
SELECT DISTINCT c.user_concurrent_program_name,
    round(((sysdate-a.actual_start_date)*24*60),2) AS run_time_minutes,
     a.request_id, a.parent_request_id, a.request_date,
     a.actual_start_date, a.actual_completion_date,
      (a.actual_completion_date-a.request_date)*24*60 AS end_to_end_minutes,
      (a.actual_start_date-a.request_date)*24*60*60 AS lag_time,
      d.user_name, a.phase_code,a.status_code,a.argument_text,a.priority
FROM apps.fnd_concurrent_requests a,
    apps.fnd_concurrent_programs b ,
    apps.FND_CONCURRENT_PROGRAMS_TL c,
    apps.fnd_user d
WHERE a.concurrent_program_id = b.concurrent_program_id AND
    b.concurrent_program_id = c.concurrent_program_id AND
    a.requested_by = d.user_id AND
    status_code = 'R'
    order by run_time_minutes desc;
```

Note: some columns are omitted in the output below to improve readability.

|               USER_CONCURRENT_PROG 	| RUN_TIME 	|  REQ_ID 	|  PAR_ID 	|    REQUEST_DATE 	|       ACT_START 	|
|-----------------------------------:	|---------:	|--------:	|--------:	|----------------:	|----------------:	|
|           Gather Schema Statistics 	|     6.98 	| 5221663 	|      -1 	| 7/20/2013 21:24 	| 7/20/2013 21:25 	|
|      Function Security Menu Report 	|     0.03 	| 5221704 	| 5221702 	| 7/20/2013 21:31 	| 7/20/2013 21:31 	|
| Function Security Navigator Report 	|     0.02 	| 5221705 	| 5221702 	| 7/20/2013 21:31 	| 7/20/2013 21:31 	|

---

A common task for an Oracle E-Business Suite Apps DBA is to monitor the currently running concurrent requests (batch jobs).  
I prefer the above SQL query which gives this information along with some helpful details.  
Note that I ran the first "alter session" command to get the dates formatted as shown in the example.  
This query is generally useful for seeing the current state of the concurrent requests,  
but I frequently use it when investigating reports from users about concurrent requests being "slow" or  
before issuing the "adstpall.sh" or "adcmctl.sh stop" commands which stop the concurrent managers  
(if there is a request running, I must be prepared to wait for it to complete or to kill it).

`USER_CONCURRENT_PROGRAM_NAME`: The name of the concurrent request.

`RUN_TIME_MINUTES`: The time in minutes that the request has been running.

`REQUEST_ID`: The unique ID for each request.

`PARENT_REQUEST_ID`: If the request is a child spawned by a parent request, the parent's request ID will be here. If the request did not have a parent, it will be -1.

`REQUEST_DATE`: The time and date that the request was submitted.

`ACTUAL_START_DATE`: The time and date that the request actually started running.

`ACTUAL_COMPLETION_DATE`: The time and date that the request completed. Note that this should be null since the query shows only running requests, but potentially useful if the where clause is modified (something other than STATUS_CODE = 'R').

`END_TO_END_MINUTES`: The interval in minutes between when the request actually completed and when the user submitted it. Note that this should be null for requests that are currently running, but this is potentially useful if the where clause is modified.

`LAG_TIME`: The interval in seconds between when the request was submitted and when it actually started running. This should normally be a few seconds, but if it is large it can possibly indicate a backlog of concurrent requests.

`USER_NAME`: The application user that submitted the request.

`PHASE_CODE`: The phase of the request.

`STATUS_CODE`: The status of request. Note that 'R' means it is currently running.

`ARGUMENT_TEXT`: The parameters with which the request was submitted.

`PRIORITY`: The priority of the concurrent request.
