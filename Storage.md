# 数据持久化技术
Android主要提供三种方式简单地实现了数据持久化功能，即文件存储、SharedPreference存储以及数据库存储。

## 文件存储
文件存储只适合简单的文本文件和二进制文件的存储，如果要存储较为复杂的文本文件，则要定义自己的格式规范。

### 将数据存储到文件中
`Context`类提供了一个`openFileoutput()`方法用于将数据存储到文件中。它的第一个参数是文件名，Android中存储到本地文件是不可以指定路径的，默认存放在`/data/data<package name>/files/`目录下。第二个参数是文件的操作模式，主要有`MODE_PRIVATE`和`MODE_APPEND`。默认操作模式为`MODE_PRIVATE`，它们分别是覆盖写和追加写。

### 从文件中读取数据
`Context`类还提供了`openFileInput()`方法从文件中读取数据。它只接受一个文件名参数，系统会自动到`/data/data<package name>/files/`目录下加载这个文件。并返回`FileInputStream`对象。

## SharedPreferences存储
不同于文件存储，`SharedPreferences`是通过键值对的方式来存储数据的。

### 将数据存储到SharedPreference中
Android主要提供了三种方法来获取`SharedPreferences`对象。

1. `Context`类中的`getSharedPreferences()`方法。这个方法接收两个参数，第一个参数用来指定`SharedPreferences`文件名，不存在则会创建一个，SharedPreferences文件都是存放在`/data/data<package name>/shared_prefs/`目录下的。第二个参数用来指定文件的操作模式，目前只有`MODE_PRIVATE`可选。
2. `Activity`类中的`getPreferences()`方法。它只接收一个操作模式参数，它会自动用当前活动的类名作为`SharedPreferences`的文件名。
3. `PreferencesManager`类中的`getDefaultSharedPreferences()`方法。这是个静态方法，它接收一个`Context`参数，自动使用包名作为前缀来命名`SharedPreferences`文件。得到`SharedPreferences`对象后就可以向其中存储数据了，主要分为三步实现。
	* 调用`SharedPreferences`对象的edit()方法来获取一个SharedPreferences.Editor对象。
	* 向	SharedPreferences.Editor对象中添加数据。
	* 调用apply()方法将添加的数据提交，从而完成存储操作。

## SQLite数据库存储
### 创建数据库
Android为了让我们方便管理数据库，专门提供了一个SQLiteOpenHelper帮助类，借助这个类就可以很简单地对数据库进行创建和升级。

SQLiteOpenHelper是一个抽象类，其中有两个抽象方法，分别是onCreate()和onUpgrade()，我们必须重写这两个方法，然后分别在这两个方法中去创建和升级数据库逻辑。
SQLite中还有getReadableDatabase()和getWritableDatabase()来返回可对数据库进行读写的对象，不同的是当数据库不可写入的时候（如磁盘空间已满），前者将以只读的方式去打开数据库，而后者则抛出异常。

数据库文件会存放在`/data/data<package name>/databases/`目录下。
重写SQLiteOpenHelper类:
    
    public class MyDatabaseHelper extends SQLiteOpenHelper {
    
    	private Context mContext;
    	public static final String CREATE_BOOK = "create table book ("
            + "id integer primary key autoincrement, "
            + "author text, "
            + "price real, "
            + "pages integer, "
            + "name text)";

    	public MyDatabaseHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version){
        	super(context, name, factory, version);
       		mContext = context;
   	 	}

    	@Override
    	public void onCreate(SQLiteDatabase db){
        	db.execSQL(CREATE_BOOK);
        	Toast.makeText(mContext, "Create Succeeded", Toast.LENGTH_SHORT).show();
   	 	}

    	@Override
    	public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion){}

    }
MinActivity:

	public class MainActivity extends AppCompatActivity {

    		private static final String TAG = "MainActivity";

    		private  MyDatabaseHelper dbHelper;

    		@Override
    		protected void onCreate(Bundle saveInstanceState){
        		super.onCreate(saveInstanceState);
        		setContentView(R.layout.activity_main);
        		dbHelper = new MyDatabaseHelper(this, "BookStore.db", null, 1);
        		Button createDB = findViewById(R.id.create_db_button);
        		createDB.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dbHelper.getWritableDatabase();
            }
       	 });
    	}
    }

### 升级数据库
将MainActivity中MyDatabaseHelper创建的版本号传入一个更高的值如:

	dbHelper = new MyDatabaseHelper(this, "BookStore.db", null, 2);

MyDatabaseHelper类修改为:

	public class MyDatabaseHelper extends SQLiteOpenHelper {
    		public static final String CREATE_BOOK = "create table book ("
            		+ "id integer primary key autoincrement, "
            		+ "author text, "
            		+ "price real, "
            		+ "pages integer, "
            		+ "name text)";

    		public static final String CREATE_CATEGORY = "create table category ("
            		+ "id integer primary key autoincrement, "
            		+ "category_name text, "
            		+ "categoty_code integer)";

    		private Context mContext;

    		public MyDatabaseHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version){
        		super(context, name, factory, version);
        		mContext = context;
    		}

    		@Override
    		public void onCreate(SQLiteDatabase db){
        		db.execSQL(CREATE_BOOK);
        		db.execSQL(CREATE_CATEGORY);
        		Toast.makeText(mContext, "Create Succeeded", Toast.LENGTH_SHORT).show();
    		}

    		@Override
    		public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion){
        		db.execSQL("drop table if exists book");
        		db.execSQL("drop table if exists category");
        		onCreate(db);
   		}

	}
### SQLite增删改查
...

### LitePal
...



