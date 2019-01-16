# 源码 - Spring Boot - 初始化SpringApplication

从启动入口开始：

    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }

进入SpringApplication的run方法中：

    public static ConfigurableApplicationContext run(Class<?> primarySource,
    			String... args) {
    		return run(new Class<?>[] { primarySource }, args);
    }
    
    public static ConfigurableApplicationContext run(Class<?>[] primarySources,
    			String[] args) {
    		return new SpringApplication(primarySources).run(args);
    }

> ??为何要将我们的资源类转成数据的形式

可以看到接下来是实例化一个SpringApplication，然后在调用run方法。

一步步来看，先看SpringApplication的构造函数中做了什么：

    public SpringApplication(Class<?>... primarySources) {
    		this(null, primarySources);
    }
    
    @SuppressWarnings({ "unchecked", "rawtypes" })
    public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    		// 1. resourceLoader 为null
    		this.resourceLoader = resourceLoader;
    		Assert.notNull(primarySources, "PrimarySources must not be null");
    		// 2. 将资源类存放入集合
    		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    		// 3. ??推断web应用类型
    		this.webApplicationType = deduceWebApplicationType();
    		// 4. 初始化应用上下文
    		setInitializers((Collection) getSpringFactoriesInstances(
    				ApplicationContextInitializer.class));
    		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    		this.mainApplicationClass = deduceMainApplicationClass();
    }

> ??在推断应用类型的方法中ClassUtils.isPresent的含义

在第三个步骤中：

    private WebApplicationType deduceWebApplicationType() {
    		if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
    				&& !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)) {
    			return WebApplicationType.REACTIVE;
    		}
    		for (String className : WEB_ENVIRONMENT_CLASSES) {
    			if (!ClassUtils.isPresent(className, null)) {
    				return WebApplicationType.NONE;
    			}
    		}
    		return WebApplicationType.SERVLET;
    }

前面3个步骤都是填充SpringApplication中的一些属性，我们从第4个步骤开始看：

    private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
    		return getSpringFactoriesInstances(type, new Class<?>[] {});
    }
    
    private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
    		Class<?>[] parameterTypes, Object... args) {
    		//
    		ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
    		// Use names and ensure unique to protect against duplicates
    		Set<String> names = new LinkedHashSet<>(
    				SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    		List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
    				classLoader, args, names);
    		AnnotationAwareOrderComparator.sort(instances);
    		return instances;
    }

注意这里type的值是之前传入`ApplicationContextInitializer`

SpringFactoriesLoader.loadFactoryNames(type, classLoader) 中做了什么：

    public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
        String factoryClassName = factoryClass.getName();
        return (List)loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
    }
    
    private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
        // 
    		MultiValueMap<String, String> result = (MultiValueMap)cache.get(classLoader);
        if (result != null) {
            return result;
        } else {
            try {
                Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
                LinkedMultiValueMap result = new LinkedMultiValueMap();
    
                while(urls.hasMoreElements()) {
                    URL url = (URL)urls.nextElement();
                    UrlResource resource = new UrlResource(url);
                    Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                    Iterator var6 = properties.entrySet().iterator();
    
                    while(var6.hasNext()) {
                        Entry<?, ?> entry = (Entry)var6.next();
                        List<String> factoryClassNames = Arrays.asList(StringUtils.commaDelimitedListToStringArray((String)entry.getValue()));
                        result.addAll((String)entry.getKey(), factoryClassNames);
                    }
                }
    
                cache.put(classLoader, result);
                return result;
            } catch (IOException var9) {
                throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var9);
            }
        }
    }

根据类路径下的 META-INF/spring.factories 文件解析并获取 ApplicationContextInitializer 接口的所有配置的类路径名称。

可以看到最终获取到的 ApplicationContextInitializer 的实现：

1. org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer
2. org.springframework.boot.context.ContextIdApplicationContextInitializer
3. org.springframework.boot.context.config.DelegatingApplicationContextInitializer
4.  org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer
5.  org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer
6. org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

接下来根据名称的集合去实例化类：

    private <T> List<T> createSpringFactoriesInstances(Class<T> type,
    			Class<?>[] parameterTypes, ClassLoader classLoader, Object[] args,
    			Set<String> names) {
    		List<T> instances = new ArrayList<>(names.size());
    		for (String name : names) {
    			try {
    				Class<?> instanceClass = ClassUtils.forName(name, classLoader);
    				Assert.isAssignable(type, instanceClass);
    				Constructor<?> constructor = instanceClass
    						.getDeclaredConstructor(parameterTypes);
    				T instance = (T) BeanUtils.instantiateClass(constructor, args);
    				instances.add(instance);
    			}
    			catch (Throwable ex) {
    				throw new IllegalArgumentException(
    						"Cannot instantiate " + type + " : " + name, ex);
    			}
    		}
    		return instances;
    }

最后回到setInitializers，其实就是将上述返回的实例化列表存在入SpringApplication的initializers属性中：

    public void setInitializers(
    			Collection<? extends ApplicationContextInitializer<?>> initializers) {
    		this.initializers = new ArrayList<>();
    		this.initializers.addAll(initializers);
    }

初始化监听器列表操作其实和设置上下文列表差不多，只不过传入的type不一样。

最后是推断主应用入口：

    private Class<?> deduceMainApplicationClass() {
    		try {
    			StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
    			for (StackTraceElement stackTraceElement : stackTrace) {
    				if ("main".equals(stackTraceElement.getMethodName())) {
    					return Class.forName(stackTraceElement.getClassName());
    				}
    			}
    		}
    		catch (ClassNotFoundException ex) {
    			// Swallow and continue
    		}
    		return null;
    }

这个推断入口应用类的方式有点特别，通过构造一个运行时异常，再遍历异常栈中的方法名，获取方法名为 main 的栈帧，从来得到入口类的名字再返回该类。