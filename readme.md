[![](https://jitpack.io/v/majunm/http.svg)](https://jitpack.io/#majunm/http)

rxjava2+okhttp3+retrofit2 组合的一种方式

缓存|断点续传暂未实现 ..
https://jitpack.io/#majunm/http  使用方式

	allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}

	dependencies {
    	        compile 'com.github.majunm.http:http:v1.0.2'
    }


http://www.baidu.com?data=json 可以调用该形式的接口

UpdateRequest 必须继承 CommonRequest
        |@ObtainPath("version/version") 请求后缀填入注解


1.返回String,自己手动解析

        Porgress mPorgress = new Porgress(this);
        mPorgress.setProgress(100);
        mPorgress.setMessage("加载呢");
        HttpRequestFactory.doPost(new UpdateRequest(), new ResultCallbackAdapterIs<String>(this) {
            @Override
            public void doOnError(ApiException ex) {
                super.doOnError(ex);

            }

            @Override
            public void doOnResponse(String response) {
                super.doOnResponse(response);
                // 自己手动解析
            }
        }, mPorgress);

2.根据接口返回实体对象

            ILoading ll = new ILoading(this);
            ll.setMessage("网络请求");
            // data 是 对象 , 泛型是 HttpCommonObjResp<UpdateResp>
            // data 是 数组 , 泛型是 HttpCommonResp<UpdateResp>
            // 要求返回格式一致,否则请在CommonRespWrapI->onNext(String resp) 处理 参考代码 faultTolerance(String resp);
            HttpRequestFactory.doPost(new UpdateRequest(), new ResultCallbackAdapterIs<HttpCommonObjResp<UpdateResp>>(this) {
                @Override
                public void doOnError(ApiException ex) {
                    super.doOnError(ex);
                    //mTst.setText(ex + "#error#");
                }

                @Override
                public void doOnResponse(HttpCommonObjResp<UpdateResp> response) {
                    super.doOnResponse(response);
                    if (response.isSuccess()) {
                        UpdateResp resp = response.getData();
                        //mTst.setText(resp + "");
                    } else {
                        //mTst.setText(response + "#结果#");
                    }
                }
            }, ll);

=======

~~~
public interface Api {
    String KEY = "data";
    String PATH = "path";

    @FormUrlEncoded
    @POST("{path}")
    Observable<String> runPost(@Path(value = PATH, encoded = true) String path,
                               @Field(KEY) String json);

    @GET("{path}")
    Observable<String> runGet(@Path(value = PATH, encoded = true) String path,
                              @Query(KEY) String json);

    @Streaming
    @GET
    Observable<ResponseBody> downFile(@Url() String url, @QueryMap Map<String, String> maps);

    @Multipart
    //@POST("/")
    @POST
    Observable<ResponseBody> uploadFiles(@Url() String url, @Part() List<MultipartBody.Part> parts);
}
~~~
=======

上传参考:

~~~
final ProgressDialog dialog = new ProgressDialog(MainActivity.this);
        dialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);// 设置水平进度条
        dialog.setCancelable(true);// 设置是否可以通过点击Back键取消
        dialog.setCanceledOnTouchOutside(false);// 设置在点击Dialog外是否取消Dialog进度条
        // dialog.setIcon(R.drawable.ic_launcher);// 设置提示的title的图标，默认是没有的
        dialog.setTitle("上传");
        dialog.setMax(100);
        dialog.show();
        // 内置SD卡路径：/storage/emulated/0
        // 外置SD卡路径：/storage/extSdCard
        if (TextUtils.isEmpty(path)) {
            path = "/sdcard/Downloads/102_c5ce43c969a9684369f673838d84a447.apk";
        }
        // Environment.getExternalStorageDirectory().getPath()
        File file = new File(path);
        mTst4.setText(file.length() + "");
        HttpRequestFactory.uploadFiles("http://server.jeasonlzy.com/OkHttpUtils/upload", null, new UCallback() {
            @Override
            public void onProgress(final long currentLength, final long totalLength, final float percent) {
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        mTst4.setText(currentLength + "|" + totalLength + "|" + percent);
                    }
                });
                dialog.show();
                dialog.setProgress((int) percent);
                dialog.setMessage("上传中....");
            }

            @Override
            public void onFail(final int errCode, final String errMsg) {
                dialog.dismiss();
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        mTst4.setText(errMsg + "" + errCode);
                    }
                });
            }
        }, new ResultCallbackAdapterIs<ResponseBody>() {
            @Override
            public void doOnResponse(ResponseBody response) {
                super.doOnResponse(response);
                try {
                    dialog.dismiss();
                    mTst4.setText(response + "|" + response.contentLength());
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

            @Override
            public void doOnError(ApiException ex) {
                super.doOnError(ex);
                dialog.dismiss();
                mTst4.setText(ex + "#error#");
            }
        }, new File(path));
    }
~~~

下载参考:

~~~
final ProgressDialog dialog = new ProgressDialog(MainActivity.this);
        dialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);// 设置水平进度条
        dialog.setCancelable(true);// 设置是否可以通过点击Back键取消
        dialog.setCanceledOnTouchOutside(false);// 设置在点击Dialog外是否取消Dialog进度条
        // dialog.setIcon(R.drawable.ic_launcher);// 设置提示的title的图标，默认是没有的
        dialog.setTitle("提示");
        dialog.setMax(100);
        dialog.show();
        HttpRequestFactory.downFile("http://download.fir.im/v2/app/install/595c5959959d6901ca0004ac?download_token=1a9dfa8f248b6e45ea46bc5ed96a0a9e&source=update", new HashMap<String, String>(), new UCallback() {
            @Override
            public void onProgress(final long currentLength, final long totalLength, final float percent) {
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        dialog.setProgress((int) percent);
                        if (percent == 100) {
                            dialog.dismiss();
                        }
                        mTst3.setText(currentLength + "|" + totalLength + "|" + percent);
                    }
                });

            }

            @Override
            public void onFail(final int errCode, final String errMsg) {
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        dialog.dismiss();
                        mTst3.setText(errMsg + "" + errCode);
                    }
                });
            }
        }, new ResultCallbackAdapterIs<ResponseBody>() {
            @Override
            public void doOnResponse(final ResponseBody response) {
                super.doOnResponse(response);
                System.out.println("成功了");
                try {
                    mTst3.setText("|" + response.contentLength());
                } catch (Exception e) {
                    e.printStackTrace();
                    System.out.println("成功了" + e);
                }
                new Thread() {
                    @Override
                    public void run() {
                        super.run();
                        try {
                            InputStream is = response.byteStream();
                            File file = new File(Environment.getExternalStorageDirectory(), "12345.apk");
                            FileOutputStream fos = new FileOutputStream(file);
                            BufferedInputStream bis = new BufferedInputStream(is);
                            byte[] buffer = new byte[1024];
                            int len;
                            while ((len = bis.read(buffer)) != -1) {
                                fos.write(buffer, 0, len);
                                fos.flush();
                            }
                            fos.close();
                            bis.close();
                            is.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                            System.out.println(e);
                        }
                    }
                }.start();

            }

            @Override
            public void doOnError(ApiException ex) {
                super.doOnError(ex);
                dialog.dismiss();
                mTst3.setText(ex + "");
            }
        });
~~~