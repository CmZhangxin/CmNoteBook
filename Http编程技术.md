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
* [3.1 HTTP协议详解之响应](#2.1)
* [4.1 HTTP协议详解之消息报头](#2.1)

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





