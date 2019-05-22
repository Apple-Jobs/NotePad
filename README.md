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

### 实验结果：
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.2.png)<br>


## 实现进行UI美化，可设置背景颜色 
### 关键代码：
```

```

### 实验结果：
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.3.png)<br>
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.4.png)<br>

## 实现导出笔记功能
### 关键代码：
```

```

### 实验结果：
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.5.png)<br>
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.6.png)<br>
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.7.png)<br>
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.8.png)<br>
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepad1.9.png)<br>


## 实现排序功能
### 关键代码：
```

```

### 实验结果：
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepadsort2.png)<br>
#### 按创建时间排序
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/sort.png)<br>
#### 按修改时间排序 
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/sort2.png)<br>
#### 按颜色排序
![Image text](https://github.com/Apple-Jobs/img-folder/blob/master/notepadsort4.png)<br>







