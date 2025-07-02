WARNING: unable to locate formfactor file - /u01/app/oracle/oem_agent/agent_13.5.0.0.0/agent_13.5.0.0.0/eons/conf/.formfactor
java.lang.Exception: Stack trace
        at java.lang.Thread.dumpStack(Thread.java:1336)
        at oracle.sysman.gcagent.tmmain.diag.DiagManager.handleOutOfMemError(DiagManager.java:1732)
        at oracle.sysman.gcagent.tmmain.diag.DiagManager.handleException(DiagManager.java:1006)
        at oracle.sysman.gcagent.tmmain.diag.DiagManager.handleException(DiagManager.java:969)
        at oracle.sysman.gcagent.tmmain.diag.DiagManager.handleException(DiagManager.java:741)
        at oracle.sysman.gcagent.tmmain.diag.DiagManager$1.uncaughtException(DiagManager.java:616)
        at oracle.sysman.gcagent.util.system.ThreadManager$ThreadUncaughtExceptionHandler.uncaughtException(ThreadManager.java:558)
        at java.lang.Thread.dispatchUncaughtException(Thread.java:1959)
Agent is going down due to an OutOfMemoryError
----- 2025-07-02 15:25:04,063::297216::Checking status of EMAgent : 151968 -----
----- 2025-07-02 15:25:04,063::297216::EMAgent exited at 2025-07-02 15:25:04,063 with return value 57. -----
----- 2025-07-02 15:25:04,063::297216::EMAgent will be restarted because of an Out of Memory Exception. -----
----- 2025-07-02 15:25:04,063::297216::writeAbnormalExitTimestampToAgntStmp: exitCause=OOM : restartRequired=1 -----
----- 2025-07-02 15:25:04,064::297216::Restarting EMAgent. -----
----- 2025-07-02 15:25:04,528::297216::Auto tuning the agent at time 2025-07-02 15:25:04,528 -----
_OOMEThresholdPctIncr=-1006632
agentJavaDefines=-Xmx319607488M -XX:MaxMetaspaceSize=256M
UploadMaxNumberXML=5000
MaxInComingConnections=128
propComputeParallelization=96
inMemoryLoggingSize=14680064
MaxThreads=12
UploadMaxMegaBytesXML=50.0
SchedulerRandomSpreadMins=5
_SchedulePersistTimer=30
Auto tuning was successful
----- 2025-07-02 15:25:07,986::297216::Finished auto tuning the agent at time 2025-07-02 15:25:07,986 -----
----- 2025-07-02 15:25:07,988::297216::Launching the JVM with following options: -Xmx1524M -XX:MaxMetaspaceSize=224M -server -Djava.security.egd=file:///dev/./urandom -Dsun.lang.ClassLoader.allowArraySyntax=true -XX:-UseLargePages -XX:+UseLinuxPosixThreadCPUClocks -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+UseCompressedOops -DHTTPClient.dontSeekTerminatingChunk=true -----
----- 2025-07-02 15:25:07,989::297216::Agent Launched with PID 322969 at time 2025-07-02 15:25:07,989 -----
----- 2025-07-02 15:25:07,990::322969::Execing EMAgent process is taking longer than expected 120 secs -----
----- 2025-07-02 15:25:07,990::322969::Time elapsed between Launch of Watchdog process and execing EMAgent is 5359352 secs -----
EONSPROVIDER: oracle.eons.proxy.impl.ONSFactoryImpl
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
Jul 02, 2025 3:25:43 PM oracle.eons.proxy.impl.ConnectionManagerImpl readFormFactor
WARNING: unable to locate formfactor file - /u01/app/oracle/oem_agent/agent_13.5.0.0.0/agent_13.5.0.0.0/eons/conf/.formfactor
