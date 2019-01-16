layout: post
title: 碎片代码
date: 2016-04-28 11:50:50
category:
tags:
---
## 前言
记录一些开发中有用的代码片段(配置、技巧都可以)
## SpringMVC日期处理
配置文件
<!-- more -->
```xml
     <mvc:annotation-driven>
		<!-- 处理responseBody 里面日期类型 -->
		<mvc:message-converters>
			<bean
				class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
				<property name="objectMapper">
					<bean class="com.fasterxml.jackson.databind.ObjectMapper">
						<property name="dateFormat">
							<bean class="java.text.SimpleDateFormat">
								<constructor-arg type="java.lang.String" value="yyyy-MM-dd" />
							</bean>
						</property>
					</bean>
				</property>
			</bean>
		</mvc:message-converters>
	</mvc:annotation-driven>
```
还需要在entity时间字段中添加注解:
```java
	@DateTimeFormat(pattern="yyyy-MM-dd")
	private Date startTime;
```
## Easyui Treegrid选择
1.4.5版本之下(1.4.5新增级联属性),处理Treegrid复选框问题(加载完数据默认勾选,能够单独勾选父节点、子节点:通过操作DOM节点实现),返回的JSON格式带checked
```javascript
            onLoadSuccess: function(row,data){
                $(this).treegrid("expandAll");
                /*
                $(this).treegrid("clearChecked");
                var rows = data.rows;
                for(i in rows){
                    console.info(rows[i].checked)
                    if(rows[i].checked==true){
                        //$(this).treegrid("checkRow",rows[i].id);
                    }else {
                        $(this).treegrid("uncheckRow",rows[i].id);
                    }
                }
                */

            },
            onCheck: function(row){
                var kids = $(this).treegrid("getChildren",row.id);
                for(var i in kids){
                    $(this).treegrid("checkRow",kids[i].id);
                }
            },
            onUncheck: function(row){
                /*
                var parent = $(this).treegrid("getParent",row.id);
                while(parent){
                    var tr = $("[node-id="+parent.id+"]");
                    tr.removeClass("datagrid-row-checked")
                    tr.find("div.datagrid-cell-check input[type=checkbox]").prop("checked",false);
                    //$(this).treegrid("uncheckRow",parent.id);
                    parent = $(this).treegrid("getParent",parent.id);
                }
                */
                var kids = $(this).treegrid("getChildren",row.id);

                for(var i in kids){
                    var tr = $("[node-id="+kids[i].id+"]");
                    tr.removeClass("datagrid-row-checked")
                    tr.find("div.datagrid-cell-check input[type=checkbox]").prop("checked",false);
                    $(this).treegrid("uncheckRow",kids[i].id);
                }
            }

```