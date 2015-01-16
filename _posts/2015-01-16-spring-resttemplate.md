---
layout: post
title: Spring 中 RestTemplate 的使用
categories: [Java]
tags: [Java, Spring, RestTemplate]
description: 最近 CMS 系统在做升级，从 0.2 的老架构一跃升级到 2.0 的新架构，2.0 中一项大的改变就是通过 Shiro 来管理权限。为了让 0.2 的功能可以继续使用，同时让 2.0 的权限管理生效，我们决定把 2.0 作为中间层，把前端指向 0.2 的请求，用 2.0 进行转发。为了把 Spring 用到底，故采用 Spring-Web-4.0 包中的 RestTemplate。

---

##背景
最近 CMS 系统在做升级，从 0.2 的老架构一跃升级到 2.0 的新架构，2.0 中一项大的改变就是通过 Shiro 来管理权限。为了让 0.2 的功能可以继续使用，同时让 2.0 的权限管理生效，我们决定把 2.0 作为中间层，把前端指向 0.2 的请求，用 2.0 进行转发。为了把 Spring 用到底，故采用 Spring-Web-4.0 包中的 RestTemplate。

	浏览器打开 2.0 页面（和 0.2 完全一样） → 提交到 2.0 后台 → 转发到 0.2 后台 → 2.0 接收返回值 → 浏览器渲染

##1. 获取 HttpMethod

HttpMethod 是指前台提交到后台的数据是 Get 方法，还是 Post 方法。RestTemplate 对于 Get 和 Post 的处理方式是有区别的，这里我先得到 HttpMethod 的值。

{% highlight java %}
private HttpMethod httpMethod(HttpServletRequest httpServletRequest){
	String method = httpServletRequest.getMethod();
	return HttpMethod.valueOf(method);
}
{% endhighlight %}

##2. 获取 HttpHeaders

HttpHeaders 中包含 cookie，contentType 等信息，这里将 HttpHeaders 提取出来，供 RestTemplate 使用。

{% highlight java %}
private HttpHeaders httpHeaders(HttpServletRequest httpServletRequest) {
	HttpHeaders httpHeaders = new HttpHeaders();
	Enumeration headerNames = httpServletRequest.getHeaderNames();
	while (headerNames.hasMoreElements()) {
		String key = (String) headerNames.nextElement();
		String value = httpServletRequest.getHeader(key);
		httpHeaders.add(key, value);
	}
	return httpHeaders;
}
{% endhighlight %}

##3. 处理普通请求

普通请求是指 Get 请求，或者是 Post 表单提交，提交的数据不包含图片视频等流信息，返回的可以是 Json 数据，也可以是页面。

在请求中有可能遇到 302 的情况，我这里只做了简单处理，就是取得转发的 location 后再请求一次（希望第二次请求不在碰上 302 吧）。

另外就是注意 Tomcat 和 Weblogic 对于 302 的处理也是有区别的，返回值要根据不同的容器，进行相应的解析。

{% highlight java %}
private ResponseEntity<String> excute(String url, HttpHeaders headers, MultiValueMap bodies, HttpMethod httpMethod) {
	logger.debug("request headers = " + JSON.toJSONString(headers));
	logger.debug("request bodies = " +JSON.toJSONString(bodies));
	logger.debug("request url = " + url);
	HttpEntity requestEntity = new HttpEntity(bodies, headers);
	if (httpMethod == HttpMethod.GET) {
		url = (UriComponentsBuilder.fromHttpUrl(url).queryParams(bodies)).build().toUriString();
	}
	
	ResponseEntity result;
	try {
		result = restTemplate.exchange(url, httpMethod, requestEntity, String.class);
	} catch (RestClientException e) {
		// tomcat 和 weblogic 对于 302 的处理不一样，tomcat 中 body 为普通字符串，weblogic 为流
		result = restTemplate.exchange(url, httpMethod, requestEntity, byte[].class);
	}
	
	HttpHeaders resultHeaders = new HttpHeaders();
	if (result.getHeaders().getContentType() != null) {
		resultHeaders.setContentType(result.getHeaders().getContentType());
	}
	
	if (HttpStatus.FOUND == result.getStatusCode()) {
		url = result.getHeaders().getLocation().toString();
		requestEntity = new HttpEntity(null, headers);
		result = restTemplate.exchange(url, HttpMethod.GET, requestEntity, String.class);
	}
	
	logger.debug("response headers = " + JSON.toJSONString(result.getHeaders()));
	logger.debug("response bodies = " +JSON.toJSONString(result.getBody()));
	logger.debug("response statusCode = " + result.getStatusCode());
	
	return new ResponseEntity(result.getBody(), resultHeaders, result.getStatusCode());
}
{% endhighlight %}

##4. 处理流

处理流最重要的一点就是能取到 HttpServletRequest.getInputStream()。但这个 InputStream 很特殊，一旦之前被调用过一次取值方法（不止一个 getInputStream()），那么再取值就必定为空，因此我首先要保证 HttpServletRequest 是没有被拦截到的。

在 Spring 中，如果配置了Bean：CommonsMultipartResolver，那么对于 ContentType 是 multipart/form-data 的 HttpServletRequest，一定会转换为它自己的 Request，这期间必定会读取 InputStream。因此要保证 Spring 的配置文件中没有这个 Bean。

{% highlight java %}
@SuppressWarnings("unchecked")
private ResponseEntity<String> excute(String url, final HttpServletRequest httpServletRequest) {
	logger.debug("request url = " + url);
	ResponseEntity result;
	
	SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
	requestFactory.setBufferRequestBody(false);     
	restTemplate4Stream.setRequestFactory(requestFactory); 
	final RequestCallback requestCallback = new RequestCallback() {
		@Override
		public void doWithRequest(final ClientHttpRequest clientHttpRequest) throws IOException {
			clientHttpRequest.getHeaders().setAll(httpHeaders(httpServletRequest).toSingleValueMap());
			IOUtils.copy(httpServletRequest.getInputStream(), clientHttpRequest.getBody());
		}
	};
	final TransferResponseEntityResponseExtractor responseExtractor= new TransferResponseEntityResponseExtractor(String.class, restTemplate4Stream.getMessageConverters());
	result = (ResponseEntity)restTemplate4Stream.execute(
			url, 
			HttpMethod.POST, 
			requestCallback, 
			responseExtractor);
	
	HttpHeaders resultHeaders = new HttpHeaders();
	if (result.getHeaders().getContentType() != null) {
		resultHeaders.setContentType(result.getHeaders().getContentType());
	}
	
	logger.debug("response headers = " + JSON.toJSONString(result.getHeaders()));
	logger.debug("response bodies = " +JSON.toJSONString(result.getBody()));
	logger.debug("response statusCode = " + result.getStatusCode());
	
	return new ResponseEntity(result.getBody(), resultHeaders, result.getStatusCode());
}
{% endhighlight %}

TransferResponseEntityResponseExtractor 是我自己定义的一个对象，它能把 restTemplate4Stream.execute 最终执行的结果转换为 ResponseEntity。

##5. 拦截请求
处理普通请求和处理流，使用了不同的 RestTemplate，这是因为 restTemplate4Stream.execute 进行回调的时候，将 callback 写入了本身，此时如果用 同一个 RestTemplate 进行 exchange 是有问题的。

{% highlight java %}
@RequestMapping("**")
public ResponseEntity<String> visit(
		@CurrentUser CoreUser coreUser, 
		@RequestParam MultiValueMap bodies, 
		HttpServletRequest httpServletRequest) {
	
	validatePermit(httpServletRequest);
	String url = urlPrefix + "/" + urlSuffix(httpServletRequest);
	if (httpServletRequest.getContentType()!=null && httpMethod(httpServletRequest)==HttpMethod.POST &&
			(httpServletRequest.getContentType().contains("multipart/form-data") || httpServletRequest.getContentType().contains("application/octet-stream"))) {
		return excute(url, httpServletRequest);
	}else {
		return excute(url, httpHeaders(httpServletRequest), bodies, httpMethod(httpServletRequest));
	}
}
{% endhighlight %}

