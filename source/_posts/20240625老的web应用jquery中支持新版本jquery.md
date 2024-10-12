20240625老的web应用jquery中支持新版本jquery

老的项目中使用了1.6.x的jquery，需要升级其中的弹窗为layui，但是layui需要至少1.10.x，修改方式如下，将原来单行的jquery引用，改为保护多版本jquery，添加layui引用，最后恢复原来的jquery版本号，确保不影响原来其它jquery使用的地方。

```
<script src="../js/jquery-1.3.2.min.js" type="text/javascript"></script>
  
```

```
 <script type="text/javascript" src="../js/lib/jquery.js"></script>
<script type="text/javascript" src="../js/jquerynew.js"></script>
<script type="text/javascript" src="../js/layer/js/layer.js"></script>
<script  type="text/javascript" >var $jq = jQuery.noConflict(true);console.log($().jquery);</script>

```

顺便做一个批量替换的脚本

```python
# -*- coding:utf-8 -*-
#@Author: whx

import os
import re
import glob
#构建一个文本文件内容查找函数
def replace_text(file_path, search_str):
    try:
        with open(file_path, 'r', encoding='utf-8') as f ,open("%s.bak"%file_path, 'w', encoding='utf-8') as fw:
            data = f.read()
            res = re.findall(search_str, data)
            if res :
                newData = re.sub(search_str, r'<script type="text/javascript" src="\g<2>/jquery-\g<3>.js"></script>\n<script type="text/javascript" src="\g<2>/jquerynew.js"></script>\n<script type="text/javascript" src="\g<2>/layer.js"></script>\n<script  type="text/javascript" >var $jq = jQuery.noConflict(true);console.log($().jquery);</script>' , data)
                fw.write(newData)
                f.close()
                fw.close()
                os.remove(file_path)
                os.rename("%s.bak" % file_path, file_path)
            else:
                f.close()
                fw.close()
                os.remove("%s.bak" % file_path)
 
    except UnicodeDecodeError:
        with open(file_path, 'r', encoding='gbk') as f ,open("%s.bak"%file_path, 'w', encoding='gbk') as fw:
            data = f.read()
            res = re.findall(search_str, data)
            if res :
                newData = re.sub(search_str, r'<script type="text/javascript" src="\g<2>/jquery-\g<3>.js"></script>\n<script type="text/javascript" src="\g<2>/jquerynew.js"></script>\n<script type="text/javascript" src="\g<2>/layer.js"></script>\n<script  type="text/javascript" >var $jq = jQuery.noConflict(true);console.log($().jquery);</script>' , data)
                fw.write(newData)
                f.close()
                fw.close()
                os.remove(file_path)
                os.rename("%s.bak" % file_path, file_path)
            else:
                f.close()
                fw.close()
                os.remove("%s.bak" % file_path)
files = glob.glob("D:\source\project/**/*.aspx*",recursive=True)
search_str = "<script(.*)src=\"(.*)/jquery-(.*).js\"(.*)>(.*)</script>"
search_str_reg = re.compile(search_str,flags=re.IGNORECASE)

for file_path in files:
   replace_text(file_path, search_str_reg )
  

```

