# Content Provider
## 运行时权限
Android在6.0系统后加入了运行时权限，对于那些触及隐私或者对设备安全性造成影响的权限会在运行时由用户手动点击授权。

|  权限组名 | 权限名 |
| :-----:| :----: |
| CALENDAR | READ\_CALENDAR、WRITE\_CANLENDAR |
| CAMERA | CAMERA |
| CALENDAR | READ\_CONTACTS、WRITE\_CONTACTS |
|                  | GET\_ACCOUNTS |
| LOCATION | ACCESS\_FINE\_LOCATION、ACCESS\_COARSE\_LOCATION |
| MICROPHONE | RECORD\_AUDIO |
| PHONE | READ\_PHONE\_STATE、CALL_PHONE |
|            | READ\_CALL\_LOG、WRITE\_CALL\_LOG |
|            | ADD\_VOICEMAIL、USE\_SIP、PROCESS\_OUTGOING\_CALLS |
| SENSORS | BODY\_SENSORS |
| SMS | SEND\_SMS、RECEIVE\_SMS、READ\_SMS |
|        | RECEIVE\_WAP\_PUSH、RECEIVE_MMS |
| STORAGE | READ_EXTERNAL_STORAGE、WRITE_EXTERNAL_STORAGE |

## 在程序运行时申请权限
通过申请CALL_PHONE权限来测试。
首先需在AndroidMainifest.xml文件中声明:

	<uses-permission android:name="android.permission.CALL_PHONE"/>

MainActivity:

	 public class MainActivity extends AppCompatActivity {

    		@Override
    		protected void onCreate(Bundle savedInstanceState) {
        		super.onCreate(savedInstanceState);
        		setContentView(R.layout.activity_main);

        		Button make_call = findViewById(R.id.make_call);
        		make_call.setOnClickListener(new View.OnClickListener() {
            			@Override
            			public void onClick(View v) {
                			if(ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.CALL_PHONE)
                        			!= PackageManager.PERMISSION_GRANTED) {
                    			ActivityCompat.requestPermissions(MainActivity.this,
                            			new String[] { Manifest.permission.CALL_PHONE }, 1);
               	 			} else {
                    			call();
                			}
            			}
        		});
    		}

    		private void call(){
        		try {
            		//ACTION_DIAL打开拨号界面，不需要权限
            		//ACTION_CALL直接拨打电话，需要CALL_PHONE权限
            		Intent intent = new Intent(Intent.ACTION_CALL);
            		intent.setData(Uri.parse("tel:123456"));
            		startActivity(intent);
        		} catch (SecurityException e) {
            		e.printStackTrace();
       			 }
    		}

    		@Override
    		public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults){
        		switch (requestCode){
            			case 1:
                			if(grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                    			call();
                			} else {
                    			Toast.makeText(this, "You denied a permission request", Toast.LENGTH_SHORT).show();
                			}
                			break;
            		default:
       		 }
    	}
    }

## 访问其他程序中的数据
内容提供器一般有两种用法，一种是使用现有的内容提供器来读取和操作相应的程序中的数据，另一种是创建自己的内容提供器给我们程序的数据提供外部访问接口。
### ContentResolver的基本用法
在程序中可以通过Context类的getContentResolver()方法来获得ContentResolver对象。它的CRUD操作和SQOLite只有参数上的不同。

不同于SQLiteDatabase，ContentResolver不接收表名参数，而是使用一个Uri参数代替，这个参数被称为内容URI。内容URI给内容提供器中的数据建立了唯一的标识符，它由两个部分组成:

* authority，它是对不同应用程序作划分的，一般为了避免冲突，都会用程序包名的方式来进行命名。如某个程序的包名是com.example.app，那么authority就可以命名为com.example.app.priovider。
* path，这个参数是对某个程序中的数据库不同的表作区分的。例如com.example.app.priovider/`table name`。

我们还需要在头部加上协议声明，因此标准格式写法为：content://com.example.app.provider/`table name`
查询方法如下:
	
	Cursor cursor = getContentResolver().query(
					uri,
					projection,
					selection,
					selectionArgs,
					sortOrder);
| query()方法参数  | 对应SQL部分 | 描述 |
| :-----------:  | :----------: | :-: |
| uri | from table_name | 指定应用程序下的一张表 |
| projection | select column1, column2 | 指定查询列名 |
| selection | where column = value | 指定where约束条件 |
| selectionArgs | - | 为where占位符提供具体的值 |
| orderBy | order by column1, column2 | 指定查询结果的排序方式 |

CRUD操作略... 

### 读取系统联系人
首先需在AndroidMainifest.xml文件中声明:

	<uses-permission android:name="android.permission.CALL_PHONE"/>
MainActivity:
	
	public class MainActivity extends AppCompatActivity {

    			ArrayAdapter<String> adapter;
    			List<String> contactsList = new ArrayList<>();

    			@Override
    			protected void onCreate(Bundle savedInstanceState) {
        			super.onCreate(savedInstanceState);
        			setContentView(R.layout.activity_main);

        			ListView contactsListView = findViewById(R.id.contacts_list);
        			adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, contactsList);
        			contactsListView.setAdapter(adapter);
        			if(ContextCompat.checkSelfPermission(this, Manifest.permission.READ_CONTACTS)
                			!= PackageManager.PERMISSION_GRANTED) {
            			ActivityCompat.requestPermissions(this,
                    			new String[]{ Manifest.permission.READ_CONTACTS }, 1);
        			}
        			else {
            			readContacts();
        			}			
    			}

    			private void readContacts(){
        			Cursor cursor = null;
       			 try {
            			cursor =  getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
                    			null, null, null, null);
            			if (cursor != null) {
                			while (cursor.moveToNext()) {
                    			//获取联系人姓名
                    			String displayName = cursor.getString(cursor.getColumnIndex(
                            			ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
                    			//获取联系人号码
                    			String number = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
                    			contactsList.add(displayName + "\n" + number);
               			 }
               			 adapter.notifyDataSetChanged();
          			  }
        			} catch (Exception e){
           			 e.printStackTrace();
        			} finally {
           			 if(cursor != null){
               			 cursor.close();
            			}
        			}
    			}

    			@Override
    			public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] results){
        			switch (requestCode){
           			 case 1:
                			if(results.length > 0 && results[0] == PackageManager.PERMISSION_GRANTED){
                    			readContacts();
               			 } else {
                    			Toast.makeText(this, "You denied read contacts permission", Toast.LENGTH_SHORT).show();
               			 }
                			break;
            			default:
       			 }
    		}
	}
## 自定义内容提供器
...