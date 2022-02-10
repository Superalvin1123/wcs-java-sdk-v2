# Preparation
Please pre-install Java, Java 8 or above recommended. You can check your Java version with `java -version`.

## Install SDK
### Add dependencies to Maven project
```
<dependency>
  <groupId>com.chinanetcenter.wcs.sdk</groupId>
  <artifactId>wcs-java-sdk-v2</artifactId>
  <version>1.0.3</version>
</dependency>
```
### Import jars on Intellij IDEA

You may install as follows:

1. Download Java SDK package from github.
2. decompress the package.
3. Copy wcs-java-sdk-rest-v2-1.0.0.jar and all jar packages under `lib` to your project.
4. Select your project on Intellij IDEA: File->Project Structure->Modules->Dependencies->+->JARs or directories.
5. Select all the jars you copied on step 3, import them to External Libraries.

### Initialization
`WosClient` is a Java client, which is used to manage bucket and object.When using Java SDK, you should initialize and adjust default settings in `WosConfiguration` first.

**example**
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
```

### Create an instance of WosClient
```
WosClient wosClient = new WosClient(ak, sk, config, regionName);

//close wosClient
wosClient.close();
```

Note: `Endpoint` should be specified when creating an WosClient. You can have one or more WosClients and use them concurrently in your project.

## Configure WosClient

You can use `WosConfiguration` to configure `WosClient`, including timeout, maximum numbers of connection, etc. See details in the list below:

| Configuration | Description | method |
| -- | -- | -- |
|connectionTimeout | Set timeout for HTTP/HTTPS connetion (unit:ms), 60000 by default.	| WosConfiguration.setConnectionTimeout
|socketTimeout | Set timeout for Socket(unit:ms), 60000 by default.	| WosConfiguration.setSocketTimeout
|idleConnectionTime| 	An idle connection will be closed if it has been not working for a time more than idleConnectionTime (unit:ms). 30000 by default.	| WosConfiguration.setIdleConnectionTime
|maxIdleConnections	| Maximum number of idle connections, 1000 by default.	| WosConfiguration.setMaxIdleConnections
|maxConnections	| Maximum number of concurrencies for HTTP request, 1000 by default. | WosConfiguration.setMaxConnections
|maxErrorRetry	| Retries when request failed, 3 by default.| WosConfiguration.setMaxErrorRetry
|endPoint| 	Service address to access Object Storage, which may contain protocol, domain name and port. For example: https://your-endpoint:443	| WosConfiguration.setEndPoint
|readBufferSize	| Buffer size of Socket stream when reading (unit:byte), -1 by default, indicating no buffer.	| WosConfiguration.setReadBufferSize
|writeBufferSize	| Buffer size of Socket stream when writing(unit:byte), -1 by default, indicating no buffer.	| WosConfiguration.setWriteBufferSize
|pathStyle | Style of domain name as `bucketName.endpoint` is used when pathStyle is false, otherwise, 'endpoint/bucketName' style is used. |WosConfiguration.setPathStyle

# Quick Start

## 1. Upload an object

```
wosClient.putObject("bucketname", "objectname", new ByteArrayInputStream("Hello WOS".getBytes()));
```

## 2. Get an object
```
WosObject wosObject = wosClient.getObject("bucketname", "objectname");
InputStream content = wosObject.getObjectContent();
if (content != null)
{
    BufferedReader reader = new BufferedReader(new InputStreamReader(content));
    while (true)
    {
        String line = reader.readLine();
        if (line == null)
            break;
        System.out.println("\n" + line);
    }
    reader.close();
}
```
Note：
- An instance of WosObject with contents and properties will be returned after calling `wosObject.getObject`
- You can get content of an object by calling `wosObject.getObjectContent`. Remember to close it when finished.

## 3. List objects in a bucket

```
ObjectListing objectListing = wosClient.listObjects("bucketname");
for(WosObject wosObject : objectListing.getObjects()){
    System.out.println(" - " + wosObject .getObjectKey() + "  " +  "(size = " + wosObject .getMetadata().getContentLength() + ")");
}
```
Note：
- An instance of ObjectListing is returned after calling `wosClient.listObjects`. You can get information of objects by calling `ObjetListing.getObjects`.
- 1000 objects will be listed by default.

## 4. Delete an object

```
wosClient.deleteObject("bucketname", "objectname");
```
## 5. Get URL with signature

```
TemporarySignatureRequest req = new TemporarySignatureRequest(HttpMethodEnum.GET, 300);
req.setBucketName(bucketName);
req.setObjectKey(objectKey);
TemporarySignatureResponse res = wosClient.createTemporarySignature(req);
System.out.println("Getting object using temporary signature url:");
System.out.println("\t" + res.getSignedUrl());
```

## 6. An example of WosClient

```
// You may keep only one global instance of WosClient.
// WosClient is thread-safe, which can be used concurrently.
WosClient wosClient = null; 
try
{
    String endPoint = "https://your-endpoint";
    String ak = "*** Provide your Access Key ***";
    String sk = "*** Provide your Secret Key ***";
    // Create an instance of WosClient
    wosClient = new WosClient(ak, sk, endPoint, regionName);
    // Put an object to the bucket
    HeaderResponse response = wosClient.putObject("bucketname", "objectname", new File("localfile"));  
    System.out.println(response);
}
catch (WosException e)
{
    System.out.println("HTTP Code: " + e.getResponseCode());
    System.out.println("Error Code:" + e.getErrorCode());
    System.out.println("Error Message: " + e.getErrorMessage());
    
    System.out.println("Request ID:" + e.getErrorRequestId());
    System.out.println("Host ID:" + e.getErrorHostId());
}finally{
    // Close the client. It's not required to close the client after calling if it's a global instance of WosClient.
    // WosClient is unavailable after calling WosClient.close()
    if(wosClient != null){
        try
        {
            // wosClient.close();
        }
        catch (IOException e)
        {
        }
    }
}
```

# Bucket Management
## 1. List buckets
You can list the buckets with `WosClient.listBuckets`.
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

WosClient wosClient = new WosClient(ak, sk, config, regionName);

// List buckets
List<WosBucket> buckets = wosClient.listBuckets();
for(WosBucket bucket : buckets){
    System.out.println("BucketName:" + bucket.getBucketName());
    System.out.println("CreationDate:" + bucket.getCreationDate());
    System.out.println("Endpoint:" + bucket.getEndpoint());
}
```
Note：
- The bucket list is ordered by bucket name dictionary.

## 2. Query existence of a bucket
You know whether a bucket exits by calling `WosClient.headBucket`.
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

WosClient wosClient = new WosClient(ak, sk, config, regionName);

boolean exists = wosClient.headBucket("bucketname");
```

# Upload

Object is the basic unit in Object Storage. There are some ways to put objects to Object Storage:

- Stream upload
- Object upload
- Multipart upload
- Form-based upload

Object within 5GB can be put to Object Storage by SDK.

For **stream upload** and **object upload**, the object to upload should not be over 5GB.

For large objects, please use **multipart upload**. And every part to upload should not be  over 5GB as well.

**Form-based upload** allows to upload by way of browser form.

SDK supports to upload objects of 0~5GB. For stream upload and object upload, content size should not be over 5GB; if you want to upload a large file, please use multipart upload. And form-based upload provides a way to upload objects based on browser forms.

Anonymous users can access the objects when read permission is set to  anonymous user’s reading. This way, the URL of an object will be like: https://bucket name.domain name/folder name/object name.

## 1. Stream upload
##java.io .InputStream## is used as the resource for stream upload. You can use 'WosClient.putObject' to perform stream upload.

```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);


String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

WosClient wosClient = new WosClient(ak, sk, config, regionName);

String content = "Hello WOS";
wosClient.putObject("bucketname", "objectname", new ByteArrayInputStream(content.getBytes()));

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

WosClient wosClient = new WosClient(ak, sk, config, regionName);

InputStream inputStream = new URL("http://www.a.com").openStream();
wosClient.putObject("bucketname", "objectname", inputStream);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

WosClient wosClient = new WosClient(ak, sk, config, regionName);

FileInputStream fis = new FileInputStream(new File("localfile"));  // 待上传的本地文件路径，需要指定到具体的文件名
wosClient.putObject("bucketname", "objectname", fis);
```
Note:
- For large localfiles, you should always use object upload not stream upload.
- For files over 5GB, multipart upload is recommended.

## 2. File upload
File upload takes local files as resources.
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

WosClient wosClient = new WosClient(ak, sk, config, regionName);

wosClient.putObject("bucketname", "objectname", new File("localfile")); // localfile为待上传的本地文件路径，需要指定到具体的文件名
```
Note:
-  Files to upload should not be larger than 5GB.

## 3. Create a folder
There is no 'folder' in Object Storage, what stored in bucket are objects actually. When you created a 'folder', you have created an object whose size is 0 and name ends by '/'. These 'folders' have no difference with objects, you can download and delete them as well. And they are showed as folders you see on Object Storage.

```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

WosClient wosClient = new WosClient(ak, sk, config, regionName);

final String keySuffixWithSlash = "parent_directory/";
wosClient.putObject("bucketname", keySuffixWithSlash, new ByteArrayInputStream(new byte[0]));


wosClient.putObject("bucketname", keySuffixWithSlash + "objectname", new ByteArrayInputStream("Hello WOS".getBytes()));
```
Note:
- Just create the last level of multi-level folders, for example, to create an folder "src1/src2/src3/", no need to create "src1/", "src1/src2/".

## 4. Set properties of the object
You can set properties of an object before upload, including length, MD5 value, customized metadata, etc.

For details of properties, see in the following:

| Name | Description | Default value |
| -- | -- | -- |
|Object length </br>(Content-Length) | The length of the uploaded object will be truncated if it exceeds the length of the stream/file.	| Actual length of stream/file
|Object MIME type </br>(Content-Type) | The MIME type of the object defines the type of the object and the encoding of the web page, and determines in what form and encoding the browser will read the object.	| application/octet-stream
|Object MD5 value </br>(Content-MD5)| The MD5 value of the object data (encoded by Base64) is provided to the WOS server to verify the integrity of the data. | N/A
|Customized metadata	| The user describes the custom attributes of the objects uploaded to the space in order to customize the management of the objects. | N/A

## Set object length
You can set the object length with 'ObjectMetadata.setContentLength'. 
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

WosClient wosClient = new WosClient(ak, sk, config, regionName);

ObjectMetadata metadata = new ObjectMetadata();
metadata.setContentLength(1024 * 1024L);// 1MB
wosClient.putObject("bucketname", "objectname", new File("localfile"), metadata);
```

## Set MIME type
You can set MIME type with 'ObjectMetadata.setContentType'.
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

WosClient wosClient = new WosClient(ak, sk, config, regionName);

ObjectMetadata metadata = new ObjectMetadata();
metadata.setContentType("image/jpeg");
wosClient.putObject("bucketname", "objectname.jpg", new File("localimage.jpg"), metadata);
```
Note: If MIME type is not set, SDK will set MIMI type by the suffix of an object. For example, "application/xml" is set as MIME type of ".xml"; "text/html" is set as MIME type of ".html".

## Set MD5 value
You can set MD5 with 'ObjectMetadata.setContentMd5'.
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

WosClient wosClient = new WosClient(ak, sk, config, regionName);

ObjectMetadata metadata = new ObjectMetadata();
metadata.setContentMd5("your md5 which should be encoded by base64");
wosClient.putObject("bucketname", "objectname", new File("localimage.jpg"), metadata);
```
Note:

- MD5 value must be Base64 encoded
- 'HTTP 400' is returned when MD5 does not match with MD5 computed by object content.
- MD5 is not verified when MD5 of the object is not set.
- You can directly compute the Content-MD5 header field through 'WosClient.base64Md5'.

## Set customized metadata
You can set customized metadata with 'ObjectMetadata.addUserMetadata'.
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

WosClient wosClient = new WosClient(ak, sk, config, regionName);

ObjectMetadata metadata = new ObjectMetadata();
metadata.addUserMetadata("property1", "property-value1");
metadata.getMetadata().put("property2", "property-value2");
wosClient.putObject("bucketname", "objectname", new File("localfile"), metadata);
```
Note:
- In the code above, two customized metadata are set, they are “property1” and “property2”.
- An object can have multiple metadata,but total size of which cannot exceed 8KB.
- You can get customized metadata with 'WosClient.getObjectMetadata', see details in Get Metadata.
- Customized metadata will be download with the object when using 'WosClient.getObject'.

## 5.分段上传
对于较大文件上传，可以切分成段上传。用户可以在如下的应用场景内（但不仅限于此），使用分段上传的模式：

上传超过100MB大小的文件。
网络条件较差，和WOS服务端之间的链接经常断开。
上传前无法确定将要上传文件的大小。

**使用分段上传具有以下优势：**
- 提高吞吐量—您可以并行上传段以提高吞吐量。
- 从任何网络问题中快速恢复—较小的段大小可以将由于网络错误而需重启失败的上传所产生的影响降至最低。
- 暂停和恢复对象上传—您可以随时上传对象段。启动多段上传后，不存在过期期限；您必须显式地完成或取消多段上传任务。
- 在得知最终对象大小之前开始上传—您可以在创建对象的同时上传对象。

**分段上传分为如下3个步骤：**
1. 初始化分段上传任务（WosClient.initiateMultipartUpload）。
2. 逐个或并行上传段（WosClient.uploadPart）。
3. 合并段（WosClient.completeMultipartUpload）或取消分段上传任务(WosClient.abortMultipartUpload)。

**初始化分段上传任务**
使用分段上传方式传输数据前，必须先通知WOS初始化一个分段上传任务。该操作会返回一个WOS服务端创建的全局唯一标识（Upload ID），用于标识本次分段上传任务。您可以根据这个唯一标识来发起相关的操作，如取消分段上传任务、列举分段上传任务、列举已上传的段等。

您可以通过WosClient.initiateMultipartUpload初始化一个分段上传任务：

该接口可设置的参数如下：

<table class="relative-table wrapped confluenceTable"><colgroup><col style="width: 16.3683%;" /><col style="width: 44.6292%;" /><col style="width: 39.0026%;" /></colgroup><tbody><tr><th class="confluenceTh">参数</th><th class="confluenceTh">作用</th><th class="confluenceTh">WOS Java SDK对应方法</th></tr><tr><td class="confluenceTd">bucketName</td><td class="confluenceTd">空间名称</td><td class="confluenceTd"><br />initiateMultipartUpload.setBucketName</td></tr><tr><td class="confluenceTd">objectKey</td><td class="confluenceTd">设置分段上传任务所属的对象名</td><td class="confluenceTd">initiateMultipartUpload.setObjectKey</td></tr><tr><td class="confluenceTd">expires</td><td class="confluenceTd">设置分段上传任务最终生成对象的过期时间，正整数。</td><td class="confluenceTd">initiateMultipartUpload.setExpires</td></tr><tr><td class="confluenceTd">metadata</td><td class="confluenceTd">设置对象属性，支持content-type，用户自定义元数据</td><td class="confluenceTd">initiateMultipartUpload.setMetadata</td></tr></tbody></table>

```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

InitiateMultipartUploadRequest request = new InitiateMultipartUploadRequest("bucketname", "objectname");
ObjectMetadata metadata = new ObjectMetadata();
metadata.addUserMetadata("property", "property-value");
metadata.setContentType("text/plain");
request.setMetadata(metadata);
InitiateMultipartUploadResult result = wosClient.initiateMultipartUpload(request);

String uploadId = result.getUploadId();
System.out.println("\t" + uploadId);
```
说明：
- 用InitiateMultipartUploadRequest指定上传对象的名称和所属空间。
- 在InitiateMultipartUploadRequest中，您可以设置对象MIME类型、对象存储类型、对象自定义元数据等对象属性。
- InitiateMultipartUploadResult.getUploadId返回分段上传任务的全局唯一标识（Upload ID），在后面的操作中将用到它。

### 上传段
初始化一个分段上传任务之后，可以根据指定的对象名和Upload ID来分段上传数据。每一个上传的段都有一个标识它的号码——分段号（Part Number，范围是1~10000）。对于同一个Upload ID，该分段号不但唯一标识这一段数据，也标识了这段数据在整个对象内的相对位置。如果您用同一个分段号上传了新的数据，那么WOS上已有的这个段号的数据将被覆盖。除了最后一段以外，其他段的大小范围是100KB~5GB；最后段大小范围是0~5GB。每个段不需要按顺序上传，甚至可以在不同进程、不同机器上上传，WOS会按照分段号排序组成最终对象。

您可以通过WosClient.uploadPart上传段：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
String uploadId = "upload id from initiateMultipartUpload";
// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

List<PartEtag> partEtags = new ArrayList<PartEtag>();
// 上传第一段
UploadPartRequest request = new UploadPartRequest("bucketname", "objectname");
// 设置Upload ID
request.setUploadId(uploadId);
// 设置分段号，范围是1~10000，
request.setPartNumber(1);
// 设置将要上传的大文件
request.setFile(new File("localfile"));

// 设置分段大小
request.setPartSize(5 * 1024 * 1024L);
UploadPartResult result = wosClient.uploadPart(request);
partEtags.add(new PartEtag(result.getEtag(), result.getPartNumber()));

// 上传第二段
request = new UploadPartRequest("bucketname", "objectname");
// 设置Upload ID
request.setUploadId(uploadId);
// 设置分段号
request.setPartNumber(2);
// 设置将要上传的大文件
request.setFile(new File("localfile"));
// 设置第二段的段偏移量
request.setOffset(5 * 1024 * 1024L);
// 设置分段大小
request.setPartSize(5 * 1024 * 1024L);
result = wosClient.uploadPart(request);
partEtags.add(new PartEtag(result.getEtag(), result.getPartNumber()));
```
说明：
- 上传段接口要求除最后一段以外，其他的段大小都要大于100KB。但是上传段接口并不会立即校验上传段的大小（因为不知道是否为最后一块）；只有调用合并段接口时才会校验。
- WOS会将服务端收到段数据的ETag值（段数据的MD5值）返回给用户。
- 为了保证数据在网络传输过程中不出现错误，可以通过设置UploadPartRequest.setAttachMd5为true来让SDK自动计算每段数据的MD5值（仅在数据源为本地文件时有效），并放到Content-MD5请求头中；WOS服务端会计算上传数据的MD5值与SDK计算的MD5值比较，保证数据完整性。
- 可以通过UploadPartRequest.setContentMd5直接设置上传数据的MD5值，提供给WOS服务端用于校验数据完整性。如果设置了该值，UploadPartRequest.setAttachMd5参数会被忽略。
- 分段号的范围是1~10000。如果超出这个范围，WOS将返回400 Bad Request错误。

### 合并段
所有分段上传完成后，需要调用合并段接口来在WOS服务端生成最终对象。在执行该操作时，需要提供所有有效的分段列表（包括分段号和分段ETag值）；WOS收到提交的分段列表后，会逐一验证每个段的有效性。当所有段验证通过后，WOS将把这些分段组合成最终的对象。

该接口可设置的参数如下：

<table class="wrapped confluenceTable"><colgroup><col /><col /><col /></colgroup><tbody><tr><th class="confluenceTh">参数</th><th class="confluenceTh">作用</th><th class="confluenceTh">WOS Java SDK对应方法</th></tr><tr><td class="confluenceTd">bucketName</td><td class="confluenceTd">空间名称</td><td class="confluenceTd">completeMultipartUpload.setBucketName</td></tr><tr><td class="confluenceTd">objectKey</td><td class="confluenceTd">设置分段上传任务所属的对象名</td><td class="confluenceTd">completeMultipartUpload.setObjectKey</td></tr><tr><td class="confluenceTd">uploadId</td><td class="confluenceTd">设置分段上传任务的ID号</td><td class="confluenceTd">completeMultipartUpload.setUploadId</td></tr><tr><td class="confluenceTd">partEtag</td><td class="confluenceTd">设置待合并的段列表</td><td class="confluenceTd">completeMultipartUpload.setPartEtag</td></tr></tbody></table>

您可以通过WosClient.completeMultipartUpload合并段：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
String uploadId = "upload id from initiateMultipartUpload";
// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

List<PartEtag> partEtags = new ArrayList<PartEtag>();
// 第一段
PartEtag part1 = new PartEtag();
part1.setPartNumber(1);
part1.seteTag("etag1");
partEtags.add(part1);

// 第二段
PartEtag part2 = new PartEtag();
part2.setPartNumber(2);
part2.setEtag("etag2");
partEtags.add(part2);

CompleteMultipartUploadRequest request = new CompleteMultipartUploadRequest("bucketname", "objectname", uploadId, partEtags);

wosClient.completeMultipartUpload(request);
```
说明：
- 上面代码中的 partEtags是进行上传段后保存的分段号和分段ETag值的列表。
- 分段可以是不连续的。

### 并发分段上传
分段上传的主要目的是解决大文件上传或网络条件较差的情况。下面的示例代码展示了如何使用分段上传并发上传大文件：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
final String bucketName = "bucketname";
final String objectKey = "objectname";
// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

// 初始化线程池
ExecutorService executorService = Executors.newFixedThreadPool(20);
final File largeFile = new File("localfile");

// 初始化分段上传任务
InitiateMultipartUploadRequest request = new InitiateMultipartUploadRequest(bucketName, objectKey);
InitiateMultipartUploadResult result = wosClient.initiateMultipartUpload(request);

final String uploadId = result.getUploadId();
System.out.println("\t"+ uploadId + "\n");

// 每段上传100MB
long partSize = 100 * 1024 * 1024L;
long fileSize = largeFile.length();

// 计算需要上传的段数
long partCount = fileSize % partSize == 0 ? fileSize / partSize : fileSize / partSize + 1;

final List<PartEtag> partEtags = Collections.synchronizedList(new ArrayList<PartEtag>());

// 执行并发上传段
for (int i = 0; i < partCount; i++)
{
    // 分段在文件中的起始位置
    final long offset = i * partSize;
    // 分段大小
    final long currPartSize = (i + 1 == partCount) ? fileSize - offset : partSize;
    // 分段号
    final int partNumber = i + 1;
    executorService.execute(new Runnable()
    {
        @Override
        public void run()
        {
            UploadPartRequest uploadPartRequest = new UploadPartRequest();
            uploadPartRequest.setBucketName(bucketName);
            uploadPartRequest.setObjectKey(objectKey);
            uploadPartRequest.setUploadId(uploadId);
            uploadPartRequest.setFile(largeFile);
            uploadPartRequest.setPartSize(currPartSize);
            uploadPartRequest.setOffset(offset);
            uploadPartRequest.setPartNumber(partNumber);
            
            UploadPartResult uploadPartResult;
            try
            {
                uploadPartResult = wosClient.uploadPart(uploadPartRequest);
                System.out.println("Part#" + partNumber + " done\n");
                partEtags.add(new PartEtag(uploadPartResult.getEtag(), uploadPartResult.getPartNumber()));
            }
            catch (WosException e)
            {
                e.printStackTrace();
            }
        }
    });
}

// 等待上传完成
executorService.shutdown();
while (!executorService.isTerminated())
{
    try
    {
        executorService.awaitTermination(5, TimeUnit.SECONDS);
    }
    catch (InterruptedException e)
    {
        e.printStackTrace();
    }
}
// 合并段
CompleteMultipartUploadRequest completeMultipartUploadRequest = new CompleteMultipartUploadRequest(bucketName, objectKey, uploadId, partEtags);
wosClient.completeMultipartUpload(completeMultipartUploadRequest);
```
说明：
- 大文件分段上传时，使用UploadPartRequest.setOffset和UploadPartRequest.setPartSize来设置每段的起始结束位置。

### 取消分段上传任务
分段上传任务可以被取消，当一个分段上传任务被取消后，就不能再使用其Upload ID做任何操作，已经上传段也会被WOS删除。

采用分段上传方式上传对象过程中或上传对象失败后会在空间内产生段，这些段会占用您的存储空间，您可以通过取消该分段上传任务来清理掉不需要的段，节约存储空间。

您可以通过WosClient.abortMultipartUpload取消分段上传任务：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
String uploadId = "upload id from initiateMultipartUpload";
// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

AbortMultipartUploadRequest request = new AbortMultipartUploadRequest("bucketname", "objectname", uploadId);

wosClient.abortMultipartUpload(request);
```

### 列举已上传的段
您可使用WosClient.listParts列举出某一分段上传任务所有已经上传成功的段。

该接口可设置的参数如下：

<table class="wrapped confluenceTable"><colgroup><col /><col /><col /></colgroup><tbody><tr><th class="confluenceTh">参数</th><th class="confluenceTh">作用</th><th class="confluenceTh">WOS Java SDK对应方法</th></tr><tr><td class="confluenceTd">bucketName</td><td class="confluenceTd">分段上传任务所属的空间名称。</td><td class="confluenceTd">ListPartsRequest.setBucketName</td></tr><tr><td class="confluenceTd">key</td><td class="confluenceTd">分段上传任务所属的对象名。</td><td class="confluenceTd">ListPartsRequest.setKey</td></tr><tr><td class="confluenceTd">uploadId</td><td class="confluenceTd">分段上传任务全局唯一标识，从WosClient.initiateMultipartUpload返回的结果获取。</td><td class="confluenceTd">ListPartsRequest.setUploadId</td></tr><tr><td class="confluenceTd">maxParts</td><td class="confluenceTd">表示列举已上传的段返回结果最大段数目，即分页时每一页中段数目。</td><td class="confluenceTd">ListPartsRequest.setMaxParts</td></tr><tr><td class="confluenceTd">partNumberMarker</td><td class="confluenceTd">表示待列出段的起始位置，只有Part Number大于该参数的段会被列出。</td><td class="confluenceTd">ListPartsRequest.setPartNumberMarker</td></tr></tbody></table>

**简单列举**
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
String uploadId = "upload id from initiateMultipartUpload";
// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

//列举已上传的段，其中uploadId来自于initiateMultipartUpload        
ListPartsRequest request = new ListPartsRequest("bucketname", "objectname");
request.setUploadId(uploadId);
ListPartsResult result = wosClient.listParts(request);

for(Multipart part : result.getMultipartList()){
    // 分段号，上传时候指定
   System.out.println("\t"+part.getPartNumber());
    // 段数据大小
   System.out.println("\t"+part.getSize());
    // 分段的ETag值
   System.out.println("\t"+part.getEtag());
    // 段的最后上传时间
   System.out.println("\t"+part.getLastModified());
}
```
说明：
- 列举段至多返回1000个段信息，如果指定的Upload ID包含的段数量大于1000，则返回结果中ListPartsResult.isTruncated为true表明本次没有返回全部段，并可通过ListPartsResult.getNextPartNumberMarker获取下次列举的起始位置。
- 如果想获取指定Upload ID包含的所有分段，可以采用分页列举的方式。

**列举所有段**
由于WosClient.listParts只能列举至多1000个段，如果段数量大于1000，列举所有分段请参考如下示例：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
String uploadId = "upload id from initiateMultipartUpload";
// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

// 列举所有已上传的段，其中uploadId来自于initiateMultipartUpload        
ListPartsRequest request = new ListPartsRequest("bucketname", "objectname");
request.setUploadId(uploadId);
ListPartsResult result;

do{
    result = wosClient.listParts(request);
    for(Multipart part : result.getMultipartList()){
        // 分段号，上传时候指定
       System.out.println("\t"+part.getPartNumber());
        // 段数据大小
       System.out.println("\t"+part.getSize());
        // 分段的ETag值
       System.out.println("\t"+part.getEtag());
        // 段的最后上传时间
       System.out.println("\t"+part.getLastModified());
    }
    request.setPartNumberMarker(Integer.parseInt(result.getNextPartNumberMarker()));
}while(result.isTruncated());
```

### 列举分段上传任务
您可以通过WosClient.listMultipartUploads列举分段上传任务。列举分段上传任务可设置的参数如下：

<table class="relative-table wrapped confluenceTable"><colgroup><col style="width: 12.3737%;" /><col style="width: 51.0101%;" /><col style="width: 36.6162%;" /></colgroup><tbody><tr><th class="confluenceTh">参数</th><th class="confluenceTh">作用</th><th class="confluenceTh">WOS Java SDK对应方法</th></tr><tr><td class="confluenceTd">bucketName</td><td class="confluenceTd">空间名称</td><td class="confluenceTd">ListMultipartUploadsRequest.setBucketName</td></tr><tr><td class="confluenceTd">prefix</td><td class="confluenceTd">限定返回的分段上传任务中的对象名必须带有prefix前缀。</td><td class="confluenceTd">ListMultipartUploadsRequest.setPrefix</td></tr><tr><td class="confluenceTd">delimiter</td><td class="confluenceTd">用于对分段上传任务中的对象名进行分组的字符。对于对象名中包含delimiter的任务，其对象名（如果请求中指定了prefix，则此处的对象名需要去掉prefix）中从首字符至第一个delimiter之间的字符串将作为一个分组并作为commonPrefix返回。</td><td class="confluenceTd">ListMultipartUploadsRequest.setDelimiter</td></tr><tr><td class="confluenceTd">maxUploads</td><td class="confluenceTd">列举分段上传任务的最大数目，取值范围为1~1000，当超出范围时，按照默认的1000进行处理。</td><td class="confluenceTd">ListMultipartUploadsRequest.setMaxUploads</td></tr><tr><td class="confluenceTd">keyMarker</td><td class="confluenceTd">表示列举时返回指定的keyMarker之后的分段上传任务。</td><td class="confluenceTd">ListMultipartUploadsRequest.setKeyMarker</td></tr><tr><td class="confluenceTd">uploadIdMarker</td><td class="confluenceTd">只有与keyMarker参数一起使用时才有意义，用于指定返回结果的起始位置，即列举时返回指定keyMarker的uploadIdMarker之后的分段上传任务。</td><td class="confluenceTd">ListMultipartUploadsRequest.setUploadIdMarker</td></tr></tbody></table>

### 简单列举分段上传任务
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
String uploadId = "upload id from initiateMultipartUpload";
// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

ListMultipartUploadsRequest request = new ListMultipartUploadsRequest("bucketname");

MultipartUploadListing result = wosClient.listMultipartUploads(request);
for(MultipartUpload upload : result.getMultipartTaskList()){
    System.out.println("\t" + upload.getUploadId());
    System.out.println("\t" + upload.getObjectKey());
    System.out.println("\t" + upload.getInitiatedDate());
}
```
说明：
- 列举分段上传任务至多返回1000个任务信息，如果指定的空间包含的分段上传任务数量大于1000，则MultipartUploadListing.isTruncated为true表明本次没有返回全部结果，并可通过MultipartUploadListing.getNextKeyMarker和MultipartUploadListing.getNextUploadIdMarker获取下次列举的起点。
- 如果想获取指定空间包含的所有分段上传任务，可以采用分页列举的方式。

### 分页列举全部分段上传任务
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
String uploadId = "upload id from initiateMultipartUpload";
// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

ListMultipartUploadsRequest request = new ListMultipartUploadsRequest("bucketname");
MultipartUploadListing result;

do{
    result = wosClient.listMultipartUploads(request);
    for(MultipartUpload upload : result.getMultipartTaskList()){
        System.out.println("\t" + upload.getUploadId());
        System.out.println("\t" + upload.getObjectKey());
        System.out.println("\t" + upload.getInitiatedDate());
    }
    request.setKeyMarker(result.getNextKeyMarker());
    request.setUploadIdMarker(result.getNextUploadIdMarker());
}while(result.isTruncated());
```

## 基于表单上传
基于表单上传是使用HTML表单形式上传对象到指定空间中，对象最大不能超过5GB。

您可以通过WosClient.createPostSignature生成基于表单上传的请求参数。使用代码模拟表单上传的完整代码示例，参见PostObjectSample。您也可以通过如下步骤进行表单上传：

使用WosClient.createPostSignature生成用于鉴权的请求参数。
准备表单HTML页面。
将生成的请求参数填入HTML页面。
选择本地文件，进行表单上传。
说明：
使用SDK生成用于鉴权的请求参数包括两个：
```
policy，对应表单中policy字段。
signature，对应表单中的signature字段。
以下代码展示了如何生成基于表单上传的请求参数：

WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "http://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

PostSignatureRequest request = new PostSignatureRequest();
// 设置表单参数
Map<String, Object> formParams = new HashMap<String, Object>();
// 设置对象访问权限为公共读
formParams.put("x-wos-acl", "public-read");
// 设置对象MIME类型
formParams.put("content-type", "text/plain");

request.setFormParams(formParams);
// 设置表单上传请求有效期，单位：秒
request.setExpires(3600);
PostSignatureResponse response = wosClient.createPostSignature(request);
formParams.put("key", objectKey);
formParams.put("policy", response.getPolicy());
formParams.put("accesskeyid", ak);
formParams.put("x-wos-credential", response.getCredential());
formParams.put("x-wos-algorithm", response.getAlgorithm());
formParams.put("bucket", bucketName);
formParams.put("x-wos-date", response.getDate());
formParams.put("x-wos-signature", response.getSignature());

// 获取表单上传请求参数
System.out.println("\t" + response.getPolicy());
System.out.println("\t" + response.getSignature());
```

示例表单HTML代码如下：
```
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>

<form action="http://bucketname.your-endpoint/" method="post" enctype="multipart/form-data">
Object key
<!-- 对象名 -->
<input type="text" name="key" value="objectname" />
<p>
ACL
<!-- 对象ACL权限 -->
<input type="text" name="x-wos-acl" value="public-read" />
<p>
Content-Type
<!-- 对象MIME类型 -->
<input type="text" name="content-type" value="text/plain" />
<p>
<!-- policy的base64编码值 -->
<input type="hidden" name="policy" value="*** Provide your policy ***" />
<!-- AK -->
<input type="hidden" name="AccessKeyId" value="*** Provide your access key ***"/>
<!-- 签名串信息 -->
<input type="hidden" name="signature" value="*** Provide your signature ***"/>

<input name="file" type="file" />
<input name="submit" value="Upload" type="submit" />
</form>
</body>
</html>
```
说明：
- HTML表单中的policy，signature的值均是从WosClient.createPostSignature的返回结果中获取。
- 您可以直接下载表单HTML示例PostDemo。

## 下载对象
对象下载简介
WOS Java SDK提供了丰富的对象下载接口，您可以通过WosClient.getObject下载对象。

### 1.流式下载
以下代码展示了如何进行流式下载：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

WosObject wosObject = WosClient.getObject("bucketname", "objectname");

// 读取对象内容
System.out.println("Object content:");
InputStream input = wosObject.getObjectContent();
byte[] b = new byte[1024];
ByteArrayOutputStream bos = new ByteArrayOutputStream(); 
int len;
while ((len=input.read(b)) != -1){
       bos.write(b, 0, len);
}
 
System.out.println(new String(bos.toByteArray()));
bos.close();
input.close();
```
说明：
- WosClient.getObject的返回实例WosObject实例包含对象所在的空间、对象名、对象属性、对象输入流等。
- 通过操作对象输入流可将对象的内容读取到本地文件或者内存中。

须知：
- WosObject.getObjectContent获取的对象输入流一定要显式关闭，否则会造成资源泄露。

## 2.范围下载
如果只需要下载对象的其中一部分数据，可以使用范围下载，下载指定范围的数据。如果指定的下载范围是0~1000，则返回第0到第1000个字节的数据，包括第1000个，共1001字节的数据，即[0， 1000]。如果指定的范围无效，则返回整个对象的数据。以下代码展示了如何进行范围下载：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

GetObjectRequest request = new GetObjectRequest("bucketname", "objectname");
// 指定开始和结束范围
request.setRangeStart(0l);
request.setRangeEnd(1000l);
WosObject wosObject = wosClient.getObject(request);

// 读取数据
byte[] buf = new byte[1024];
InputStream in = wosObject.getObjectContent();
for (int n = 0; n != -1; ) {
    n = in.read(buf, 0, buf.length);
}

in.close();
```
说明：
- 如果指定的范围无效（比如开始位置、结束位置为负数，大于文件大小），则会返回整个对象。
- 可以利用范围下载并发下载大对象，详细代码示例请参考ConcurrentDownloadObjectSample。

## 3.限定条件下载
下载对象时，可以指定一个或多个限定条件，满足限定条件时则进行下载，否则抛出异常，下载对象失败。

您可以使用的限定条件如下：

<table class="wrapped confluenceTable"><colgroup><col /><col /><col /></colgroup><tbody><tr><th class="confluenceTh">参数</th><th class="confluenceTh">作用</th><th class="confluenceTh">WOS Java SDK对应方法</th></tr><tr><td class="confluenceTd">If-Modified-Since</td><td class="confluenceTd">如果对象的修改时间晚于该参数值指定的时间，则返回对象内容，否则抛出异常。</td><td class="confluenceTd">GetObjectRequest.setIfModifiedSince</td></tr><tr><td class="confluenceTd">If-Unmodified-Since</td><td class="confluenceTd">如果对象的修改时间早于该参数值指定的时间，则返回对象内容，否则抛出异常。</td><td class="confluenceTd">GetObjectRequest.setIfUnmodifiedSince</td></tr><tr><td class="confluenceTd">If-Match</td><td class="confluenceTd">如果对象的ETag值与该参数值相同，则返回对象内容，否则抛出异常。</td><td class="confluenceTd">GetObjectRequest.setIfMatchTag</td></tr><tr><td class="confluenceTd">If-None-Match</td><td class="confluenceTd">如果对象的ETag值与该参数值不相同，则返回对象内容，否则抛出异常。</td><td class="confluenceTd">GetObjectRequest.setIfNoneMatchTag</td></tr></tbody></table>

说明：
- 对象的ETag值是指对象数据的MD5校验值。
- 如果包含If-Unmodified-Since并且不符合或者包含If-Match并且不符合，抛出异常中HTTP状态码为：412 precondition failed。
- 如果包含If-Modified-Since并且不符合或者包含If-None-Match并且不符合，抛出异常中HTTP状态码为：304 Not Modified。

以下代码展示了如何进行限定条件下载：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

GetObjectRequest request = new GetObjectRequest("bucketname", "objectname");
request.setRangeStart(0l);
request.setRangeEnd(1000l);

request.setIfModifiedSince(new SimpleDateFormat("yyyy-MM-dd").parse("2016-01-01"));
WosObject wosObject = wosClient.getObject(request);

wosObject.getObjectContent().close();
```

## 4.重写响应头
下载对象时，可以重写部分HTTP/HTTPS响应头信息。可重写的响应头信息见下表：

<table class="wrapped confluenceTable"><colgroup><col /><col /><col /></colgroup><tbody><tr><th class="confluenceTh">参数</th><th class="confluenceTh">作用</th><th class="confluenceTh">WOS Java SDK对应方法</th></tr><tr><td class="confluenceTd">contentType</td><td class="confluenceTd">重写HTTP/HTTPS响应中的Content-Type</td><td class="confluenceTd">ObjectRepleaceMetadata.setContentType</td></tr><tr><td class="confluenceTd">contentLanguage</td><td class="confluenceTd">重写HTTP/HTTPS响应中的Content-Language</td><td class="confluenceTd">ObjectRepleaceMetadata.setContentLanguage</td></tr><tr><td class="confluenceTd">expires</td><td class="confluenceTd">重写HTTP/HTTPS响应中的Expires</td><td class="confluenceTd">ObjectRepleaceMetadata.setExpires</td></tr><tr><td class="confluenceTd">cacheControl</td><td class="confluenceTd">重写HTTP/HTTPS响应中的Cache-Control</td><td class="confluenceTd">ObjectRepleaceMetadata.setCacheControl</td></tr><tr><td class="confluenceTd">contentDisposition</td><td class="confluenceTd">重写HTTP/HTTPS响应中的Content-Disposition</td><td class="confluenceTd">ObjectRepleaceMetadata.setContentDisposition</td></tr><tr><td class="confluenceTd">contentEncoding</td><td class="confluenceTd">重写HTTP/HTTPS响应中的Content-Encoding</td><td class="confluenceTd">ObjectRepleaceMetadata.setContentEncoding</td></tr></tbody></table>

以下代码展示了如何重写响应头：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

GetObjectRequest request = new GetObjectRequest("bucketname", "objectname");
ObjectRepleaceMetadata replaceMetadata = new ObjectRepleaceMetadata();
replaceMetadata.setContentType("image/jpeg");
request.setReplaceMetadata(replaceMetadata);

WosObject wosObject = wosClient.getObject(request);
System.out.println(wosObject.getMetadata().getContentType());

wosObject.getObjectContent().close();
```

## 5.获取自定义元数据
下载对象成功后会返回对象的自定义元数据。以下代码展示了如何获取自定义元数据：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

// 上传对象，设置自定义元数据
PutObjectRequest request = new PutObjectRequest("bucketname", "objectname");
ObjectMetadata metadata = new ObjectMetadata();
metadata.addUserMetadata("property", "property-value");
request.setMetadata(metadata);
wosClient.putObject(request);

// 下载对象，获取对象自定义元数据
WosObject wosObject = wosClient.getObject("bucketname", "objectname");
System.out.println(wosObject.getMetadata().getUserMetadata("property"));

wosObject.getObjectContent().close();
```

## 6.下载归档存储对象
如果要下载归档存储对象，需要先将归档存储对象取回，您可以通过WosClient.restoreObject取回归档存储对象。以下代码展示了如何下载归档存储对象：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

RestoreObjectRequest request = new RestoreObjectRequest();
request.setBucketName("bucketname");
request.setObjectKey("objectname");
request.setDays(1);
wosClient.restoreObject(request);

// 等待对象取回
Thread.sleep(60 * 6 * 1000);

// 下载对象
WosObject wosObject = wosClient.getObject("bucketname", "objectname");

wosObject.getObjectContent().close();
```
说明：
- WosClient.restoreObject中指定的对象必须是归档存储类型，否则调用该接口会抛出异常。

# 管理对象
## 1.设置对象属性
您可以通过WosClient.setObjectMetadata来设置对象属性，包括对象自定义元数据等信息。以下代码展示了如何设置对象属性：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

SetObjectMetadataRequest request = new SetObjectMetadataRequest("bucketname", "objectname");
request.addUserMetadata("property1", "property-value1");
// 设置自定义元数据
ObjectMetadata metadata = wosClient.setObjectMetadata(request);

System.out.println("\t" + metadata.getUserMetadata("property1"));
```

## 2.获取对象属性
您可以通过WosClient.getObjectMetadata来获取对象属性，包括对象长度，对象MIME类型，对象自定义元数据等信息。以下代码展示了如何获取对象属性：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

ObjectMetadata metadata = wosClient.getObjectMetadata("bucketname", "objectname");
System.out.println("\t" + metadata.getContentType());
System.out.println("\t" + metadata.getContentLength());
System.out.println("\t" + metadata.getUserMetadata("property"));
```

## 3.获取对象访问权限
您可以通过WosClient.getObjectAcl获取对象的访问权限。以下代码展示如何获取对象访问权限：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

AccessControlList acl = wosClient.getObjectAcl("bucketname", "objectname");
System.out.println(acl);
```

## 4.列举对象
您可以通过WosClient.listObjects列举出空间里的对象。

该接口可设置的参数如下：

<table class="relative-table wrapped confluenceTable"><colgroup><col style="width: 10.8287%;" /><col style="width: 62.9834%;" /><col style="width: 26.1878%;" /></colgroup><tbody><tr><th class="confluenceTh">参数</th><th class="confluenceTh">作用</th><th class="confluenceTh">WOS Java SDK对应方法</th></tr><tr><td class="confluenceTd">bucketName</td><td class="confluenceTd">空间名称</td><td class="confluenceTd">ListObjectsRequest.setBucketName</td></tr><tr><td class="confluenceTd">prefix</td><td class="confluenceTd">限定返回的对象名必须带有prefix前缀。</td><td class="confluenceTd">ListObjectsRequest.setPrefix</td></tr><tr><td class="confluenceTd">marker</td><td class="confluenceTd">列举对象的起始位置，返回的对象列表将是对象名按照字典序排序后该参数以后的所有对象。</td><td class="confluenceTd">ListObjectsRequest.setMarker</td></tr><tr><td class="confluenceTd">maxKeys</td><td class="confluenceTd">列举对象的最大数目，取值范围为1~1000，当超出范围时，按照默认的1000进行处理。</td><td class="confluenceTd">ListObjectsRequest.setMaxKeys</td></tr><tr><td class="confluenceTd">delimiter</td><td class="confluenceTd">用于对对象名进行分组的字符。对于对象名中包含delimiter的对象，其对象名（如果请求中指定了prefix，则此处的对象名需要去掉prefix）中从首字符至第一个delimiter之间的字符串将作为一个分组并作为commonPrefix返回。</td><td class="confluenceTd">ListObjectsRequest.setDelimiter</td></tr></tbody></table>

### 简单列举
以下代码展示如何简单列举对象，最多返回1000个对象：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

ObjectListing result = wosClient.listObjects("bucketname");
for(WosObject wosObject : result.getObjects()){
    System.out.println("\t" + wosObject.getObjectKey());
    System.out.println("\t" + wosObject.getOwner());
}
```
说明：
- 每次至多返回1000个对象，如果指定空间包含的对象数量大于1000，则返回结果中ObjectListing.isTruncated为true表明本次没有返回全部对象，并可通过ObjectListing.getNextMarker获取下次列举的起始位置。
- 如果想获取指定空间包含的所有对象，可以采用分页列举的方式。

### 指定数目列举
以下代码展示如何指定数目列举对象：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

ListObjectsRequest request = new ListObjectsRequest("bucketname");
// 只列举100个对象
request.setMaxKeys(100);
ObjectListing result = wosClient.listObjects(request);
for(WosObject wosObject : result.getObjects()){
    System.out.println("\t" + wosObject.getObjectKey());
    System.out.println("\t" + wosObject.getOwner());
}
```

### 指定前缀列举

以下代码展示如何指定前缀列举对象：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

ListObjectsRequest request = new ListObjectsRequest("bucketname");
// 设置列举带有prefix前缀的100个对象
request.setMaxKeys(100);
request.setPrefix("prefix");
ObjectListing result = wosClient.listObjects(request);
for(WosObject wosObject : result.getObjects()){
    System.out.println("\t" + wosObject.getObjectKey());
    System.out.println("\t" + wosObject.getOwner());
}
```

### 指定起始位置列举
以下代码展示如何指定起始位置列举对象：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

ListObjectsRequest request = new ListObjectsRequest("bucketname");
// 设置列举对象名字典序在"test"之后的100个对象
request.setMaxKeys(100);
request.setMarker("test");
ObjectListing result = wosClient.listObjects(request);
for(WosObject wosObject : result.getObjects()){
    System.out.println("\t" + wosObject.getObjectKey());
    System.out.println("\t" + wosObject.getOwner());
}
```

### 分页列举全部对象
以下代码展示分页列举全部对象：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

ListObjectsRequest request = new ListObjectsRequest("bucketname");
// 设置每页100个对象
request.setMaxKeys(100);

ObjectListing result;
do{
    result = wosClient.listObjects(request);
    for(WosObject wosObject : result.getObjects()){
        System.out.println("\t" + wosObject.getObjectKey());
        System.out.println("\t" + wosObject.getOwner());
    }
    
    request.setMarker(result.getNextMarker());
}while(result.isTruncated());
```

### 列举文件夹中的所有对象
WOS本身是没有文件夹的概念的，空间中存储的元素只有对象。文件夹对象实际上是一个大小为0且对象名以“/”结尾的对象，将这个文件夹对象名作为前缀，即可模拟列举文件夹中对象的功能。以下代码展示如何列举文件夹中的对象：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

ListObjectsRequest request = new ListObjectsRequest("bucketname");
// 设置文件夹对象名"dir/"为前缀
request.setPrefix("dir/");
request.setMaxKeys(1000);

ObjectListing result;

do{
    result = wosClient.listObjects(request);
    for (WosObject wosObject : result.getObjects())
    {
        System.out.println("\t" + wosObject.getObjectKey());
        System.out.println("\t" + wosObject.getOwner());
    }
    request.setMarker(result.getNextMarker());
}while(result.isTruncated());
```

### 按文件夹分组列举所有对象
以下代码展示如何按文件夹分组，列举空间内所有对象：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

ListObjectsRequest request = new ListObjectsRequest("bucketname");
request.setMaxKeys(1000);
// 设置文件夹分隔符"/"
request.setDelimiter("/");
ObjectListing result = wosClient.listObjects(request);
System.out.println("Objects in the root directory:");
for(WosObject wosObject : result.getObjects()){
    System.out.println("\t" + wosObject.getObjectKey());
    System.out.println("\t" + wosObject.getOwner());
}
listObjectsByPrefix(wosClient, request, result);
```

### 递归列出子文件夹中对象的函数listObjectsByPrefix的示例代码如下：
```
static void listObjectsByPrefix(WosClient wosClient, ListObjectsRequest request, ObjectListing result) throws WosException
{
      for(String prefix : result.getCommonPrefixes()){
          System.out.println("Objects in folder [" + prefix + "]:");
          request.setPrefix(prefix);
          result  = wosClient.listObjects(request);
          for(WosObject wosObject : result.getObjects()){
              System.out.println("\t" + wosObject.getObjectKey());
              System.out.println("\t" + wosObject.getOwner());
          }
          listObjectsByPrefix(wosClient, request, result);
      }
}
```
说明：
- 以上代码示例没有考虑文件夹中对象数超过1000个的情况。
- 由于是需要列举出文件夹中的对象和子文件夹，且文件夹对象总是以“/”结尾，因此delimiter总是为“/”。
- 每次递归的返回结果中ObjectListing.getObjects包含的是文件夹中的对象；ObjectListing.getCommonPrefixes包含的是文件夹的子文件夹。

## 5.删除对象
### 删除单个对象
您可以通过WosClient.deleteObject删除单个对象。以下代码展示如何删除单个对象：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);
wosClient.deleteObject("bucketname", "objectname");
```

### 批量删除对象
您可以通过WosClient.deleteObjects批量删除对象，每次最多删除1000个对象。
以下代码展示了如何进行批量删除空间内所有对象：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
WosClient wosClient = new WosClient(ak, sk, config, regionName);

DeleteObjectsRequest deleteRequest = new DeleteObjectsRequest("bucketname");
wosClient.deleteObjects(deleteRequest);
```

## 6.复制对象
复制对象即为WOS上已经存在的对象创建一个副本。

您可以通过WosClient.copyObject来复制对象。复制对象时，可重新指定新对象的属性和设置对象权限，且支持条件复制。

操作限制

- 用户有待复制的源对象的读权限。
- 不支持跨区域复制。
- 待复制的源对象的大小不能超过5GB。小于1GB时，建议使用简单复制；大于1GB时，建议使用分段复制。
- 如果待复制的源对象是归档存储类型，则必须先取回源对象才能进行复制。

### 简单复制
以下代码展示了如何进行简单复制：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

try{
    CopyObjectResult result = wosClient.copyObject("sourcebucketname", "sourceobjectname", "destbucketname", "destobjectname");

    System.out.println("\t" + result.getStatusCode());
    System.out.println("\t" + result.getEtag());
}
catch (WosException e)
{
    // 复制失败
    System.out.println("HTTP Code: " + e.getResponseCode());
    System.out.println("Error Code:" + e.getErrorCode());
    System.out.println("Error Message: " + e.getErrorMessage());

    System.out.println("Request ID:" + e.getErrorRequestId());
    System.out.println("Host ID:" + e.getErrorHostId());
}
```

### 重写对象属性
以下代码展示了如何在复制对象时重写对象属性：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

CopyObjectRequest request = new CopyObjectRequest("sourcebucketname", "sourceobjectname", "destbucketname", "destobjectname");
// 设置进行对象属性重写
request.setReplaceMetadata(true);
ObjectMetadata newObjectMetadata = new ObjectMetadata();
newObjectMetadata.setContentType("image/jpeg");
newObjectMetadata.addUserMetadata("property", "property-value");
newObjectMetadata.setObjectStorageClass(StorageClassEnum.WARM);
request.setNewObjectMetadata(newObjectMetadata);
CopyObjectResult result = wosClient.copyObject(request);
System.out.println("\t" + result.getEtag());
```
说明：
- CopyObjectRequest.setReplaceMetadata需与CopyObjectRequest.setNewObjectMetadata配合使用。

### 限定条件复制
复制对象时，可以指定一个或多个限定条件，满足限定条件时则进行复制，否则抛出异常，复制对象失败。

您可以使用的限定条件如下：

 <table class="wrapped confluenceTable"><colgroup><col /><col /><col /></colgroup><tbody><tr><th class="confluenceTh">&nbsp;参数</th><th class="confluenceTh">作用</th><th class="confluenceTh">WOS Java SDK对应方法</th></tr><tr><td class="confluenceTd">Copy-Source-If-Modified-Since</td><td class="confluenceTd">如果源对象的修改时间晚于该参数值指定的时间，则进行复制，否则抛出异常。</td><td class="confluenceTd">CopyObjectRequest.setIfModifiedSince</td></tr><tr><td class="confluenceTd">Copy-Source-If-Unmodified-Since</td><td class="confluenceTd">如果源对象的修改时间早于该参数值指定的时间，则进行复制，否则抛出异常。</td><td class="confluenceTd">CopyObjectRequest.setIfUnmodifiedSince</td></tr><tr><td class="confluenceTd">Copy-Source-If-Match</td><td class="confluenceTd">如果源对象的ETag值与该参数值相同，则进行复制，否则抛出异常。</td><td class="confluenceTd">CopyObjectRequest.setIfMatchTag</td></tr><tr><td class="confluenceTd">Copy-Source-If-None-Match</td><td class="confluenceTd">如果源对象的ETag值与该参数值不相同，则进行复制，否则抛出异常。</td><td class="confluenceTd">CopyObjectRequest.setIfNoneMatchTag</td></tr></tbody></table>

说明：
- 源对象的ETag值是指源对象数据的MD5校验值。
- 如果包含Copy-Source-If-Unmodified-Since并且不符合，或者包含Copy-Source-If-Match并且不符合，或者包含Copy-Source-If-Modified-Since并且不符合，或者包含Copy-Source-If-None-Match并且不符合，则复制失败，抛出异常中HTTP状态码为：412 precondition failed。
- Copy-Source-If-Modified-Since和Copy-Source-If-None-Match可以一起使用；Copy-Source-If-Unmodified-Since和Copy-Source-If-Match可以一起使用。

以下代码展示了如何进行限定条件复制：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

CopyObjectRequest request = new CopyObjectRequest("sourcebucketname", "sourceobjectname", "destbucketname", "destobjectname");

request.setIfModifiedSince(new SimpleDateFormat("yyyy-MM-dd").parse("2016-01-01"));
request.setIfNoneMatchTag("none-match-etag");

CopyObjectResult result = wosClient.copyObject(request);
System.out.println("\t" + result.getEtag());
```

### 分段复制
分段复制是分段上传的一种特殊情况，即分段上传任务中的段通过复制WOS指定空间中现有对象（或对象的一部分）来实现。您可以通过WosClient.copyPart来复制段。以下代码展示了如何使用分段复制模式复制大对象：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

final String destBucketName = "destbucketname";
final String destObjectKey = "destobjectname";
final String sourceBucketName = "sourcebucketname";
final String sourceObjectKey = "sourceobjectname";
// 创建WosClient实例
final WosClient wosClient = new WosClient(ak, sk, config, regionName);

// 初始化线程池
ExecutorService executorService = Executors.newFixedThreadPool(20);

// 初始化分段上传任务
InitiateMultipartUploadRequest request = new InitiateMultipartUploadRequest(destBucketName, destObjectKey);
InitiateMultipartUploadResult result = wosClient.initiateMultipartUpload(request);

final String uploadId = result.getUploadId();
System.out.println("\t"+ uploadId + "\n");

// 获取大对象信息
ObjectMetadata metadata = wosClient.getObjectMetadata(sourceBucketName, sourceObjectKey);
// 每段复制100MB
long partSize = 100 * 1024 * 1024L;
long objectSize = metadata.getContentLength();

// 计算需要复制的段数
long partCount = objectSize % partSize == 0 ? objectSize / partSize : objectSize / partSize + 1;

final List<PartEtag> partEtags = Collections.synchronizedList(new ArrayList<PartEtag>());

// 执行并发复制段
for (int i = 0; i < partCount; i++)
{
    // 复制段起始位置
    final long rangeStart = i * partSize;
    // 复制段结束位置
    final long rangeEnd = (i + 1 == partCount) ? objectSize - 1 : rangeStart + partSize - 1;
    // 分段号
    final int partNumber = i + 1;
    executorService.execute(new Runnable()
    {
        
        @Override
        public void run()
        {
            CopyPartRequest request = new CopyPartRequest();
            request.setUploadId(uploadId);
            request.setSourceBucketName(sourceBucketName);
            request.setSourceObjectKey(sourceObjectKey);
            request.setDestinationBucketName(destBucketName);
            request.setDestinationObjectKey(destObjectKey);
            request.setByteRangeStart(rangeStart);
            request.setByteRangeEnd(rangeEnd);
            request.setPartNumber(partNumber);
            CopyPartResult result;
            try
            {
                result = wosClient.copyPart(request);
                System.out.println("Part#" + partNumber + " done\n");
                partEtags.add(new PartEtag(result.getEtag(), result.getPartNumber()));
            }
            catch (WosException e)
            {
                e.printStackTrace();
            }
        }
    });
}

// 等待复制完成
executorService.shutdown();
while (!executorService.isTerminated())
{
    try
    {
        executorService.awaitTermination(5, TimeUnit.SECONDS);
    }
    catch (InterruptedException e)
    {
        e.printStackTrace();
    }
}

// 合并段
CompleteMultipartUploadRequest completeMultipartUploadRequest = new CompleteMultipartUploadRequest(destBucketName, destObjectKey, uploadId, partEtags);
wosClient.completeMultipartUpload(completeMultipartUploadRequest);
```

# 生命周期管理
生命周期管理简介
WOS允许您对空间设置生命周期规则，实现自动转换对象的存储类型、自动淘汰过期的对象，以有效利用存储特性，优化存储空间。针对不同前缀的对象，您可以同时设置多条规则。一条规则包含：

- 规则ID，用于标识一条规则，不能重复。
- 受影响的对象前缀，此规则只作用于符合前缀的对象。
- 最新版本对象的转换策略，指定方式为：
- 指定满足前缀的对象创建后第几天时转换为指定的存储类型。
- 直接指定满足前缀的对象转换为指定的存储类型的日期。
- 最新版本对象过期时间，指定方式为：
- 指定满足前缀的对象创建后第几天时过期。
- 直接指定满足前缀的对象过期日期。
- 是否生效标识。

## 1.设置生命周期规则
您可以通过WosClient.setBucketLifecycleConfiguration设置空间的生命周期规则。
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";

// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

final String ruleId = "delete obsoleted files";
final String matchPrefix = "obsoleted/";

StringBuffer sb = new StringBuffer();

LifecycleConfiguration lifecycleConfig = new LifecycleConfiguration();
LifecycleConfiguration.Rule rule = lifecycleConfig.new Rule();
rule.setEnabled(true);
rule.setId(ruleId);
rule.setPrefix(matchPrefix);
LifecycleConfiguration.Expiration expiration = lifecycleConfig.new Expiration();
expiration.setDays(10);

rule.setExpiration(expiration);
lifecycleConfig.addRule(rule);

sb.append("Setting bucket lifecycle\n\n");
HeaderResponse headerResponse = wosClient.setBucketLifecycleConfiguration(bucketName, lifecycleConfig);
System.out.println(headerResponse);
```

## 2.查看生命周期规则
您可以通过WosClient.getBucketLifecycle查看空间的生命周期规则。以下代码展示了如何查看空间的生命周期规则：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

LifecycleConfiguration config = wosClient.getBucketLifecycle("bucketname");

for (Rule rule : config.getRules())
{
    System.out.println(rule.getId());
    System.out.println(rule.getPrefix());
    for(Transition transition : rule.getTransitions()){
        System.out.println(transition.getDays());
        System.out.println(transition.getStorageClass());
    }
    System.out.println(rule.getExpiration() != null ? rule.getExpiration().getDays() : "");
}
```

## 3.删除生命周期规则
您可以通过WosClient.deleteBucketLifecycle删除空间的生命周期规则。以下代码展示了如何删除空间的生命周期规则：
```
WosConfiguration config = new WosConfiguration();
config.setSocketTimeout(30000);
config.setConnectionTimeout(10000);
config.setEndPoint(endPoint);

String endPoint = "https://your-endpoint";
String ak = "*** Provide your Access Key ***";
String sk = "*** Provide your Secret Key ***";
// 创建WosClient实例
WosClient wosClient = new WosClient(ak, sk, config, regionName);

wosClient.deleteBucketLifecycle("bucketname");
```
![image](https://user-images.githubusercontent.com/98135632/153330698-be129814-63e1-48fd-b1ad-1540c4609f2a.png)
