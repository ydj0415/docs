# DBA Script

#### 기본 Configuration
SELECT NAME ,setting ,unit ,short_desc ,extra_desc ,boot_val ,reset_val ,sourcefile ,sourceline ,context ,vartype ,source ,min_val ,max_val FROM pg_settings ORDER BY NAME;

#### 기본이 아닌 변경된 Configuration
SELECT NAME ,setting ,unit ,short_desc ,extra_desc ,context ,vartype ,source ,min_val ,max_val FROM pg_settings WHERE source <> 'default' ORDER BY NAME;


로그파일 이름, 크기, 로그정보	SELECT * FROM ( SELECT pg_ls_dir('pg_log')) AS tmp (filename) WHERE filename LIKE '%csv' AND EXISTS (SELECT 1 FROM pg_stat_file('pg_log/'||filename) WHERE not isdir) ORDER BY 1 DESC LIMIT 1 -- to get the latest filename
SELECT size FROM pg_stat_file('pg_log/enterprisedb-2015-06-10.csv') -- to get the size of the latest filename
SELECT pg_read_file('pg_log/enterprisedb-2015-06-10.csv', 0, 5918) as contents -- to get the contents of the latest filename										
데이터베이스 정보확인	SELECT datname, pg_get_userbyid(datdba) AS dba, pg_catalog.pg_encoding_to_char(encoding) AS encoding, datcollate, datctype, datistemplate, datallowconn, datconnlimit, datlastsysoid, datfrozenxid, spcname as tablespace, pg_size_pretty(pg_database_size(datname)) AS size, datacl, age(datfrozenxid) AS freezeage, ROUND(100*(age(datfrozenxid)/freez::float)) AS perc FROM pg_database, pg_tablespace JOIN (SELECT setting AS freez FROM pg_settings WHERE name = 'autovacuum_freeze_max_age') AS param ON (true) WHERE dattablespace = pg_tablespace.oid ORDER BY datname;										
캐시 된 데이터베이스 정보확인	SELECT dba, datname, size, buffers, buffers * 8192 AS buffer_size, case when buffers = 0 then 0 else round(buffers::numeric*8192/size::numeric*100,2) end AS "Cache %" FROM (SELECT pg_get_userbyid(datdba) AS dba, datname, pg_database_size(reldatabase) AS size, count(*) AS buffers FROM pg_buffercache, pg_database WHERE reldatabase=pg_database.oid GROUP BY 1, 2, 3) ORDER BY 1, 2, 3;										
Roles 정보확인	SELECT rolname, rolsuper, rolinherit, rolcreaterole, rolcreatedb, rolcatupdate, rolcanlogin, rolconnlimit, rolreplication, rolvaliduntil, rolconfig FROM pg_roles ORDER BY rolname;										
유저 Object 확인	SELECT pg_get_userbyid(relowner) AS rolname, CASE WHEN relkind='r' THEN 'table' WHEN relkind='i' THEN 'index' WHEN relkind='S' THEN 'sequence' WHEN relkind='v' THEN 'view' WHEN relkind='c' THEN 'composite type' WHEN relkind='t' THEN 'TOAST table' ELSE '' END AS kind, COUNT(*) AS total FROM pg_class GROUP BY 1, 2 ORDER BY 1, 2;										
유저 Object 사이즈 확인	SELECT pg_get_userbyid(relowner) AS rolname, CASE WHEN relkind='r' THEN 'table' WHEN relkind='i' THEN 'index' WHEN relkind='S' THEN 'sequence' WHEN relkind='t' THEN 'TOAST table' ELSE '' END AS kind, pg_size_pretty(SUM(pg_relation_size(pg_class.oid))::int8) AS size FROM pg_class WHERE relkind IN ('r', 't', 'i', 'S') GROUP BY 1, 2 ORDER BY 1, 2;										
데이터베이스 / Role 셋팅	SELECT db.datname, r.rolname, setconfig FROM pg_db_role_setting LEFT JOIN pg_database db ON db.oid=setdatabase LEFT JOIN pg_roles r ON r.oid=setrole;										
테이블스페이스 사이즈 확인	SELECT spcname, pg_get_userbyid(spcowner) AS owner, CASE WHEN length(pg_tablespace_location(oid)) = 0 THEN (SELECT setting FROM pg_settings WHERE name='data_directory') ELSE pg_tablespace_location(oid) END AS spclocation, spcacl, spcoptions , pg_size_pretty(pg_tablespace_size(spcname)) AS size FROM pg_tablespace ORDER BY spcname;										
테이블스페이스 Objects 갯수확인	SELECT pg_get_userbyid(spcowner) AS rolname, spcname, CASE WHEN relkind='r' THEN 'Tables' ELSE 'Index' END AS kind, count(*) AS total FROM pg_class, pg_tablespace WHERE pg_tablespace.oid=reltablespace AND relkind IN ('r', 'i') GROUP BY 1, 2, 3 ORDER BY 1, 2, 3;;										
대용량 Objects 확인	SELECT loid, pg_get_userbyid(lomowner) AS owner, lomacl, totalblocks FROM pg_largeobject_metadata md, (SELECT loid, count(*) AS totalblocks FROM pg_largeobject GROUP BY 1 ORDER BY 1) pg_largeobject WHERE md.oid=loid ORDER BY 1;										
WAL 파일 확인	SELECT * FROM pg_ls_dir('pg_xlog') WHERE pg_ls_dir ~ E'^[0-9A-F]{24}$' ORDER BY 1;
SELECT size, access, modification, change, creation, isdir FROM pg_stat_file('pg_xlog/000000010000000000000002');
SELECT size, access, modification, change, creation, isdir FROM pg_stat_file('pg_xlog/000000010000000000000003');										
스키마 확인	SELECT nspname, pg_get_userbyid(nspowner) AS owner, nspacl FROM pg_namespace ORDER BY nspname;										
기본 ACL 확인	SELECT r.rolname, nsp.nspname, defaclobjtype, defaclacl FROM pg_default_acl LEFT JOIN pg_namespace nsp ON nsp.oid=defaclnamespace LEFT JOIN pg_roles r ON r.oid=defaclrole;										
테이블 정보확인	SELECT relname, nspname AS schema, pg_get_userbyid(relowner) AS owner, reloftype, relam, relfilenode, (select spcname from pg_tablespace where oid=reltablespace) as tablespace, relpages, reltuples, relallvisible, reltoastrelid, relhasindex, relisshared, relkind, relnatts, relchecks, CASE WHEN relpersistence='t' THEN 'Temporary' ELSE 'Unknown' END AS relpersistence, relhasoids, relhaspkey, relhasrules, relhassubclass, relfrozenxid, relacl, reloptions, pg_size_pretty(pg_relation_size(pg_class.oid)) AS size FROM pg_class, pg_namespace WHERE relkind = 'r' AND relnamespace = pg_namespace.oid ORDER BY relname;										
캐시 된 테이블 정보확인	SELECT nspname, relname, size, buffers, buffers * 8192 AS buffer_size, case when buffers = 0 then 0 else round(buffers::numeric*8192/size::numeric*100,2) end AS "Cache %" FROM (SELECT n.nspname, c.relname, pg_relation_size(c.oid) AS size, count(*) AS buffers FROM pg_buffercache b, pg_class c, pg_namespace n WHERE b.relfilenode = c.relfilenode AND c.relnamespace = n.oid AND c.relkind = 'r' AND b.reldatabase IN (0, (SELECT oid FROM pg_database WHERE datname = current_database())) GROUP BY n.nspname, c.relname, size) ORDER BY 3 DESC;										
테이블의 oid, 스키마 확인	SELECT pg_class.oid, nspname, relname FROM pg_class, pg_namespace WHERE relkind IN ('r', 't') AND relnamespace = pg_namespace.oid ORDER BY relname;										
PK가 없는 테이블 정보확인	SELECT pg_get_userbyid(relowner) AS owner, nspname, relname, pg_size_pretty(pg_relation_size(pg_class.oid)) AS size FROM pg_class, pg_namespace WHERE relkind='r' AND relhaspkey IS false AND relnamespace = pg_namespace.oid ORDER BY relowner, relname;										
인덱스가 5개 이상인 테이블 확인	SELECT schemaname, tablename, count(*) as total FROM pg_indexes GROUP BY 1, 2 HAVING count(*)>=5 ORDER BY 1, 2;										
클러스터된 테이블 확인	SELECT s.relname, s.indexrelname, a.attname, p.correlation FROM pg_stats p, pg_stat_user_indexes s, pg_index i, pg_attribute a WHERE i.indisclustered AND s.indexrelid = i.indexrelid AND p.tablename = s.relname AND a.attnum = ANY (indkey) AND a.attrelid = i.indrelid AND p.attname = a.attname ORDER BY 1, 2, 3;										
FK 제약사항 정보확인	SELECT pg_get_userbyid(relowner) AS tableowner, nspname, relname AS tablename, conname, pg_get_constraintdef(pg_constraint.oid, true) as condef FROM pg_constraint, pg_class, pg_namespace WHERE conrelid=pg_class.oid AND relnamespace=pg_namespace.oid AND contype = 'f' ORDER BY 1, 2, 3;										
뷰 정보확인	SELECT relname, nspname AS schema, pg_get_userbyid(relowner) AS owner, relam, relfilenode, (select spcname from pg_tablespace where oid=reltablespace) as tablespace, relpages, reltuples, reltoastrelid, relhasindex, relisshared, relkind, relnatts, relchecks, relhasoids, relhaspkey, relhasrules, relhassubclass, relfrozenxid, relacl, reloptions, pg_size_pretty(pg_relation_size(pg_class.oid)) AS size FROM pg_class, pg_namespace WHERE relkind = 'v' AND relnamespace = pg_namespace.oid ORDER BY relname;										
시퀀스 정보확인	SELECT relname, nspname AS schema, pg_get_userbyid(relowner) AS owner, relam, relfilenode, (select spcname from pg_tablespace where oid=reltablespace) as tablespace, relpages, reltuples, reltoastrelid, relhasindex, relisshared, relkind, relnatts, relchecks, CASE WHEN relpersistence='t' THEN 'Temporary' ELSE 'Unknown' END AS relpersistence, relhasoids, relhaspkey, relhasrules, relhassubclass, relfrozenxid, relacl, reloptions, pg_size_pretty(pg_relation_size(pg_class.oid)) AS size FROM pg_class, pg_namespace WHERE relkind = 'S' AND relnamespace = pg_namespace.oid ORDER BY relname;										
시퀀스 메타데이터 정보확인	SELECT 'sys' AS schema, sequence_name, last_value, start_value, increment_by, max_value, min_value, cache_value, log_cnt, is_cycled, is_called FROM "sys"."plsql_profiler_runid" UNION SELECT 'sys' AS schema, sequence_name, last_value, start_value, increment_by, max_value, min_value, cache_value, log_cnt, is_cycled, is_called FROM "sys"."snapshot_num_seq" ORDER BY sequence_name;										
인덱스 정보확인	SELECT relname, nspname AS schema, pg_get_userbyid(relowner) AS owner, relam, relfilenode, (select spcname from pg_tablespace where oid=reltablespace) as tablespace, relpages, reltuples, reltoastrelid, relhasindex, relisshared, relkind, relnatts, relchecks, CASE WHEN relpersistence='t' THEN 'Temporary' ELSE 'Unknown' END AS relpersistence, relhasoids, relhaspkey, relhasrules, relhassubclass, relfrozenxid, relacl, reloptions, pg_size_pretty(pg_relation_size(pg_class.oid)) AS size FROM pg_class, pg_namespace WHERE relkind = 'i' AND relnamespace = pg_namespace.oid ORDER BY relname;										
테이블보다 큰 인덱스 확인	SELECT n.nspname AS schemaname, c.relname AS tablerelname, i.relname AS indexrelname, pg_size_pretty(pg_relation_size(c.oid)) AS tablesize, pg_size_pretty(pg_relation_size(i.oid)) AS indexsize FROM pg_class c JOIN pg_index x ON c.oid = x.indrelid JOIN pg_class i ON i.oid = x.indexrelid LEFT JOIN pg_namespace n ON n.oid = c.relnamespace WHERE c.relkind IN ('r', 't') AND pg_relation_size(c.oid) < pg_relation_size(i.oid) AND pg_relation_size(c.oid) > 1048576 ORDER BY 1, 2, 3;										
쓸모없는 인덱스 확인	SELECT idstat.schemaname AS schema_name, idstat.relname AS table_name, indexrelname AS index_name, idstat.idx_scan AS times_used, pg_size_pretty(pg_relation_size(idstat.relid)) AS table_size, pg_size_pretty(pg_relation_size(indexrelid)) AS index_size, n_tup_upd + n_tup_ins + n_tup_del as num_writes, indexdef AS definition FROM pg_stat_user_indexes AS idstat JOIN pg_indexes ON indexrelname = indexname JOIN pg_stat_user_tables AS tabstat ON idstat.relname = tabstat.relname WHERE idstat.idx_scan < 200 AND indexdef !~* 'unique' AND pg_relation_size(idstat.indexrelid) > 1048576 ORDER BY idstat.relname, indexrelname;										
Relations Bloat	SELECT schemaname, tablename, reltuples::bigint, relpages::bigint, otta, ROUND(CASE WHEN otta=0 THEN 0.0 ELSE sml.relpages/otta::numeric END,1) AS tbloat, relpages::bigint - otta AS wastedpages, bs*(sml.relpages-otta)::bigint AS wastedbytes, pg_size_pretty((bs*(relpages-otta))::bigint) AS wastedsize, iname, ituples::bigint, ipages::bigint, iotta, ROUND(CASE WHEN iotta=0 OR ipages=0 THEN 0.0 ELSE ipages/iotta::numeric END,1) AS ibloat, CASE WHEN ipages < iotta THEN 0 ELSE ipages::bigint - iotta END AS wastedipages, CASE WHEN ipages < iotta THEN 0 ELSE bs*(ipages-iotta) END AS wastedibytes, CASE WHEN ipages < iotta THEN pg_size_pretty(0::bigint) ELSE pg_size_pretty((bs*(ipages-iotta))::bigint) END AS wastedisize FROM ( SELECT schemaname, tablename, cc.reltuples, cc.relpages, bs, CEIL((cc.reltuples*((datahdr+ma- (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::float)) AS otta, COALESCE(c2.relname,'?') AS iname, COALESCE(c2.reltuples,0) AS ituples, COALESCE(c2.relpages,0) AS ipages, COALESCE(CEIL((c2.reltuples*(datahdr-12))/(bs-20::float)),0) AS iotta FROM ( SELECT ma,bs,schemaname,tablename, (datawidth+(hdr+ma-(case when hdr%ma=0 THEN ma ELSE hdr%ma END)))::numeric AS datahdr, (maxfracsum*(nullhdr+ma-(case when nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2 FROM ( SELECT schemaname, tablename, hdr, ma, bs, SUM((1-null_frac)*avg_width) AS datawidth, MAX(null_frac) AS maxfracsum, hdr+( SELECT 1+count(*)/8 FROM pg_stats s2 WHERE null_frac<>0 AND s2.schemaname = s.schemaname AND s2.tablename = s.tablename ) AS nullhdr FROM pg_stats s, ( SELECT (SELECT current_setting('block_size')::numeric) AS bs, CASE WHEN substring(v,12,3) IN ('8.0','8.1','8.2') THEN 27 ELSE 23 END AS hdr, CASE WHEN v ~ 'mingw32' THEN 8 ELSE 4 END AS ma FROM (SELECT version() AS v) AS foo ) AS constants GROUP BY 1,2,3,4,5 ) AS foo ) AS rs JOIN pg_class cc ON cc.relname = rs.tablename JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname = rs.schemaname LEFT JOIN pg_index i ON indrelid = cc.oid LEFT JOIN pg_class c2 ON c2.oid = i.indexrelid ) AS sml WHERE sml.relpages - otta > 0 OR ipages - iotta > 10 ORDER BY wastedbytes DESC, wastedibytes DESC;										
함수 확인	SELECT n.nspname, p.proname, CASE WHEN p.proretset THEN 'setof ' ELSE '' END || pg_catalog.format_type(p.prorettype, NULL) as returntype, CASE WHEN proallargtypes IS NOT NULL THEN pg_catalog.array_to_string(ARRAY( SELECT CASE WHEN p.proargmodes[s.i] = 'i' THEN '' WHEN p.proargmodes[s.i] = 'o' THEN 'OUT ' WHEN p.proargmodes[s.i] = 'b' THEN 'INOUT ' END || CASE WHEN COALESCE(p.proargnames[s.i], '') = '' THEN '' ELSE p.proargnames[s.i] || ' ' END || pg_catalog.format_type(p.proallargtypes[s.i], NULL) FROM pg_catalog.generate_series(1, pg_catalog.array_upper(p.proallargtypes, 1)) AS s(i) ), ', ') ELSE pg_catalog.array_to_string(ARRAY( SELECT CASE WHEN COALESCE(p.proargnames[s.i+1], '') = '' THEN '' ELSE p.proargnames[s.i+1] || ' ' END || pg_catalog.format_type(p.proargtypes[s.i], NULL) FROM pg_catalog.generate_series(0, pg_catalog.array_upper(p.proargtypes, 1)) AS s(i) ), ', ') END AS args, CASE WHEN p.provolatile = 'i' THEN 'immutable' WHEN p.provolatile = 's' THEN 'stable' WHEN p.provolatile = 'v' THEN 'volatile' END as volatility, pg_get_userbyid(proowner) AS rolname, proleakproof, l.lanname FROM pg_catalog.pg_proc p LEFT JOIN pg_catalog.pg_namespace n ON n.oid = p.pronamespace LEFT JOIN pg_catalog.pg_language l ON l.oid = p.prolang WHERE p.prorettype <> 'pg_catalog.cstring'::pg_catalog.regtype AND (p.proargtypes[0] IS NULL OR p.proargtypes[0] <> 'pg_catalog.cstring'::pg_catalog.regtype) AND NOT p.proisagg ORDER BY 1, 2, 3, 4;										
언어 확인	SELECT lanname, lanispl, lanpltrusted, lanacl FROM pg_language ORDER BY lanname;										
Extensions 확인	SELECT e.name, e.default_version, x.extversion AS installed_version, r.rolname as owner, n.nspname as namespace, x.extrelocatable, x.extconfig, x.extcondition, e.comment FROM pg_available_extensions() e(name, default_version, comment) LEFT JOIN pg_extension x ON e.name = x.extname LEFT JOIN pg_roles r ON r.oid=x.extowner LEFT JOIN pg_namespace n ON n.oid=x.extnamespace ORDER BY 1;										
세션 확인	SELECT name, setting FROM pg_settings WHERE name IN ('max_connections', 'autovacuum_max_workers') UNION ALL SELECT 'actual sessions', COUNT(*)::text FROM pg_stat_activity;										
전체 프로세스 확인	SELECT datname, pid, usename, query, date_trunc('second', query_start) as query_start, client_hostname, client_addr, waiting, date_trunc('second', xact_start) as xact_start, date_trunc('second', backend_start) as backend_start, application_name FROM pg_stat_activity ORDER BY datname, pid;										
Active 세션 확인	SELECT extract(epoch FROM (now() - query_start))::numeric(10,2) AS age, pid, usename, client_addr, application_name, query FROM pg_stat_activity WHERE query <> '' ORDER BY 1										
Replication 프로세스 확인	SELECT pid, usename, application_name, client_addr, client_hostname, client_port, date_trunc('second', backend_start) as backend_start, state, sent_location, write_location, flush_location, replay_location, sync_priority, sync_state FROM pg_stat_replication ORDER BY application_name, pid;										
커서 정보확인	SELECT name, statement, is_holdable, is_binary, is_scrollable, creation_time FROM pg_cursors ORDER BY name;										
준비된 statements	SELECT name, statement, prepare_time, from_sql FROM pg_prepared_statements ORDER BY name;										
준비된 트렌젝션	SELECT transaction, gid, prepared, owner, database FROM pg_prepared_xacts ORDER BY owner, database;										
전체 락 정보확인	SELECT locktype, CASE WHEN datname IS NOT NULL THEN datname ELSE database::text END AS database, nspname, relname, page, tuple, virtualxid, transactionid, classid, objid, objsubid, virtualtransaction, pid, mode, fastpath, granted FROM pg_locks LEFT JOIN pg_database ON pg_database.oid = database LEFT JOIN pg_class ON pg_class.oid = relation LEFT JOIN pg_namespace ON pg_namespace.oid=pg_class.relnamespace;										
Exclusive 락 정보확인	SELECT locktype, CASE WHEN datname IS NOT NULL THEN datname ELSE database::text END AS database, nspname, relname, page, tuple, virtualxid, transactionid, classid, objid, objsubid, virtualtransaction, pid, granted FROM pg_locks LEFT JOIN pg_database ON pg_database.oid = database LEFT JOIN pg_class ON pg_class.oid = relation LEFT JOIN pg_namespace ON pg_namespace.oid=pg_class.relnamespace WHERE mode='ExclusiveLock' AND locktype NOT IN ('virtualxid', 'transactionid');										
bgwriter 정보확인	SELECT checkpoints_timed, checkpoints_req, buffers_checkpoint, buffers_clean, maxwritten_clean, buffers_backend, buffers_backend_fsync, buffers_alloc, date_trunc('second', stats_reset) as stats_reset FROM pg_stat_bgwriter;										
캐시 적중률	SELECT datname, blks_read, blks_hit, round((blks_hit::float/(blks_read+blks_hit+1)*100)::numeric, 2) as cachehitratio FROM pg_stat_database ORDER BY datname, cachehitratio;										
데이터베이스 통계	SELECT datname, numbackends, xact_commit, xact_rollback, blks_read, blks_hit, tup_returned, tup_fetched, tup_inserted, tup_updated, tup_deleted, conflicts, date_trunc('second', stats_reset) as stats_reset FROM pg_stat_database ORDER BY datname;										
데이터베이스 충돌 정보	SELECT datname, confl_tablespace, confl_lock, confl_snapshot, confl_bufferpin, confl_deadlock FROM pg_stat_database_conflicts ORDER BY datname;										
테이블 통계	SELECT schemaname, relname, seq_scan, seq_tup_read, idx_scan, idx_tup_fetch, n_tup_ins, n_tup_upd, n_tup_del, n_tup_hot_upd, n_live_tup, n_dead_tup, last_vacuum, last_autovacuum, last_analyze, last_autoanalyze, vacuum_count, autovacuum_count, analyze_count, autoanalyze_count FROM pg_stat_all_tables ORDER BY schemaname, relname;										
테이블 IO 통계	SELECT schemaname, relname, heap_blks_read, heap_blks_hit, idx_blks_read, idx_blks_hit, toast_blks_read, toast_blks_hit, tidx_blks_read, tidx_blks_hit FROM pg_statio_all_tables ORDER BY schemaname, relname;										
마지막으로 청소된 테이블확인	SELECT schemaname, relname, max(greatest(last_vacuum, last_autovacuum)) as lastvac FROM pg_stat_all_tables WHERE last_vacuum IS NOT NULL OR last_autovacuum IS NOT NULL GROUP BY 1, 2 ORDER BY 3 DESC;										
마지막으로 분석된 테이블확인	SELECT schemaname, relname, max(greatest(last_analyze, last_autoanalyze)) as lastanalyze FROM pg_stat_all_tables WHERE last_analyze IS NOT NULL OR last_autoanalyze IS NOT NULL GROUP BY 1, 2 ORDER BY 3 DESC;										
인덱스 통계	SELECT schemaname, relname, indexrelname, idx_scan, idx_tup_read, idx_tup_fetch FROM pg_stat_all_indexes ORDER BY schemaname, relname;										
인덱스 IO 통계	SELECT schemaname, relname, blks_read, blks_hit FROM pg_statio_all_sequences ORDER BY schemaname, relname;										
시퀀스 IO 통계	SELECT schemaname, relname, blks_read, blks_hit FROM pg_statio_all_sequences ORDER BY schemaname, relname;										
함수 상태 확인	SELECT schemaname, funcname, calls, total_time, self_time FROM pg_stat_user_functions ORDER BY schemaname, funcname;										