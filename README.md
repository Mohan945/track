SET LINESIZE 200
SELECT shared_pool_size_for_estimate AS "Estimated Size (MB)",
       estd_lc_size AS "Est LC Size",
       estd_lc_memory_object_hits AS "Hits",
       estd_lc_time_saved AS "Time Saved (ms)"
FROM v$shared_pool_advice
ORDER BY shared_pool_size_for_estimate;
