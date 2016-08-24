---
title: JSP 总结 
tags: 
grammar_cjkRuby: true
---

[TOC]


##  JSTL

* Java 中一个定制标记库集
* 代码复用
* jstl.jar 和 standard.jar,版本和servlet版本相关

JSP 使用
* <%@ taglib prefix="c" uri="...">
* <c:out value="hello world">

四大分类
* 核心标签
	* 表达式控制标签:out set remove catch
	* 流程控制标签: if choose when otherwise
	* 循环标签: forEach forTokens
	* URL 操作标签: import url redirect 
* 格式化标签
* SQL 标签
* XML 标签

EL 表达式:Expression Language
- 普通写法 <%=session.getValue("name") %>
- EL 表达式 <c:out value="${sessionScope.name}" />
- EL 表达式格式
	- $ 定界,内容在花括符{}中
	- 含特殊字符用 ${user["first-name"]},不是 ${user.first-name}
	- 通过变量取值 ${user[param]}

EL 变量
JSP 内置变量:Page,Request,Session,Application
对应 EL 名称:PageScope,RequestScope,SessionScope,ApplicationScope
查找顺序 Page->Request->Session->Application,没有则为空

EL 自动类型转换

EL 运算符
算术运算符:+-*/
关系运算符:--,!=
逻辑运算符:&&,||
验证运算符:empty

**set**
- 传值到 scope 中
<c:set value="today" var="day" scop="session"></c:set>
<c:set var="day" scop="session">today</c:set>

- 传值到 javaBean 中
<jsp:userBean id="person" class="com.jqk.person">
<c:set target="${person}" property="name" value="jqk">

**remove**
* var 属性必选
* scope 可选

**catch**
<c:catch var=error>
	<c:set target="aa" property=bb">
</c:catch>

<c:out value="${error}"> // 上面的变量

### 流程控制 if
**if**
```html
<form action="f.jsp" method="post">
	<input type="text" name="score" value="${param.score}" />
	<input type="submit" />
</form>
<c:if test="${param.score>=90}" var="result">
	<c:out>恭喜</c:out>
</c:if>
```

**choose/when/otherwise**
```jsp
<c:choose>
	<c:when test="条件" />
	<c:when test ...
	<c:otherwise >
</c:choose>
```

**循环控制标签**
forEach 标签
* var 设定变量名用于从集合中取出元素
* items 要便利的集合
* begin,end 起始地址,终止地址
* step 步长

```jsp
<c:forEach var="fruit" items="${fruits}">
	<c:out value="${fruit}" >


<c:forEach var="fruit" items="${fruits}" begin="1" end="4">
	<c:out value="${fruit}" >


<c:forEach var="fruit" items="${fruits}" begin="1" end="4" step="2">
	<c:out value="${fruit}" >



<c:forEach var="fruit" items="${fruits}" begin="1" end="4" varStatus="fru">
	<c:out value="${fruit}" >
	<c:out value="index属性:${fru.index}"
	<c:out value="count属性:${fru.count}"
	<c:out value="first属性:${fru.first}"
	<c:out value="last属性:${fru.last}"
```

**forTokens**
* 浏览字符串,截取字符串
	* items 被迭代的字符串
	* delims 分隔符
	* var 结果

```jsp
<c:forTokens items="010-202-51-4" delims="-" var="num">
	<c:out value="${num}"
```

### URL 操作
**import**
* 可以把其他静态或动态文件包含在 JSP 中
* <jsp:include> 只包含同一个web应用中文件,<c:import>包含网络中资源
```jsp
<c:import url context var charEncoding varReader
```
- url:资源路劲
- context:其他web工程
- var:以String类型存入被包含文件
- Scope var 变量的jsp范围
- charEncoding 编码
- varReader 以Reader类型存储被包含文件内容


**redirect**
重定向

```jsp
<c:redirect url="abc.jsp">
	<c:param name="username" >lily</c:param>

//获取
<c:out value="${param.username}"
```

**url**
动态生成 URL
* value 表示 URL 路径值
* var 将 url 存储在变量中
* scope var 变量的范围
```jsp
<c:url value="http...${url}" var="newUrl" scope="session">
```

##  jsp 函数
```jsp
<c:out value="${fn:contains("abc","ab")}"

<c:out value="${indexOf("abc","ab")}"

<c:out value="${fn:escapeXml('<book>jk</book>')}"
```


JSP（全称Java Server Pages）是由Sun Microsystems公司倡导和许多公司参与共同创建的一种使软件开发者可以响应客户端请求，而动态生成HTML、XML或其他格式文档的Web网页的技术标准



##  自定义标签

* 提高开发效率
* 节省开发时间
* JSP 页面消除Java代码

1. 继承 TagSupport 类,复写 doStartTag 方法,使用 pageContext.getOut.print(data)
2. 定义标签库描述文件 tld,放在WEB-INF 文件夹下面或子目录下
```xml
/url

	date
	com.jqk.ok
	empty // jsp:接受java表达式,scriptless:文本ER表达式jsp动作,tagdependent:内容直接写入body-content

```







