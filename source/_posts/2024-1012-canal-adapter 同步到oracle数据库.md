canal-adapter 同步到oracle数据库



canal-adapter同步mysql数据库到所有的rds数据库都没问题，但是到oracle遇到了麻烦

### 第一个坑-表名字段名称大写

oracle数据库的表名和字段名必须是大写，如果小写的话，会出现找不到表和找不到字段的提示。

所以务必保证oracle数据表大写。

我当时是把mysql的表复制到oracel，结果所有表名、字段名都是小写，就没法同步。

解决的办法：

在navicat中打开菜单重的数据传输，目标选择文件，sql格式插掉与“原服务器相同”，然后选择oracle，版本选择11g（根据目标数据库格式选择），编码选择unicode（utf-8）。然后生成的文件用文本编辑器打开，把建表部分的语句全部转为大写，插入语句也替换成大写。

然后直接用sql导入数据库就行。

### 第二个坑-字段类型

我的数据表的字段是 bit ,自动会转成nvarchar2，然后导入数据的时候会提示大小超长之类的，我就把它加长，但是提示有数据不能改。所以还是在数据传输的时候把表盖好。



### 第三个坑-全量同步

canal经常需要做全量同步,执行的命令就是

```
curl http://127.0.0.1:8081/etl/rdb/keyoracle/oracle-baseinfo-company.yml -X POST
```

遇到的问题就是canal-adapter源代码中增量同步和全量同步是分别写的，增量同步中对目标数据库类型的处理没有问题，但是全量同步的时候会遇到各种sql错误，检查发现是源代码中对目标数据库格式的判断都错误的使用了源数据库格式，导致生成的语句错误。

另外oracle setFetchSize的时候不能设置为负数，所以这里改成 0

详见如下patch：



```java
Subject: [PATCH] 修复全量同步数据库时，目标数据库与源数据库不一致时获取数据库类型不一致的错误
---
Index: client-adapter/common/src/main/java/com/alibaba/otter/canal/client/adapter/support/Util.java
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
diff --git a/client-adapter/common/src/main/java/com/alibaba/otter/canal/client/adapter/support/Util.java b/client-adapter/common/src/main/java/com/alibaba/otter/canal/client/adapter/support/Util.java
--- a/client-adapter/common/src/main/java/com/alibaba/otter/canal/client/adapter/support/Util.java	(revision ea8949298fc74310990f24c864db218c7ccac312)
+++ b/client-adapter/common/src/main/java/com/alibaba/otter/canal/client/adapter/support/Util.java	(revision 1f4d326bea570e0db95794d412075c478e7f05ea)
@@ -38,7 +38,7 @@
     public static Object sqlRS(DataSource ds, String sql, Function<ResultSet, Object> fun) {
         try (Connection conn = ds.getConnection();
                 Statement stmt = conn.createStatement(ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY)) {
-            stmt.setFetchSize(Integer.MIN_VALUE);
+            stmt.setFetchSize(0);
             try (ResultSet rs = stmt.executeQuery(sql)) {
                 return fun.apply(rs);
             }
@@ -52,7 +52,7 @@
         try (Connection conn = ds.getConnection()) {
             try (PreparedStatement pstmt = conn
                 .prepareStatement(sql, ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY)) {
-                pstmt.setFetchSize(Integer.MIN_VALUE);
+                pstmt.setFetchSize(0);
                 if (values != null) {
                     for (int i = 0; i < values.size(); i++) {
                         pstmt.setObject(i + 1, values.get(i));
@@ -80,6 +80,8 @@
             consumer.accept(rs);
         } catch (SQLException e) {
             logger.error(e.getMessage(), e);
+            logger.error("sqlRs has error, sql: {} ", sql);
+
         }
     }
 
Index: client-adapter/rdb/src/main/java/com/alibaba/otter/canal/client/adapter/rdb/service/RdbEtlService.java
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
diff --git a/client-adapter/rdb/src/main/java/com/alibaba/otter/canal/client/adapter/rdb/service/RdbEtlService.java b/client-adapter/rdb/src/main/java/com/alibaba/otter/canal/client/adapter/rdb/service/RdbEtlService.java
--- a/client-adapter/rdb/src/main/java/com/alibaba/otter/canal/client/adapter/rdb/service/RdbEtlService.java	(revision ea8949298fc74310990f24c864db218c7ccac312)
+++ b/client-adapter/rdb/src/main/java/com/alibaba/otter/canal/client/adapter/rdb/service/RdbEtlService.java	(revision 1f4d326bea570e0db95794d412075c478e7f05ea)
@@ -59,9 +59,11 @@
             Map<String, Integer> columnType = new LinkedHashMap<>();
             DruidDataSource dataSource = (DruidDataSource) srcDS;
             String backtick = SyncUtil.getBacktickByDbType(dataSource.getDbType());
+            DruidDataSource targetDataSource = (DruidDataSource) targetDS;
+            String targetbacktick = SyncUtil.getBacktickByDbType(targetDataSource.getDbType());
 
             Util.sqlRS(targetDS,
-                "SELECT * FROM " + SyncUtil.getDbTableName(dbMapping, dataSource.getDbType()) + " LIMIT 1 ",
+                "SELECT * FROM " + SyncUtil.getDbTableName(dbMapping, targetDataSource.getDbType()) + SyncUtil.getLimitOneByDbType( targetDataSource.getDbType()),
                 rs -> {
                 try {
 
@@ -89,10 +91,10 @@
 
                     StringBuilder insertSql = new StringBuilder();
                     insertSql.append("INSERT INTO ")
-                        .append(SyncUtil.getDbTableName(dbMapping, dataSource.getDbType()))
+                        .append(SyncUtil.getDbTableName(dbMapping, targetDataSource.getDbType()))
                         .append(" (");
                     columnsMap
-                        .forEach((targetColumnName, srcColumnName) -> insertSql.append(backtick).append(targetColumnName).append(backtick).append(","));
+                        .forEach((targetColumnName, srcColumnName) -> insertSql.append(targetbacktick).append(targetColumnName).append(targetbacktick).append(","));
 
                     int len = insertSql.length();
                     insertSql.delete(len - 1, len).append(") VALUES (");
@@ -115,8 +117,8 @@
                             // 删除数据
                             Map<String, Object> pkVal = new LinkedHashMap<>();
                             StringBuilder deleteSql = new StringBuilder(
-                                "DELETE FROM " + SyncUtil.getDbTableName(dbMapping, dataSource.getDbType()) + " WHERE ");
-                            appendCondition(dbMapping, deleteSql, pkVal, rs, backtick);
+                                "DELETE FROM " + SyncUtil.getDbTableName(dbMapping, targetDataSource.getDbType()) + " WHERE ");
+                            appendCondition(dbMapping, deleteSql, pkVal, rs, targetbacktick);
                             try (PreparedStatement pstmt2 = connTarget.prepareStatement(deleteSql.toString())) {
                                 int k = 1;
                                 for (Object val : pkVal.values()) {
Index: client-adapter/rdb/src/main/java/com/alibaba/otter/canal/client/adapter/rdb/support/SyncUtil.java
IDEA additional info:
Subsystem: com.intellij.openapi.diff.impl.patch.CharsetEP
<+>UTF-8
===================================================================
diff --git a/client-adapter/rdb/src/main/java/com/alibaba/otter/canal/client/adapter/rdb/support/SyncUtil.java b/client-adapter/rdb/src/main/java/com/alibaba/otter/canal/client/adapter/rdb/support/SyncUtil.java
--- a/client-adapter/rdb/src/main/java/com/alibaba/otter/canal/client/adapter/rdb/support/SyncUtil.java	(revision ea8949298fc74310990f24c864db218c7ccac312)
+++ b/client-adapter/rdb/src/main/java/com/alibaba/otter/canal/client/adapter/rdb/support/SyncUtil.java	(revision 1f4d326bea570e0db95794d412075c478e7f05ea)
@@ -288,4 +288,24 @@
                 return "";
         }
     }
+    public static String getLimitOneByDbType(String dbTypeName){
+        DbType dbType = DbType.of(dbTypeName);
+        if (dbType == null) {
+            dbType = DbType.other;
+        }
+
+        // 只有当dbType为MySQL/MariaDB或OceanBase时返回limit
+        switch (dbType) {
+            case mysql:
+            case mariadb:
+            case oceanbase:
+                return " limit 1";
+            // 当dbType为oracle 时返回 where rownum<2
+
+            case oracle:
+                return " where rownum<2";
+            default:
+                return "";
+        }
+    }
 }

```

