---
title: Calcite文档
author: xxzuo
tags:
  - calcite
categories:
  - calcite
date: 2024-02-06 16:15:56
---

# 概述

## 背景

Apache Calcite是一个动态数据管理框架。

它包含了许多组成典型数据库管理系统的部分，但省略了一些关键功能：数据存储、处理数据的算法和存储元数据的存储库。

Calcite有意不参与数据存储和处理业务。正如我们将看到的，这使它成为在应用程序与一个或多个数据存储位置和数据处理引擎之间进行中介的绝佳选择。它也是构建数据库的完美基础：只需添加数据即可。

为了进行说明，让我们创建一个Calcite的空实例，然后将其指向一些数据。

```java
public static class HrSchema {  
    public final Employee[] emps = 0;  
    public final Department[] depts = 0;  
}  
Class.forName("org.apache.calcite.jdbc.Driver");  
Properties info = new Properties(); info.setProperty("lex","JAVA");  
Connection connection = DriverManager.getConnection("jdbc:calcite:", info);  
CalciteConnection calciteConnection = connection.unwrap(CalciteConnection.class);  
SchemaPlus rootSchema = calciteConnection.getRootSchema();  
Schema schema = new ReflectiveSchema(new HrSchema()); rootSchema.add("hr",schema);  
Statement statement = calciteConnection.createStatement();  
ResultSet resultSet = statement.executeQuery("select d.deptno, min(e.empid)\n" + "from hr.emps as e\n" + "join hr.depts as d\n" + " on e.deptno = d.deptno\n" + "group by d.deptno\n" + "having count(*) > 1");  
  
print(resultSet); resultSet.close(); statement.close(); connection.close();
```

数据库在哪里？没有数据库。连接完全为空，直到 `new ReflectiveSchema` 将Java对象注册为模式，并将其集合字段`emps`和`depts`注册为表。

Calcite不想拥有数据；它甚至没有喜欢的数据格式。此示例使用内存中的数据集，并使用 `linq4j` 库中的`groupBy`和`join`等运算符对其进行处理。但是Calcite 也可以处理其他数据格式的数据，例如JDBC。在第一个示例中，替换
```java
Schema schema = new ReflectiveSchema(new HrSchema());
```

为
```java
BasicDataSource dataSource = new BasicDataSource();  
dataSource.setUrl("jdbc:mysql://localhost");  
dataSource.setUsername("username");  
dataSource.setPassword("password");  
Schema schema = JdbcSchema.create(rootSchema, "hr", dataSource,  
        null, "name");
```

Calcite将在JDBC中执行相同的查询。对于应用程序来说，数据和API是相同的，但在后台实现是非常不同的。Calcite使用优化器规则将 `JOIN` 和 `GROUP BY` 操作推送到源数据库。

内存和JDBC只是两个熟悉的例子。Calcite可以处理任何数据源和数据格式。要添加数据源，您需要编写一个适配器，该适配器告诉Calcite应该将数据源中的哪些集合视为 `table`。

对于更高级的集成，您可以编写优化器规则。优化器规则允许Calcite访问新格式的数据，允许您注册新的运算符（例如更好的 `Join` 算法），也允许Calcite优化查询转换为运算符的方式。Calcite将您的规则和运算符与内置规则和运算符相结合，使用基于成本的优化模型，生成一个高效的计划。


### 编写一个适配器

`example/csv` 目录下的子项目提供了一个功能齐全、可用于应用程序的 CSV 适配器。它也足够简单，如果您正在编写自己的适配器，它可以作为一个很好的模板。

关于使用 CSV 适配器和编写其他适配器的信息，请参考[教程](https://calcite.apache.org/docs/tutorial.html)

关于使用其他适配器以及常规使用 Calcite 的更多信息，请参考[如何去做](https://calcite.apache.org/docs/howto.html)

### 状态

以下功能已完成
- 查询解析器、验证器和优化器
- 支持读取 JSON 格式的模型
- 许多标准函数和聚合函数
- 针对 Linq4j 和 JDBC 后端的JDBC查询
- Linq4j 前端
- SQL功能：`SELECT`、`FROM`（包括 `JOIN` 语法）、`WHERE`、`GROUP BY`（包括`GROUPING SETS`）、聚合函数（包括`COUNT(DISTINCT …)`和 `FILTER` ）、`HAVING`、`ORDER BY`（包括`NULLS FIRST/LAST`）、集操作（`UNION`、`INTERSECT`、`MINUS`）、子查询（包括相关子查询）、窗口聚合、LIMIT(如 [Postgres](https://www.postgresql.org/docs/8.4/static/sql-select.html#SQL-LIMIT) 语法)——更多详细信息参考[SQL参考](https://calcite.apache.org/docs/reference.html)
- 本地和远程JDBC驱动程序；参考 [Avatica](https://calcite.apache.org/docs/avatica_overview.html)
- 几个[适配器](https://calcite.apache.org/docs/adapter.html)



## 教程

这是一个循序渐进的教程，展示了如何构建和连接Calcite。它使用一个简单的适配器，使CSV文件的目录看起来像是包含表的模式。Calcite完成了剩下的工作，并提供了一个完整的SQL接口。

`Calcite-example-CSV` 是Calcite的一个功能齐全的适配器，用于读取[CSV（逗号分隔值）](https://en.wikipedia.org/wiki/Comma-separated_values)格式的文本文件。 值得注意的是，几百行Java代码就足以提供完整的SQL查询功能。

CSV还可以作为构建其他数据格式适配器的模板。尽管代码行不多，但它涵盖了几个重要概念：
- 使用SchemaFactory和schema接口的用户定义模式;
- 在 JSON 格式的模型文件中声明模式;
- 在 JSON 格式的模型文件中声明视图;
- 使用 `Table` 接口的用户自定义表;
- 确定表的记录类型;
- 表的一个简单实现，使用`ScannableTable`接口，直接枚举所有行;
- 一个更高级的实现，它实现了`FilterableTable`，并且可以根据简单的谓词过滤掉行;
- 表的高级实现，使用`TranslatableTable`，使用计划器规则转换为关系运算符;

### 下载和构建

你需要 `Java`（版本 8、9 或 10）和 `Git`
```shell
$ git clone https://github.com/apache/calcite.git 
$ cd calcite/example/csv 
$ ./sqlline
```

#### 第一个查询

现在，让我们使用 `sqlline` 连接到Calcite，[sqlline](https://github.com/julianhyde/sqlline) 是该项目中包含的SQL shell。

```shell
$ ./sqlline
sqlline> !connect jdbc:calcite:model=src/test/resources/model.json admin admin
```

（如果您运行的是Windows，则命令为`sqlline.bat`。）

执行元数据查询：
```shell
sqlline> !tables
+------------+--------------+-------------+---------------+----------+------+
| TABLE_CAT  | TABLE_SCHEM  | TABLE_NAME  |  TABLE_TYPE   | REMARKS  | TYPE |
+------------+--------------+-------------+---------------+----------+------+
| null       | SALES        | DEPTS       | TABLE         | null     | null |
| null       | SALES        | EMPS        | TABLE         | null     | null |
| null       | SALES        | HOBBIES     | TABLE         | null     | null |
| null       | metadata     | COLUMNS     | SYSTEM_TABLE  | null     | null |
| null       | metadata     | TABLES      | SYSTEM_TABLE  | null     | null |
+------------+--------------+-------------+---------------+----------+------+

```

JDBC 专家们注意：`sqlline` 的 `!tables` 命令只是在背后执行了  [DatabaseMetaData.getTables()](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/sql/DatabaseMetaData.html#getTables(java.lang.String,%20java.lang.String,%20java.lang.String,%20java.lang.String%5B%5D)) 方法。它也提供了其他命令，可以用来查询 JDBC 元数据，例如 `!columns` 和 `!describe`。

正如你看见的，系统中有 5 张表： `EMPS`，`DEPTS` 和 `HOBBIES` 表在当前 `SALES` 模式中，`COLUMNS` 和 `TABLES` 表在系统 `metadata` 模式中。系统表始终存在于 Calcite 中，而其他表则由模式的具体实现提供。在这个场景下，`EMPS` 和 `DEPTS` 表是基于 `resources/sales` 目录下的 `EMPS.csv` 和 `DEPTS.csv` 文件。

让我们对这些表执行一些查询，来展示 Calcite 提供的 SQL 完整实现。首先，进行表扫描：
```shell
sqlline> SELECT * FROM emps;
+--------+--------+---------+---------+----------------+--------+-------+---+
| EMPNO  |  NAME  | DEPTNO  | GENDER  |      CITY      | EMPID  |  AGE  | S |
+--------+--------+---------+---------+----------------+--------+-------+---+
| 100    | Fred   | 10      |         |                | 30     | 25    | t |
| 110    | Eric   | 20      | M       | San Francisco  | 3      | 80    | n |
| 110    | John   | 40      | M       | Vancouver      | 2      | null  | f |
| 120    | Wilma  | 20      | F       |                | 1      | 5     | n |
| 130    | Alice  | 40      | F       | Vancouver      | 2      | null  | f |
+--------+--------+---------+---------+----------------+--------+-------+---+

```

再进行关联和分组查询：
```shell
sqlline> SELECT d.name, COUNT(*) 
. . . .> FROM emps AS e JOIN depts AS d ON e.deptno = d.deptno 
. . . .> GROUP BY d.name; 
+------------+---------+ 
| NAME       | EXPR$1  | 
+------------+---------+ 
| Sales      | 1       |
| Marketing  | 2       | 
+------------+---------+
```

最后，VALUES运算符生成一行，是测试表达式和SQL内置函数的一种方便方法
```shell
sqlline> VALUES CHAR_LENGTH('Hello, ' || 'world!');
+---------+
| EXPR$0  |
+---------+
| 13      |
+---------+

```


Calcite还有许多其他SQL功能。我们没有时间在这里报道他们。你可以再写一些查询进行实验。

### 模式发现

现在，Calcite是如何找到这些表格的？记住，核心Calcite对CSV文件一无所知。（作为一个“没有存储层的数据库”，Calcite不知道任何文件格式）。Calcite 知道这些表，完全是因为我们告诉它去执行 `calcite-example-csv` 项目中的代码。

发现过程包含了几个步骤。首先，我们基于模型文件中的模式工厂类定义了一个模式。然后，模式工厂创建了一个模式，并且这个模式创建一些表，每个表都知道通过扫描 CSV 文件来获取数据。最后，在 Calcite 解析完查询并生成使用这些表的执行计划后，Calcite 会在执行查询时，调用这些表来读取数据。现在让我们更详细地了解这些步骤。

在 JDBC 连接字符串上，我们以 JSON 格式给出了模型的路径。下面是模型的内容：

```json
{
    "version": "1.0",
    "defaultSchema": "SALES",
    "schemas": [
        {
            "name": "SALES",
            "type": "custom",
            "factory": "org.apache.calcite.adapter.csv.CsvSchemaFactory",
            "operand": {
                "directory": "sales"
            }
        }
    ]
}

```

模型定义了一个名为 `SALES` 的单模式。这个模式由插件类 [org.apache.calcite.adapter.csv.CsvSchemaFactory](https://github.com/apache/calcite/blob/main/example/csv/src/main/java/org/apache/calcite/adapter/csv/CsvSchemaFactory.java)  提供支持，它是 `calcite-example-csv` 项目的一部分，并实现了 Calcite  [SchemaFactory](https://calcite.apache.org/javadocAggregate/org/apache/calcite/schema/SchemaFactory.html)  接口。它的 `create` 方法，通过从模型文件中传入的 `directory` 参数，实例化了模式：
```java
public Schema create(SchemaPlus parentSchema, String name,  
                     Map<String, Object> operand) {  
    final String directory = (String) operand.get("directory");  
    final File base =  
            (File) operand.get(ModelHandler.ExtraOperand.BASE_DIRECTORY.camelName);  
    File directoryFile = new File(directory);  
    if (base != null && !directoryFile.isAbsolute()) {  
        directoryFile = new File(base, directory);  
    }  
    String flavorName = (String) operand.get("flavor");  
    CsvTable.Flavor flavor;  
    if (flavorName == null) {  
        flavor = CsvTable.Flavor.SCANNABLE;  
    } else {  
        flavor = CsvTable.Flavor.valueOf(flavorName.toUpperCase(Locale.ROOT));  
    }  
    return new CsvSchema(directoryFile, flavor);  
}
```

在模型的驱动下，模式工厂实例化一个名为 `SALES` 的单模式。该模式是[org.apache.calcite.adapter.csv.CsvSchema](https://github.com/apache/calcite/blob/main/example/csv/src/main/java/org/apache/calcite/adapter/csv/CsvSchema.java) 的一个实例，并实现 calcite [Schema](https://calcite.apache.org/javadocAggregate/org/apache/calcite/schema/Schema.html) 接口。

模式的一项工作是生成一系列的表（它还可以生成子模式和表函数，但这些是高级功能，`calcite-example-csv` 不支持它们）。这些表实现了 Calcite [Table](https://calcite.apache.org/javadocAggregate/org/apache/calcite/schema/Table.html) 接口。`CsvSchema` 生成的表是  [CsvTable](https://github.com/apache/calcite/blob/main/example/csv/src/main/java/org/apache/calcite/adapter/csv/CsvTable.java) 及其子类的实例。

下面是 `CsvSchema` 的相关代码，它重写了 `AbstractSchema` 基类中的 [getTableMap()](https://calcite.apache.org/javadocAggregate/org/apache/calcite/schema/impl/AbstractSchema.html#getTableMap()) 方法。
```java
private Map<String, Table> createTableMap() {  
    // Look for files in the directory ending in ".csv", ".csv.gz", ".json",  
    // ".json.gz".    final Source baseSource = Sources.of(directoryFile);  
    File[] files = directoryFile.listFiles((dir, name) -> {  
        final String nameSansGz = trim(name, ".gz");  
        return nameSansGz.endsWith(".csv")  
                || nameSansGz.endsWith(".json");  
    });  
    if (files == null) {  
        System.out.println("directory " + directoryFile + " not found");  
        files = new File[0];  
    }  
    // Build a map from table name to table; each file becomes a table.  
    final ImmutableMap.Builder<String, Table> builder = ImmutableMap.builder();  
    for (File file : files) {  
        Source source = Sources.of(file);  
        Source sourceSansGz = source.trim(".gz");  
        final Source sourceSansJson = sourceSansGz.trimOrNull(".json");  
        if (sourceSansJson != null) {  
            final Table table = new JsonScannableTable(source);  
            builder.put(sourceSansJson.relative(baseSource).path(), table);  
        }  
        final Source sourceSansCsv = sourceSansGz.trimOrNull(".csv");  
        if (sourceSansCsv != null) {  
            final Table table = createTable(source);  
            builder.put(sourceSansCsv.relative(baseSource).path(), table);  
        }  
    }  
    return builder.build();  
}  
  
/** Creates different sub-type of table based on the "flavor" attribute. */  
private Table createTable(Source source) {  
    switch (flavor) {  
        case TRANSLATABLE:  
            return new CsvTranslatableTable(source, null);  
        case SCANNABLE:  
            return new CsvScannableTable(source, null);  
        case FILTERABLE:  
            return new CsvFilterableTable(source, null);  
        default:  
            throw new AssertionError("Unknown flavor " + this.flavor);  
    }  
}
```

模式扫描目录，找到所有具有适当扩展名的文件，并为它们创建表。在这种情况下，目录是 `sales`，包含文件 `EMPS.csv.gz`、`DEPTS.csv` 和 `SDEPTS.csv` ，这些文件成为表`EMPS`、`DEPTS`和`SDEPTS`。


### 模式中的表和视图

请注意，我们不需要在模型中定义任何表;架构 自动生成表。

您可以定义额外的表， 除了那些自动创建的， 使用架构的属性。`tables`

让我们看看如何创建 一种重要且有用的表类型，即视图。

在编写查询时，视图看起来像一个表，但它不存储数据。 它通过执行查询来派生其结果。 在规划查询时，视图会展开，因此查询优化器通常可以执行优化，删除那些在最终结果中未使用的 SELECT 子句表达式。

下面是定义视图的架构：
```json
{
  version: '1.0',
  defaultSchema: 'SALES',
  schemas: [
    {
      name: 'SALES',
      type: 'custom',
      factory: 'org.apache.calcite.adapter.csv.CsvSchemaFactory',
      operand: {
        directory: 'sales'
      },
      tables: [
        {
          name: 'FEMALE_EMPS',
          type: 'view',
          sql: 'SELECT * FROM emps WHERE gender = \'F\''
        }
      ]
    }
  ]
}
```

`"type": "view"` 这行将 `FEMALE_EMPS` 标记为视图，而不是常规表或自定义表。JSON 并不能简单地书写长字符串，因此 Calcite 支持另一种可选的语法。如果你的视图有很长的 SQL 语句，你可以将单个字符串改为多行列表
```json
{
  name: 'FEMALE_EMPS',
  type: 'view',
  sql: [
    'SELECT * FROM emps',
    'WHERE gender = \'F\''
  ]
}
```
现在，我们已经定义了一个视图，我们可以像使用表一样，在查询中使用它：
```shell
sqlline> SELECT e.name, d.name FROM female_emps AS e JOIN depts AS d on e.deptno = d.deptno;
+--------+------------+
|  NAME  |    NAME    |
+--------+------------+
| Wilma  | Marketing  |
+--------+------------+
```

### 自定义表

自定义表是那些由用户自定义的代码驱动的表。他们不需要存在于自定义模式中。

在 `model-with-custom-table.json` 模型文件中，有一个自定义表的例子：
```json
{
  version: '1.0',
  defaultSchema: 'CUSTOM_TABLE',
  schemas: [
    {
      name: 'CUSTOM_TABLE',
      tables: [
        {
          name: 'EMPS',
          type: 'custom',
          factory: 'org.apache.calcite.adapter.csv.CsvTableFactory',
          operand: {
            file: 'sales/EMPS.csv.gz',
            flavor: "scannable"
          }
        }
      ]
    }
  ]
}
```

我们可以使用常规的方式查询自定义表：
```shell
sqlline> !connect jdbc:calcite:model=src/test/resources/model-with-custom-table.json admin admin
sqlline> SELECT empno, name FROM custom_table.emps;
+--------+--------+
| EMPNO  |  NAME  |
+--------+--------+
| 100    | Fred   |
| 110    | Eric   |
| 110    | John   |
| 120    | Wilma  |
| 130    | Alice  |
+--------+--------+
```

该模式是一个常规的模式，包含一个由[org.apache.calcate.adapter.csv.CsvTableFactory](https://github.com/apache/calcite/blob/main/example/csv/src/main/java/org/apache/calcite/adapter/csv/CsvTableFactory.java)提供支持的自定义表，实现Calcite接口[TableFactory](https://calcite.apache.org/javadocAggregate/org/apache/calcite/schema/TableFactory.html). 它的方法实例化，从模型文件中传入参数:`create` `CsvScannableTable` `file`
```java
public CsvTable create(SchemaPlus schema, String name,  
                       Map<String, Object> operand, @Nullable RelDataType rowType) {  
    String fileName = (String) operand.get("file");  
    final File base =  
            (File) operand.get(ModelHandler.ExtraOperand.BASE_DIRECTORY.camelName);  
    final Source source = Sources.file(base, fileName);  
    final RelProtoDataType protoRowType =  
            rowType != null ? RelDataTypeImpl.proto(rowType) : null;  
    return new CsvScannableTable(source, protoRowType);  
}
```

实现自定义表，通常是实现自定义模式的一个更简单方法。这两种方法可能最终都会创建类似的 `Table` 接口实现，但对于自定义表，你不需要实现元数据发现。`CsvTableFactory` 创建一个 `CsvScannableTable`，就像 `CsvSchema` 所做的那样，但表的实现不会扫描文件系统来查找 `.csv` 文件。

自定义表需要模型的开发者做更多的工作，需要明确指定每个表及其文件，但也给开发者提供了更多的控制权，例如，为每个表提供不同的参数。

### 模型中的注释

模型可以使用 `/* ... */` 和 `//` 语法来包含注释：
```json
{
  version: '1.0',
  /* Multi-line
     comment. */
  defaultSchema: 'CUSTOM_TABLE',
  // Single-line comment.
  schemas: [
    ..
  ]
}
```

注释不是标准的 JSON，而是一种无害的扩展。


### 使用优化器规则优化查询

我们到目前为止看到的表实现都不错,只要表不包含大量的数据。但是,如果您的客户表有 100 列和 100 万行,您会更喜欢系统在每个查询中不检索所有数据。您希望 Calcite 与适配器协商并找到一种更有效的访问数据的方式。

这种协商是查询优化的一种简单形式。Calcite 通过添加 _优化器规则_ 来支持查询优化。优化器规则的工作方式是在查询解析树中查找模式(例如某种表解析树顶部的投影),并用一组新的节点替换匹配的节点来实现优化。

优化器规则也是可扩展的,就像 schema 和表一样。因此,如果您有一个想通过 SQL 访问的数据存储,您首先要定义一个自定义表或 schema,然后定义一些规则来进行高效访问。

要查看它的实际操作,我们使用一个规划器规则来访问 CSV 文件中的一部分列。让我们对两个非常相似的 schema 运行相同的查询:
```shell
sqlline> !connect jdbc:calcite:model=src/test/resources/model.json admin admin
sqlline> explain plan for select name from emps;
+-----------------------------------------------------+
| PLAN                                                |
+-----------------------------------------------------+
| EnumerableCalc(expr#0..9=[{inputs}], NAME=[$t1])    |
|   EnumerableTableScan(table=[[SALES, EMPS]])        |
+-----------------------------------------------------+
sqlline> !connect jdbc:calcite:model=src/test/resources/smart.json admin admin
sqlline> explain plan for select name from emps;
+-----------------------------------------------------+
| PLAN                                                |
+-----------------------------------------------------+
| CsvTableScan(table=[[SALES, EMPS]], fields=[[1]])   |
+-----------------------------------------------------+
```

是什么导致了执行计划的差异？让我们跟着证据的线索走。在 `smart.json` 模型文件中，只有一行：
```json
flavor: "translatable"
```

这个配置会使用 `flavor = TRANSLATABLE` 来创建 `CsvSchema`，它的 `createTable` 方法创建了 [CsvTranslatableTable](https://github.com/apache/calcite/blob/main/example/csv/src/main/java/org/apache/calcite/adapter/csv/CsvTranslatableTable.java) 而不是 `CsvScannableTable`。

`CsvTranslatableTable` 实现了 [TranslatableTable.toRel()](https://calcite.apache.org/javadocAggregate/org/apache/calcite/schema/TranslatableTable.html#toRel()) 方法，用来创建 [CsvTableScan](https://github.com/apache/calcite/blob/main/example/csv/src/main/java/org/apache/calcite/adapter/csv/CsvTableScan.java)。表扫描是查询操作树的叶子节点。通常实现是 [EnumerableTableScan](https://calcite.apache.org/javadocAggregate/org/apache/calcite/adapter/enumerable/EnumerableTableScan.html)，但我们创建了一个独特的子类型，它将导致规则触发。

下面是完整的规则实现：
```java
public class CsvProjectTableScanRule
    extends RelRule<CsvProjectTableScanRule.Config> {

  /** Creates a CsvProjectTableScanRule. */
  protected CsvProjectTableScanRule(Config config) {
    super(config);
  }

  @Override public void onMatch(RelOptRuleCall call) {
    final LogicalProject project = call.rel(0);
    final CsvTableScan scan = call.rel(1);
    int[] fields = getProjectFields(project.getProjects());
    if (fields == null) {
      // Project contains expressions more complex than just field references.
      return;
    }
    call.transformTo(
        new CsvTableScan(
            scan.getCluster(),
            scan.getTable(),
            scan.csvTable,
            fields));
  }

  private static int[] getProjectFields(List<RexNode> exps) {
    final int[] fields = new int[exps.size()];
    for (int i = 0; i < exps.size(); i++) {
      final RexNode exp = exps.get(i);
      if (exp instanceof RexInputRef) {
        fields[i] = ((RexInputRef) exp).getIndex();
      } else {
        return null; // not a simple projection
      }
    }
    return fields;
  }

  /** Rule configuration. */
  @Value.Immutable(singleton = false)
  public interface Config extends RelRule.Config {
    Config DEFAULT = ImmutableCsvProjectTableScanRule.Config.builder()
        .withOperandSupplier(b0 ->
            b0.operand(LogicalProject.class).oneInput(b1 ->
                b1.operand(CsvTableScan.class).noInputs()))
        .build();

    @Override default CsvProjectTableScanRule toRule() {
      return new CsvProjectTableScanRule(this);
    }
  }
}
```

规则的默认实例驻留在 `CsvRules` 的持有类中：
```java
public abstract class CsvRules {
  public static final CsvProjectTableScanRule PROJECT_SCAN =
      CsvProjectTableScanRule.Config.DEFAULT.toRule();
}
```




## 代数
