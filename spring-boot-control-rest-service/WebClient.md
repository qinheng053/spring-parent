### Spring boot WebClient

WebClient是从spring webflux5.0版本开始提供的一个非阻塞型的HTTP请求客户端工具，它是基于Reactor响应式编程；

#### 1.WebClient支持7种请求方式
```

public interface WebClient {
    WebClient.RequestHeadersUriSpec<?> get();

    WebClient.RequestHeadersUriSpec<?> head();

    WebClient.RequestBodyUriSpec post();

    WebClient.RequestBodyUriSpec put();

    WebClient.RequestBodyUriSpec patch();

    WebClient.RequestHeadersUriSpec<?> delete();

    WebClient.RequestHeadersUriSpec<?> options();
```

#### 2.WebClient对象的创建提供了三种方式
* 使用默认方式创建
```
WebClient webClient = WebClient.create();
```
* 指定基础baseUrl，即访问同一个应用的相同URL部分
```
 WebClient webClient = WebClient.create(baseUrl);
```
* 使用build建造者模式构件
```
WebClient webClient = WebClient.builder().build();
```
* 创建WebClient相关源码
```
    static WebClient create() {
        return (new DefaultWebClientBuilder()).build();
    }

    static WebClient create(String baseUrl) {
        return (new DefaultWebClientBuilder()).baseUrl(baseUrl).build();
    }

    static WebClient.Builder builder() {
        return new DefaultWebClientBuilder();
    }
```

#### 3.WebClient传递URL一共提供了四种方式

```
    public interface UriSpec<S extends WebClient.RequestHeadersSpec<?>> {
        S uri(URI var1);

        S uri(String var1, Object... var2);

        S uri(String var1, Map<String, ?> var2);

        S uri(Function<UriBuilder, URI> var1);
    }
```

* 不带参数测试用例
```
    @Test
    public void test(){
        String baseUrl = "http://172.30.67.122:9000/handler/";
        WebClient webClient = WebClient.create(baseUrl);
        Mono<String> result = webClient.post().uri("client").retrieve().bodyToMono(String.class);
        //或者URI
        //Mono<String> result = webClient.post().uri(URI.create("http://172.30.67.122:9000//handler/client")).retrieve().bodyToMono(String.class);
        System.out.println(result.block());
    }
```
* 带参数的测试用例
```
  @Test
    public void test(){
        String baseUrl = "http://172.30.67.122:9000//handler/";
        WebClient webClient = WebClient.create(baseUrl);
        Map<String, Object> param = Maps.newHashMap();
        param.put("path", "client");
        Mono<String> result = webClient.post().uri("{path}", param).retrieve().bodyToMono(String.class);
        System.out.println(result.block());
    }
```
或者
```
    @Test
    public void test(){
        String baseUrl = "http://172.30.67.122:9000//handler/";
        WebClient webClient = WebClient.create(baseUrl);
        Mono<String> result = webClient.post().uri("{path}", "client").retrieve().bodyToMono(String.class);
        System.out.println(result.block());
    }
```

* 使用UriBuilder构造url
```
    @Test
    public void testUrlBuilder(){
        String baseUrl = "http://172.30.67.122:9000//handler/";
        WebClient webClient = WebClient.create(baseUrl);
        Mono<String> result = webClient.post()
                                    .uri(uriBuilder -> uriBuilder.path("client")
                                                                .queryParam("name", "李磊")
                                                                .build())
                                    .retrieve()
                                    .bodyToMono(String.class);
        System.out.println(result.block());
    }
```

#### 4.UriBuilder
```
package org.springframework.web.util;

import java.net.URI;
import java.util.Map;

import org.springframework.lang.Nullable;
import org.springframework.util.MultiValueMap;

public interface UriBuilder {

	/**
	 * 设置请求协议如：http、https
	 */
	UriBuilder scheme(@Nullable String scheme);

	/**
	 * Set the URI user info which may contain URI template variables, and
	 * may also be {@code null} to clear the user info of this builder.
	 * @param userInfo the URI user info
	 */
	UriBuilder userInfo(@Nullable String userInfo);

	/**
	 * 设置请求的域名或者IP地址
	 */
	UriBuilder host(@Nullable String host);

	/**
	 * 设置端口号
	 */
	UriBuilder port(int port);

	/**
	 * 设置端口号
	 */
	UriBuilder port(@Nullable String port);

	/**
	 * 设置接口路由
	 */
	UriBuilder path(String path);

	/**
	 * 替换请求路由
	 */
	UriBuilder replacePath(@Nullable String path);

	/**
	 * Append path segments to the existing path. Each path segment may contain
	 * URI template variables and should not contain any slashes.
	 * Use {@code path("/")} subsequently to ensure a trailing slash.
	 * @param pathSegments the URI path segments
	 */
	UriBuilder pathSegment(String... pathSegments) throws IllegalArgumentException;

	/**
	 * Append the given query to the existing query of this builder.
	 * The given query may contain URI template variables.
	 * <p><strong>Note:</strong> The presence of reserved characters can prevent
	 * correct parsing of the URI string. For example if a query parameter
	 * contains {@code '='} or {@code '&'} characters, the query string cannot
	 * be parsed unambiguously. Such values should be substituted for URI
	 * variables to enable correct parsing:
	 * <pre class="code">
	 * builder.query(&quot;filter={value}&quot;).uriString(&quot;hot&amp;cold&quot;);
	 * </pre>
	 * @param query the query string
	 */
	UriBuilder query(String query);

	/**
	 * Set the query of this builder overriding all existing query parameters.
	 * @param query the query string, or {@code null} to remove all query params
	 */
	UriBuilder replaceQuery(@Nullable String query);

	/**
	 * 设置查询参数，参数值可以为单个也可以是一个数组
	 */
	UriBuilder queryParam(String name, Object... values);

	/**
	 * 设置查询参数，参数值可以为数组模式MultiValueMap
	 */
	UriBuilder queryParams(MultiValueMap<String, String> params);

	/**
	 * 设置查询参数值，覆盖同一参数的所有现有查询值。如果没有给定值，则删除查询参数
	 */
	UriBuilder replaceQueryParam(String name, Object... values);

	/**
	 * Set the query parameter values overriding all existing query values.
	 * @param params the query parameter name
	 */
	UriBuilder replaceQueryParams(MultiValueMap<String, String> params);

	/**
	 * Set the URI fragment. The given fragment may contain URI template variables,
	 * and may also be {@code null} to clear the fragment of this builder.
	 * @param fragment the URI fragment
	 */
	UriBuilder fragment(@Nullable String fragment);

	/**
	 * 构件URI对象，参数uriVariables用来替换url中的变量
	 */
	URI build(Object... uriVariables);

	/**
	 * 构件URI对象，参数uriVariables用来替换url中的变量
	 */
	URI build(Map<String, ?> uriVariables);

}
```

