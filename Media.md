# 运用Android多媒体
## 使用通知
当程序向用户发出一些提示信息，而程序在后台运行时，就可以借助通知来实现。
### 通知的基本用法
通知的用法比较灵活，既可以在活动中创建，也可以在广播接收器中创建。还可以在服务中创建。

首先需要建立一个NotificationManager来对通知进行管理，可以调用Context的getSystemService()方法获得。getSystemService()方法通过接收字符串参数来确定获取系统的哪个服务，这里我们传入Context.NOTIFICATION_SERVICE即可。

	NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);

为了使我们的通知在所有Android系统版本上都能正常运行，则需要使用support-v4中的NotificationCompat类，使用这个类的构造器来创建Notification对象。
	
	Notification notification = new NotificationCompat.Bulder(context)
								.setContentTitle("This is content title")
								.setContentText("This is context text")
								.setWhen(System.currentTimeMillis())
								.setSmallIcon(R.drawable.small_icon)
								.setLargeIcon(BitmapFactory.decodeResource(getResources(),
									R.drawable.large_icon))
								.build();

以上工作完成后，只需调用NotificationManager的notify()方法就可以让通知显示出来了。notify()方法接收两个参数，第一个参数是id，要保证每个i通知所指定的d是唯一的。第二个参数是Notification对象。
	
	manager.notify(1, notification);

### 补充
Android在API26后引入了NotificationChannel，消息需要设置NotificationChannel。
	
			  		String channelId = "channel_1";
                    String description = "143";
                    int importance = NotificationManager.IMPORTANCE_LOW;
                    NotificationChannel channel = new NotificationChannel(channelId, description, importance);
                    manager.createNotificationChannel(channel);
                    Notification notification = new Notification.Builder(this, channelId)
                            .setContentTitle("This is content title")
                            .setContentText("this is content text")
                            .setWhen(System.currentTimeMillis())
                            .setSmallIcon(R.mipmap.ic_launcher)
                            .setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher))
                            .build();
 
### 设置PendingIntent
目前我们的通知点击不会有任何效果，要实现点击效果就需要使用PendingIntent。它和Intent不同的地方在于Inetnt倾向于立即执行某个动作，而PendingIntent更倾向于在某个合适的时机去执行某个动作。

PendingIntent主要通过getActivity()、getBroadcast()、getService()这三个静态方法来获取。这三个方法参数是一样的，第一个参数是context，第二个参数一般用不到，通常传入0即可。第三个参数是一个Intent对象，我们通过这个对象来构建PendingIntent。第四个参数用于确定PendingIntent行为，主要有FLAG\_ONE\_SHOT、FLAG\_NO\_CREATE、FLAG\_CANCEL\_CURRENT、FLAG\_UPDATE\_CURRENT，通常情况下传入0。

创建方式:
	
	Intent intent = new Intent(this, NotificationActivity.class);
    PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);
    
    //创建过程中添加.setContentIntent(pendingIntent)来执行Intent
    //添加.setAutoCancel(true)来设置点击后自动删除通知
    .setContentIntent(pendingIntent)
    .setAutoCancel(true)
    

### 通知进阶技巧
	
	//设置通知铃声
	.setSound(Uri.fromFile(new File("/system/media/audio/ringtones/Luna.ogg")))
	//设置震动，下标为偶数表示静止时长，下标为奇数表示震动时长
    .setVibrate(new long[]{0, 1000, 1000, 1000, 1000})
    //第一个参数是灯的颜色，前一个数表示亮起时长，后一个数表示熄灭时长
    .setLights(Color.GREEN, 1000, 1000)
    
    //还可以使用当前环境默认设置
    .setDefaults(NotificationCompat.DEFAULT_ALL)



## 播放多媒体文件
...
### 播放音频文件
MeidPlayer

### 