# HTTP编程 #

* [1.1 HTTP协议详解之URL](#1.1)
 * [1.1.1 URIBuilder工具类](#1.1.1)
 * [1.1.2 UrlEncodedFormEntity工具类](#1.1.2)
* [2.1 HTTP协议详解之请求](#2.1)
 * [2.1.1 HttpClient执行Http请求](#2.1.1)
 * [2.1.2 HttpClient连接管理](#2.1.2)
 * [2.1.3 HttpClient状态管理](#2.1.3)
 * [2.1.4 HttpClient重定向](#2.1.4)
 * [2.1.5 HttpClient授权](#2.1.4)
* [3.1 HTTP协议详解之响应](#3.1)
 * [3.1.1 Http响应](#3.1.1)
 * [3.1.2 Http ResponseHandler处理响应](#3.1.2)
* [4.1 HTTP协议详解之消息报头](#4.1)
  * [4.1.1 Http消息头](#4.1.1)
  * [4.1.2 Http消息体](#4.1.2)
* [5.1 HTTP运行上下文](#5.1)


## 1.1.1 URIBuilder工具类
HttpClient提供URIBuilder工具类进行URL的创建和修改。

     URIBuilder uri = new URIBuilder();
     uri.setScheme("http")
		    .setHost("10.128.91.38:20501")
		    .setPath("/serviceAgent/rest/iot/valid/validAcctCheck")
		    .setParameter("recNumType", "10")
		    .setParameter("accNum", "057465578757")
		    .setParameter("regionId", "8330200")
		    .setParameter("srcSysID", "1111");
		System.out.print(uri.toString());`

## 1.1.2 UrlEncodedFormEntity工具类

        List<NameValuePair>nameValuePairs=new ArrayList<NameValuePair>();
		nameValuePairs.add(new BasicNameValuePair("recNumType", "10"));
		nameValuePairs.add(new BasicNameValuePair("accNum", "057465578757"));
		nameValuePairs.add(new BasicNameValuePair("regionId", "8330200"));
		nameValuePairs.add(new BasicNameValuePair("srcSysID", "1111"));
		UrlEncodedFormEntity entity=new UrlEncodedFormEntity(nameValuePairs,"UTF-8");
		String url=uri.toString()+"?"+EntityUtils.toString(entity);
		System.out.println(url);

##2.1.1 HttpClient执行Http请求

   所有的 HTTP 请求都有一个请求行，包含一个方法名，请求 URI 和 HTTP 协议版本。
HttpClient 支持所有的定义在 HTTP/1.1 规范的 HTTP 原装方法： GET, HEAD, POST, PUT, DELETE, TRACE 和 OPTIONS。每一个方法都有相应的类：HttpGet, HttpHead, HttpPost, HttpPut, HttpDelete, HttpTrace 和 HttpOptions。

    //方式一
    CloseableHttpClient httpClient1=HttpClients.createDefault();//方式1
    HttpGet httpGet=new HttpGet(Url);//字符串和URI类似都可以
    CloseableHttpResponse httpResponse=httpClient1.execute(httpGet);
    httpResponse.close()//关闭流
  
    //方式二
    RequestConfig config = RequestConfig.custom().setConnectTimeout(60000).setSocketTimeout(15000).build();//增加对Http连接的配置
    CloseableHttpClient httpClient1=HttpClientBuilder.create().setDefaultRequestConfig(config).build();
    CloseableHttpResponse httpResponse=httpClient2.execute(httpGet);
    httpResponse.close()//关闭流

##3.1.1 Http响应
HTTP响应是服务器在服务器收到并解析客户端请求返回给客户端的信息。

    CloseableHttpResponse response = httpClient.execute(httpGet);
    int statusCode = response.getStatusLine().getStatusCode();//获取响应状态码
    if (statusCode != 200) {
		httpGet.abort();
		throw new RuntimeException("HttpClient,error status code :"+ statusCode);
	}
	HttpEntity entity = response.getEntity();
	String result = null;
	if (entity != null) {
		result = EntityUtils.toString(entity, "utf-8");
	}
	EntityUtils.consume(entity);
	response.close();
##3.1.2 Http ResponseHandler处理响应
响应处理器（ResponseHandler）来处理HTTP响应。这是执行HTTP请求和处理HTTP响应的推荐方式。这种做法使调用者将注意力集中在处理HTTP响应内容的过程中，并委派任务释放HttpClient所占用的系统资源。ResponseHandler能够保证在任何情况下都会将底层的HTTP连接释放回连接管理器

            HttpGet httpget = new HttpGet(url);
            //打印请求地址
            System.out.println("executing request " + httpget.getURI());
            //创建响应处理器处理服务器响应内容
            ResponseHandler<String> responseHandler = new BasicResponseHandler();
            //执行请求并获取结果
            String responseBody = httpclient.execute(httpget, responseHandler);

##4.1.1 Http消息头
HttpClient 提供了检索、添加、移除和枚举头部的方法，请求和返回操作方式相同。

       //添加报文头
		httpGet.addHeader("X-APP-ID1", "xAppId");
		httpGet.addHeader("X-APP-ID2", "xAppId");
		httpGet.addHeader("X-APP-key1", "xAppkey");
		httpGet.addHeader("X-APP-key2", "xAppkey");
		//删除
		httpGet.removeHeaders("X-APP-ID1");
		httpGet.removeHeaders("X-APP-ID2");
		//检索 fist 和last效果一样都是按名称取head
		Header h1=httpGet.getFirstHeader("X-APP-ID2");//遍历head
		Header h2=httpGet.getLastHeader("X-APP-ID2");
		System.out.println(h1.toString());
		System.out.println(h2.toString());
		//遍历方式
		Header[] heads=httpGet.getAllHeaders();
		for(int i=0;i<heads.length;i++){
			System.out.println(heads[i].toString());
		}
		//采用迭代器的方式
		HeaderIterator it=httpGet.headerIterator();
		while(it.hasNext()){
			System.out.println(it.next().toString());
		}
		
##4.1.2 http消息体
http消息能够携带与请求和响应相关的实体。对于请求一般Post和put通常含有请求封装的实体。对于响应实体一般都含有响应实体，head响应是不包含实体。
HttpClient常用的是用流实现的，在读取流后要关闭流。同时可以使用使用流式实体时，可以调用 EntityUtils#consume(HttpEntity) 方法来保证实体内容被完全消耗，也关闭了基础连接。

  方法1： entity.getContent();

    CloseableHttpClient httpclient = HttpClients.createDefault();
    HttpGet httpget = new HttpGet("http://localhost/");
    CloseableHttpResponse response = httpclient.excute(httpget);
    try {
    HttpEntity entity = response.getEntity();
    if(entity != null) {
        InputStream instream = entity.getContent();
        try {
            // do something useful
        } finally {
            instream.close();
        }
    }
    } finally {
    response.close();
    }
    EntityUtils#consume(HttpEntity)

   方法2： entity.writeTo(outstream);

    CloseableHttpClient httpclient = HttpClients.createDefault();
    HttpGet httpget = new HttpGet("http://localhost/");
    CloseableHttpResponse response = httpclient.excute(httpget);
    try {
    HttpEntity entity = response.getEntity();
    if(entity != null) {
        	OutputStream outstream=null;
			entity.writeTo(outstream);
        try {
            // do something useful
        } finally {
            instream.close();
        }
    }
    } finally {
    response.close();
    }
    EntityUtils#consume(HttpEntity)

方法3：EntityUtils.toString(entity)实现

     if (entity != null) {
				result = EntityUtils.toString(entity, "utf-8");
			}`
    EntityUtils#consume(HttpEntity)



HttpClient 提供几个类，适用于大多数常见的数据，诸如字符串，字节数组，输入流和文件： StringEntity, ByteArrayEntity,
InputStreamEntity 和 FileEntity。若是需要缓存，最简单的方式是将原始实体用 BufferedHttpEntity 类封装；对于请求和返回可以使用同样的实体。

    //BufferedHttpEntity	
    CloseableHttpResponse response = httpclient.excute(httpget);
    HttpEntity entity = response.getEntity();
    if(entity != null) {
    entity = new BufferedHttpEntity(entity);
    }

    //FileEntity
    File file = new File("somefile.txt");
    FileEntity entity = new FileEntity(file, ContentType.create("text/plain", "UTF-8"));
    HttpPost httppost = new HttpPost("http://localhost/action.do");
    httppost.setEntity(entity);

## 5.1HTTP运行上下文
HttpClient允许HTTP请求在一个特定的执行上下文中来执行--称为HTTP上下文。如果相同的上下文在连续请求之间重用，那么多种逻辑相关的请求可以参与到一个逻辑会话中。HTTP上下文功能和java.util.Map<String,Object>很相似。 它仅仅是任意命名参数值的集合。应用程序可以在请求之前填充上下文属性，也可以在执行完成之后来检查上下文属性。
HttpContext能够包含任意的对象，因此在两个不同的线程中共享上下文是不安全的，建议每个线程都一个它自己执行的上下文。
在HTTP请求执行的这一过程中， HttpClient添加了下列属性到执行上下文中：
`HttpConnection实例代表连接到目标服务器的当前连接。
`HttpHost实例代表连接到目标服务器的当前连接。
`HttpRoute实例代表了完整的连接路由。
`HttpRequest实例代表了当前的HTTP请求。最终的HttpRequest对象在执行上下文中总是准确代表了状态信息，因为它已经发送给了服务器。每个默认的HTTP/1.0 和 HTTP/1.1使用相对的请求URI，可是，请求以non-tunneling模式通过代理发送时，URI会是绝对的。
`HttpResponse实例代表当前的HTTP响应。
`java.lang.Boolean对象是一个标识，它标志着当前请求是否完整地传输给连接目标。
`RequestConfig代表当前请求配置。
`java.util.List<URI>对象代表一个含有执行请求过程中所有的重定向地址。
你可以使用HttpClientContext适配器类来简化上下文状态的活动。

    http上下文来绑定cookie:
    CloseableHttpClient httpclient = HttpClients.createDefault();
        try {
            // Create a local instance of cookie store
            CookieStore cookieStore = new BasicCookieStore();
 
            // Create local HTTP context
            HttpClientContext localContext = HttpClientContext.create();
            // Bind custom cookie store to the local context
            localContext.setCookieStore(cookieStore);
 
            HttpGet httpget = new HttpGet("http://localhost/");
            System.out.println("Executing request " + httpget.getRequestLine());
 
            // Pass local context as a parameter
            CloseableHttpResponse response = httpclient.execute(httpget, localContext);
            try {
                System.out.println("----------------------------------------");
                System.out.println(response.getStatusLine());
                List<Cookie> cookies = cookieStore.getCookies();
                for (int i = 0; i < cookies.size(); i++) {
                    System.out.println("Local cookie: " + cookies.get(i));
                }
                EntityUtils.consume(response.getEntity());
            } finally {
                response.close();
            }
        } finally {
            httpclient.close();
        }







#参考资料：#
 * [https://blog.csdn.net/u011179993/article/details/47279521] HttpClient执行上下文HttpContext