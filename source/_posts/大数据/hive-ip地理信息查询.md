---
title: hive-ip地理信息查询
tags:
  - hive
categories:
  - 大数据
date: 2022-06-20 23:04:06
---

# HIVE UDF IP查询



利用 hive-udf 自定义 IP 查询函数

借助 ipip 提供的 ipdb

https://www.ipip.net/product/ip.html#ipv4city
将 .ipdb 文件放在 resources 目录下

代码如下

```java
import net.ipip.ipdb.City;
import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentLengthException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.StringObjectInspector;
 
import java.io.IOException;
 
/**
 * @description: ip查询城市
 * @author: xxzuo
 * @email: 1293378490@qq.com
 **/
public class IpLocationCity extends GenericUDF {
    private static City IPDB;
    private transient StringObjectInspector allCgi;
 
    /**
     * Initialize this GenericUDF. This will be called once and only once per
     * GenericUDF instance.
     *
     * @param arguments The ObjectInspector for the arguments
     * @return The ObjectInspector for the return value
     * @throws UDFArgumentException Thrown when arguments have wrong types, wrong length, etc.
     */
    @Override
    public ObjectInspector initialize(ObjectInspector[] arguments) throws UDFArgumentException {
        ObjectInspector arg = arguments[0];
        if (arguments.length != 1) {
            throw new UDFArgumentLengthException(
                    "The operator 'SubstrCgi' accepts one arguments.");
        }
        try {
            IPDB = new City(this.getClass().getResourceAsStream("/ipipfree.ipdb"));
        } catch (IOException e) {
        }
        this.allCgi = (StringObjectInspector) arg;
        return PrimitiveObjectInspectorFactory.javaStringObjectInspector;
    }
 
    /**
     * Evaluate the GenericUDF with the arguments.
     *
     * @param arguments The arguments as DeferedObject, use DeferedObject.get() to get the
     *                  actual argument Object. The Objects can be inspected by the
     *                  ObjectInspectors passed in the initialize call.
     * @return The
     */
    @Override
    public Object evaluate(GenericUDF.DeferredObject[] arguments) throws HiveException {
        String cgi = allCgi.getPrimitiveJavaObject(arguments[0].get());
        if(null == cgi) {
            return null;
        }
        String ipInfo = "";
        try {
            ipInfo = IPDB.find(cgi.toString(), "CN")[2];
        }
        catch (Exception e) {
        }
        return ipInfo;
    }
 
    /**
     * Get the String to be displayed in explain.
     *
     * @param children
     */
    @Override
    public String getDisplayString(String[] children) {
        return "Usage: SubstrCgi(String cgi)";
    }
}
```

