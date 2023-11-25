---
title: Java获取应用的所有Class对象
tags: [软件开发, Java]
---

## 代码实例

获取系统类加载器（Application ClassLoader）的类路径，遍历类路径的所有子目录，获取所有类名，并加载

```java
@SneakyThrows  
private Set<Class<?>> getAllClasses(File file, String classPathRoot) {  
    Set<Class<?>> classSet = new HashSet<>();  
  
    File[] files = file.listFiles();  
    if (files != null) {  
        for (File childFile : files) {  
            if (childFile.isDirectory()) {  
                Set<Class<?>> childClassSet = getAllClasses(childFile, classPathRoot);  
                classSet.addAll(childClassSet);  
            } else {  
                String path = childFile.getPath();  
                if (path.endsWith(".class")) {  
                    path = path.replace(classPathRoot, "").replace(".class", "").replace("\\", ".");  
                    Class<?> clazz = Class.forName(path);  
                    classSet.add(clazz);  
                }  
            }  
        }  
    }  
    return classSet;  
}
```

```java
ClassLoader classLoader = this.getClass().getClassLoader();  
// 这里假如classpath里有多个文件夹，可能会返回其中一个文件夹
// getResource方法使用双亲委派原则，父classLoader的classpath里没有文件夹，返回null，最后到Launcher$AppClassLoader获取到文件夹路径
URL resource = classLoader.getResource("");  
if (resource != null) {  
    String path = resource.getPath();  
    path = path.substring(1).replace("/", "\\");  
    File file = new File(path);  
    Set<Class<?>> allClasses = getAllClasses(file, path);  
    System.out.println(allClasses);  
}
```