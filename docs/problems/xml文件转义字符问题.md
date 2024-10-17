### 问题描述

```shell
Unexpected character '=' (code 61); expected a semi-colon after the reference for entity 'useUnicode
```

xml 文件中出现特殊字符，需要转义。

### 错误原因

在XML文档中的所有文本都会被解析器解析，非法的 XML 字符必须被替换为实体引用（entity reference）。

假如您在 XML 文档中放置了一个类似 "&" 字符，那么这个文档会产生一个错误，这是因为解析器会把它解释为新元素的开始。因此你不能这样写：

```xml
<message>aaa & bbb</message>
```


为了避免此类错误，需要把字符 "&" 替换为实体引用，就像这样：

```xml
<message> aaa &amp; bbb </message>
```

### 解决方法

Xml文件中不能使用&，要使用他的转义&amp;来代替。

其余转义字符：

转义字符	特殊符号	 

| 转义字符 | 特殊符号 | 中文名称 |
| -------- | -------- | -------- |
| <        | <        | 小于     |
| &le;     | <=       | 小于等于 |
| &gt;     | >        | 大于     |
| &ge;     | >=       | 大于等于 |
| &amp;    | &        | 且       |
| &apos;   | '        | 单引     |
| &quot;   | "        | 双引     |


**注意点**
转义序列各字符间不能有空格；
转义序列必须以 ";" 结束；
单独的&不被认为是转义开始；
区分大小写。
