# NotePad

## 实验内容：
1.实现显示条目增加时间戳<br>
2.实现添加笔记标题查询功能 <br>
3.实现进行UI美化，可设置背景颜色<br>
4.实现导出笔记功能<br>
5.实现排序功能<br>

## 显示条目增加时间戳
### 关键代码：
修改noteslist_item.xml布局，添加时间栏<br>
```
    <TextView
    android:id="@+id/text2"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textAppearance="?android:attr/textAppearanceLarge"
    android:textColor="#3771cf"
    android:textSize="20sp" />
```

修改NotePadProvider代码中的数据库结构<br>
```
       @Override
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //颜色
                   + ");");
       }
```
修改NotePadProvider代码中的添加函数<br>
```
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.CHINA);
        format.setTimeZone(TimeZone.getTimeZone("GMT+00:00"));
        String dateTime = format.format(date);
        //System.out.println(dateTime);

        if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
        }

        // If the values map doesn't contain the creation date, sets the value to the current time.
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateTime);
        }

        // If the values map doesn't contain the modification date, sets the value to the current
        // time.
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
        }
```

NodesList填充数据<br>
```
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//修改时间
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
```
修改NoteEditor中的updateNote<br>
```
 Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        String dateTime = format.format(date);

        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
```

### 实验结果：
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notePad1.1.png)<br>

## 添加笔记标题查询功能 
### 关键代码：
list_options_menu.xml，添加一个搜索的item
```
    <item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@android:drawable/ic_menu_search"
        android:showAsAction="always"
        tools:ignore="AppCompatResource">
    </item>
```
在NoteList中找到onOptionsItemSelected方法，在switch中添加搜索的case语句:
```
case R.id.menu_search:
            Intent intent = new Intent();
            intent.setClass(NotesList.this, NoteSearch.class);
            NotesList.this.startActivity(intent);
            return true;
```

添加note_search布局<br>
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="请输入搜索内容..."
        android:layout_alignParentTop="true">
    </SearchView>
    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
```
添加NoteSearch<br>
```
package com.example.android.notepad;

import android.app.ListActivity;
import android.content.ContentUris;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SearchView.OnQueryTextListener;

public class NoteSearch extends  ListActivity implements OnQueryTextListener {

    private static  final  String[] PROJECTION = new String[] {
            NotePad.Notes._ID,
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view);
        //为查询文本框注册监听器
        searchview.setOnQueryTextListener(NoteSearch.this);
    }

    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }

    @Override
    public boolean onQueryTextChange(String newText) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " like ? ";
        String[] selectionArgs = {"%" + newText + "%"};
        Cursor cursor = managedQuery(
                getIntent().getData(),
                PROJECTION,
                selection,
                selectionArgs,
                NotePad.Notes.DEFAULT_SORT_ORDER
        );
        String[] dataColumns = {NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE};
        int[] viewIDs = {android.R.id.text1, R.id.text2};
        MyCursorAdapter adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return  true;
    }

    @Override
    protected void onListItemClick(ListView listView, View view, int position, long id) {
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);
        // Gets the action from the incoming Intent
        String action = getIntent().getAction();
        // Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
            // Sets the result to return to the component that called this Activity. The
            // result contains the new URI
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {
            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
            // Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }

}

```

最后要在AndroidManifest.xml注册NoteSearch：
```
        <activity
            android:name=".NoteSearch"
            android:label="Search"></activity>
```

### 实验结果：
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.2.png)<br>


## 实现进行UI美化，可设置背景颜色 
### 关键代码：
在NotePad契约类中添加:
```
        public static final String COLUMN_NAME_BACK_COLOR = "color";

        public static final int DEFAULT_COLOR = 0; //白
        public static final int YELLOW_COLOR = 1; //黄
        public static final int BLUE_COLOR = 2; //蓝
        public static final int GREEN_COLOR = 3; //绿
        public static final int RED_COLOR = 4; //红
```
NotePadProvider创建数据库表地方添加颜色的字段：
```
       @Override
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //颜色
                   + ");");
       }
```
定义MyCursor
```
package com.example.android.notepad;

import android.content.Context;
import android.database.Cursor;
import android.graphics.Color;
import android.view.View;
import android.widget.SimpleCursorAdapter;

/**
 * Created by 林延萍 on 2019/4/22.
 */

public class MyCursorAdapter extends SimpleCursorAdapter {
    public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
        super(context, layout, c, from, to);
    }
    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));

        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
   }
}

```
NoteList中的PROJECTION添加颜色项,并且将NoteList中用的SimpleCursorAdapter改使用MyCursorAdapter：
```
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//修改时间
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };

```
```
        MyCursorAdapter adapter
            = new MyCursorAdapter (
                      this,                             // The Context for the ListView
                      R.layout.noteslist_item,          // Points to the XML for a list item
                      cursor,                           // The cursor to get items from
                      dataColumns,
                      viewIDs
              );
  ```
  
在菜单文件editor_options_menu.xml中添加一个更改背景的选项:
```
  <item android:id="@+id/menu_color"
        android:title="Color"
        android:icon="@drawable/ic_menu_color"
        android:showAsAction="always"
        tools:ignore="AppCompatResource"/>
```
在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：
```
        case R.id.menu_color:
            changeColor();
            break;
```
在NoteEditor中添加函数changeColor()：
```
    //跳转改变颜色的activity，将uri信息传到新的activity
    private final void changeColor() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteColor.class);
        NoteEditor.this.startActivity(intent);
    }
```


最后要在AndroidManifest.xml注册NoteColor：
```
        <activity
            android:name=".NoteColor"
            android:label="ChangeColor"
            android:theme="@android:style/Theme.Holo.Dialog"
            android:windowSoftInputMode="stateVisible" />
```
### 实验结果：
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.3.png)<br>
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.4.png)<br>

## 实现导出笔记功能
### 关键代码:

在菜单文件editor_options_menu.xml中添加一个导出笔记的选项:
```
    <item android:id="@+id/menu_color"
        android:title="Color"
        android:icon="@drawable/ic_menu_color"
        android:showAsAction="always"
        tools:ignore="AppCompatResource"/>
```
在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：
```
        case R.id.menu_output:
            outputNote();
            break;
```
在NoteEditor中添加函数outputNote()：

```
    //跳转导出activity
    private final void outputNote() {
        Intent intent = new Intent(null, mUri);
        intent.setClass(NoteEditor.this, OutputText.class);
        NoteEditor.this.startActivity(intent);
    }
```
新建布局output_text.xml:
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
    <EditText android:id="@+id/output_name"
        android:maxLines="1"
        android:layout_marginTop="2dp"
        android:layout_marginBottom="15dp"
        android:layout_width="wrap_content"
        android:ems="25"
        android:layout_height="wrap_content"
        android:autoText="true"
        android:capitalize="sentences"
        android:scrollHorizontally="true" />
    <Button android:id="@+id/output_ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:text="Output"
        android:onClick="OutputOk" />
</LinearLayout>
```
创建OutputText的Acitvity:
```
package com.example.android.notepad;

import android.app.Activity;
import android.database.Cursor;
import android.net.Uri;
import android.os.Environment;

import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;

public class OutputText extends Activity {
    //要使用的数据库中笔记的信息
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_NOTE, // 2
            NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 3
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 4
    };
    //读取出的值放入这些变量
    private String TITLE;
    private String NOTE;
    private String CREATE_DATE;
    private String MODIFICATION_DATE;
    //读取该笔记信息
    private Cursor mCursor;
    //导出文件的名字
    private EditText mName;
    //NoteEditor传入的uri，用于从数据库查出该笔记
    private Uri mUri;

    //关于返回与保存按钮的一个特殊标记，返回的话不执行导出，点击按钮才导出
    private boolean flag = false;
    private static final int COLUMN_INDEX_TITLE = 1;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.output_text);
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
        mName = (EditText) findViewById(R.id.output_name);
    }
    @Override
    protected void onResume(){
        super.onResume();
        if (mCursor != null) {
            // The Cursor was just retrieved, so its index is set to one record *before* the first
            // record retrieved. This moves it to the first record.
            mCursor.moveToFirst();
            //编辑框默认的文件名为标题，可自行更改
            mName.setText(mCursor.getString(COLUMN_INDEX_TITLE));
        }
    }
    @Override
    protected void onPause() {
        super.onPause();
        if (mCursor != null) {
            //从mCursor读取对应值
            TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
            NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
            CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
            MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
            //flag在点击导出按钮时会设置为true，执行写文件
            if (flag == true) {
                write();
            }
            flag = false;
        }
    }
    public void OutputOk(View v){
        flag = true;
        finish();
    }
    private void write()
    {
        try
        {
            // 如果手机插入了SD卡，而且应用程序具有访问SD的权限
            if (Environment.getExternalStorageState().equals(
                    Environment.MEDIA_MOUNTED)) {
                // 获取SD卡的目录
                File sdCardDir = Environment.getExternalStorageDirectory();
                //创建文件目录
                File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
                //写文件
                PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                ps.println(TITLE);
                ps.println(NOTE);
                ps.println("创建时间：" + CREATE_DATE);
                ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                ps.close();
                Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}

```
在AndroidManifest.xml中将这个Acitvity主题定义为对话框样式，并且加入权限：
```
    <!-- 在SD卡中创建与删除文件权限 -->
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
```
```
        <!--添加导出activity-->
        <activity android:name="OutputText"
            android:label="Output"
            android:theme="@android:style/Theme.Holo.Dialog"
            android:windowSoftInputMode="stateVisible">
        </activity>

```

### 实验结果：
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.5.png)<br>
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.6.png)<br>
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.7.png)<br>
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.8.png)<br>
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.9.png)<br>


## 实现排序功能
### 关键代码：
list_options_menu.xml中添加：
```
<item
        android:id="@+id/menu_sort"
        android:title="Sort"
        android:icon="@android:drawable/ic_menu_sort_by_size"
        android:showAsAction="always"
        tools:ignore="AppCompatResource">
        <menu>
            <item
                android:id="@+id/menu_sort1"
                android:title="By CreateTime"/>
            <item
                android:id="@+id/menu_sort2"
                android:title="By UpdateTime"/>
            <item
                android:id="@+id/menu_sort3"
                android:title="By Color"/>
        </menu>
    </item>
```
在NotesList菜单switch下添加case：
```
//排序
            //按创建时间
        case R.id.menu_sort1:
            cursor = managedQuery(
                    getIntent().getData(),
                    PROJECTION,
                    null,
                    null,
                    NotePad.Notes._ID
            );
            adapter = new MyCursorAdapter(
                    this,
                    R.layout.noteslist_item,
                    cursor,
                    dataColumns,
                    viewIDs
            );
            setListAdapter(adapter);
            return true;
            //修改时间排序
        case R.id.menu_sort2:
            cursor = managedQuery(
                    getIntent().getData(),
                    PROJECTION,
                    null,
                    null,
                    NotePad.Notes.DEFAULT_SORT_ORDER
            );
            adapter = new MyCursorAdapter(
                    this,
                    R.layout.noteslist_item,
                    cursor,
                    dataColumns,
                    viewIDs
            );
            setListAdapter(adapter);
            return true;
        //颜色排序
        case R.id.menu_sort3:
            cursor = managedQuery(
                    getIntent().getData(),
                    PROJECTION,
                    null,
                    null,
                    NotePad.Notes.COLUMN_NAME_BACK_COLOR
            );
            adapter = new MyCursorAdapter(
                    this,
                    R.layout.noteslist_item,
                    cursor,
                    dataColumns,
                    viewIDs
            );
            setListAdapter(adapter);
            return true;
```

### 实验结果：
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepadsort2.png)<br>
#### 按创建时间排序
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/sort.png)<br>
#### 按修改时间排序 
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/sort2.png)<br>
#### 按颜色排序
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepadsort4.png)<br>







