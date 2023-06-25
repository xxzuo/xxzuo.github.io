---
title: List<Integer>添加一个String
author: xxzuo
categories:
  - 其他
date: 2022-08-30 00:02:07
tags:
---



利用反射魔法 就可以在 ArrayList<Integer> 中添加一个 String类型的元素了

```java
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.List;

public class Test2 {
    public static void main(String[] args) {
        try {
            List<Integer> test = new ArrayList<>();
            test.add(1);
            Method method = test.getClass().getMethod("add", Object.class);
            method.invoke(test, "t");
            for (Object num : test) {
                System.out.println(num);
            }
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}

```

输出结果为

```
1
t
```

