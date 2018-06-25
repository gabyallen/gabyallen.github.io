---
layout:     post                    # 使用的布局（不需要改）
title:       Android Okhttp使用
    # 标题 
subtitle:     Okhttp详解  #副标题
date:       2018-6-25             # 时间
author:     BY  wangchuanwen         # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android
    - Http
    - Okhttp教程
    - okhttp
---

# OKHTTP使用详解
在Android 开发中，发送Http请求时很常见的。SDK中自带的HttpURLConnection虽然能基本满足需要，但是在使用上有诸多不便，为此，square公司实现了一个HTTP客户端，可以在android和Java应用程序中使用，其具体有以下特征：
1.API设计轻巧，基本上通过几行代码的链式调用即可获取结果。

2.即支持同步请求，也支持异步请求。同步请求会阻塞当前线程，异步请求不会阻塞当前线程，异步执行完成后执行相应的回调方法。

3.其支持HTTP/2协议，那么通过HTTP/2，可以让客户端中到同一服务器的所有请求共用同一个Scoket连接。

4.如果请求不支持HTTP/2协议，那么Okhttp会在内部维护一个连接池，通过该连接池，可以对HTTP/1.X的连接进行重用，减少了延迟。

5.透明的GZIP处理降低了下载数据大小。

6.请求的数据会进行相应的缓存处理，下次在进行请求时，如果服务器 告知304（表明数据没有发生变化），那么就直接从缓存中读取数据，降低了重复请求的数量。

当前okttp最新的版本是3.x，现在是3.10了，支持android2.3+。想要在Java应用程序中使用Okhttp，jre的最低版本是1.7.

Okttp API：
<!--[](http://square.github.io/okhttp/3.x/okhttp/)-->
<http://square.github.io/okhttp/3.x/okhttp/>

本文中的代码来自于Okhttp的官方Wiki中的示例代码。
<https://github.com/square/okhttp/wiki/Recipes/>
<!--[](https://github.com/square/okhttp/wiki/Recipes)-->


##下载Okttp

1.通过Maven获取
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>3.3.1</version>
</dependency>

2.也可以在Gradle中进行如下配置以便在Android Studio中使用：
dependencies{
compile 'com.squareup.okhttp3:okhttp:3.10.0'
}

添加上述依赖会自动下载两个库，一个是Okhht库，一个是Okio库，后者是前者的通信基础。

##核心类

我们在使用okhttp进行开发的时候，主要牵扯到以下几个核心类：OkHttpClient，Request,Call,和Response

1.OkhttpClient
OkhttpClient表示了Http请求的客户端类，在绝大多数的app中，我们只应该执行一次new OkhttpClient（），将其作为全局的实例进行保存，从而在app的各处都只使用这一个实例对象，这样所有的HTTP请求都可以共用Response缓存、共用线程池以及共用连接池。

  *默认情况下，直接执行OkhttpClient client =new OkhttpClient() 就可以实例化一个OKhttpClient对象。
  
  *可以配置OkhttpClient 的一些参数，比如超时时间、缓存目录、代理、Authenticator等，那么就需要用到内部类OkHttpClient.Builder,设置如下所示：
  
  
###OkHttpClient client = new OkHttpClient.Builder().
        readTimeout(30, TimeUnit.SECONDS).
        cache(cache).
        proxy(proxy).
        authenticator(authenticator).
        build();

OkhttpClent本身不能设置参数，需要借助于其内部类Builder设置参数，参数设置完成后，调用Builder的build方法得到一个配置好参数的okHttpClient对象。这些配置的参数会对该OkhttpClient对象所生产的所有HTTP请求都有影响。

有些时候我们想单独给某个网络请求设置特别的几个参数，比如只想让某个请求的超时时间设置为30秒，但是还想保持Okhttpclient对象中的其他参数设置，那么就可以调用OkHttpClient对象的newBulider（）方法，代码如下：

###OkHttpClient client = ...
OkHttpClient clientWith60sTimeout = client.newBuilder().
        readTimeout(30, TimeUnit.SECONDS).
        build();

clientWith60sTimeout中的参数来自于client中的配置参数，只不过它覆盖了读取超时时间这一个参数，其余参数与client中的一致。


2.Request
Request类封装了请求报文信息：请求url地址、请求的方法（如GET,POST等）、各种请求头（如Content-type、Cookie）以及可选的请求体。一般通过内部类Request.builder的链式调用生成的Request对象。

3.Call
Call代表了一个实际的Http请求，它是连接Request和Response的桥梁，通过Request对象的newCll()方法可以得到一个Call对象。Call对象即支持同步获取数据，也可以异步获取数据。
  
  *执行Call对象的execute（）方法，会阻塞当前线程去获取数据，该方法返回一个Response对象。
  
  *执行Call对象的execute()方法，不会阻塞当前线程，该方法接收一个Callback对象，当异步获取到数据之后，会回调执行Callback对象的相应方法。如果请求成功，则执行Callback对象的onResponse方法，并将Response对象传入该方法中；如果请求失败，则执行Callback对象的onFalilure方法。
  
  4.Response
  Response类封装了响应报文信息：状态码（200、404等）、响应头（Content-Type、Server等）以及可选的响应体。可以通过Call对象的execute（）方法获得Response对象，异步回调执行Callback对象的onResponse方法时也可以获取Response对象。
  
  
##同步GET
以下示例了如何同步发送GET请求，输出响应头以及将响应体转换为字符串。

private final OkHttpClient client = new OkHttpClient();
public void run() throws Exception {

  Request request = new Request.Builder()
      .url("http://publicobject.com/helloworld.txt")
      .build();
      
  Response response = client.newCall(request).execute();
  if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

  Headers responseHeaders = response.headers();
  
  for (int i = 0; i < responseHeaders.size(); i++) {
    System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));
    
  }

  System.out.println(response.body().string());
  
}

下面对以上代码进行简单说明：

* Clent执行newCall方法会得到一个Call对象，表示一个新的网络请求。
 
* Call对象的execute方法是同步方法，会阻塞当前线程，其返回Response对象。

* 通过Response对象的isSuccessful()方法可以判断请求是否成功。

*通过Response的headers（）方法可以得到响应头Headers对象，可以通过for循环索引遍历所有的响应头的名称和值，可以通过Headers.name（index）方法获取响应头的名称，通过Headers.value(index)方法获取响应头

*为了解决同时获取多个name相同的响应头的值，Headers中提供了一个public List<String> values(String name)方法，该方法会返回一个List<String>对象，所以此处通过Headers对象的values(‘Set-Cookie’)可以获取全部的Cookie信息，如果调用Headers对象的get(‘Set-Cookie’)方法，那么只会获取最后一条Cookie信息。

*通过Response对象的body()方法可以得到响应体ResponseBody对象，调用其string()方法可以很方便地将响应体中的数据转换为字符串，该方法会将所有的数据放入到内存之中，所以如果数据超过1M，最好不要调用string()方法以避免占用过多内存，这种情况下可以考虑将数据当做Stream流处理。

##异步GET

下面示例演示了如何异步发送GET网络请求，代码如下所示：

private final OkHttpClient client = new OkHttpClient();

  public void run() throws Exception {
    Request request = new Request.Builder()
        .url("http://publicobject.com/helloworld.txt")
        .build();

    client.newCall(request).enqueue(new Callback() {
      @Override
      public void onFailure(Call call, IOException e) {
        e.printStackTrace();
      }

      @Override
      public void onResponse(Call call, Response response) throws IOException {
        if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

        Headers responseHeaders = response.headers();
        for (int i = 0, size = responseHeaders.size(); i < size; i++) {
          System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));
        }

        System.out.println(response.body().string());
      }
    });
  }
  
### 下面对以上代码进行一下说明：

*要想异步执行网络请求，需要执行Call对象的enqueue方法，该方法okhttp3.Callback对象，enqueue方法不会阻塞当前线程，会新开一个工作线程，让实际的网络请求在工作线程中执行。一般情况下这个工作线程的名字以“okhttp”开头，并包含连接的host信息，比如上面例子中的工作线程的name是"Okhttp http://publicobject.com/..."

*当异步请求成功后，会回调Callback对象的onResponse方法，在该方法中可以获取Response对象。当异步请求失败或者调用了Call对象的cancel方法时，会回调Callback对象的onFailure方法。onResponse和onFailure这两个方法都是在工作线程中执行的。

####   请求头和响应头
典型的HTTP请求头、响应头都是类似于Map<String, String>，每个name对应一个value值。不过像我们之前提到到的，也会存在多个name重复的情况，比如相应结果中就有可能存在多个Set-Cookie响应头，同样的，也可能同时存在多个名称一样的请求头。响应头的读取我们在上文已经说过了，在此不再赘述。一般情况下，我们只需要调用header(name, value)方法就可以设置请求头的name和value，调用该方法会确保整个请求头中不会存在多个名称一样的name。如果想添加多个name相同的请求头，应该调用addHeader(name, value)方法，这样可以添加重复name的请求头，其value可以不同，例如如下所示：


  private final OkHttpClient client = new OkHttpClient();
  
  public void run() throws Exception {
    Request request = new Request.Builder()
        .url("https://api.github.com/repos/square/okhttp/issues")
        .header("User-Agent", "OkHttp Headers.java")
        .addHeader("Accept", "application/json; q=0.5")
        .addHeader("Accept", "application/vnd.github.v3+json")
        .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println("Server: " + response.header("Server"));
    System.out.println("Date: " + response.header("Date"));
    System.out.println("Vary: " + response.headers("Vary"));
  }
  
上面的代码通过addHeader方法添加了两个Accept请求头
且二者的值不同，这样服务器收到客户端发来的请求后，就知道客户端即支持application/json类型的数据，也支持application/vnd.github.v3+json类型的数据。

##用POST发送String

可以使用Post方法发送请求体。下面的示例演示了如何将markdown文本作为请求体发送到服务器，服务器会将其转换成html文档，并发送给客户端。

  public static final MediaType MEDIA_TYPE_MARKDOWN
      = MediaType.parse("text/x-markdown; charset=utf-8");

  private final OkHttpClient client = new OkHttpClient();

  public void run() throws Exception {
    String postBody = ""
        + "Releases\n"
        + "--------\n"
        + "\n"
        + " * _1.0_ May 6, 2013\n"
        + " * _1.1_ June 15, 2013\n"
        + " * _1.2_ August 11, 2013\n";

    Request request = new Request.Builder()
        .url("https://api.github.com/markdown/raw")
        .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, postBody))
        .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());
  }

下面对以上代码进行说明：

*Request.Builder的post方法接收一个RequestBody对象。

*RequestBody就是请求体，一般可通过调用该类的5个重载的的static的Create（）方法的第二个参数可以是String、File、byte[]或okio.ByteString。除了调用create()方法，还可以调用RequestBody的writeTo()方法向其写入数据，writeTo方法一般在用POST发送Stream流的时候使用。

*MediaType指的是要传递的数据的MIME类型，mediatype对象包含了三种信息：type 、subtype以及CharSet，一般将这些信息传入parse（）方法中，这样就能解析出mediatype对象，比如上例总text/x-markdown; charset=utf-8，type值是text，表示是文本这一大类；/后面的x-markdown是subtype，表示是文本这一大类下的markdown这一小类；charset=utf-8则表示采用UTF-8编码。如果不知道某种类型数据的MIME类型，可以参见连接Media Types和MIME 参考手册，较详细地列出了所有的数据的MIME类型。以下是几种常见数据的MIME类型值：

son ：application/json
xml：application/xml
png：image/png
jpg： image/jpeg
gif：image/gif

mediatypes：<https://www.iana.org/assignments/media-types/media-types.xhtml/>

mime: <http://www.w3school.com.cn/media/media_mimeref.asp/>

注意：在该例中，请求体会放置在内存中，所以应该避免用该API发送超过1M的数据

##用POST发送Stream流
下面的示例演示了如何使用POST发送Stream流。

  public static final MediaType MEDIA_TYPE_MARKDOWN
      = MediaType.parse("text/x-markdown; charset=utf-8");

  private final OkHttpClient client = new OkHttpClient();

  public void run() throws Exception {
    RequestBody requestBody = new RequestBody() {
      @Override public MediaType contentType() {
        return MEDIA_TYPE_MARKDOWN;
      }

      @Override public void writeTo(BufferedSink sink) throws IOException {
        sink.writeUtf8("Numbers\n");
        sink.writeUtf8("-------\n");
        for (int i = 2; i <= 997; i++) {
          sink.writeUtf8(String.format(" * %s = %s\n", i, factor(i)));
        }
      }

      private String factor(int n) {
        for (int i = 2; i < n; i++) {
          int x = n / i;
          if (x * i == n) return factor(x) + " × " + i;
        }
        return Integer.toString(n);
      }
    };

    Request request = new Request.Builder()
        .url("https://api.github.com/markdown/raw")
        .post(requestBody)
        .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());
  }
  
  下面对以上代码进行说明：
  
  *以上代码在实例化RequestBost对象的时候重写了contentType（）和writeTo方法。
  
  *覆写contentTyoe方法，返回markdown类型的mediatyoe
  
  *覆写writeTo方法，该方法会传入一个OKia的BufferedSink类型的对象，可以通过BufferedSink的各种write方法向其写入各种类型的数据，此例中用其writeUTF8方法向其中写入UTF-8的文本数据。也可以通过它的outputStream（）方法，得到输出流OutputStream，从而通过OutputSteram向BufferedSink写入数据。
  
##用POST发送File
下面的代码演示了如何用POST发送文件。

  public static final MediaType MEDIA_TYPE_MARKDOWN
      = MediaType.parse("text/x-markdown; charset=utf-8");

  private final OkHttpClient client = new OkHttpClient();

  public void run() throws Exception {
    File file = new File("README.md");

    Request request = new Request.Builder()
        .url("https://api.github.com/markdown/raw")
        .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, file))
        .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());
  }
  
我们之前提到，RequestBody.create()静态方法可以接收File参数，将File转换成请求体，将其传递给post()方法。
  
##用POST发送Form表单中的键值对
如果想用POST发送键值对字符串，可以使用在post（）方法中传入FormBody对象，FormBody继承自RequestBody，类似于Web前端中的Form表单。可以通过FromBody.Builder构建FromBody.

  private final OkHttpClient client = new OkHttpClient();

  public void run() throws Exception {
    RequestBody formBody = new FormBody.Builder()
        .add("search", "Jurassic Park")
        .build();
    Request request = new Request.Builder()
        .url("https://en.wikipedia.org/w/index.php")
        .post(formBody)
        .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());
  }
  
  需要注意的是，在发送数据之前，android会对非ASCLL码字符调用encodeURIComponent方法进行编码，例如”Jurassic Park”会编码成”Jurassic%20Park”，其中的空格符被编码成%20了，服务器端会其自动解码。

##用POST发送multipart数据
我们可以通过WEB前端的Form表单上传一个或多个文件，OKhttp也提供了对应的功能，如果我们想同时发送多个From表单形式的文件，就可以使用在post（）方法中传入multipartBody对象。MultipartBody对象。MultipartBody继承自RequestBody，也表示请求体。只不过MultipartBody的内部是由多个part组成的，每个part就单独包含了一个RequestBody请求体，所以可以把MultipartBody看成是一个RequestBody的数组，而且可以分别给每个RequestBody单独设置请求头。

示例代码如下所示：

  private static final String IMGUR_CLIENT_ID = "...";
  private static final MediaType MEDIA_TYPE_PNG = MediaType.parse("image/png");

  private final OkHttpClient client = new OkHttpClient();

  public void run() throws Exception {
    // Use the imgur image upload API as documented at https://api.imgur.com/endpoints/image
    RequestBody requestBody = new MultipartBody.Builder()
        .setType(MultipartBody.FORM)
        .addFormDataPart("title", "Square Logo")
        .addFormDataPart("image", "logo-square.png",
            RequestBody.create(MEDIA_TYPE_PNG, new File("website/static/logo-square.png")))
        .build();

    Request request = new Request.Builder()
        .header("Authorization", "Client-ID " + IMGUR_CLIENT_ID)
        .url("https://api.imgur.com/3/image")
        .post(requestBody)
        .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());
  }
  
  下面对以上代码进行说明：

MultipartBody要通过其内部类MultipartBody.Builder进行构建。

通过MultipartBody.Builder的setType()方法设置MultipartBody的MediaType类型，一般情况下，将该值设置为MultipartBody.FORM，即W3C定义的multipart/form-data类型，详见Forms in HTML documents。

通过MultipartBody.Builder的方法addFormDataPart(String name, String value)或addFormDataPart(String name, String filename, RequestBody body)添加数据，其中前者添加的是字符串键值对数据，后者可以添加文件。

MultipartBody.Builder还提供了三个重载的addPart方法，其中通过addPart(Headers headers, RequestBody body)方法可以在添加RequestBody的时候，同时为其单独设置请求头。

##用Gson处理JSON响应

Gson是Google开源的一个用于进行JSON处理的JAVA库，通过Gson可以很方面地在JSON和jAVA对象之间进行转换。我们可以将OKhttp和Gson一起使用，用Gson解析返回的Json结果。

下面的示例代码演示了如何使用Gson解析GITHUB api的返回结果。

  private final OkHttpClient client = new OkHttpClient();
  private final Gson gson = new Gson();

  public void run() throws Exception {
    Request request = new Request.Builder()
        .url("https://api.github.com/gists/c2a7c39532239ff261be")
        .build();
    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    Gist gist = gson.fromJson(response.body().charStream(), Gist.class);
    for (Map.Entry<String, GistFile> entry : gist.files.entrySet()) {
      System.out.println(entry.getKey());
      System.out.println(entry.getValue().content);
    }
  }

  static class Gist {
    Map<String, GistFile> files;
  }

  static class GistFile {
    String content;
  }

下面对以上代码进行说明：

[](https://api.github.com/gists/c2a7c39532239ff261be)

*Gist类对应着整个JSON的返回结果，Gist中的Map<String, GistFile> files对应着JSON中的files。

*files中的每一个元素都是一个key-value的键值对，key是String类型，value是GistFile类型，并且GistFile中必须包含一个String content。OkHttp.txt就对应着一个GistFile对象，其content值就是GistFile中的content。

*通过代码Gist gist = gson.fromJson(response.body().charStream(), Gist.class)，将JSON字符串转换成了Java对象。将ResponseBody的charStream方法返回的Reader传给Gson的fromJson方法，然后传入要转换的Java类的class。

charStream：
<http://square.github.io/okhttp/3.x/okhttp/okhttp3/ResponseBody.html#charStream--/>

##缓存响应结果

如果想缓存响应结果，我们就需要为Okthhp配置缓存目录，Oktthp可以写入和读取该缓存目录，并且我们需要限制该缓存目录的大小。Okhttp的缓存目录应该是私有化的，不能被其他应用访问。

Okhttp中，多个缓存示例同时访问同一个缓存目录会出错，大部分的应用只应该调用一次new OKHttpClient（），然后为其配置缓存目录，然后在App的各个地方都使用这一个OkttpClient实例对象，否则两个缓存实例会相互影响，导致app崩溃。

缓存示例代码如下所示：

  private final OkHttpClient client;

  public CacheResponse(File cacheDirectory) throws Exception {
    int cacheSize = 10 * 1024 * 1024; // 10 MiB
    String okhttpCachePath = getCacheDir().getPath() + File.separator + "okhttp";
    File okhttpCache = new File(okhttpCachePath);
    if(!okhttpCache.exists()){
        okhttpCache.mkdirs();
    }

    Cache cache = new Cache(okhttpCache, cacheSize);

    client = new OkHttpClient.Builder()
        .cache(cache)
        .build();
  }

  public void run() throws Exception {
    Request request = new Request.Builder()
        .url("http://publicobject.com/helloworld.txt")
        .build();

    Response response1 = client.newCall(request).execute();
    if (!response1.isSuccessful()) throw new IOException("Unexpected code " + response1);

    String response1Body = response1.body().string();
    System.out.println("Response 1 response:          " + response1);
    System.out.println("Response 1 cache response:    " + response1.cacheResponse());
    System.out.println("Response 1 network response:  " + response1.networkResponse());

    Response response2 = client.newCall(request).execute();
    if (!response2.isSuccessful()) throw new IOException("Unexpected code " + response2);

    String response2Body = response2.body().string();
    System.out.println("Response 2 response:          " + response2);
    System.out.println("Response 2 cache response:    " + response2.cacheResponse());
    System.out.println("Response 2 network response:  " + response2.networkResponse());

    System.out.println("Response 2 equals Response 1? " + response1Body.equals(response2Body));
  }

下面对以上代码进行说明：

*我们在App的cache目录下创建了一个子目录okhttp，将其作为Okhhtp专门用于缓存的目录，并设置其上限为10M，Okhttp需要能够读写该目录。

*不要让多个缓存实例同时访问同一个缓存目录，因为多个缓存实例会相互影响，导致出错，甚至系统崩溃。在绝大多数的App中，我们只应该执行一次new OkHttpClient()，将其作为全局的实例进行保存，从而在App的各处都只使用这一个实例对象，这样所有的HTTP请求都可以共用Response缓存。

*上面代码，我们对于同一个URL，我们先后发送了两个HTTP请求。第一次请求完成后，Okhttp将请求到的结果写入到了缓存目录中，进行了缓存。response1.networkResponse()返回了实际的数据，response1.cacheResponse()返回了null，这说明第一次HTTP请求的得到的响应是通过发送实际的网络请求，而不是来自于缓存。然后对同一个URL进行了第二次HTTP请求，response2.networkResponse()返回了null，response2.cacheResponse()返回了缓存数据，这说明第二次HTTP请求得到的响应来自于缓存，而不是网络请求。

*如果想让某次请求禁用缓存，可以调用request.cacheControl(CacheControl.FORCE_NETWORK)方法，这样即便缓存目录有对应的缓存，也会忽略缓存，强制发送网络请求，这对于获取最新的响应结果很有用。如果想强制某次请求使用缓存的结果，可以调用request.cacheControl(CacheControl.FORCE_CACHE)，这样不会发送实际的网络请求，而是读取缓存，即便缓存数据过期了，也会强制使用该缓存作为响应数据，如果缓存不存在，那么就返回504 Unsatisfiable Request错误。也可以向请求中中加入类似于Cache-Control: max-stale=3600之类的请求头，Okhttp也会使用该配置对缓存进行处理。

##取消请求
当请求不再需要的时候，我们应该中止请求，比如退出当前的Activity了，那么在Activity中发出的请求应该被中止，可以通过调用Call的cancel方法立即中止请求，如果线程正在写入Request或读取Response，那么会抛出IOException异常。同步请求和异步请求都可以被取消。

 private final ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
  private final OkHttpClient client = new OkHttpClient();

  public void run() throws Exception {
    Request request = new Request.Builder()
        .url("http://httpbin.org/delay/2") // This URL is served with a 2 second delay.
        .build();

    final long startNanos = System.nanoTime();
    final Call call = client.newCall(request);

    // Schedule a job to cancel the call in 1 second.
    executor.schedule(new Runnable() {
      @Override public void run() {
        System.out.printf("%.2f Canceling call.%n", (System.nanoTime() - startNanos) / 1e9f);
        call.cancel();
        System.out.printf("%.2f Canceled call.%n", (System.nanoTime() - startNanos) / 1e9f);
      }
    }, 1, TimeUnit.SECONDS);

    try {
      System.out.printf("%.2f Executing call.%n", (System.nanoTime() - startNanos) / 1e9f);
      Response response = call.execute();
      System.out.printf("%.2f Call was expected to fail, but completed: %s%n",
          (System.nanoTime() - startNanos) / 1e9f, response);
    } catch (IOException e) {
      System.out.printf("%.2f Call failed as expected: %s%n",
          (System.nanoTime() - startNanos) / 1e9f, e);
    }
  }

上述请求，服务器端会有两秒的延时，在客户端发出请求1秒之后，请求还未完成，这时候通过cancel方法中止了Call，请求中断，并触发IOException异常。

##设置超时

一次HTTP请求实际上可以分为三步：
1.客户端与服务器建立连接
2.客户端发送请求数据到服务器，即数据上传
3.服务器将响应数据发送给客户端，即数据下载

由于网络、服务器等各种原因，这三步中的每一步都有可能耗费很长时间，导致整个HTTP请求花费很长时间或不能完成。

为此，可以通过OkHttpClient.Builder的connectTimeout()方法设置客户端和服务器建立连接的超时时间，通过writeTimeout()方法设置客户端上传数据到服务器的超时时间，通过readTimeout()方法设置客户端从服务器下载响应数据的超时时间。

在2.5.0版本之前，okhttp默认不设置任何的超时时间，从2.5.0版本开始，okhttp将连接超时，写入超时（上传数据）、读取超时（下载数据）的超时时间都默认设置为10秒。如果HTTP请求需要更长时间，那么需要我们手动设置超时时间。

示例代码如下所示:

  private final OkHttpClient client;

  public ConfigureTimeouts() throws Exception {
    client = new OkHttpClient.Builder()
        .connectTimeout(10, TimeUnit.SECONDS)
        .writeTimeout(10, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .build();
  }

  public void run() throws Exception {
    Request request = new Request.Builder()
        .url("http://httpbin.org/delay/2") // This URL is served with a 2 second delay.
        .build();

    Response response = client.newCall(request).execute();
    System.out.println("Response completed: " + response);
  }
  
  如果HTTP请求的某一部分超时了，那么就会触发异常。

##处理身份验证
有些网络请求是需要用户名密码登录的，如果没提供登录需要的信息，那么会得到401 Not Authorized未授权的错误，这时候Okhttp会自动查找是否配置了Authenticator，如果配置过Authenticator，会用Authenticator中包含的登录相关的信息构建一个新的Request，尝试再次发送HTTP请求。

  private final OkHttpClient client;

  public Authenticate() {
    client = new OkHttpClient.Builder()
        .authenticator(new Authenticator() {
          @Override public Request authenticate(Route route, Response response) throws IOException {
            System.out.println("Authenticating for response: " + response);
            System.out.println("Challenges: " + response.challenges());
            String credential = Credentials.basic("jesse", "password1");
            return response.request().newBuilder()
                .header("Authorization", credential)
                .build();
          }
        })
        .build();
  }

  public void run() throws Exception {
    Request request = new Request.Builder()
        .url("http://publicobject.com/secrets/hellosecret.txt")
        .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println(response.body().string());
  }
  
上面对以上代码进行说明：

1.OkHttpClient.Builder的authenticator()方法接收一个Authenticator对象，我们需要实现Authenticator对象的authenticate()方法，该方法需要返回一个新的Request对象，该新的Request对象基于原始的Request对象进行拷贝，而且要通过header("Authorization", credential)方法对其设置登录授权相关的请求头信息。

2.通过Response对象的challenges()方法可以得到第一次请求失败的授权相关的信息。如果响应码是401 unauthorized，那么会返回”WWW-Authenticate”相关信息，这种情况下，要执行OkHttpClient.Builder的authenticator()方法，在Authenticator对象的authenticate()中 对新的Request对象调用header("Authorization", credential)方法，设置其Authorization请求头；如果Response的响应码是407 proxy unauthorized，那么会返回”Proxy-Authenticate”相关信息，表示不是最终的服务器要求客户端登录授权信息，而是客户端和服务器之间的代理服务器要求客户端登录授权信息，这时候要执行OkHttpClient.Builder的proxyAuthenticator()方法，在Authenticator对象的authenticate()中 对新的Request对象调用header("Proxy-Authorization", credential)方法，设置其Proxy-Authorization请求头。

3.如果用户名密码有问题，那么okhttp会一直用这个错误的登录信息尝试登录，我们应该判断如果之前已经用该用户名登录失败了，就不应该再次登录，这种情况下需要让authenticator对象的authenticate()方法返回null，这就避免了没必要的重复尝试，代码片段如下所示：

if (credential.equals(response.request().header("Authorization"))) {
   return null; 
}


##ResponseBody
通过Response的body（）方法可以得到响应体ResponseBody，响应体必须最终要被关闭，否则会导致资源泄露、app运行变慢甚至崩溃。

ResponseBody和Response都实现了Closeable和AutoCloseable接口，它们都有close()方法，Response的close()方法内部直接调用了ResponseBody的close()方法，无论是同步调用execute()还是异步回调onResponse()，最终都需要关闭响应体，可以通过如下方法关闭响应体：

Response.close()；
Response.body().close()；
Response.body().source().close()；
Response.body().charStream().close()；
Response.body().byteString().close()；
Response.body().bytes()；
Response.body().string()；

对于同步调用，确保响应体被关闭的最简单的方式是使用try代码块，如下所示：

Call call = client.newCall(request);
 try (Response response = call.execute()) {
   ... // Use the response.
 }
 
将Response response = call.execute()放入到try()的括号之中，由于Response 实现了Closeable和AutoCloseable接口，这样对于编译器来说，会隐式地插入finally代码块，编译器会在该隐式的finally代码块中执行Response的close()方法。

也可以在异步回调方法onResponse()中，执行类似的try代码块，try()代码块括号中的ResponseBody也实现了Closeable和AutoCloseable接口，这样编译器也会在隐式的finally代码块中自动关闭响应体，代码如下所示：

   Call call = client.newCall(request);
   call.enqueue(new Callback() {
     public void onResponse(Call call, Response response) throws IOException {
       try (ResponseBody responseBody = response.body()) {
         ... // Use the response.
       }
     }

     public void onFailure(Call call, IOException e) {
       ... // Handle the failure.
     }
   });
   
响应体中的数据有可能很大，应该只读取一次响应体的数据。调用ResponseBody的bytes()或string()方法会将整个响应体数据写入到内存中，可以通过source()、byteStream()或charStream()进行流式处理。



参考资料转载于：
<https://blog.csdn.net/iispring/article/details/51661195/>
参考：
<http://square.github.io/okhttp/3.x/okhttp/>
<https://github.com/square/okhttp/wiki/Recipes/>
<https://github.com/square/okhttp/blob/master/CHANGELOG.md/>







