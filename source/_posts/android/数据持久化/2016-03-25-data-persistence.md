---
author: ivyxjc
date: 2016-03-25
title: 数据持久化
category: Android
tags: [android,data_persistence]
keywords:
description: 存储数据主要有三种方式:<br>1. 存储到文件<br>2. SharedPreferences存储<br>3. SQLite
---

## 数据存储到文件

`Context`类提供了一个`openFileOutput()`方法，可以将数据存储到指定的文件中。接受两个参数，
1. 第一个参数是文件名(无需包含路径)，默认存储 到`/data/data/<package name>/files/`目录下。
2. 第二个参数是文件的操作模式，主要有两种：`MODE_PRIVATE`和`MODE_APPEND`。其中`MODE_PRIVATE`是默认的操作模式，表示当指定同样文件名的时候，所写入的内容将会覆盖原文件中的内容。`MODE_APPEND`则表示如果该文件已存在就往文件里面追加内容。还有另外两种模式：`MODE_WORLD_READABLE`和`MODE_WORLD_WRITEABLE`表示允许其它的应用程序对该程序中的文件进行读写操作，已被废弃。

```java
public void save(String inputText){
    FileOutputStream out=null;
    BufferedWriter writer=null;
    try{
        out=openFileOutput("data", Context.MODE_PRIVATE);
        writer=new BufferedWriter(new OutputStreamWriter(out));
        writer.write(inputText);
    }catch (IOException e){
        e.printStackTrace();
    }finally {
        try{
            if(writer!=null){
                writer.close();
            }
        }catch (IOException e){
            e.printStackTrace();
        }
    }
}

public String load(){
    FileInputStream in=null;
    BufferedReader reader=null;
    StringBuilder content=new StringBuilder();

    try{
        in=openFileInput("data");
        reader=new BufferedReader(new InputStreamReader(in));
        String line="";
        while ((line=reader.readLine())!=null){
            content.append(line);
        }
    }catch (IOException e){
        e.printStackTrace();
    }finally {
        if(reader!=null){
            try{
                reader.close();
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }
    return content.toString();
}

```

## SharedPreferences存储

```java
//存储数据
SharedPreferences.Editor editor=getSharedPreferences("data",MODE_PRIVATE).edit();
editor.putString("name","Tom");
editor.putInt("age", 28);
editor.putBoolean("married", false);
editor.commit();

//读取数据

SharedPreferences preferences=getSharedPreferences("data",MODE_PRIVATE);
String name=preferences.getString("name", "");
boolean married=preferences.getBoolean("married", false);
int age=preferences.getInt("age",0);
```

## SQLite

安卓提供了一个`SQLiteOpenHelper`帮助类，可以借助这个类对数据库进行创建和升级。

### 建表

```sql
//建表
create table Book{
	id integer primary key autoincrement,
    author text,
    price real,
    pages integer,
    name text,
}
```

1. `integer`:整型
2. `real`：浮点型
3. `text`：文本类型
4. `blob`：二进制类型

### 添加数据

```java
SQLiteDatabase db=dbHelper.getWritableDatabase();
ContentValues values=new ContentValues();
values.put("author","Dan Brown");
values.put("name","The Da Vinci Code");
values.put("pages",454);
values.put("price", 16.96);
db.insert("Book",null,values);
values.clear();
```

向数据库中添加数据

### 更新数据

可以使用
`db.update(String Table,ContenValues values,String WhereClauses,String[] WhereArgs)`

第三个参数对应SQL语句中的`where`，其中用`?`作占位符，第四个参数则表示`?`的内容。

```java
//表示将name为The Da Vinci Code 的价格改为10.99
ConteValues values=new ContentValues();
values.put("price",10.99);
db.update("Book",values,"name=?",new String[]{"The Da Vinci Code"})；
```

### 删除数据

可以使用`db.delete(String Table,String WhereClauses,String[] WhereArgs)`
第二，三两个参数用来约束删除某一行或几行的数据，不指定的话就默认删除所有行。

### 查询数据

query()方法参数 | 对应SQL部分|描述
--------|--------|--------
table |   from table_name|  指定查询的表名
columns  |   select column1,column2 |  指定查询的列名
selection  |    where column=value| 指定where的约束条件
selectionArgs |   -  |  为where中的占位符提供具体的数值
groupBy   |    group by column    |  指定需要group by的列
having |   having column=value |对group by后的结果进一步约束
orderBy | order by column1, column2 |  指定查询结果的排序方式


`Cursor cursor = db.query("Book", null, null, null, null, null, null);`
上式查询表中所有数据


### 直接使用SQL语句

除查询之外的所有方法可以条用`db.execSQL(...)`接执行SQL语句
查询数据的方法时可调用它`db.rawQuery(...)`接执行SQL语句

## 代码

```java
public class MyDatabaseHelper extends SQLiteOpenHelper {
    private Context mContext;

    public static final String CREATE_BOOK="create table Book (" +
            "id integer primary key autoincrement," +
            "author text," +
            "price real," +
            "pages integer," +
            "name text)";

    public MyDatabaseHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
        mContext=context;
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(CREATE_BOOK);
        //db.execSQL(CREATE_CATEGORY);
        Toast.makeText(mContext,"Create succeeded", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("drop table if exists Book");
        db.execSQL("drop table if exists Category");
        onCreate(db);
    }
}
```

```java
public class MainActivity extends AppCompatActivity {

    private MyDatabaseHelper dbHelper;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

//最后一个数是版本号，若版本号比原先高，则会调用  SQLiteOpenHelper中的onUpgrade()。
        dbHelper=new MyDatabaseHelper(this,"BookStore.db",null,5);

        Button button=(Button)findViewById(R.id.button);

        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dbHelper.getWritableDatabase();
            }
        });

        Button addData=(Button)findViewById(R.id.add_data);
        addData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db=dbHelper.getWritableDatabase();

                //向数据库中添加数据
                ContentValues values=new ContentValues();
                values.put("author","Dan Brown");
                values.put("name","The Da");
                values.put("pages",454);
                values.put("price", 16.96);
                db.insert("Book",null,values);
                values.clear();
            }
        });
    }
}
```
