# Spring-Boot-Study

Spring-Boot 源码解析，版本2.0.0.RELEASE。

### 目录结构

```

.
├── docs
│   └── 源码解析
│       ├── 源码-springboot-SpringApplication.md
│       └── 源码-springboot-SpringApplication.md
└── spring-boot-source 源码目录
    └── spring-boot-project spring-boot项目核心目录
        └── spring-boot-examplep 自定义的测试启动项目
        
```

### 源码来源

spring-boot的源码来自[官方github](https://github.com/spring-projects/spring-boot)，git拉下来后，本地执行：

```shell
git tag
git checkout -b v2.0.0.RELEASE v2.0.0.RELEASE
```

最终获取到代码的就是和spring-boot-source目录中一样。

### 项目内容

* :white_check_mark: 增加源码的注释
* :white_check_mark: SpringBoot项目启动源码解析
* :ballot_box_with_check: spring-boot-starters 解析
* :ballot_box_with_check: ​设计模式解析

### 参考资源

1. [IDEA 编译运行 Spring Boot 2.0 源码](https://my.oschina.net/dabird/blog/1942112)
2. [Spring Boot 2.x 启动全过程源码分析（上）入口类剖析](https://segmentfault.com/a/1190000015899405)
3. [涨姿势：Spring Boot 2.x 启动全过程源码分析](https://segmentfault.com/a/1190000015998105)
4. [一看你就懂，超详细java中的ClassLoader详解](https://blog.csdn.net/briblue/article/details/54973413)
5. [轻松学，Java 中的代理模式及动态代理](https://blog.csdn.net/briblue/article/details/73928350)

