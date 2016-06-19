---
title:
  JSON 解析
date:
  2015/7/18 22:57
categories: JSON
tags: [Java,JSON,解析]
---
![image](http://ww1.sinaimg.cn/mw690/7c2c72d3gw1f2ojpvulfkj20dg073gm0.jpg)
# JSON (JavaScript Object Notation) -- 一种轻量级的数据交换格式
JSON 的特点是纯文本，易于人理解和编写，并且机器也易于解析和生成。比 XML（可扩展标记语言）更小、更快，更易解析。
## JSON 语法规则
* 数据在名称/值对中
* 数据由逗号分隔
* 花括号保存对象
* 方括号保存数组

## JSON 对象
对象是一个无序的“‘名称/值’对”集合。一个对象以“{”（左括号）开始，“}”（右括号）结束。每个“名称”后跟一个“:”（冒号）；“‘名称/值’ 对”之间使用“,”（逗号）分隔。
```json
{ "firstName":"John" , "lastName":"Doe" }
```
<!-- more -->
## JSON 数组
数组是值（value）的有序集合。一个数组以“[”（左中括号）开始，“]”（右中括号）结束。值之间使用“,”（逗号）分隔。
```json
{
	"employees": [
		{ "firstName":"John" , "lastName":"Doe" },
		{ "firstName":"Anna" , "lastName":"Smith" },
		{ "firstName":"Peter" , "lastName":"Jones" }
	]
}
```
## JSON 解析
在 Java 中对 JSON 进行解析的封装有很多，例如 org.json , json-lib 等众多工具。
### org.json 
[源码下载](https://github.com/douglascrockford/JSON-java )
[文档下载](http://www.json.org/javadoc/org/json/package-summary.html )
* JSONObject 类
 通过 JSONObject 类的构造方法，可以将 String 类型的 JSON 文本转为一个 JSONObject 类对象，如果 JSON 不是一个 JSONObject 对象，报 JSONException 异常。JSONObejct 类提供了很多种方法对该对象进行操作。
```JAVA
public static void main(String[] args) {
	try {
		String json = "{ 'employees': { 'firstName':'John' , 'lastName':'Doe' }}";
		JSONObject jsonObj = new JSONObject(json);
		JSONObject childJSONObj = jsonObj.getJSONObject("employees");
		System.out.println(childJSONObj.getString("firstName")); //John
		childJSONObj.put("age", 18);
		System.out.println(childJSONObj.toString()); //{"firstName":"John","lastName":"Doe","age":18}
		System.out.println(childJSONObj.length()); //3
		System.out.println(childJSONObj.has("age")); //true
		Iterator it = childJSONObj.keys();
		while (it.hasNext()) {
			Object o = it.next();
			System.out.print(o.toString() + " "); //firstName lastName age 
		}
		System.out.println();
		childJSONObj.remove("age");
		System.out.println(childJSONObj.length()); //2
		System.out.println(childJSONObj.has("age")); //false
		childJSONObj.put("firstName", new String("Bob"));
		System.out.println(childJSONObj.toString()); //{"firstName":"Bob","lastName":"Doe"}
		System.out.println(childJSONObj.isNull("birthday")); //true
	} catch (JSONException e) {
		e.printStackTrace();
	}
}
```
 JSONObject 类中分别有两种方法得到 JSON 文本中的值，getXXX(String key) 与 optXXX(String key)。两者的区别在于通过 getXXX 方法取值时，如果键不存在，就会报 JSONException 异常。而通过 optXXX 方法取值时，如果键不存在，会返回空值。该方法还提供了 optXXX(String key,String defaultValue) 方法。如果键不存在，就会返回默认值。
```JAVA
public static void main(String[] args) {
	try {
		String json = "{ 'employees': { 'firstName':'John' , 'lastName':'Doe' , 'age':18 }}";
		JSONObject jsonObj = new JSONObject(json);
		JSONObject childJSONObj = jsonObj.getJSONObject("employees");
		System.out.println(childJSONObj.optString("age")); //18
		System.out.println(childJSONObj.optString("age", "20")); //18
		System.out.println(childJSONObj.optString("birthday")); //
		System.out.println(childJSONObj.optString("birthday", "1979-01-01")); //1979-01-01
		System.out.println(childJSONObj.getString("age")); //18
		System.out.println(childJSONObj.getString("birthday")); //org.json.JSONException: JSONObject["birthday"] not found.
	} catch (JSONException e) {
		e.printStackTrace();
	}
}
```
* JSONArray 类
 通过 JSONArray 类的构造方法，可以将 String 类型的 JSON 文本转为一个 JSONArray 类对象，如果 JSON 不是一个 JSONArray 对象，报 JSONException 异常。JSONArray 类也提供了很多种方法对该对象进行操作，其中大部分方法和 JSONObejct 类提供方法相似。
```JAVA
public static void main(String[] args) {
	try {
		String json = "[{'firstName':'John','lastName':'Doe'},{'firstName':'Anna','lastName':'Smith'},{'firstName':'Peter','lastName':'Jones'}]";
		JSONArray jsonArray = new JSONArray(json);
		System.out.println(jsonArray.length()); //3
		for (int i = 0; i < jsonArray.length(); i++) {
			System.out.print(jsonArray.get(i) + " "); //{"firstName":"John","lastName":"Doe"} {"firstName":"Anna","lastName":"Smith"} {"firstName":"Peter","lastName":"Jones"} 
		}
		System.out.println();
	} catch (JSONException e) {
		e.printStackTrace();
	}
}
```
* JSONStringer 类
 JSONStringer 类提供了一个快捷和方便的构造JSON文本的方法。
 JSONStringer 的 object() 方法表示JSON文本是以一个对象开始。key() 方法表示添加一个键，value() 方法表示添加一个键对应的值。endObject() 方法表示对象的结束。对应的 Array() 与 endArray() 方法表示数组的开始与结束。
```JAVA
public static void main(String[] args) {
	try {
		JSONStringer jsonStr = new JSONStringer();
		JSONObject jsonObj = new JSONObject();
		JSONObject jsonHeader = new JSONObject();
		jsonHeader.put("userid", "0001");
		JSONObject jsonBody = new JSONObject();
		jsonBody.put("username", "simon");
		jsonBody.put("age", 20);
		jsonObj.put("header", jsonHeader);
		jsonObj.put("body", jsonBody);
		System.out.println(jsonStr.object().key("root").value(jsonObj).endObject().toString()); //{"root":{"header":{"userid":"0001"},"body":{"age":20,"username":"simon"}}}
	} catch (JSONException e) {
		e.printStackTrace();
	}
}
```
* JSONTokener 类
 与 JSONStringer 类对应的就是 JSONTokener 类，该类用于读取JSON格式的文件。
 从文件中读取一个JSONObject;
 ```JAVA
 JSONObject obj = new JSONObject( new JSONTokener(java.io.Reader));
 ```
 从文件中读取一个JSONArray;
 ```JAVA
 JSONArray obj = new JSONArray( new JSONTokener(java.io.Reader));
 ```
### json-lib
[jar包与源码下载](http://sourceforge.net/projects/json-lib/files/json-lib/json-lib-2.4/ )
注意:使用json-lib需要的依赖包
1. commons-lang [下载](http://commons.apache.org/lang/download_lang.cgi )
2. commons-beanutils [下载](http://commons.apache.org/proper/commons-beanutils/download_beanutils.cgi )
3. commons-collections [下载](http://commons.apache.org/proper/commons-collections/download_collections.cgi )
4. commons-logging [下载](http://commons.apache.org/proper/commons-logging/download_logging.cgi )
5. ezmorph [下载](http://sourceforge.net/projects/ezmorph/ )

建议使用 Maven 对项目进行管理,免去到处下载依赖包的麻烦。参见 [Maven 入门](http://blog.saisimon.net/2015/07/12/maven/ )

json-lib 与 org.json 在JSON文本解析时略有不同，json-lib 使用 JSONObject 类中的静态方法 formObject(Object object) 得到 JSONObject 对象。JSONObject 类中封装了常用的一些方法。
```JAVA
public static void main(String[] args) {
	String json = "{ 'employees': { 'firstName':'John' , 'lastName':'Doe' }}";
	JSONObject jsonObj = JSONObject.fromObject(json);
	JSONObject childJSONObj = jsonObj.getJSONObject("employees");
	System.out.println(childJSONObj.names());//["firstName","lastName"]
	Iterator it = childJSONObj.keys();
	while(it.hasNext()) {
		Object obj = it.next();
		System.out.print(obj + "=" + childJSONObj.get(obj) + " "); //firstName=John lastName=Doe 
	}
	System.out.println();
}
```
其他大部分的封装都与 org.json 中的差不太多，这里就不一一赘述了。

## 参考资料
* [JSON 教程](http://www.w3school.com.cn/json/ )
* [介绍 JSON](http://www.json.org/json-zh.html )
* [org.json文档](http://www.json.org/javadoc/org/json/package-summary.html )