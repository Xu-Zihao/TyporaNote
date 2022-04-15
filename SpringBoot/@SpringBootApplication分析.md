### @SpringBootApplication

包名：

```java
package org.springframework.boot.autoconfigure;
```

@SpringBootApplication属于SpringBoot的注解

#### 源码分析：

```java
@Target({ElementType.TYPE})//java注解，说明该注解可以用于哪些目标之上（如：类、接口、方法...）
@Retention(RetentionPolicy.RUNTIME)//java注解，表明该注解可以保留的时间
@Documented//java注解，只是用来标注生成javadoc的时候是否会被记录。
@Inherited//java注解，如果标注了@Inherited，则标注了该注解（@SpringBootApplication）的类的子类会继承
//该注解@SpringBootApplication）

@SpringBootConfiguration//SpringBoot定义的注解，表明这是一个主注解类
@EnableAutoConfiguration//SpringBoot定义的注解，开启自动配置
@ComponentScan//开启组件扫描，将@Component类注解的类注册进容器
(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    Class<?>[] exclude() default {};

    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    String[] excludeName() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "nameGenerator"
    )
    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}

```

重点是这三个注解：@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan。

下面一一进行分析：

##### @SpringBootConfiguration

源码：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration//表示这是一个注解类
@Indexed//为注解扫描添加索引，提高扫描性能
public @interface SpringBootConfiguration {
    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}
```

##### @EnableAutoConfiguration

源码：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage//
@Import({AutoConfigurationImportSelector.class})//
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

最主要的两条注解是：`@AutoConfigurationPackage`和`@Import({AutoConfigurationImportSelector.class})`下面一一对其进行分析：

###### @AutoConfigurationPackage

1. 看到注解@AutoConfigurationPackage，观察其源码：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})//
public @interface AutoConfigurationPackage {
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};
}
```

2. 继续看`@Import({Registrar.class})`发现其使用import向容器中导入`Registrar.class`，查看`Registrar.class`的源码：

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
    
    ....
    ....
    ....
    
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
        Registrar() {
        }

        public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
            AutoConfigurationPackages.register(registry, (String[])(new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames().toArray(new String[0]));
        }
    //new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames()得到的就是启动类所在包的包名，
    //最后以数组的形式返回，表示将该包下的所有组件注册进入容器中,所以在SpringBoot中，默认只有在Application程序
    //同包或者其子包的组件会被注册进容器

        public Set<Object> determineImports(AnnotationMetadata metadata) {
            return Collections.singleton(new AutoConfigurationPackages.PackageImports(metadata));
        }
    }
    
    ....
    ....
    
}
```

3. 这个类是`AutoConfigurationImportSelector`这个类的内部类，而`AutoConfigurationImportSelector`这个类已经`@EnableAutoConfiguration`的源码`@Import({AutoConfigurationImportSelector.class})`中向容器中注册。

   这个类的`registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry)`方法有两个参数，第一个参数metadate是该注解的元信息，标记这这个注解所注解的位置，属性的设值等；第二个参数用于向容器中注册组件。

###### @Import({AutoConfigurationImportSelector.class})

该注解向容器中导入了一个类：`AutoConfigurationImportSelector.class`查看这个类的源代码：

这个类有一个核心方法：

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
            AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);//通过getAutoConfigurationEntry得到一些数据
            return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
            //把这些数据转化为字符串数组，返回到@Import中作为参数
        }
    }
```

通过上面，还需要继续查看`getAutoConfigurationEntry`方法的源码：

```java
 protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        } else {
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            //获取所有候选配置，下面的几步进行去重等处理再封装返回。
            configurations = this.removeDuplicates(configurations);
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.getConfigurationClassFilter().filter(configurations);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
        }
    }
```

上面的代码，可以发现

`List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);`这一行代码是核心，继续查看`getCandidateConfigurations`方法的源代码：

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
    //使用SpringFactoriesLoader加载loadFactoryNames
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
```

继续看`loadFactoryNames`的源码：

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
        ClassLoader classLoaderToUse = classLoader;
        if (classLoader == null) {
            classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();//
        }

        String factoryTypeName = factoryType.getName();
        return (List)loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
    }
```

该方法返回的是`classLoaderToUse`经过处理的封装成的列表，为了找出`classLoaderToUse`来自哪里，继续查看`SpringFactoriesLoader`的源码：

该类有一个方法：`private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) `

```java
private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
        Map<String, List<String>> result = (Map)cache.get(classLoader);
        if (result != null) {
            return result;
        } else {
            HashMap result = new HashMap();

            try {
                Enumeration urls = classLoader.getResources("META-INF/spring.factories");

                while(urls.hasMoreElements()) {
                    URL url = (URL)urls.nextElement();
                    UrlResource resource = new UrlResource(url);
                    Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                    Iterator var6 = properties.entrySet().iterator();

                    while(var6.hasNext()) {
                        Entry<?, ?> entry = (Entry)var6.next();
                        String factoryTypeName = ((String)entry.getKey()).trim();
                        String[] factoryImplementationNames = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
                        String[] var10 = factoryImplementationNames;
                        int var11 = factoryImplementationNames.length;

                        for(int var12 = 0; var12 < var11; ++var12) {
                            String factoryImplementationName = var10[var12];
                            ((List)result.computeIfAbsent(factoryTypeName, (key) -> {
                                return new ArrayList();
                            })).add(factoryImplementationName.trim());
                        }
                    }
                }

                result.replaceAll((factoryType, implementations) -> {
                    return (List)implementations.stream().distinct().collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList));
                });
                cache.put(classLoader, result);
                return result;
            } catch (IOException var14) {
                throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var14);
            }
        }
    }
```

其中最值得关注的一行代码是：`Enumeration urls = classLoader.getResources("META-INF/spring.factories");`至此可以知道SpringBoot扫描每个包的META-INF/spring.factories文件，如下图：这些自动配置类都被写死在该文件中

![image-20220228172403803](C:\Users\XuZihao\AppData\Roaming\Typora\typora-user-images\image-20220228172403803.png)

###### 结论(按需加载)：

在spring-boot-autoconfigure包下的目录\META-INF\spring.factories文件中定义了所有需要被加载的自动配置类。

但是该文件中定义的配置类不会全部生效，因为这些类设置了许多@Conditional条件装载的注解，只有这些类被真使用时才会在扫描的时侯被加载容器(按需装配)，如果该配置类生效，该配置类定义的组件也会被加入到容器中。

> 这些自动配置类会以用户配置的优先，只要用户自行配置了，就会使用用户的配置。这些配置类的配置信息配置文件绑定。

如`AopAutoConfiguration`类中的配置：

```java
@ConditionalOnProperty//当配置文件中存在满足以下条件的配置信息时注册该组件
(
        prefix = "spring.aop",//该配置与application.properties文件以spring.aop为前缀
        name = {"proxy-target-class"},
        havingValue = "true",
        matchIfMissing = true//如果配置文件没有匹配的配置信息，照常注册zu'jian
    )
```







------



##### @ComponentScan

为组件扫描设置属性