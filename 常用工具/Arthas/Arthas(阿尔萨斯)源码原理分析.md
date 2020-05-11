- [官方介绍:](#-----)
- [基本原理](#----)
- [源码分析主要内容](#--------)
  * [启动代码](#----)
    + [代码位置](#----)
  * [服务端代码](#-----)

## 官方介绍:

> Arthas 是Alibaba开源的Java诊断工具，深受开发者喜爱。在线排查问题，无需重启；动态跟踪Java代码；实时监控JVM状态。
>
> 官方文档地址：https://alibaba.github.io/arthas/
>
> GitHub地址：https://github.com/alibaba/arthas/

## 基本原理

试用一下以后就被深深吸引了,以前只知道使用Debug(原理是使用了JPDA技术,[参见官方文档](https://docs.oracle.com/javase/8/docs/technotes/guides/jpda/index.html))可以使用远程调试代码,生产不开JPDA端口则无可奈何.

而arthas直接使用了Java Agent技术.

> SE 5 的时候使用java.lang.instrucment可以在启动时加载Agent对JVM底层组件进行访问
>
> 在SE 6 之后则可以通过进程间通讯的方法,先VirtualMachine.attach(pid),然后动态加载自定义的agent代理virtualMachine.loadAgent.[参考文档](https://www.cnblogs.com/LittleHann/p/4783581.html)

## 源码分析主要内容

- 启动代码
- Agent分析
- 命令执行过程

### 启动代码

3.0.5之前的版本推荐的是脚本启动方式as.sh.

3.1.0开始推荐使用arthas-boot启动.

```java
wget https://alibaba.github.io/arthas/arthas-boot.jar
java -jar arthas-boot.jar
```

#### 代码位置



```css
com.taobao.arthas.boot.Bootstrap
```

1. 使用了阿里开源的组件[cli](https://github.com/alibaba/cli),对参数进行了解析.
    **代码块**



```java
        Bootstrap bootstrap = new Bootstrap();
        CLI cli = CLIConfigurator.define(Bootstrap.class);
        CommandLine commandLine = cli.parse(Arrays.asList(args));
        try {
            CLIConfigurator.inject(commandLine, bootstrap);
        } catch (Throwable e) {
            e.printStackTrace();
            System.out.println(usage(cli));
            System.exit(1);
        }
```

1. 对传入的参数进行处理,如调整日志级别,设置RepoMirror地址,Java版本,telnet/http的端口检查
    **代码块**



```cpp
//非重要代码,详见com.taobao.arthas.boot.Bootstrap.main()方法
```

1. 如果在传入参数中没有pid,则会调用本地jps命令,列出java进程(当然会排除本身)
    **代码块**



```dart
    private static Map<Integer, String> listProcessByJps(boolean v) {
        Map<Integer, String> result = new LinkedHashMap<Integer, String>();

        String jps = "jps";
        File jpsFile = findJps();
        if (jpsFile != null) {
            jps = jpsFile.getAbsolutePath();
        }

        AnsiLog.debug("Try use jps to lis java process, jps: " + jps);

        String[] command = null;
        if (v) {
            command = new String[] { jps, "-v", "-l" };
        } else {
            command = new String[] { jps, "-l" };
        }

        List<String> lines = ExecutingCommand.runNative(command);

        int currentPid = Integer.parseInt(PidUtils.currentPid());
        for (String line : lines) {
            String[] strings = line.trim().split("\\s+");
            if (strings.length < 1) {
                continue;
            }
            int pid = Integer.parseInt(strings[0]);
            if (pid == currentPid) {
                continue;
            }
            if (strings.length >= 2 && isJspProcess(strings[1])) { // skip jps
                continue;
            }

            result.put(pid, line);
        }

        return result;
    }
```

1. 进入主逻辑,会在用户目录下建立.arthas目录,同时下载arthas-core和arthas-agent等lib文件,然后启动客户端和服务端.启动命令如下:
    **代码块**



```csharp
        if (telnetPortPid > 0 && pid == telnetPortPid) {
            AnsiLog.info("The target process already listen port {}, skip attach.", bootstrap.getTelnetPort());
        } else {
            // start arthas-core.jar
            List<String> attachArgs = new ArrayList<String>();
            attachArgs.add("-jar");
            attachArgs.add(new File(arthasHomeDir, "arthas-core.jar").getAbsolutePath());
            attachArgs.add("-pid");
            attachArgs.add("" + pid);
            attachArgs.add("-target-ip");
            attachArgs.add(bootstrap.getTargetIp());
            attachArgs.add("-telnet-port");
            attachArgs.add("" + bootstrap.getTelnetPort());
            attachArgs.add("-http-port");
            attachArgs.add("" + bootstrap.getHttpPort());
            attachArgs.add("-core");
            attachArgs.add(new File(arthasHomeDir, "arthas-core.jar").getAbsolutePath());
            attachArgs.add("-agent");
            attachArgs.add(new File(arthasHomeDir, "arthas-agent.jar").getAbsolutePath());
            if (bootstrap.getSessionTimeout() != null) {
                attachArgs.add("-session-timeout");
                attachArgs.add("" + bootstrap.getSessionTimeout());
            }

            AnsiLog.info("Try to attach process " + pid);
            AnsiLog.debug("Start arthas-core.jar args: " + attachArgs);
            // 启动服务端
            ProcessUtils.startArthasCore(pid, attachArgs);
            AnsiLog.info("Attach process {} success.", pid);
        }
```

1. 最后通过反射的方式来启动字符客户端,等待用户输入指令.
    **代码块**



```dart
URLClassLoader classLoader = new URLClassLoader(
                        new URL[] { new File(arthasHomeDir, "arthas-client.jar").toURI().toURL() });
Class<?> telnetConsoleClas = classLoader.loadClass("com.taobao.arthas.client.TelnetConsole");
Method mainMethod = telnetConsoleClas.getMethod("main", String[].class);
```

### 服务端代码

可以打开pom.xml文件，使用assembly进行打包的时候制定了manifest文件的一些信息,指定了mainClass为`com.taobao.arthas.core.Arthas`.

1. 使用`VirutalMachine.attach(pid)`来连接进程,同时使用`virtualMachine.loadAgent`加载自定义的agent.
    **代码块**



```java
    private void attachAgent(Configure configure) throws Exception {
       //省略部分代码
      //attach进程
      virtualMachine = VirtualMachine.attach("" + configure.getJavaPid());

       //省略部分代码
      //动态加载Agent
      virtualMachine.loadAgent(configure.getArthasAgent(),
                            configure.getArthasCore() + ";" + configure.toString());
    }
```

1. arthas-agent包指定了`Agent-Class`为`com.taobao.arthas.agent.AgentBootstrap`.
    **代码块**



```xml
<manifestEntries>
    <Premain-Class>com.taobao.arthas.agent.AgentBootstrap</Premain-Class>
    <Agent-Class>com.taobao.arthas.agent.AgentBootstrap</Agent-Class>
</manifestEntries>
```

1. `main()`方法中对于`arthas-spy`(简单理解为勾子类,类似于spring aop的前置方法,后置方法)进行了加载.
    **代码块**



```dart
            final ClassLoader agentLoader = getClassLoader(inst, spyJarFile, agentJarFile);
            initSpy(agentLoader);
```

1. 首先将spyJar添加到了BootstrapClassLoader(启动类加载器)，优先加载启动类加载器，spy可以在各个ClassLoader中使用.
    **代码块**



```java
    private static ClassLoader getClassLoader(Instrumentation inst, File spyJarFile, File agentJarFile) throws Throwable {
        // 将Spy添加到BootstrapClassLoader
        inst.appendToBootstrapClassLoaderSearch(new JarFile(spyJarFile));

        // 构造自定义的类加载器，尽量减少Arthas对现有工程的侵蚀
        return loadOrDefineClassLoader(agentJarFile);
    }
```

1. 同时加载`com.taobao.arthas.core.advisor.AdviceWeaver`类，并将里面的`methodOnBegin`、`methodOnReturnEnd`、`methodOnThrowingEnd`等方法取出赋值给Spy类对应的方法。
    **代码块**



```kotlin
    private static void initSpy(ClassLoader classLoader) throws ClassNotFoundException, NoSuchMethodException {
        Class<?> adviceWeaverClass = classLoader.loadClass(ADVICEWEAVER);
        Method onBefore = adviceWeaverClass.getMethod(ON_BEFORE, int.class, ClassLoader.class, String.class,
                String.class, String.class, Object.class, Object[].class);
        Method onReturn = adviceWeaverClass.getMethod(ON_RETURN, Object.class);
        Method onThrows = adviceWeaverClass.getMethod(ON_THROWS, Throwable.class);
        Method beforeInvoke = adviceWeaverClass.getMethod(BEFORE_INVOKE, int.class, String.class, String.class, String.class);
        Method afterInvoke = adviceWeaverClass.getMethod(AFTER_INVOKE, int.class, String.class, String.class, String.class);
        Method throwInvoke = adviceWeaverClass.getMethod(THROW_INVOKE, int.class, String.class, String.class, String.class);
        Method reset = AgentBootstrap.class.getMethod(RESET);
        Spy.initForAgentLauncher(classLoader, onBefore, onReturn, onThrows, beforeInvoke, afterInvoke, throwInvoke, reset);
    }
```

1. 异步调用bind()方法,启动服务端,监听端口,和客户端进行通讯.
    **代码块**



```java
            Thread bindingThread = new Thread() {
                @Override
                public void run() {
                    try {
                        bind(inst, agentLoader, agentArgs);
                    } catch (Throwable throwable) {
                        throwable.printStackTrace(ps);
                    }
                }
            };
```

1. 同时启动服务器监听,http和telnet的.
    ***代码块\***



```css
                shellServer.registerTermServer(new TelnetTermServer(configure.getIp(), configure.getTelnetPort(),
                                options.getConnectionTimeout()));
                shellServer.registerTermServer(new HttpTermServer(configure.getIp(), configure.getHttpPort(),
                                options.getConnectionTimeout()));
```

1. 服务器端用了一个阿里开源的命令行程序开发框架[termd](https://github.com/alibaba/termd)
    ***代码块\***



```kotlin
    public ShellServer listen(final Handler<Future<Void>> listenHandler) {
        final List<TermServer> toStart;
        synchronized (this) {
            if (!closed) {
                throw new IllegalStateException("Server listening");
            }
            toStart = termServers;
        }
        final AtomicInteger count = new AtomicInteger(toStart.size());
        if (count.get() == 0) {
            setClosed(false);
            listenHandler.handle(Future.<Void>succeededFuture());
            return this;
        }
        Handler<Future<TermServer>> handler = new TermServerListenHandler(this, listenHandler, toStart);
        for (TermServer termServer : toStart) {
            termServer.termHandler(new TermServerTermHandler(this));
            termServer.listen(handler);
        }
        return this;
    }
```



