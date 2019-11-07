---
title: JSP转HTML中EL表达式替换方式
categories:
  - 前端
tags:
  - JSP
  - EL表达式
date: 2019-11-06 17:12:20
---

### 目录
* [1.缘由](#1-缘由)
* [2.目的](#2-目的)
* [3.具体实施](#3-具体实施)
    * [3.1 事前准备](#3-1事前准备)
    * [3.2 实施并验证](#3-2实施并验证)
* [4.为什么采用以上方式](#4-为什么采用以上方式)

## 1.缘由

工作中遇到老项目写的jsp采用了较多EL表达式导致无法前后分离，需要替代掉EL表达式，转化为html后再进行前后分离。

## 2.目的

能把包含EL表达式的JSP转化为HTML

## 3.具体实施

采用HandleBar前端插件替换EL表达式。

### 3.1事前准备

HandleBar基础用法

### 3.2实施并验证

**将一个用EL表达式写的导航转化为HandleBar来做**

**源EL表达式：**
```
<!--菜单输出开始-->
<ul id="menu_ul">
<c:forEach var="firstMenu" items="${menus}" varStatus="status">
	<li class="second" <c:if test="${!empty firstMenu.URL}"> onclick="navClick('${firstMenu.ID}','${firstMenu.NAME}',
		<c:choose>
			<c:when test="${fn:startsWith(firstMenu.URL, 'http')}">'${firstMenu.URL}'</c:when>
			<c:otherwise>webRoot + '${firstMenu.URL}'</c:otherwise>
		</c:choose> )"</c:if>>
		<!--一级菜单输出-->
		<c:choose>
			<c:when test="${fn:contains(firstMenu.iconClass, '/')}">
				<img src="<%=webRoot%>${firstMenu.iconClass}" />
			</c:when>
			<c:otherwise>
				<i class='<c:if test="${!empty firstMenu.iconClass}">${firstMenu.iconClass}</c:if><c:if test="${empty firstMenu.iconClass}">icon-map-marker</c:if>' title='${firstMenu.NAME}'></i>
			</c:otherwise>
		</c:choose>
		<p>${firstMenu.NAME}</p>

		<!-- 二层 -->
		<ul>
			<c:forEach var="secondMenu" items="${firstMenu.senconds}">
				<li <c:if test="${!empty secondMenu.URL}"> onclick="navClick('${secondMenu.ID}','${secondMenu.NAME}',
					<c:choose>
						<c:when test="${fn:startsWith(secondMenu.URL, 'http')}">'${secondMenu.URL}'</c:when>
						<c:otherwise>webRoot + '${secondMenu.URL}'</c:otherwise>
					</c:choose> )"</c:if>>
					<i class='<c:if test="${!empty secondMenu.iconClass}">${secondMenu.iconClass}</c:if><c:if test="${empty secondMenu.iconClass}">icon-info-sign</c:if>'></i>
					<span>${secondMenu.NAME}
						<c:if test="${!empty secondMenu.thirds}">
							<i class="icon-angle-right"></i>
						</c:if>
					</span>
					<!--二级菜单输出-->
					<c:if test="${!empty secondMenu.thirds}">

						<c:if test="${secondMenu.haveFourth!=1}">

							<!-- 三层 -->
							<ul class='fourth'>

								<c:forEach var="thirdMenu" items="${secondMenu.thirds}">
									<!--三级菜单输出-->
									<li class='third orange width2 height1' <c:if test="${!empty thirdMenu.URL}"> onclick="navClick('${thirdMenu.ID}','${thirdMenu.NAME}',
										<c:choose>
											<c:when test="${fn:startsWith(thirdMenu.URL, 'http')}">'${thirdMenu.URL}'</c:when>
											<c:otherwise>webRoot + '${thirdMenu.URL}'</c:otherwise>
										</c:choose> )"</c:if>>

						<span>${thirdMenu.NAME}</span>
						<img src="<%=webRoot%>/page/images/img.png" onerror="this.src='<%=webRoot%>/page/images/unvisited.png'" />



						</li>
						</c:forEach>
						</ul>

					</c:if>

					<c:if test="${secondMenu.haveFourth==1}">
						<ul class="thirds">
							<c:forEach var="thirdMenu" items="${secondMenu.thirds}">
								<!--三级菜单输出-->
								<li <c:if test="${!empty thirdMenu.URL}"> onclick="navClick('${thirdMenu.ID}','${thirdMenu.NAME}',
									<c:choose>
										<c:when test="${fn:startsWith(thirdMenu.URL, 'http')}">'${thirdMenu.URL}'</c:when>
										<c:otherwise>webRoot + '${thirdMenu.URL}'</c:otherwise>
									</c:choose> )"</c:if>>
					<i class='<c:if test="${!empty thirdMenu.iconClass}">${thirdMenu.iconClass}</c:if><c:if test="${empty thirdMenu.iconClass}">icon-info-sign</c:if>'></i>
					<span>${thirdMenu.NAME}
						<c:if test="${!empty thirdMenu.fourths}">
							<i class="icon-angle-right"></i>
						</c:if>
					</span>
					<!--二级菜单输出-->
					<c:if test="${!empty thirdMenu.fourths}">

						<ul class='fourth'>
							<c:forEach var="fourthMenu" items="${thirdMenu.fourths}">
								<li class='third orange width2 height1' <c:if test="${!empty fourthMenu.URL}"> onclick="navClick('${fourthMenu.ID}','${fourthMenu.NAME}',
									<c:choose>
										<c:when test="${fn:startsWith(fourthMenu.URL, 'http')}">'${fourthMenu.URL}'</c:when>
										<c:otherwise>webRoot + '${fourthMenu.URL}'</c:otherwise>
									</c:choose> )"</c:if>>
					<span>${fourthMenu.NAME}</span>
					<img src="<%=webRoot%>/page/images/img.png"  onerror="this.src='<%=webRoot%>/page/images/unvisited.png'" />

					</li>
					</c:forEach>
					</ul>
					</c:if>
					</li>
					</c:forEach>
					</ul>
					</c:if>
					</c:if>
				</li>

			</c:forEach>
		</ul>
	</li>
</c:forEach>
</ul>
```

**更改之后的HandleBar表达式**
```
<script id="menu_html" type="text/x-handlebars-template">
    {{!-- 菜单输出开始 --}}
    {{#each menus}}

    {{!-- 一级菜单输出 --}}
    {{#if URL}}
    <li class="second" onclick="navClick('{{ID}}','{{NAME}}','{{URL}}')">
        {{else}}
    <li class="second">
        {{/if}}

        {{#if iconImg}}
        <img src="{{iconImg}}"/>
        {{else}}
        {{#if iconClass}}
        <i class="{{iconClass}}" title="{{NAME}}"></i>
        {{else}}
        <i class="icon-map-marker" title="{{NAME}}"></i>
        {{/if}}
        {{/if}}


        <p>{{NAME}}</p>

        {{!-- 二级菜单输出 --}}
        <ul>
            {{#each seconds}}

            {{#if URL}}
            <li onclick="navClick('{{ID}}','{{NAME}}','{{URL}}')">
                {{else}}
            <li>
                {{/if}}

                {{#if iconClass}}
                <i class="{{iconClass}}" title="{{NAME}}"></i>
                {{else}}
                <i class="icon-info-sign" title="{{NAME}}"></i>
                {{/if}}

                {{#if thirds}}
                <span>{{NAME}}<i class="icon-angle-right"></i></span>
                {{else}}
                <span>{{NAME}}</span>
                {{/if}}

                {{!-- 三级菜单输出 --}}
                {{#if haveFourth}}
                <ul class="thirds">
                    {{#each thirds}}

                    {{#if URL}}
                    <li onclick="navClick('{{ID}}','{{NAME}}','{{URL}}')">
                        {{else}}
                    <li>
                        {{/if}}

                        {{#if iconClass}}
                        <i class="{{iconClass}}" title="{{NAME}}"></i>
                        {{else}}
                        <i class="icon-info-sign" title="{{NAME}}"></i>
                        {{/if}}

                        {{#if fourths}}
                        <span>{{NAME}}<i class="icon-angle-right"></i></span>
                        <ul class='fourth'>
                            {{#each fourths}}
                            {{#if URL}}
                            <li class='third orange width2 height1' onclick="navClick('{{ID}}','{{NAME}}','{{URL}}')">
                                {{else}}
                            <li>
                                {{/if}}
                                <span>{{NAME}}</span>
                                <img src="./images/nav_default.jpg" onerror="this.src='./images/unvisited.png'"/>
                            </li>
                            {{/each}}

                        </ul>
                        {{else}}
                        <span>{{NAME}}</span>
                        {{/if}}
                        {{/each}}
                </ul>

                {{else}}

                {{#if thirds}}
                <ul class='fourth'>
                    {{#each thirds}}

                    {{#if URL}}
                    <li class='third orange width2 height1' onclick="navClick('{{ID}}','{{NAME}}','{{URL}}')">
                        {{else}}
                    <li>
                        {{/if}}
                        <span>{{NAME}}</span>
                        <img src="./images/nav_default.jpg" onerror="this.src='./images/unvisited.png'"/>
                    </li>
                    {{/each}}
                </ul>
                {{/if}}

                {{/if}}

            </li>
            {{/each}}
        </ul>

    </li>
    {{/each}}
</script>
```

**HandleBar模板以及原始数据的js处理：**
```

    $.post('http://xxx?menus', '', function (data) {
        eachFirstNav(data.menus);

        var menuHTML = $("#menu_html").html();
        var template = Handlebars.compile(menuHTML);
        var html = template(data);

        let menuDom = $('#menu_ul');
        menuDom.html(html);

        navInit01();
    }, 'json')

    function eachFirstNav(data) {

        for(let item in data){
            if(data[item].URL === undefined) continue;
            if (data[item].iconClass.indexOf('/') !== -1){
                data[item].iconImg = webRoot + data[item].iconClass
            }
            if (data[item].URL !== '' ) {
                if (data[item].URL.indexOf('http') === -1) {
                    data[item].URL = webRoot + data[item].URL ;
                }
            } else {
                eachSecondsNav(data[item].seconds);
            }
        }
    }

    function eachSecondsNav(data) {
        for(let item in data){
            //若有4层导航，样式等需要替换，由于HandleBar的IF/ELSE判断无法判断具体逻辑，只能判断非空，故作此处理
            if (data[item].haveFourth === -1) {
                data[item].haveFourth = '';
            }
            if(data[item].URL === undefined) continue;
            if (data[item].URL !== '') {
                if (data[item].URL.indexOf('http') === -1) {
                    data[item].URL = webRoot + data[item].URL ;
                }
            } else {
                eachThirdsNav(data[item].thirds);
            }
        }
    }

    function eachThirdsNav(data) {
        for(let item in data){
            if(data[item].URL === undefined) continue;
            if (data[item].URL !== '') {
                if (data[item].URL.indexOf('http') === -1) {
                    data[item].URL = webRoot + data[item].URL ;
                }
            } else {
                eachFourthNav(data[item].thirds);
            }
        }
    }

    function eachFourthNav(data) {
        for(let item in data){
            if(data[item].URL === undefined) continue;
            if (data[item].URL !== '') {
                if (data[item].URL.indexOf('http') === -1) {
                    data[item].URL = webRoot + data[item].URL ;
                }
            } else {
                return;
            }
        }
    }
```

**源测试数据:**
```
{
    "menus": [
        
        {
            "ID": "a25b2t4aastb",
            "CREATEDATETIME": 1484647359000,
            "DESCRIPTION": "",
            "ICONCLS": "",
            "NAME": " 视图管理",
            "SEQ": 5,
            "TARGET": "",
            "UPDATEDATETIME": 1503631792000,
            "URL": "",
            "SYRESOURCE_ID": "",
            "SYRESOURCETYPE_ID": "0",
            "color": "",
            "iconClass": "icon-sitemap\t",
            "seconds": [
                {
                    "ID": "ab542b5b25a",
                    "CREATEDATETIME": 1517818707000,
                    "DESCRIPTION": "",
                    "ICONCLS": "ext-icon-folder_user",
                    "NAME": "用户视图",
                    "SEQ": 4,
                    "TARGET": "",
                    "UPDATEDATETIME": 1517818707000,
                    "URL": "http://www.baidu.com",
                    "SYRESOURCE_ID": "haerhh123515",
                    "SYRESOURCETYPE_ID": "0",
                    "haveFourth": -1,
                    "thirds": []
                }
            ]
        },
        {
            "ID": "fawfaf244",
            "CREATEDATETIME": 1458248395000,
            "DESCRIPTION": "管理系统的资源、角色、机构、用户等信息",
            "ICONCLS": "",
            "NAME": "系统管理",
            "SEQ": 40,
            "TARGET": "",
            "UPDATEDATETIME": 1502706964000,
            "URL": "",
            "SYRESOURCE_ID": "",
            "SYRESOURCETYPE_ID": "0",
            "color": "xxx-color-blue",
            "iconClass": "/page/image/icon_leftmenu09.png",
            "seconds": [
                {
                    "ID": "afawfa2",
                    "CREATEDATETIME": 1503633639000,
                    "DESCRIPTION": "",
                    "ICONCLS": "",
                    "NAME": "系统维护",
                    "SEQ": 4,
                    "TARGET": "",
                    "UPDATEDATETIME": 1503633639000,
                    "URL": "",
                    "SYRESOURCE_ID": "a22ab3wb2",
                    "SYRESOURCETYPE_ID": "0",
                    "haveFourth": 1,
                    "thirds": [
                        {
                            "ID": "afa2fa",
                            "CREATEDATETIME": 1466414417000,
                            "DESCRIPTION": "",
                            "ICONCLS": "",
                            "NAME": "功能使用统计",
                            "SEQ": 4,
                            "TARGET": "",
                            "UPDATEDATETIME": 1503633686000,
                            "URL": "/xxx.do?ba32b522",
                            "SYRESOURCE_ID": "a2b5235",
                            "SYRESOURCETYPE_ID": "0",
                            "color": "",
                            "iconClass": "icon-unlock-alt",
                            "fourths": []
                        },
                        {
                            "ID": "afa2f2a445",
                            "CREATEDATETIME": 1504236483000,
                            "DESCRIPTION": "数据库监控相关操作",
                            "ICONCLS": "",
                            "NAME": "数据库监控",
                            "SEQ": 14,
                            "TARGET": "",
                            "UPDATEDATETIME": 1504236483000,
                            "URL": "",
                            "SYRESOURCE_ID": "SYRESOURCE_IDxx",
                            "SYRESOURCETYPE_ID": "0",
                            "fourths": [
                                {
                                    "ID": "2bab525",
                                    "CREATEDATETIME": 1504236647000,
                                    "DESCRIPTION": "数据库信息查看",
                                    "ICONCLS": "",
                                    "NAME": "数据库信息查看",
                                    "SEQ": 1,
                                    "TARGET": "",
                                    "UPDATEDATETIME": 1504236782000,
                                    "URL": "/xxx.do?xxx",
                                    "SYRESOURCE_ID": "SYRESOURCE_IDawdwa",
                                    "SYRESOURCETYPE_ID": "0"
                                }
                            ]
                        }
                    ]
                }
            ]
        }
    ]
}
```

## 4.为什么采用以上方式

此案例中，本质都是js拼接html，但如果纯用js拼接html，会导致html标签结构被破坏，不利于维护，而HandleBar能够解决这个问题。不过要注意的是HandleBar并没有提供复杂的逻辑判断，所以本次处理对源数据用js做了处理以适配HandleBar。


