---
title: JAVA操作XML
date: 2016-12-09 10:56:58
categories: Java
---

### XML文件
``` XML
<template>
    <task id="ddd">aaa</task>
</template>
```

### 构建DOM

``` JAVA
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;
-----------------------
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.parse(this.getClass().getResourceAsStream("/template.xml"));//这里值的是src目录下
```

### 增加元素

``` JAVA
Element element = doc.createElement("task");//创建元素
element.setAttribute("id","AAA");//添加属性
element.appendChild(doc.createTextNode("DDD"));//添加文本节点
doc.getDocumentElement().appendChild(element);//追加到根节点元素中
TransformerFactory tFactory =TransformerFactory.newInstance();
Transformer transformer = tFactory.newTransformer();
DOMSource source = new DOMSource(doc);
StreamResult result = new StreamResult(new FileOutputStream(this.getClass().getResource("/template.xml").getPath()));
transformer.transform(source, result);//写入到文件
```
### 删除元素

``` JAVA
doc.getDocumentElement().removeChild(doc.getElementsByTagName("task").item(0));//删除根节点中第一个task元素
TransformerFactory tFactory =TransformerFactory.newInstance();
Transformer transformer = tFactory.newTransformer();
DOMSource source = new DOMSource(doc);
StreamResult result = new StreamResult(new FileOutputStream(this.getClass().getResource("/template.xml").getPath()));
transformer.transform(source, result);//写入到文件
```


### 修改元素

``` JAVA
doc.getElementsByTagName("task").item(0).getFirstChild().setNodeValue("ccc");//修改第一个task元素中的内容
TransformerFactory tFactory =TransformerFactory.newInstance();
Transformer transformer = tFactory.newTransformer();
DOMSource source = new DOMSource(doc);
StreamResult result = new StreamResult(new FileOutputStream(this.getClass().getResource("/template.xml").getPath()));
transformer.transform(source, result);//写入到文件
```

### 查看元素
``` JAVA
NodeList tasks = doc.getDocumentElement().getChildNodes();
for(int i = 0;i < tasks.getLength();i++){
    System.out.println(((Element)tasks.item(i)).getAttribute("id"));//输出元素的id属性值
    System.out.println(tasks.item(i).getNodeValue);//输出元素的内容值
}
```





