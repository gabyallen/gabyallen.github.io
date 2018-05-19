---
layout:     post                    # 使用的布局（不需要改）
title:      android快速切换到主线程     # 标题 
subtitle:   Hello World, Android #副标题
date:       2018-5-19              # 时间
author:     BY  gabyallen          # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - ANDROID
---

##创建android快速切换到主线程更新UI的几种方法
看过很多网上的信息，相信大家都对在子线程中切换到主线程中掌握了一定知识;我这边再来总结一下;以便可以帮助到大家
>方法一：view.post（Runnable action）;
view.post（new Runnable（）{
 @Override
 public void run（）{
 View.setText（“更新textView”）;
}
}）;
##同时查看还有一个方法view.postDelayed（Runnable action，long delayMillis）;用来延迟发送
>方法二：activity.runOnUiThread（Runnable action）;
假如该方法在子线程中
注意：context对象要是主线程中的MainActivity，这样强转才可以。
 public void UIupdate（final Context context）{
（（MainActivity）context）.runOnUiThread（new Runnable（）{
 @Override
 public void run（）{
//此时已在主线程中，可以更新UI了
}
}）;
}
    
##如果没有上下文（context），试试下面的方法： 
1.用view.getContext（）可以得到上下文。 
2.跳过上下文直接用new Activity（）。runOnUiThread（Runnable action）来切换到主线程。
>方法三：处理程序机制
首先在主线程中定义Handler，Handler mainHandler = new Handler（）;
（必须要在主线程中定义才能操作主线程，如果想在其他地方定义声明时要这样写Handler mainHandler = new Handler（Looper.getMainLooper（）），
来获取主线程的Looper和Queue）
获取到Handler后就很简单了，用handler.post（Runnable r）方法把消息处理放在该处理程序依附的消息队列中（也就是主线程消息队列）。

（1）：假如该方法是在子线程中
Handler mainHandler = new Handler(Looper.getMainLooper());
   mainHandler.post(new Runnable() {
    @Override
       public void run() {
           //已在主线程中，可以更新UI
       }
   });
   
（2） 假设在主线程中

   Handler myHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            switch(msg.what) {
                case 0:
                   //更新UI等
                   break;
               case 1:
                     //更新UI等
                    break;
               default:
                   break;
            }
        }
  }

之后可以把 mainHandler 当做参数传递在各个类之间，当需要更新UI时，
   可以调用sendMessage一系列方法来执行handleMessage里的操作。 
  
      /**
     *获取消息，尽量用obtainMessage()方法，查看源码发现，该方法节省内存。
     *不提倡用Messenger msg=new Messenger()这种方法，每次都去创建一个对象，肯定不节省内存啦！
     *至于为什么该方法还存在，估计还是有存在的必要吧。（留作以后深入研究）
     */
    Message msg = myHandler.obtainMessage();
    msg.what = 0; //消息标识
    myHandler.sendMessage(msg); //发送消息
  
  如上代码，只是发送了个消息标识，并没有传其他参数。 
如果想传递参数，可以这样：

     msg.what = 1;  //消息标识
     msg.arg1=2;   //存放整形数据，如果携带数据简单，优先使用arg1和arg2，比Bundle更节省内存。
      msg.arg2=3;   //存放整形数据
     Bundle bundle=new Bundle();
     bundle.putString("dd","adfasd");
     bundle.putInt("love",5);
     msg.setData(bundle);
     msg.obj=bundle;   //用来存放Object类型的任意对象
      myHandler.sendMessage(msg); //发送消息
     
 主线程中接收消息：
 public Handler myHandler = new Handler(Looper.getMainLooper()) {
       @Override
       public void handleMessage(Message msg) {
           super.handleMessage(msg);
           switch (msg.what) {
                case 1:
                   String dd=msg.getData().getString("dd");
                   int love =msg.getData().getInt("love");
                    break;
           }
        }
    };

总结： msg.obj它的功能比较强大一下，至于它和利用Bundle传递数据，那个会效率高一些，更节省内存一些。
个人认为：从传递数据的复杂程度看，由简单到复杂依次使用，arg1， setData(), obj。会比较好一些。

当然可以用简化方法sendEmptyMessage(int what)来减少不必要的代码，这样写：myHandler.sendEmptyMessage(0); //其实内部实现还是和上面一样

发送消息的其他方法有：
endEmptyMessageAtTime(int what, long uptimeMillis); //定时发送空消息
sendEmptyMessageDelayed(int what, long delayMillis); //延时发送空消息
sendMessageAtTime(Message msg, long uptimeMillis); //定时发送消息
sendMessageDelayed(Message msg, long delayMillis); //延时发送消息
sendMessageAtFrontOfQueue(Message msg); //最先处理消息（慎用）

>方法四： 使用AsyncTask
/**
  * 该类中方法的执行顺序依次为：onPreExecute, doInBackground, onPostExecute
  */
    private class MyAsyncTask extends AsyncTask<String, Integer, String> {
        /**
         * 主线程中执行
         * 在execute()被调用后首先执行
         * 一般用来在执行后台任务前对UI做一些标记
         */
        @Override
        protected void onPreExecute() {
            super.onPreExecute();
            System.out.println("MyAsyncTask.onPreExecute");
        }

        /**
         * 子线程中执行，执行一些耗时操作，关键方法
         * 在执行过程中可以调用publishProgress(Progress... values)来更新进度信息。
         */
        @Override
        protected String doInBackground(String... params) {
            System.out.println("MyAsyncTask.doInBackground");
            //只是模拟了耗时操作
            int count = 0;
            for (int i = 0; i < 10; i++) {
                try {
                    count++;
                    publishProgress((count % 100) * 10);
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // publishProgress((int) ((count / (float) total) * 100));
            return "耗时操作执行完毕";
        }

        /**
         * 主线程中执行
         * 在调用publishProgress(Progress... values)时，此方法被执行，直接将进度信息更新到UI组件中
         */
        @Override
        protected void onProgressUpdate(Integer... values) {
            super.onProgressUpdate(values);
            progressBar.setProgress(values[0]);
            textView.setText("loading..." + values[0] + "%");
            System.out.println("MyAsyncTask.onProgressUpdate");
        }

        /**
         * 在主线程中，当后台操作结束时，此方法将会被调用
         * 计算结果将做为参数传递到此方法中，直接将结果显示到UI组件上。
         */
        @Override
        protected void onPostExecute(String aVoid) {
            super.onPostExecute(aVoid);
            System.out.println("MyAsyncTask.onPostExecute aVoid=" + aVoid);
            textView.setText(aVoid);
        }


        /**
         * 主线程中执行
         * 当异步任务取消后的，会回调该函数。在该方法内可以更新UI
         */
        @Override
        protected void onCancelled() {
            super.onCancelled();
            System.out.println("MyAsyncTask.onCancelled");
            progressBar.setProgress(0);
            textView.setText("0");
        }

        @Override
        protected void onCancelled(String s) {
            super.onCancelled(s);
        }
    }
    
    注意：doInBackground方法是在子线程中，所以，我们在这个方法里面执行耗时操作。
    同时，由于其返回结果会传递到onPostExecute方法中，
    而onPostExecute方法工作在UI线程，这样我们就在这个方法里面更新ui，达到了异步更新ui的目的。

对于android的异步加载数据及更新ui，我们不仅可以选择AsyncTask异步任务，还可以选择许多开源的网络框架，如： 点击进入了解更多 xUtils3，AsyncHttpClient，Okhttp，Volley，…， 
这些优秀的网络框架让我们异步更新ui变得非常简单，而且，效率和性能也非常高。当然这些网络框架绝不仅仅就这么个功能，异步更新UI这只是他们的冰山一角！

参考资料：<https://blog.csdn.net/da_caoyuan/article/details/52931007/>
