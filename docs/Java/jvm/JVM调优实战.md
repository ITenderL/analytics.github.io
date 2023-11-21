# JVM调优实战

### 调优实战相关命令

#### `jps` Java processes 进程信息

- jps -l 进程信息

```bash
E:\workSpace\IdeaProjects\javastudy>jps -l
1648 org.jetbrains.jps.cmdline.Launcher
6480 sun.tools.jps.Jps
# 进程id 进程名称
2324 com.itender.juc.lock8.DeathLock
3524
8180 org.jetbrains.idea.maven.server.RemoteMavenServer36
```

#### `jinfo` Java information java的jvm信息

```bash
Attaching to process ID 2324, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.92-b14
Java System Properties:

java.runtime.name = Java(TM) SE Runtime Environment
java.vm.version = 25.92-b14
sun.boot.library.path = D:\Java\Jdk_1.8\jre\bin
java.vendor.url = http://java.oracle.com/
java.vm.vendor = Oracle Corporation
path.separator = ;
file.encoding.pkg = sun.io
java.vm.name = Java HotSpot(TM) 64-Bit Server VM
sun.os.patch.level =
sun.java.launcher = SUN_STANDARD
user.script =
user.country = CN
user.dir = E:\workSpace\IdeaProjects\javastudy
java.vm.specification.name = Java Virtual Machine Specification
java.runtime.version = 1.8.0_92-b14
java.awt.graphicsenv = sun.awt.Win32GraphicsEnvironment
os.arch = amd64
java.endorsed.dirs = D:\Java\Jdk_1.8\jre\lib\endorsed
line.separator =

java.io.tmpdir = C:\Users\YUANHE~1\AppData\Local\Temp\
java.vm.specification.vendor = Oracle Corporation
user.variant =
os.name = Windows 10
sun.jnu.encoding = GBK
java.library.path = D:\Java\Jdk_1.8\bin;C:\WINDOWS\Sun\Java\bin;C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem;C:\WINDOWS\S
ystem32\WindowsPowerShell\v1.0\;C:\WINDOWS\System32\OpenSSH\;D:\Java\Jdk_1.8\bin;D:\Java\Jdk_1.8\jre\bin;M2_HOME \bin;D:\Java\apache-maven-3.6.3\bin;D:\mysql-5.6.4
6-winx64\bin;D:\Java\Openssl\OpenSSL-Win64\bin;D:\Git\Git\cmd;D:\Java\MySQL\bin;D:\Java\apache-tomcat-8.5.24\bin;"D:\Java\erlang\erl9.3\erl9.3\bin;";D:\tortoiseGit
\bin;D:\nodejs\;D:\nodejs\node_global;D:\tools\LxRunOffline-v3.5.0-mingw;D:\tools\netcat-win32-1.12;C:\Users\yuanhewei\AppData\Local\Microsoft\WindowsApps;E:\Bandi
zip\;C:\Users\yuanhewei\AppData\Local\Programs\Fiddler;C:\Users\yuanhewei\AppData\Local\Microsoft\WindowsApps;C:\Users\yuanhewei\AppData\Roaming\npm;.
java.specification.name = Java Platform API Specification
java.class.version = 52.0
sun.management.compiler = HotSpot 64-Bit Tiered Compilers
os.version = 10.0
user.home = C:\Users\yuanhewei
user.timezone = Asia/Shanghai
java.awt.printerjob = sun.awt.windows.WPrinterJob
file.encoding = UTF-8
java.specification.version = 1.8
user.name = yuanhewei
java.class.path = D:\Java\Jdk_1.8\jre\lib\charsets.jar;D:\Java\Jdk_1.8\jre\lib\deploy.jar;D:\Java\Jdk_1.8\jre\lib\ext\access-bridge-64.jar;D:\Java\Jdk_1.8\jre\lib\
ext\cldrdata.jar;D:\Java\Jdk_1.8\jre\lib\ext\dnsns.jar;D:\Java\Jdk_1.8\jre\lib\ext\jaccess.jar;D:\Java\Jdk_1.8\jre\lib\ext\jfxrt.jar;D:\Java\Jdk_1.8\jre\lib\ext\lo
caledata.jar;D:\Java\Jdk_1.8\jre\lib\ext\nashorn.jar;D:\Java\Jdk_1.8\jre\lib\ext\sunec.jar;D:\Java\Jdk_1.8\jre\lib\ext\sunjce_provider.jar;D:\Java\Jdk_1.8\jre\lib\
ext\sunmscapi.jar;D:\Java\Jdk_1.8\jre\lib\ext\sunpkcs11.jar;D:\Java\Jdk_1.8\jre\lib\ext\zipfs.jar;D:\Java\Jdk_1.8\jre\lib\javaws.jar;D:\Java\Jdk_1.8\jre\lib\jce.ja
r;D:\Java\Jdk_1.8\jre\lib\jfr.jar;D:\Java\Jdk_1.8\jre\lib\jfxswt.jar;D:\Java\Jdk_1.8\jre\lib\jsse.jar;D:\Java\Jdk_1.8\jre\lib\management-agent.jar;D:\Java\Jdk_1.8\
jre\lib\plugin.jar;D:\Java\Jdk_1.8\jre\lib\resources.jar;D:\Java\Jdk_1.8\jre\lib\rt.jar;E:\workSpace\IdeaProjects\javastudy\jucthread\target\classes;D:\Java\maven_
repos\org\springframework\boot\spring-boot-starter-web\2.6.4\spring-boot-starter-web-2.6.4.jar;D:\Java\maven_repos\org\springframework\boot\spring-boot-starter\2.6
.4\spring-boot-starter-2.6.4.jar;D:\Java\maven_repos\org\springframework\boot\spring-boot\2.6.4\spring-boot-2.6.4.jar;D:\Java\maven_repos\org\springframework\boot\
spring-boot-autoconfigure\2.6.4\spring-boot-autoconfigure-2.6.4.jar;D:\Java\maven_repos\org\springframework\boot\spring-boot-starter-logging\2.6.4\spring-boot-star
ter-logging-2.6.4.jar;D:\Java\maven_repos\ch\qos\logback\logback-classic\1.2.10\logback-classic-1.2.10.jar;D:\Java\maven_repos\ch\qos\logback\logback-core\1.2.10\l
ogback-core-1.2.10.jar;D:\Java\maven_repos\org\apache\logging\log4j\log4j-to-slf4j\2.17.1\log4j-to-slf4j-2.17.1.jar;D:\Java\maven_repos\org\apache\logging\log4j\lo
g4j-api\2.17.1\log4j-api-2.17.1.jar;D:\Java\maven_repos\org\slf4j\jul-to-slf4j\1.7.36\jul-to-slf4j-1.7.36.jar;D:\Java\maven_repos\jakarta\annotation\jakarta.annota
tion-api\1.3.5\jakarta.annotation-api-1.3.5.jar;D:\Java\maven_repos\org\yaml\snakeyaml\1.29\snakeyaml-1.29.jar;D:\Java\maven_repos\org\springframework\boot\spring-
boot-starter-json\2.6.4\spring-boot-starter-json-2.6.4.jar;D:\Java\maven_repos\com\fasterxml\jackson\core\jackson-databind\2.13.1\jackson-databind-2.13.1.jar;D:\Ja
va\maven_repos\com\fasterxml\jackson\core\jackson-annotations\2.13.1\jackson-annotations-2.13.1.jar;D:\Java\maven_repos\com\fasterxml\jackson\core\jackson-core\2.1
3.1\jackson-core-2.13.1.jar;D:\Java\maven_repos\com\fasterxml\jackson\datatype\jackson-datatype-jdk8\2.13.1\jackson-datatype-jdk8-2.13.1.jar;D:\Java\maven_repos\co
m\fasterxml\jackson\datatype\jackson-datatype-jsr310\2.13.1\jackson-datatype-jsr310-2.13.1.jar;D:\Java\maven_repos\com\fasterxml\jackson\module\jackson-module-para
meter-names\2.13.1\jackson-module-parameter-names-2.13.1.jar;D:\Java\maven_repos\org\springframework\boot\spring-boot-starter-tomcat\2.6.4\spring-boot-starter-tomc
at-2.6.4.jar;D:\Java\maven_repos\org\apache\tomcat\embed\tomcat-embed-core\9.0.58\tomcat-embed-core-9.0.58.jar;D:\Java\maven_repos\org\apache\tomcat\embed\tomcat-e
mbed-el\9.0.58\tomcat-embed-el-9.0.58.jar;D:\Java\maven_repos\org\apache\tomcat\embed\tomcat-embed-websocket\9.0.58\tomcat-embed-websocket-9.0.58.jar;D:\Java\maven
_repos\org\springframework\spring-web\5.3.16\spring-web-5.3.16.jar;D:\Java\maven_repos\org\springframework\spring-beans\5.3.16\spring-beans-5.3.16.jar;D:\Java\mave
n_repos\org\springframework\spring-webmvc\5.3.16\spring-webmvc-5.3.16.jar;D:\Java\maven_repos\org\springframework\spring-aop\5.3.16\spring-aop-5.3.16.jar;D:\Java\m
aven_repos\org\springframework\spring-context\5.3.16\spring-context-5.3.16.jar;D:\Java\maven_repos\org\springframework\spring-expression\5.3.16\spring-expression-5
.3.16.jar;D:\Java\maven_repos\org\slf4j\slf4j-api\1.7.36\slf4j-api-1.7.36.jar;D:\Java\maven_repos\org\springframework\spring-core\5.3.16\spring-core-5.3.16.jar;D:\
Java\maven_repos\org\springframework\spring-jcl\5.3.16\spring-jcl-5.3.16.jar;D:\Java\maven_repos\org\projectlombok\lombok\1.18.22\lombok-1.18.22.jar;D:\Java\maven_
repos\cn\hutool\hutool-all\4.1.14\hutool-all-4.1.14.jar;D:\Java\maven_repos\com\google\guava\guava\27.1-jre\guava-27.1-jre.jar;D:\Java\maven_repos\com\google\guava
\failureaccess\1.0.1\failureaccess-1.0.1.jar;D:\Java\maven_repos\com\google\guava\listenablefuture\9999.0-empty-to-avoid-conflict-with-guava\listenablefuture-9999.
0-empty-to-avoid-conflict-with-guava.jar;D:\Java\maven_repos\com\google\code\findbugs\jsr305\3.0.2\jsr305-3.0.2.jar;D:\Java\maven_repos\org\checkerframework\checke
r-qual\2.5.2\checker-qual-2.5.2.jar;D:\Java\maven_repos\com\google\errorprone\error_prone_annotations\2.2.0\error_prone_annotations-2.2.0.jar;D:\Java\maven_repos\c
om\google\j2objc\j2objc-annotations\1.1\j2objc-annotations-1.1.jar;D:\Java\maven_repos\org\codehaus\mojo\animal-sniffer-annotations\1.17\animal-sniffer-annotations
-1.17.jar;D:\Java\maven_repos\org\apache\commons\commons-lang3\3.4\commons-lang3-3.4.jar;D:\IDE\IntelliJ IDEA 2019.3.1\lib\idea_rt.jar
java.vm.specification.version = 1.8
sun.arch.data.model = 64
sun.java.command = com.itender.juc.lock8.DeathLock
java.home = D:\Java\Jdk_1.8\jre
user.language = zh
java.specification.vendor = Oracle Corporation
awt.toolkit = sun.awt.windows.WToolkit
java.vm.info = mixed mode
java.version = 1.8.0_92
java.ext.dirs = D:\Java\Jdk_1.8\jre\lib\ext;C:\WINDOWS\Sun\Java\lib\ext
sun.boot.class.path = D:\Java\Jdk_1.8\jre\lib\resources.jar;D:\Java\Jdk_1.8\jre\lib\rt.jar;D:\Java\Jdk_1.8\jre\lib\sunrsasign.jar;D:\Java\Jdk_1.8\jre\lib\jsse.jar;
D:\Java\Jdk_1.8\jre\lib\jce.jar;D:\Java\Jdk_1.8\jre\lib\charsets.jar;D:\Java\Jdk_1.8\jre\lib\jfr.jar;D:\Java\Jdk_1.8\jre\classes
java.vendor = Oracle Corporation
file.separator = \
java.vendor.url.bug = http://bugreport.sun.com/bugreport/
sun.io.unicode.encoding = UnicodeLittle
sun.cpu.endian = little
sun.desktop = windows
sun.cpu.isalist = amd64
# JVM配置信息
VM Flags:
Non-default VM flags: -XX:CICompilerCount=4 -XX:InitialHeapSize=266338304 -XX:MaxHeapSize=4246732800 -XX:MaxNewSize=1415577600 -XX:MinHeapDeltaBytes=524288 -XX:New
Size=88604672 -XX:OldSize=177733632 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:-UseLargePagesIndividualAllocation -
XX:+UseParallelGC
Command line:  -javaagent:D:\IDE\IntelliJ IDEA 2019.3.1\lib\idea_rt.jar=50373:D:\IDE\IntelliJ IDEA 2019.3.1\bin -Dfile.encoding=UTF-8

```

#### `jstat` Java statistics 统计信息

```bash
invalid argument count
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.

```

#### `jstack`

- jstack pid   pid进程下线程的信息  

```bash
2022-05-22 18:16:46
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.92-b14 mixed mode):

"DestroyJavaVM" #14 prio=5 os_prio=0 tid=0x000000001fb06800 nid=0x12dc waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Thread-1" #13 prio=5 os_prio=0 tid=0x000000001fb02000 nid=0x23b8 waiting for monitor entry [0x000000002053f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.itender.juc.lock8.MyThread.run(DeathLock.java:38)
        - waiting to lock <0x000000076bc04a98> (a java.lang.Object)
        - locked <0x000000076bc04aa8> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)

"Thread-0" #12 prio=5 os_prio=0 tid=0x000000001faf6800 nid=0x16c4 waiting for monitor entry [0x000000002043f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.itender.juc.lock8.MyThread.run(DeathLock.java:38)
        - waiting to lock <0x000000076bc04aa8> (a java.lang.Object)
        - locked <0x000000076bc04a98> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)

"Service Thread" #11 daemon prio=9 os_prio=0 tid=0x000000001f0d4800 nid=0x2820 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread3" #10 daemon prio=9 os_prio=2 tid=0x000000001f045000 nid=0xa34 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread2" #9 daemon prio=9 os_prio=2 tid=0x000000001f038000 nid=0x498 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #8 daemon prio=9 os_prio=2 tid=0x000000001f035800 nid=0x2a08 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #7 daemon prio=9 os_prio=2 tid=0x000000001f02f800 nid=0x1440 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Monitor Ctrl-Break" #6 daemon prio=5 os_prio=0 tid=0x000000001efe4000 nid=0x392c runnable [0x000000001f53e000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
        at java.net.SocketInputStream.read(SocketInputStream.java:170)
        at java.net.SocketInputStream.read(SocketInputStream.java:141)
        at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
        at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
        at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
        - locked <0x000000076bccac98> (a java.io.InputStreamReader)
        at java.io.InputStreamReader.read(InputStreamReader.java:184)
        at java.io.BufferedReader.fill(BufferedReader.java:161)
        at java.io.BufferedReader.readLine(BufferedReader.java:324)
        - locked <0x000000076bccac98> (a java.io.InputStreamReader)
        at java.io.BufferedReader.readLine(BufferedReader.java:389)
        at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:64)

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x000000001ed7b000 nid=0x2060 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x000000001edd0000 nid=0x1e38 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x000000001ed60800 nid=0xe40 in Object.wait() [0x000000001f23f000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x000000076ba08ee0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
        - locked <0x000000076ba08ee0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x000000001ce7d000 nid=0x2d80 in Object.wait() [0x000000001ed3e000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x000000076ba06b50> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x000000076ba06b50> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"VM Thread" os_prio=2 tid=0x000000001ce78800 nid=0x1c3c runnable

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x0000000003429800 nid=0x1004 runnable

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x000000000342b000 nid=0x29ac runnable

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x000000000342c800 nid=0x1fc4 runnable

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x000000000342e000 nid=0x2f8c runnable

"GC task thread#4 (ParallelGC)" os_prio=0 tid=0x0000000003430800 nid=0x26c8 runnable

"GC task thread#5 (ParallelGC)" os_prio=0 tid=0x0000000003431800 nid=0x2658 runnable

"GC task thread#6 (ParallelGC)" os_prio=0 tid=0x0000000003435800 nid=0x25c4 runnable

"GC task thread#7 (ParallelGC)" os_prio=0 tid=0x0000000003437000 nid=0x26d8 runnable

"VM Periodic Task Thread" os_prio=2 tid=0x000000001fa78800 nid=0x23f4 waiting on condition

JNI global references: 33


Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x000000001ce80b08 (object 0x000000076bc04a98, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x000000001ce83448 (object 0x000000076bc04aa8, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
        at com.itender.juc.lock8.MyThread.run(DeathLock.java:38)
        - waiting to lock <0x000000076bc04a98> (a java.lang.Object)
        - locked <0x000000076bc04aa8> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)
"Thread-0":
        at com.itender.juc.lock8.MyThread.run(DeathLock.java:38)
        - waiting to lock <0x000000076bc04aa8> (a java.lang.Object)
        - locked <0x000000076bc04a98> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)

Found 1 deadlock.
```

#### `top` Linux命令

- top -Hp pid   pid进程下所有的线程

```bash
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
    1 root      20   0     896    524    464 S   0.0   0.0   0:00.28 init
   10 root      20   0     896     80     20 S   0.0   0.0   0:00.00 init
   11 root      20   0     896     80     20 S   0.0   0.0   0:00.00 init
   12 itender   20   0   10040   5012   3304 S   0.0   0.0   0:00.03 bash
   77 itender   20   0   10876   3636   3124 R   0.0   0.0   0:00.00 top
```

#### `jmap`

- jmap -histo pid
- jmap -histo pid | head -20 展示前20个
- jmap -dump:format=b,file=20220522.hprof 2324  导出文件

```bash
E:\workSpace\IdeaProjects\javastudy>jmap -histo  2324

 num     #instances         #bytes  class name
----------------------------------------------
   1:           686        7284792  [I
   2:          2066        1577496  [B
   3:          9083        1159728  [C
   4:          6488         155712  java.lang.String
   5:           717          81936  java.lang.Class
   6:          1352          66104  [Ljava.lang.Object;
   7:           844          33760  java.util.TreeMap$Entry
   8:           714          28560  java.util.LinkedHashMap$Entry
   9:           455          22032  [Ljava.lang.String;
  10:           490          15680  java.io.File
  11:           463          14816  java.util.HashMap$Node
  12:            46          14080  [Ljava.util.HashMap$Node;
  13:           516          12384  java.lang.StringBuilder
  14:           189          12096  java.net.URL
  15:           152          10944  java.lang.reflect.Field
  16:           414           9936  java.lang.StringBuffer
  17:           217           8680  java.lang.ref.Finalizer
  18:           240           7680  java.util.Hashtable$Entry
  19:            55           5280  java.util.jar.JarFile$JarFileEntry
  20:           125           5000  java.lang.ref.SoftReference


```

#### `jvisualvm`



### `Arthas`





