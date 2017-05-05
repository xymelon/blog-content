Title: Android使用HttpClient实现下载，并监控进度
Date: 2014-07-28 19:50
Modified: 2014-07-28 19:50
Category: Technology
Tags: Android
Slug: android-httpclient-download
Author: xymelon

Android程序开发中使用网络上传下载是必不可少的，前不久使用[Apache Httpclient](http://hc.apache.org/)组件进行图片上传下载，并监控进度等功能的了解与实现，并解决了缩放图片时出现`SkImageDecoder::Factory returned null`错误。本文是在httpcore-4.3.jar,httpmime-4.3.4.jar基础上实现，文章末尾提供的Demo下载中包含了它们。关于上传，请移步[Android使用HttpClient实现文件上传到PHP服务器，并监控进度](http://www.xycoding.com/articles/2014/07/29/android-httpclient-upload/)。好记性不如烂笔头，方便以后查阅，不当之处，还请指正。

<!-- PELICAN_END_SUMMARY -->

### 监控进度实现 ###

关于HttpClient的资料，可去其官方查阅。本例简单的使用它进行图片下载，如想实现批量下载图片，请参见[Android异步批量下载图片并缓存](http://www.xycoding.com/articles/2014/07/29/android-async-images-download/)。

首先定义监听器接口，如下所示：

	/**
	 * 进度监听器接口
	 */
	public interface ProgressListener {
		public void transferred(long transferedBytes);
	}

然后自定义一个`CountingInputStream`类继承于`FilterInputStream`，重写其中的关键方法，实现进度监听的功能，如下所示：

	/**
	 * 重写FilterInputStream，实现进度监听的功能
	 * 
	 * @author Cow
	 * 
	 */
	public class CountingInputStream extends FilterInputStream {
	
		private final ProgressListener listener;
		private long transferred;
	
		protected CountingInputStream(final InputStream in,
				final ProgressListener listener) {
			super(in);
			this.listener = listener;
			this.transferred = 0;
		}
	
		@Override
		public int read() throws IOException {
			int read = in.read();
			readCount(read);
			return read;
		}
	
		@Override
		public int read(byte[] buffer) throws IOException {
			int read = in.read(buffer);
			readCount(read);
			return read;
		}
	
		@Override
		public int read(byte[] buffer, int byteOffset, int byteCount)
				throws IOException {
			int read = in.read(buffer, byteOffset, byteCount);
			readCount(read);
			return read;
		}
	
		@Override
		public long skip(long byteCount) throws IOException {
			long skip = in.skip(byteCount);
			readCount(skip);
			return skip;
		}
	
		private void readCount(long read) {
			if (read > 0) {
				this.transferred += read;
				this.listener.transferred(this.transferred);
			}
		}
	}

最后就是使用上述实现的类和Httpclient进行下载并显示进度的功能，非常简单，代码如下，使用AsyncTask异步下载。

	public class FileDownLoadAsyncTask extends
			AsyncTask<ImageView, Integer, Bitmap> {
	
		// 图片下载地址
		private String url = "http://img0.bdstatic.com/img/image/shouye/leimu/mingxing.jpg";
		private Context context;
		private ProgressDialog pd;
		private ImageView image;
		private int width = 150;
		private int height = 150;
	
		public FileDownLoadAsyncTask(Context context) {
			this.context = context;
		}
	
		@Override
		protected void onPreExecute() {
			pd = new ProgressDialog(context);
			pd.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
			pd.setMessage("下载中....");
			pd.setCancelable(false);
			pd.show();
		}
	
		/**
		 * 下载图片，并按指定高度和宽度压缩
		 */
		@Override
		protected Bitmap doInBackground(ImageView... params) {
			this.image = params[0];
			Bitmap bitmap = null;
			HttpClient httpClient = new DefaultHttpClient();
			try {
				httpClient.getParams().setParameter(
						CoreProtocolPNames.PROTOCOL_VERSION, HttpVersion.HTTP_1_1);
				HttpGet httpGet = new HttpGet(url);
				HttpResponse httpResponse = httpClient.execute(httpGet);
				if (httpResponse.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
					HttpEntity entity = httpResponse.getEntity();
					final long size = entity.getContentLength();
					CountingInputStream cis = new CountingInputStream(
							entity.getContent(), new ProgressListener() {
	
								@Override
								public void transferred(long transferedBytes) {
									Log.i("FileDownLoadAsyncTask", "总字节数：" + size
											+ " 已下载字节数：" + transferedBytes);
									publishProgress((int) (100 * transferedBytes / size));
								}
							});
					// 需将Inputstream转化为byte数组，以备decodeByteArray用
					// 如直接使用decodeStream会将stream破坏，然后第二次decodeStream时，会出现SkImageDecoder::Factory returned null错误
					// 我试过将获得的Inputstream转化为BufferedInputStream，然后使用mark、reset方法，但是我试了试没成功，不知道为啥，还请成功的各位告知
					byte[] byteIn = toByteArray(cis, (int) size);
					BitmapFactory.Options bmpFactoryOptions = new BitmapFactory.Options();
					// 第一次decode时，需设置inJustDecodeBounds属性为true,这样系统就会只读取下载图片的属性而不分配空间，并将属性存储在Options中
					bmpFactoryOptions.inJustDecodeBounds = true;
					// 第一次decode，获取图片高宽度等属性
					BitmapFactory.decodeByteArray(byteIn, 0, byteIn.length,
							bmpFactoryOptions);
					// 根据显示控件大小获取压缩比率，有效避免OOM
					int heightRatio = (int) Math.ceil(bmpFactoryOptions.outHeight
							/ height);
					int widthRatio = (int) Math.ceil(bmpFactoryOptions.outWidth
							/ width);
					if (heightRatio > 1 && widthRatio > 1) {
						bmpFactoryOptions.inSampleSize = heightRatio > widthRatio ? heightRatio
								: widthRatio;
					}
					// 第二次decode时，需设置inJustDecodeBounds属性为fasle,系统才会根据传入的BitmapFactory.Options真正的压缩图片并返回
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
			return bitmap;
		}
	
		@Override
		protected void onProgressUpdate(Integer... progress) {
			pd.setProgress((int) (progress[0]));
		}
	
		@Override
		protected void onPostExecute(Bitmap bm) {
			pd.dismiss();
			if (bm != null) {
				image.setImageBitmap(bm);
			} else {
				Toast.makeText(context, "图片下载失败", Toast.LENGTH_SHORT).show();
			}
		}
	
		/**
		 * InputStream转化为Byte数组
		 * 
		 * @param instream
		 * @param contentLength
		 * @return
		 * @throws IOException
		 */
		public byte[] toByteArray(InputStream instream, int contentLength)
				throws IOException {
			if (instream == null) {
				return null;
			}
			try {
				if (contentLength < 0) {
					contentLength = 4096;
				}
				final ByteArrayBuffer buffer = new ByteArrayBuffer(contentLength);
				final byte[] tmp = new byte[4096];
				int l;
				while ((l = instream.read(tmp)) != -1) {
					buffer.append(tmp, 0, l);
				}
				return buffer.toByteArray();
			} finally {
				instream.close();
			}
		}
	
	}

最后的最后，下载效果图如下：

![dowmload](/images/image-download.png)

有疑问或者觉得不对的地方还请指正，谢谢。

<a href="https://github.com/cowfighting/ImageWithProgress" target="_blank">GitHub源码下载：DEMO</a>