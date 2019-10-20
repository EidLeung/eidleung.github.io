---
layout:      post
classify:    "Spring Cloud"
title:       "Eureka服务界面改造"
subtitle:    "在Eureka服务界面直接控制服务实例的状态"
date:        2019-11-11
header-img: "img/work/digital-gb.jpg"
catalog:     true
author:      "EidLeung"
tags:
    - Spring Cloud
    - Eureka
---


<div align="center">
	<font size = 8><b>Eureka服务界面改造</b></font>
</div>

## 1. 拷贝相应的文件到工程
1. 将`spring-cloud-netflix-erueka-server`下`template.eureka`文件下拷贝在自己的`eureka`工程下。
![From](/img/work/eureka/sourceFrom.png)
<center>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;">spring-cloud-netflix-erueka-server</div>
</center>
![to](/img/work/eureka/sourceto.png)
<center>
	<div style="color:orange; border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">本地eureka工程</div>
</center>

<div>
	<font color=red size = 6><b>注：</b></font><code class="highlighter-rouge">template.eureka</code>下所有的文件都要拷贝。
</div>

# 2. 加入服务状态修改连接
修改**status.ftl**文件在第57行`</#if><#if instance_has_next>,</#if>`的第一个`</#if>`后面添加如下内容：
```
&emsp;&emsp;&emsp;&emsp;
<#if instanceInfo.isNotUp>
   <a href="/eureka/apps/${app.name}/${instance.id}/status?value=UP" option="PUT" class="base-option"><font color=green size=+1><b>UP</b></font></a>
<#else>
    <a href="/eureka/apps/${app.name}/${instance.id}/status?value=DOWN" option="PUT" class="base-option"><font color=red size=+1><b>DOWN</b></font></a>
</#if>
&emsp;&emsp;<a href="/eureka/apps/${app.name}/${instance.id}" option="DELETE" class="base-option"><font color=yellow size=+1><b>DELETE</b> </font></a>
```

## 3. 使用ajax发送PUT和DELETE请求
### 3.1. 引入jquery。  
在本地工程中加入jquery
![import](/img/work/eureka/importJquery.jpg)
### 3.2. 修改**status.ftl**文件
1. 在文件尾部script区域引入jquery，添加如下代码：
```
<script type="text/javascript" src="jquery-2.2.4.min.js" ></script>
```
2. 添加POST请求转换成PUT和DELETE请求的代码：
```
$(".base-option").on("click", function(event) {
	event.preventDefault();
	$.ajax({
		url: $(this).attr("href"),
		type: "POST",
		data: {_method: $(this).attr("option")},
		success: function(data) {
			location.reload();
			console.log(data);
		},
		error: function(result){
			alert(result);
		}
	});
	return false;
})
```
如下图所示：
![add](/img/work/eureka/addJquery.png)

## 4. 添加配置
通过配置`eureka.server.enable-debug`来控制是否显示，默认`false`不显示。
### 4.1. 添加**HandlerInterceptor**
```
@Component
public class EnableDebugInterceptor implements HandlerInterceptor {
	@Value("${eureka.server.enable-debug}")
	private boolean enableDebugEureka = false;
	
	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
		if (null != modelAndView) {
			modelAndView.addObject("enableDebugEureka", enableDebugEureka);
		}
	}
}
```

### 4.2. 添加**WebMvcConfigurer**
```
@Configuration
public class EnableDebugConfig implements WebMvcConfigurer {
	@Autowired
	private EnableDebugInterceptor enableDebugInterceptor;
	
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		registry.addInterceptor(enableDebugInterceptor).addPathPatterns("/**");
	}
}
```

## 5. 将连接代码包裹在`enableDebugEureka`中
![enabledebug](/img/work/eureka/enabledebug.jpg)
注意后面有一个`<br/>`，否则**UP**和**DOWN**状态会显示在一行。
## 6. 效果展示
### 6.1. 在yml中加入如下配置
```
	eureka:
	  server:
	    enable-debug: true
```
### 6.2. Eureka界面展示
![效果图](/img/work/eureka/rendering.jpg)
通过DOWN、UP来改变实例状态。
<div>
	<font color=green size = "6"><b>附：</b></font>在生产环境中不想显示时，只需要将<code class="highlighter-rouge">eureka.server.enable-debug</code>设置成<code class="highlighter-rouge">false</code>或者删除即可。
</div>