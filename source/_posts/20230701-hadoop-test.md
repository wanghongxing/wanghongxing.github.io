hadoop test



生成新的文本文件 

```
echo "I love runoob" >input.txt
echo "I love runoob" >>input.txt
echo "I love runoob" >>input.txt
echo "I love runoob" >>input.txt
```

把文件放到hdfs上：

```
hadoop fs -put input.txt /input.txt
```

