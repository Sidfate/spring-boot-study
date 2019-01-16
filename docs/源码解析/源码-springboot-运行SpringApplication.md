# 源码 - Spring Boot - 运行SpringApplication

public ConfigurableApplicationContext run(String... args) {
    		// 1、创建并启动计时监控类
    		StopWatch stopWatch = new StopWatch();
    		stopWatch.start();
    		// 2、初始化应用上下文和异常报告集合
    		ConfigurableApplicationContext context = null;
    		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    		// 3、设置系统属性 `java.awt.headless` 的值，默认值为：true
    		configureHeadlessProperty();
    		// 4、创建所有 Spring 运行监听器并发布应用启动事件
    		SpringApplicationRunListeners listeners = getRunListeners(args);
    		listeners.starting();
    		try {
    			// 5、初始化默认应用参数类
    			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
    					args);
    			// 6、根据运行监听器和应用参数来准备 Spring 环境
    			ConfigurableEnvironment environment = prepareEnvironment(listeners,
    					applicationArguments);
    			configureIgnoreBeanInfo(environment);
    			// 7、创建 Banner 打印类
    			Banner printedBanner = printBanner(environment);
    			// 8、创建应用上下文
    			context = createApplicationContext();
    			// 9、准备异常报告器
    			exceptionReporters = getSpringFactoriesInstances(
    					SpringBootExceptionReporter.class,
    					new Class[] { ConfigurableApplicationContext.class }, context);
    			// 10、准备应用上下文
    			prepareContext(context, environment, listeners, applicationArguments,
    					printedBanner);
    			// 11、刷新应用上下文
    			refreshContext(context);
    			// 12、应用上下文刷新后置处理
    			afterRefresh(context, applicationArguments);
    			// 13、停止计时监控类
    			stopWatch.stop();
    			// 14、输出日志记录执行主类名、时间信息
    			if (this.logStartupInfo) {
    				new StartupInfoLogger(this.mainApplicationClass)
    						.logStarted(getApplicationLog(), stopWatch);
    			}
    			// 15、发布应用上下文启动完成事件
    			listeners.started(context);
    			// 16、执行所有 Runner 运行器
    			callRunners(context, applicationArguments);
    		}
    		catch (Throwable ex) {
    			handleRunFailure(context, listeners, exceptionReporters, ex);
    			throw new IllegalStateException(ex);
    		}
    		// 17、发布应用上下文就绪事件
    		listeners.running(context);
    		// 18、返回应用上下文
    		return context;
    	}