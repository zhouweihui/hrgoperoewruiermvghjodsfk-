# 加载一张(bitmap)
Bitmap所占用的内存 = 图片长度 x 图片宽度 x 一个像素点占用的字节数。3个参数，任意减少一个的值，就达到了压缩的效果。
## 加载:bitmap factory
```
decodeFile()   //从SD卡文件读取
Bitmap bm = BitmapFactory.decodeFile(Environment.
getExternalStorageDirectory().getAbsolutePath()+"/photo.jpg");

decodeResource()   //从资源文件res读取
Bitmap bm = BitmapFactory.
decodeResource(this.getResources(), R.mipmap.test_pic);

decodeStream()   //从输入流读取
Bitmap bm = BitmapFactory.decodeStream(inputStream);

decodeByteArray()   //从字节数组读取
Bitmap bm = BitmapFactory.decodeByteArray(bytes, 0, bytes.length);

```
## 缩放:bitmapFactoryOptions
用于BitmapFactory的decode方法中，其属性in开头的代表的就是设置某某参数;out开头的代表的就是获取某某参数
- inJustDecodeBounds ：设置只去读图片的附加信息(宽高),不去解析真实的Bitmap
- outHeight outWidth 代表图片的宽高
- inSampleSize 缩放比例，官方推荐只能为2,4,8，16。。。

## 颜色深度:Bitmap.Config
ALPHA_8,ARGB_4444,ARGB_8888,RGB_565
```
BitmapFactory.Options options=new BitmapFactory.Options();
options.inPreferredConfig=Config.ARGB_8888;
```

## 压缩输出:Bitmap.compress()方法
使用该方法需要传三个参数进去：CompressFormat、int类型的quality、OutputStream
- CompressFormat
指定Bitmap的压缩格式，可选择JPEG、PNG、WEBP
- int类型的quality
指定Bitmap的压缩品质，范围是0 ~ 100；该值越高，画质越高。0表示画质最差，100画质最高。
- OutputStream
指定Bitmap的字节输出流。一般使用：
ByteArrayOutputStream stream = new ByteArrayOutputStream();

# 加载多张(cache)
- 缓存意义：节省流量，提高用户体验
- 缓存策略: 缓存的添加，获取，和删除；主要是删除的策略

## 内存缓存
- LruCache<K,V> ：采用Linkhashmap 以强应用的方式保存外界缓存的对象，以get和put方式完成缓存的获取，remove可以删除一个指定的缓存的对象
- 初始化: 指定泛型的类型，以缓存容量作为构造函数，重写sizeof（K,V），计算每个对象的大小，它的大小单位与缓存容量的单位一致。

## 存储设备缓存（DiskLruCache）
- 初始化:不能用构造方法，需要用open方法，传入路径，版本号，单个节点对于数据个数，缓存总字节数
- 增加: 调用edit(key),取得DiskLruCache.Editor 对象。通过Editor.newOutputStream(DISK_CACHE_INDEX)获取OutputStream对象
- 查找: 调用get(key),取得调用DiskLruCache.Snapshot,通过DiskLruCache.Snapshot的getInputStream取得输入流。

## UIL的设计
## 工作流
![1]
[1]:https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tool-lib/image-cache/universal-image-loader/image/uil-flow.png
## 模块
整个库分为ImageLoaderEngine，Cache及ImageDownloader，ImageDecoder，BitmapDisplayer，BitmapProcessor五大模块，其中Cache分为MemoryCache和DiskCache两部分。
简单的讲就是ImageLoader收到加载及显示图片的任务，并将它交给ImageLoaderEngine，ImageLoaderEngine分发任务到具体线程池去执行，任务通过Cache及ImageDownloader获取图片，中间可能经过BitmapProcessor和ImageDecoder处理，最终转换为Bitmap交给BitmapDisplayer在ImageAware中显示。
![2]
[2]:https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tool-lib/image-cache/universal-image-loader/image/overall-design.png
