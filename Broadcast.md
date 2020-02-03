# 广播

## 广播的种类

Android广播类型分为标准广播和有序广播两种。

* `标准广播`是一种完全异步执行的广播。广播接收器几乎同时收到广播，没有任何先后顺序可言。这种广播效率较高，但是无法被中途截断。
* `有序广播`则一种同步执行的广播。广播发出后，同一时刻只会有一个广播接收器能够收到这条广播消息，前面的广播接收器可以截断后面的广播接收器。

## 接收系统广播
### 动态注册
实现当网络状况可用和不可用切换时接收系统广播（需在`AndroidManifest.xml`文件中添加获取网络状态许可）。
> Class MainActivity

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        intentFilter = new IntentFilter();
        //当网络状态发生变化时，系统会发出一条android.net.conn.CONNECTIVITY_CHANGE广播
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        networkChangeReceiver = new NetWorkChangeReceiver();
        //通过动态注册，实现了网络状态监听功能
        registerReceiver(networkChangeReceiver, intentFilter);

        Button broadcastButton = findViewById(R.id.broadcast_button);
        broadcastButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent();
                //Android 8后发送广播需使用setComponen来构建intent，其中第一个是包名，第二个是接收器类名
                intent.setComponent(new ComponentName("com.mycompany.broadcasttest", "com.mycompany.broadcasttest.MyBroadcastReceiver"));
                sendBroadcast(intent);
                Log.d(TAG, "send");
            }
        });
    }

    @Override
    protected void onDestroy(){
        super.onDestroy();
        //
        unregisterReceiver(networkChangeReceiver);
    }

    class NetWorkChangeReceiver extends BroadcastReceiver{
	   //重写onReceive函数实现接收网络状态变化广播
        @Override
        public void onReceive(Context context, Intent intent){
            ConnectivityManager connectivityManager = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
            NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();
            if(networkInfo != null && networkInfo.isConnected()) {
                Toast.makeText(context, "network is available", Toast.LENGTH_SHORT).show();
            }
            else {
                Toast.makeText(context, "network is unavailable", Toast.LENGTH_SHORT).show();
            }
        }
    }
## 自定义广播
### 标准广播

自定义广播需要编写继承`BroadcastReceiver`的子类，并且在`AndroidManifest.xml`文件中添加`receiver`标签。

> Class MyBroadcastReceiver

    @Override
    public void onReceive(Context context, Intent intent){
        Log.d(TAG, "received");
        Toast.makeText(context, "Received in MyBroadacastReceiver", Toast.LENGTH_SHORT).show();
    }

> receiver标签

	<receiver android:name=".MyBroadcastReceiver"
                  	android:enabled="true"
                  	android:exported="true"/>
在`MainActivity`中添加按钮及点击事件。

### 有序广播
如需发送有序广播，使用`sendOrderedBroadcast(Intent intent, String receiverPermission)`方法。

还可以在`receiver`标签中添加`intent-filter`标签，并在该标签中添加android:priority属性，取值在(-1000, 1000)范围内，值越大优先收到广播。

在`onReceive()`方法中可以使用`abortBroadcast()`将该条广播截断。

## 本地广播
为了解决广播安全性问题，Android引入了一套本地广播机制，使用这个机制发出的广播。

> MainActivity

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        localBroadcastManager = LocalBroadcastManager.getInstance(this);


        Button broadcastButton = findViewById(R.id.broadcast_button);
        broadcastButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent("com.mycompany.broadcasttest.LOCAL_BROADCAST");
                localBroadcastManager.sendBroadcast(intent);
                Log.d(TAG, "send");
            }
        });

        intentFilter = new IntentFilter();
        intentFilter.addAction("com.mycompany.broadcasttest.LOCAL_BROADCAST");
        localReceiver = new LocalReceiver();
        //注册本地接收器
        localBroadcastManager.registerReceiver(localReceiver, intentFilter);
    }

    @Override
    protected void onDestroy(){
        super.onDestroy();
        localBroadcastManager.unregisterReceiver(localReceiver);
    }

    class LocalReceiver extends BroadcastReceiver{

        @Override
        public void onReceive(Context context, Intent intent){
            Toast.makeText(context, "received local broadcast", Toast.LENGTH_SHORT).show();
        }
    }

