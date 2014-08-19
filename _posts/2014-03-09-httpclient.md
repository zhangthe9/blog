---
layout: post
title: httpclient
description: 
- httpclient
categories:
- httpclient
tags:
- httpclient
---
##HttpClient 3 4.3 区别
3
HttpClient httpClient=new DefaultHttpClient();
4.3
CloseableHttpClient httpClient = HttpClients.createDefault();


3.X的超时设置方法
	HttpClient client = new HttpClient();
	client.setConnectionTimeout(30000); 
	client.setTimeout(30000);
	HttpClient httpClient= new HttpClient(); 
	httpClient.getHttpConnectionManager().getParams().setConnectionTimeout(5000);
4.X版本的超时设置(4.3后已过时)
	HttpClient httpClient=new DefaultHttpClient();
	httpClient.getParams().setParameter(CoreConnectionPNames.CONNECTION_TIMEOUT,2000);//连接时间
	httpClient.getParams().setParameter(CoreConnectionPNames.SO_TIMEOUT,2000);//数据传输时间
4.3版本超时设置
	CloseableHttpClient httpClient = HttpClients.createDefault();
	HttpGet httpGet=new HttpGet("http://www.baidu.com");//HTTP Get请求(POST雷同)
	RequestConfig requestConfig = RequestConfig.custom().setSocketTimeout(2000).setConnectTimeout(2000).build();//设置请求和传输超时时间
	httpGet.setConfig(requestConfig);
	httpClient.execute(httpGet);//执行请求
4.3版本不设置超时的话，一旦服务器没有响应，等待时间N久(>24小时)。
	
	
HttpConnection没有连接池的概念，多少次请求就会建立多少个IO，高并发下O可能会耗尽
###httpclient 3

<pre class="prettyprint"><xmp>
<dependency>
	<groupId>commons-httpclient</groupId>
	<artifactId>commons-httpclient</artifactId>
	<version>3.1</version>
</dependency></xmp>
</pre>

<pre class="prettyprint">
MultiThreadedHttpConnectionManager connectionManager = new MultiThreadedHttpConnectionManager();
HttpClient client = new HttpClient(connectionManager);
// and then from inside some thread executing a method
GetMethod get = new GetMethod("http://hc.apache.org/");
try {
	client.executeMethod(get);
	// print response to stdout
	// System.out.println(get.getResponseBodyAsStream());
	System.out.println(get.getResponseBodyAsString());
} finally {
	// be sure the connection is released back to the connection  manager
	get.releaseConnection();
}
</pre>

   多线程环境下应该使用一个全局单例的HttpClient，并且使用MultiThreadHttpConnectionManager来管理(复用)Connection

###两种连接池
   全局的ConnectionPool，每主机（per- host）HostConnectionPool
maxHostConnections就HostConnectionPool中表示每主机可保持 连接的连接数
maxTotalConnections是ConnectionPool中可最多保持的连接数

每主机的配置类是 HostConfiguration，HttpClient有个int executeMethod(final HostConfiguration hostConfiguration, final HttpMethod method)方法可以指定使用哪个HostConfiguration，不过多数情况都是不显示指定HostConfiguration，这样 httpclient就用了默认的HostConfiguration=null，也就是说所有的请求可以认为指自同一个主机

如果不设置这两个参 数
默认的maxHostConnections大小只有2,也就是说，并发8个线程请求数据时，实际上会有6个线程处于等待被调度
maxTotalConnections默认20


MultiThreadedHttpConnectionManager，看名字好像是多线程并发请求似的， 其实不是，但它也确实用到了多线程，那是在发现连接不够用时起个等待线程wait信号，这个名称的含义应该是多线程情况线程安全的 HttpConnectionManager





###httpclient 4.0 连接池
<pre class="prettyprint">
import org.apache.http.client.HttpClient;
import org.apache.http.conn.ClientConnectionManager;
import org.apache.http.conn.params.ConnManagerParams;
import org.apache.http.conn.params.ConnPerRouteBean;
import org.apache.http.conn.scheme.PlainSocketFactory;
import org.apache.http.conn.scheme.Scheme;
import org.apache.http.conn.scheme.SchemeRegistry;
import org.apache.http.conn.ssl.SSLSocketFactory;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.impl.conn.tsccm.ThreadSafeClientConnManager;
import org.apache.http.params.BasicHttpParams;
import org.apache.http.params.HttpConnectionParams;
import org.apache.http.params.HttpParams;

public class HttpConnectionManager {

	private static HttpParams httpParams;
	private static ClientConnectionManager connectionManager;

	/**
	 * 最大连接数
	 */
	public final static int MAX_TOTAL_CONNECTIONS = 800;
	/**
	 * 获取连接的最大等待时间
	 */
	public final static int WAIT_TIMEOUT = 60000;
	/**
	 * 每个路由最大连接数
	 */
	public final static int MAX_ROUTE_CONNECTIONS = 400;
	/**
	 * 连接超时时间
	 */
	public final static int CONNECT_TIMEOUT = 10000;
	/**
	 * 读取超时时间
	 */
	public final static int READ_TIMEOUT = 10000;

	static {
		httpParams = new BasicHttpParams();
		// 设置最大连接数
		ConnManagerParams.setMaxTotalConnections(httpParams,
				MAX_TOTAL_CONNECTIONS);
		// 设置获取连接的最大等待时间
		ConnManagerParams.setTimeout(httpParams, WAIT_TIMEOUT);
		// 设置每个路由最大连接数
		ConnPerRouteBean connPerRoute = new ConnPerRouteBean(
				MAX_ROUTE_CONNECTIONS);
		ConnManagerParams.setMaxConnectionsPerRoute(httpParams, connPerRoute);
		// 设置连接超时时间
		HttpConnectionParams.setConnectionTimeout(httpParams, CONNECT_TIMEOUT);
		// 设置读取超时时间
		HttpConnectionParams.setSoTimeout(httpParams, READ_TIMEOUT);

		SchemeRegistry registry = new SchemeRegistry();
		registry.register(new Scheme("http", PlainSocketFactory
				.getSocketFactory(), 80));
		registry.register(new Scheme("https", SSLSocketFactory
				.getSocketFactory(), 443));

		connectionManager = new ThreadSafeClientConnManager(httpParams,
				registry);
	}

	public static HttpClient getHttpClient() {
		return new DefaultHttpClient(connectionManager, httpParams);
	}

}
</pre>

###每个路由(route)最大连接数 
运行环境机器 到 目标机器的一条线路
用HttpClient的实现来分别请求 www.baidu.com 的资源和 www.bing.com 的资源会产生两个route
默认值为2，如果不设置这个参数值默认情况下对于同一个目标机器的最大并发连接只有2个！哪怕设置连接池的最大连接数为200，但是实际上还是只有2个连接在工作，其他剩余的198个连接都在等待 


###httpclient 4.0 后
从4.0版本之后ConnManagerParams被Deprecated
ConnManagerParams的功能被挪到了 ThreadSafeClientConnManager 和 HttpConnectionParams两个类
	
##httpclient 4.3 常用代码
CloseableHttpClient httpClient = HttpClients.createDefault();
String url = "http://www.baidu.com";
##Get
httpClient.execute(new HttpGet(url));

###响应
####响应码
CloseableHttpResponse response = httpClient.execute(new HttpGet(url));
response.getStatusLine().getStatusCode()
####响应的媒体类型
String contentMimeType = ContentType.getOrDefault(response.getEntity()).getMimeType();
常量值 ContentType.TEXT_HTML.getMimeType()
####响应Body	
String bodyAsString = EntityUtils.toString(response.getEntity());

##Post
httpClient.execute(new HttpPost(url));
###重定向
CloseableHttpClient instance = HttpClientBuilder.create().disableRedirectHandling().build();
CloseableHttpResponse response = instance.execute(new HttpGet(url));

###HEAD
HttpGet request = new HttpGet(url);
request.addHeader(HttpHeaders.ACCEPT, "application/xml");
####响应head
Header[] headers = response.getHeaders(HttpHeaders.CONTENT_TYPE);
for (int i = 0; i < headers.length; i++) {
	System.out.println(headers[i]);
}


###允许所有证书
SSLContextBuilder builder = new SSLContextBuilder();
	builder.loadTrustMaterial(null, new TrustSelfSignedStrategy());
	SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(
			builder.build());
	CloseableHttpClient httpclient = HttpClients.custom()
			.setSSLSocketFactory(sslsf).build();

	HttpGet httpGet = new HttpGet("https://www.google.com");
	CloseableHttpResponse response = httpclient.execute(httpGet);
	try {
		System.out.println(response.getStatusLine());
		HttpEntity entity = response.getEntity();
		EntityUtils.consume(entity);
	} finally {
		response.close();
	}

###
推荐使用HttpEntity的getConent()方法或者HttpEntity的writeTo(OutputStream)方法来消耗掉Http实体内容
当响应不太大，可用EntityUtils里的静态工具方法以字符串或者字节数组的形式读取Http实体
 CloseableHttpResponse response = httpclient.execute(httpget);
    try {
        HttpEntity entity = response.getEntity();
        if (entity != null) {
            long len = entity.getContentLength();
            if (len != -1 && len < 2048) {
                System.out.println(EntityUtils.toString(entity));
            } else {
                // Stream content out
            }
        }
    } finally {
        response.close();
    }


HttpClient 4.3教程
http://www.yeetrack.com/?p=779