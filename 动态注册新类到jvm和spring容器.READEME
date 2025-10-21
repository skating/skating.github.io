### 动态注册新类到jvm和spring 容器
``` java
package com.zfull.commons.hot;

import cn.hutool.extra.spring.SpringUtil;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.objectweb.asm.ClassReader;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.stereotype.Component;

import java.io.InputStream;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

/**
 * 动态注入新类到spring容器
 * @author zhb
 * @date 2025-10-21 16:27:38
 */
@Slf4j
@Component
public class DynamicClassSpringRegistrar {

    private final ConfigurableApplicationContext context;
    private final HotClassLoader loader;

    public DynamicClassSpringRegistrar(ConfigurableApplicationContext context) {
        this.context = context;
        loader = new HotClassLoader(getClass().getClassLoader());
    }

    /**
     *
     * @param classFile 类文件
     * @return
     */
    public Object registerByClassFile(String classFile) {
        try {

            Path path = Paths.get(classFile);
            byte[] bytes = Files.readAllBytes(path);
            try (InputStream in = Files.newInputStream(path)) {
                //通过asm 获取全限定类名
                ClassReader reader = new ClassReader(in);
                String className = reader.getClassName().replace('/', '.');
//                int lastIndexOf = className.lastIndexOf('.');
//                String beanName = className.substring(lastIndexOf + 1);
//                beanName = StringUtils.uncapitalize(beanName);
                return loadAndRegister(className, bytes, className);
            }
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
        return null;
    }

    public Object loadAndRegister(String className, byte[] classData, String beanName)
            throws Exception {

        //用自定义 ClassLoader 加载类
        Class<?> clazz = loader.defineClass(className, classData);

        //注册到 Spring 容器
        registerDynamicClass(beanName, clazz);

        System.out.println("已注册动态 Bean: " + beanName + " -> " + clazz.getName());
        return SpringUtil.getBean(beanName);
    }

    /**
     * 注册到spring容器，并且让其依赖注入等
     * @param beanName 全限定类名
     * @param clazz 需要注入的类
     */
    private void registerDynamicClass(String beanName, Class<?> clazz) {
        DefaultListableBeanFactory beanFactory = (DefaultListableBeanFactory) context.getBeanFactory();

        BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(clazz);
        beanFactory.registerBeanDefinition(beanName, builder.getRawBeanDefinition());
    }
}

```


### 自定义一个classloader
```java
package com.zfull.commons.hot;

/**
 * @author zhb
 * @date 2025-10-21 15:50:32
 */
public class HotClassLoader extends ClassLoader {

    public HotClassLoader(ClassLoader parent) {
        super(parent);
    }
    public Class<?> defineClass(String className, byte[] classData) {
        return super.defineClass(className, classData, 0, classData.length);
    }
}

```
