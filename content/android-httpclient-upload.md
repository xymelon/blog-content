Title: Android使用HttpClient实现文件上传到PHP服务器，并监控进度
Date: 2014-07-29 10:35
Modified: 2014-07-29 10:35
Category: Technology
Tags: Android
Slug: android-httpclient-upload
Author: xymelon

上一篇文章[Android使用HttpClient实现下载，并监控进度](http://www.xycoding.com/articles/2014/07/28/android-httpclient-download/)，本文继续讲解实现上传进度监控，原理其实一样，只不过这次重载的是`FilterOutputStream`，服务器端使用PHP进行文件的接收。好记性不如烂笔头，方便以后查阅，不当之处，还请指正。

<!-- PELICAN_END_SUMMARY -->

### 服务器端PHP ###

废话不多说，直接上代码：

	<?php
		$target_path  = "./tmp/";//接收文件目录  
		$target_path = $target_path.($_FILES['file']['name']);
		$target_path = iconv("UTF-8","gb2312", $target_path);
		if(move_uploaded_file($_FILES['file']['tmp_name'], $target_path)) {  
		   echo "The file ".( $_FILES['file']['name'])." has been uploaded.";
		}else{  
		   echo "There was an error uploading the file, please try again! Error Code: ".$_FILES['file']['error'];
		}
	?>

### 监控进度实现 ###

首先定义监听器接口，这和上文中是一样的，如下所示：

	/**
	 * 进度监听器接口
	 */
	public interface ProgressListener {
		public void transferred(long transferedBytes);
	}

实现监控进度的关键部分就在于记录已传输字节数，所以我们需重载`FilterOutputStream`，重写其中的关键方法，实现进度监听的功能，如下所示，本例中首先重载的是`HttpEntityWrapper`，顾名思义，就是将需发送的`HttpEntity`打包，以便计算总字节数，代码如下：

	/**
	 * ProgressOutHttpEntity：输出流(OutputStream)时记录已发送字节数
	 * 
	 * @author Cow
	 * 
	 */
	public class ProgressOutHttpEntity extends HttpEntityWrapper {
	
		private final ProgressListener listener;
	
		public ProgressOutHttpEntity(final HttpEntity entity,
				final ProgressListener listener) {
			super(entity);
			this.listener = listener;
		}
	
		public static class CountingOutputStream extends FilterOutputStream {
	
			private final ProgressListener listener;
			private long transferred;
	
			CountingOutputStream(final OutputStream out,
					final ProgressListener listener) {
				super(out);
				this.listener = listener;
				this.transferred = 0;
			}
	
			@Override
			public void write(final byte[] b, final int off, final int len)
					throws IOException {
				// NO, double-counting, as super.write(byte[], int, int)
				// delegates to write(int).
				// super.write(b, off, len);
				out.write(b, off, len);
				this.transferred += len;
				this.listener.transferred(this.transferred);
			}
	
			@Override
			public void write(final int b) throws IOException {
				out.write(b);
				this.transferred++;
				this.listener.transferred(this.transferred);
			}
	
		}
	
		@Override
		public void writeTo(final OutputStream out) throws IOException {
			this.wrappedEntity.writeTo(out instanceof CountingOutputStream ? out
					: new CountingOutputStream(out, this.listener));
		}
	}

最后就是使用上述实现的类和Httpclient进行上传并显示进度的功能，非常简单，代码如下，使用AsyncTask异步上传。

	public class FileUploadAsyncTask extends AsyncTask<File, Integer, String> {
	
		private String url = "http://192.168.83.213/receive_file.php";
		private Context context;
		private ProgressDialog pd;
		private long totalSize;
	
		public FileUploadAsyncTask(Context context) {
			this.context = context;
		}
	
		@Override
		protected void onPreExecute() {
			pd = new ProgressDialog(context);
			pd.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
			pd.setMessage("上传中....");
			pd.setCancelable(false);
			pd.show();
		}
	
		@Override
		protected String doInBackground(File... params) {
			// 保存需上传文件信息
			MultipartEntityBuilder entitys = MultipartEntityBuilder.create();
			entitys.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
			entitys.setCharset(Charset.forName(HTTP.UTF_8));
	
			File file = params[0];
			entitys.addPart("file", new FileBody(file));
			HttpEntity httpEntity = entitys.build();
			totalSize = httpEntity.getContentLength();
			ProgressOutHttpEntity progressHttpEntity = new ProgressOutHttpEntity(
					httpEntity, new ProgressListener() {
						@Override
						public void transferred(long transferedBytes) {
							publishProgress((int) (100 * transferedBytes / totalSize));
						}
					});
			return uploadFile(url, progressHttpEntity);
		}
	
		@Override
		protected void onProgressUpdate(Integer... progress) {
			pd.setProgress((int) (progress[0]));
		}
	
		@Override
		protected void onPostExecute(String result) {
			pd.dismiss();
			Toast.makeText(context, result, Toast.LENGTH_SHORT).show();
		}
	
		/**
		 * 上传文件到服务器
		 * 
		 * @param url
		 *            服务器地址
		 * @param entity
		 *            文件
		 * @return
		 */
		public static String uploadFile(String url, ProgressOutHttpEntity entity) {
			HttpClient httpClient = new DefaultHttpClient();
			httpClient.getParams().setParameter(
					CoreProtocolPNames.PROTOCOL_VERSION, HttpVersion.HTTP_1_1);
			// 设置连接超时时间
			httpClient.getParams().setParameter(
					CoreConnectionPNames.CONNECTION_TIMEOUT, 5000);
			HttpPost httpPost = new HttpPost(url);
			httpPost.setEntity(entity);
			try {
				HttpResponse httpResponse = httpClient.execute(httpPost);
				if (httpResponse.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
					return "文件上传成功";
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
			return "文件上传失败";
		}
	
	}

最后的最后，上传效果图如下：

![upload](/images/image-upload.png)

有疑问或者觉得不对的地方还请指正，谢谢。

<a href="https://github.com/cowfighting/ImageWithProgress" target="_blank">GitHub源码下载：DEMO</a>
