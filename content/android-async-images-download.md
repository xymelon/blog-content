Title: Android异步批量下载图片并缓存
Date: 2014-07-29 19:55
Modified: 2014-07-29 19:55
Category: Technology
Tags: Android
Slug: android-async-images-download
Author: xymelon

### 前言 ###

接触android开发不久，近段时间需实现一个批量下载图片并显示的小功能。在网上搜索了一圈，发现国内外网上异步加载的例子太多太杂，要么是加载大图decode时报OOM异常，要么内存急剧上升不稳定。所以在前辈们的基础上，做了一些优化，特共享出来，欢迎大家指正。这里主要参见了以下两篇文章，非常感谢：

<a href="http://blog.csdn.net/guolin_blog/article/details/9526203" target="_blank">Android照片墙应用实现，再多的图片也不怕崩溃</a>

<a href="http://blog.csdn.net/xiaanming/article/details/9825113" target="_blank">Android 异步加载图片，使用LruCache和SD卡或手机缓存，效果非常的流畅</a>

在巨人的肩膀上，主要优化了以下几个地方：

1. 下载大图decode时，可根据View大小自动缩放图片，不在出现`OOM`和`SkImageDecoder::Factory returned null`错误
1. 图片下载失败时，可自定义失败重试次数
1. 记录正在下载的任务，防止屏幕滚动时多次下载
1. 缓存目录容量大于给定限制时，清空文件缓存
1. 其他一些小问题

<!-- PELICAN_END_SUMMARY -->

### 工具类Utils ###

Utils类主要封装了一些文件操作的基本方法，包括创建、删除、获取文件大小等。

	public class Utils {
		private static final String Util_LOG = makeLogTag(Utils.class);
	
		public static String makeLogTag(Class<?> cls) {
			return cls.getName();
		}
	
		public static void showToast(Context context, String str) {
			Toast.makeText(context, str, Toast.LENGTH_SHORT).show();
		}
	
		/**
		 * 检查是否存在SD卡
		 * 
		 * @return
		 */
		public static boolean hasSdcard() {
			String state = Environment.getExternalStorageState();
			if (state.equals(Environment.MEDIA_MOUNTED)) {
				return true;
			} else {
				return false;
			}
		}
	
		/**
		 * 创建目录
		 * 
		 * @param context
		 * @param dirName
		 *            文件夹名称
		 * @return
		 */
		public static File createFileDir(Context context, String dirName) {
			String filePath;
			// 如SD卡已存在，则存储；反之存在data目录下
			if (hasSdcard()) {
				// SD卡路径
				filePath = Environment.getExternalStorageDirectory()
						+ File.separator + dirName;
			} else {
				filePath = context.getCacheDir().getPath() + File.separator
						+ dirName;
			}
			File destDir = new File(filePath);
			if (!destDir.exists()) {
				boolean isCreate = destDir.mkdirs();
				Log.i(Util_LOG, filePath + " has created. " + isCreate);
			}
			return destDir;
		}
	
		/**
		 * 删除文件（若为目录，则递归删除子目录和文件）
		 * 
		 * @param file
		 * @param delThisPath
		 *            true代表删除参数指定file，false代表保留参数指定file
		 */
		public static void delFile(File file, boolean delThisPath) {
			if (!file.exists()) {
				return;
			}
			if (file.isDirectory()) {
				File[] subFiles = file.listFiles();
				if (subFiles != null) {
					int num = subFiles.length;
					// 删除子目录和文件
					for (int i = 0; i < num; i++) {
						delFile(subFiles[i], true);
					}
				}
			}
			if (delThisPath) {
				file.delete();
			}
		}
	
		/**
		 * 获取文件大小，单位为byte（若为目录，则包括所有子目录和文件）
		 * 
		 * @param file
		 * @return
		 */
		public static long getFileSize(File file) {
			long size = 0;
			if (file.exists()) {
				if (file.isDirectory()) {
					File[] subFiles = file.listFiles();
					if (subFiles != null) {
						int num = subFiles.length;
						for (int i = 0; i < num; i++) {
							size += getFileSize(subFiles[i]);
						}
					}
				} else {
					size += file.length();
				}
			}
			return size;
		}
	
		/**
		 * 保存Bitmap到指定目录
		 * 
		 * @param dir
		 *            目录
		 * @param fileName
		 *            文件名
		 * @param bitmap
		 * @throws IOException
		 */
		public static void savaBitmap(File dir, String fileName, Bitmap bitmap) {
			if (bitmap == null) {
				return;
			}
			File file = new File(dir, fileName);
			try {
				file.createNewFile();
				FileOutputStream fos = new FileOutputStream(file);
				bitmap.compress(CompressFormat.JPEG, 100, fos);
				fos.flush();
				fos.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	
		/**
		 * 判断某目录下文件是否存在
		 * 
		 * @param dir
		 *            目录
		 * @param fileName
		 *            文件名
		 * @return
		 */
		public static boolean isFileExists(File dir, String fileName) {
			return new File(dir, fileName).exists();
		}
	}

### 图片下载管理类ImageDownLoader ###

ImageDownLoader类主要是图片下载管理，包括缓存管理、异步下载管理等。

	public class ImageDownLoader {
		private static final String ImageDownLoader_Log = Utils
				.makeLogTag(ImageDownLoader.class);
	
		/** 保存正在下载或等待下载的URL和相应失败下载次数（初始为0），防止滚动时多次下载 */
		private Hashtable<String, Integer> taskCollection;
		/** 缓存类 */
		private LruCache<String, Bitmap> lruCache;
		/** 线程池 */
		private ExecutorService threadPool;
		/** 缓存文件目录 （如无SD卡，则data目录下） */
		private File cacheFileDir;
		/** 缓存文件夹 */
		private static final String DIR_CACHE = "cache";
		/** 缓存文件夹最大容量限制（10M） */
		private static final long DIR_CACHE_LIMIT = 10 * 1024 * 1024;
		/** 图片下载失败重试次数 */
		private static final int IMAGE_DOWNLOAD_FAIL_TIMES = 2;
	
		public ImageDownLoader(Context context) {
			// 获取系统分配给每个应用程序的最大内存
			int maxMemory = (int) Runtime.getRuntime().maxMemory();
			// 给LruCache分配最大内存的1/8
			lruCache = new LruCache<String, Bitmap>(maxMemory / 8) {
				// 必须重写此方法，来测量Bitmap的大小
				@Override
				protected int sizeOf(String key, Bitmap value) {
					return value.getRowBytes() * value.getHeight() / 1024;
				}
			};
			taskCollection = new Hashtable<String, Integer>();
			// 创建线程数
			threadPool = Executors.newFixedThreadPool(10);
			cacheFileDir = Utils.createFileDir(context, DIR_CACHE);
		}
	
		/**
		 * 添加Bitmap到内存缓存
		 * 
		 * @param key
		 * @param bitmap
		 */
		private void addLruCache(String key, Bitmap bitmap) {
			if (getBitmapFromMemCache(key) == null && bitmap != null) {
				lruCache.put(key, bitmap);
			}
		}
	
		/**
		 * 从内存缓存中获取Bitmap
		 * 
		 * @param key
		 * @return
		 */
		private Bitmap getBitmapFromMemCache(String key) {
			return lruCache.get(key);
		}
	
		/**
		 * 异步下载图片，并按指定宽度和高度压缩图片
		 * 
		 * @param url
		 * @param width
		 * @param height
		 * @param listener
		 *            图片下载完成后调用接口
		 */
		public void loadImage(final String url, final int width, final int height,
				AsyncImageLoaderListener listener) {
			Log.i(ImageDownLoader_Log, "download:" + url);
			final ImageHandler handler = new ImageHandler(listener);
			Runnable runnable = new Runnable() {
				@Override
				public void run() {
					Bitmap bitmap = downloadImage(url, width, height);
					Message msg = handler.obtainMessage();
					msg.obj = bitmap;
					handler.sendMessage(msg);
					// 将Bitmap 加入内存缓存
					addLruCache(url, bitmap);
					// 加入文件缓存前，需判断缓存目录大小是否超过限制，超过则清空缓存再加入
					long cacheFileSize = Utils.getFileSize(cacheFileDir);
					if (cacheFileSize > DIR_CACHE_LIMIT) {
						Log.i(ImageDownLoader_Log, cacheFileDir
								+ " size has exceed limit." + cacheFileSize);
						Utils.delFile(cacheFileDir, false);
						taskCollection.clear();
					}
					// 缓存文件名称（ 替换url中非字母和非数字的字符，防止系统误认为文件路径）
					String urlKey = url.replaceAll("[^\\w]", "");
					// 将Bitmap加入文件缓存
					Utils.savaBitmap(cacheFileDir, urlKey, bitmap);
				}
			};
			// 记录该url，防止滚动时多次下载，0代表该url下载失败次数
			taskCollection.put(url, 0);
			threadPool.execute(runnable);
		}
	
		/**
		 * 获取Bitmap, 若内存缓存为空，则去文件缓存中获取
		 * 
		 * @param url
		 * @return 若缓存中没找到，则返回null
		 */
		public Bitmap getBitmapCache(String url) {
			// 去处url中特殊字符作为文件缓存的名称
			String urlKey = url.replaceAll("[^\\w]", "");
			if (getBitmapFromMemCache(url) != null) {
				return getBitmapFromMemCache(url);
			} else if (Utils.isFileExists(cacheFileDir, urlKey)
					&& Utils.getFileSize(new File(cacheFileDir, urlKey)) > 0) {
				// 从文件缓存中获取Bitmap
				Bitmap bitmap = BitmapFactory.decodeFile(cacheFileDir.getPath()
						+ File.separator + urlKey);
				// 将Bitmap 加入内存缓存
				addLruCache(url, bitmap);
				return bitmap;
			}
			return null;
		}
	
		/**
		 * 下载图片，并按指定高度和宽度压缩
		 * 
		 * @param url
		 * @param width
		 * @param height
		 * @return
		 */
		private Bitmap downloadImage(String url, int width, int height) {
			Bitmap bitmap = null;
			HttpClient httpClient = new DefaultHttpClient();
			try {
				httpClient.getParams().setParameter(
						CoreProtocolPNames.PROTOCOL_VERSION, HttpVersion.HTTP_1_1);
				HttpPost httpPost = new HttpPost(url);
				HttpResponse httpResponse = httpClient.execute(httpPost);
				if (httpResponse.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
					HttpEntity entity = httpResponse.getEntity();
					//解决缩放大图时出现SkImageDecoder::Factory returned null错误
					byte[] byteIn = EntityUtils.toByteArray(entity);
					BitmapFactory.Options bmpFactoryOptions = new BitmapFactory.Options();
					bmpFactoryOptions.inJustDecodeBounds = true;
					BitmapFactory.decodeByteArray(byteIn, 0, byteIn.length,
							bmpFactoryOptions);
					int heightRatio = (int) Math.ceil(bmpFactoryOptions.outHeight
							/ height);
					int widthRatio = (int) Math.ceil(bmpFactoryOptions.outWidth
							/ width);
					if (heightRatio > 1 && widthRatio > 1) {
						bmpFactoryOptions.inSampleSize = heightRatio > widthRatio ? heightRatio
								: widthRatio;
					}
					bmpFactoryOptions.inJustDecodeBounds = false;
					bitmap = BitmapFactory.decodeByteArray(byteIn, 0,
							byteIn.length, bmpFactoryOptions);
				}
			} catch (ClientProtocolException e) {
				e.printStackTrace();
			} catch (ConnectTimeoutException e) {
				e.printStackTrace();
			} catch (Exception e) {
				e.printStackTrace();
			} finally {
				if (httpClient != null && httpClient.getConnectionManager() != null) {
					httpClient.getConnectionManager().shutdown();
				}
			}
			// 下载失败，再重新下载
			// 本例是图片下载失败则再次下载，可根据需要改变，比如记录下载失败的图片URL，在某个时刻再次下载
			if (taskCollection.get(url) != null) {
				int times = taskCollection.get(url);
				if (bitmap == null
						&& times < IMAGE_DOWNLOAD_FAIL_TIMES) {
					times++;
					taskCollection.put(url, times);
					bitmap = downloadImage(url, width, height);
					Log.i(ImageDownLoader_Log, "Re-download " + url + ":" + times);
				}
			}
			return bitmap;
		}
	
		/**
		 * 取消正在下载的任务
		 */
		public synchronized void cancelTasks() {
			if (threadPool != null) {
				threadPool.shutdownNow();
				threadPool = null;
			}
		}
	
		/**
		 * 获取任务列表
		 * 
		 * @return
		 */
		public Hashtable<String, Integer> getTaskCollection() {
			return taskCollection;
		}
	
		/** 异步加载图片接口 */
		public interface AsyncImageLoaderListener {
			void onImageLoader(Bitmap bitmap);
		}
	
		/** 异步加载完成后，图片处理 */
		static class ImageHandler extends Handler {
	
			private AsyncImageLoaderListener listener;
	
			public ImageHandler(AsyncImageLoaderListener listener) {
				this.listener = listener;
			}
	
			@Override
			public void handleMessage(Message msg) {
				super.handleMessage(msg);
				listener.onImageLoader((Bitmap) msg.obj);
			}
		}
	}

### GridView适配器类ImageGridViewAdapter ###

该类为GridView适配器，并实现滚动监听等的功能。

	public class ImageGridViewAdapter extends BaseAdapter implements OnScrollListener {
	
		/** 数据源 */
		private List<String> data;
		/** 图片下载类 */
		private ImageDownLoader loader;
		/** 判定是否第一次加载 */
		private boolean isFirstEnter = true;
		/** 第一张可见Item下标 */
		private int firstVisibleItem;
		/** 每屏Item可见数 */
		private int visibleItemCount;
		/** GridView实例 */
		private GridView gridView;
		private Context context;
	
		public ImageGridViewAdapter(Context context, GridView gridView, List<String> data) {
			this.context = context;
			this.gridView = gridView;
			this.data = data;
			loader = new ImageDownLoader(context);
			this.gridView.setOnScrollListener(this);
		}
	
		@Override
		public int getCount() {
			return data.size();
		}
	
		@Override
		public Object getItem(int position) {
			return data.get(position);
		}
	
		@Override
		public long getItemId(int position) {
			return position;
		}
	
		@Override
		public View getView(int position, View convertView, ViewGroup parent) {
			String url = data.get(position);
			if (convertView == null) {
				convertView = LayoutInflater.from(context).inflate(
						R.layout.photo_item, null);
			}
			ImageView imageView = (ImageView) convertView.findViewById(R.id.photo);
			// 设置Tag，保证异步加载图片不乱序
			imageView.setTag(url);
			setImageView(imageView, url);
			return convertView;
		}
	
		private void setImageView(ImageView imageView, String url) {
			Bitmap bitmap = loader.getBitmapCache(url);
			if (bitmap != null) {
				imageView.setImageBitmap(bitmap);
			} else {
				imageView.setImageResource(R.drawable.empty_photo);
			}
		}
	
		@Override
		public void onScrollStateChanged(AbsListView view, int scrollState) {
			// 当停止滚动时，加载图片
			if (scrollState == SCROLL_STATE_IDLE) {
				loadImage(firstVisibleItem, visibleItemCount);
			}
		}
	
		@Override
		public void onScroll(AbsListView view, int firstVisibleItem,
				int visibleItemCount, int totalItemCount) {
			this.firstVisibleItem = firstVisibleItem;
			this.visibleItemCount = visibleItemCount;
			if (isFirstEnter && visibleItemCount > 0) {
				loadImage(firstVisibleItem, visibleItemCount);
				isFirstEnter = false;
			}
		}
	
		/**
		 * 加载图片，若缓存中没有，则根据url下载
		 * 
		 * @param firstVisibleItem
		 * @param visibleItemCount
		 */
		private void loadImage(int firstVisibleItem, int visibleItemCount) {
			Bitmap bitmap = null;
			for (int i = firstVisibleItem; i < firstVisibleItem + visibleItemCount; i++) {
				String url = data.get(i);
				final ImageView imageView = (ImageView) gridView
						.findViewWithTag(url);
				bitmap = loader.getBitmapCache(url);
				if (bitmap != null) {
					imageView.setImageBitmap(bitmap);
				} else {
					// 防止滚动时多次下载
					if (loader.getTaskCollection().containsKey(url)) {
						continue;
					}
					imageView.setImageDrawable(context.getResources().getDrawable(
							R.drawable.empty_photo));
					loader.loadImage(url, imageView.getWidth(),
							imageView.getHeight(),
							new ImageDownLoader.AsyncImageLoaderListener() {
	
								@Override
								public void onImageLoader(Bitmap bitmap) {
									if (imageView != null && bitmap != null) {
										imageView.setImageBitmap(bitmap);
									}
								}
	
							});
				}
			}
		}
	
		/**
		 * 取消下载任务
		 */
		public void cancelTasks() {
			loader.cancelTasks();
		}
	}


最后的最后，别忘加上权限，并附效果图如下：

	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" />
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

![upload](/images/download-images.png)

有疑问或者觉得不对的地方还请指正，谢谢。

<a href="http://download.csdn.net/detail/q541959443/7692557" target="_blank">ImagesDownLoad源码下载：DEMO</a>
