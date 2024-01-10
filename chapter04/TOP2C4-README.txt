TOP2 chapter04	28 scripts

@host_load_hist

@ash_activity           1: all | sid   2: all | sql_id
@ash_top_actions        1: time_begin  2: time_end
@ash_top_clients        1: time_begin  2: time_end
@ash_top_files          1: time_begin  2: time_end
@ash_top_modules        1: time_begin  2: time_end
@ash_top_objects        1: time_begin  2: time_end
@ash_top_plsql          1: time_begin  2: time_end
@ash_top_services       1: time_begin  2: time_end
@ash_top_sessions       1: time_begin  2: time_end
@ash_top_sqls           1: time_begin  2: time_end   3: all | sid   4: all | sql_id

@report_sql_detail      1: sql_id
@report_sql_monitor     1: sql_id

@sql                    1: sql_id
@sqlarea                1: sql_id  2: interval N seconds
@sqlstats               1: sql_id  2: interval N seconds


host_load_setup teardown
active_sessions_setup teardown
system_activity_setup teardown
time_model_setup teardown

@host_load              1: interval N minutes
@active_sessions        1: interval N minutes   2: No. samples  3: No. sessions
@system_activity        1: interval N minutes   2: No. samples
@time_model             1: interval N minutes   2: No. samples

