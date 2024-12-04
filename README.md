# NotePad
 
## Table of Contents

- [时间戳实现部分](#时间戳实现部分)
- [搜索功能实现部分](#搜索功能实现部分)
- [UI美化实现部分](#美化实现部分)
- [导出笔记功能实现部分](#导出笔记功能实现部分)
 
## 项目描述
 
此项目是基于原有NotePad进行的一次更新。
· 新增笔记时将会生成新增笔记的时间，并且显示在listview
· 实现了搜索功能，能通过关键字对笔记进行快速搜索
· 实现了UI界面的美化，包括listview的背景美化，以及NoteEditor内部背景的美化
· 添加了笔记的导出功能，能够将笔记导出置主文件夹
 
## 时间戳实现部分

1.首先添加COLUMN_NAME_MODIFICATION_DATE字段，用于存储时间戳  

```java  
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
            NotePad.Notes.COLUMN_NAME_NOTE, // 3 - 假设这里是内容列
    };
```

2.然后在视图中添加显示时间戳的代码  
     
```java  
  // 要在视图中显示的光标列的名称，初始化为标题列和修改日期列
    String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };

  // 将显示光标列的视图ID，初始化为noteslist_item2.xml中的TextView
    int[] viewIDs = { R.id.textTitel, R.id.textDate };

  // 创建ListView的后台适配器。
    SimpleCursorAdapter adapter = new SimpleCursorAdapter(
         this,                             // ListView的上下文
         R.layout.noteslist_item2,        // 指向列表项的XML布局
         cursor,                          // 用于获取项目的光标
         dataColumns,                     // 要显示的光标列
         viewIDs                          // 显示光标列的视图ID
    )
```

3.将时间戳格式改为年月日

     
```java  
   private final void updateNote(String text, String title) {

        // 创建ContentValues对象
        ContentValues values = new ContentValues();
        // 获取当前时间戳
        long currentTimeMillis = System.currentTimeMillis();
        // 将时间戳转换为Date对象
        Date date = new Date(currentTimeMillis);
        // 使用SimpleDateFormat将Date对象格式化为HH:mm:ss格式的字符串
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault());
        String formattedTime = sdf.format(date);
        // 将格式化后的时间字符串存储到数据库的相应列中
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, formattedTime);
        // If the action is to insert a new note, this creates an initial title for it.
        if (mState == STATE_INSERT) {
            // If no title was provided as an argument, create one from the note text.
            if (title == null) {
                // Get the note's length
                int length = text.length();
                // Sets the title by getting a substring of the text that is 31 characters long
                // or the number of characters in the note plus one, whichever is smaller.
                title = text.substring(0, Math.min(30, length));
                // If the resulting length is more than 30 characters, chops off any
                // trailing spaces
                if (length > 30) {
                    int lastSpace = title.lastIndexOf(' ');
                    if (lastSpace > 0) {
                        title = title.substring(0, lastSpace);
                    }
                }
            }
            // In the values map, sets the value of the title
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);
        } else if (title != null) {
            // In the values map, sets the value of the title
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, title);
        }
        getContentResolver().update(
                mUri,    // The URI for the record to update.
                values,  // The map of column names and new values to apply to them.
                null,    // No selection criteria are used, so no where columns are necessary.
                null     // No where columns are used, so no where arguments are necessary.
            );
    }
```

4.最终呈现的效果如图  

![image](https://github.com/user-attachments/assets/7e5343c6-4b57-4c9c-9f5b-4546dad8dd6f)


## 搜索功能实现部分

1.在onCreateOptionsMenu中添加搜索功能  

```java  
  @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate menu from XML resource
        super.onCreateOptionsMenu(menu);
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.list_options_menu, menu);

        // 添加搜索功能
        MenuItem searchItem = menu.findItem(R.id.menu_search);
        SearchView searchView = (SearchView) searchItem.getActionView();
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                // 提交搜索时更新查询
                searchNotes(query);
                return true;
            }

            @Override
            public boolean onQueryTextChange(String newText) {
                // 输入时实时更新查询
                searchNotes(newText);
                return true;
            }
        });

        // Add Export option
        menu.add(Menu.NONE, R.id.menu_export, Menu.NONE, "Export Notes");

        Intent intent = new Intent(null, getIntent().getData());
        intent.addCategory(Intent.CATEGORY_ALTERNATIVE);
        menu.addIntentOptions(Menu.CATEGORY_ALTERNATIVE, 0, 0,
                new ComponentName(this, NotesList.class), null, intent, 0, null);

        return super.onCreateOptionsMenu(menu);
    }
```

2.包括searchNotes方法  

```java  
  private void searchNotes(String query) {
        // 构建搜索条件：查询标题包含输入的文本
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " LIKE ?";
        String[] selectionArgs = new String[] { "%" + query + "%" };

        // 执行查询并更新适配器
        Cursor cursor = getContentResolver().query(
                getIntent().getData(),
                PROJECTION,
                selection,
                selectionArgs,
                NotePad.Notes.DEFAULT_SORT_ORDER
        );

        // 更新ListView的数据源
        SimpleCursorAdapter adapter = (SimpleCursorAdapter) getListAdapter();
        adapter.changeCursor(cursor);
    }
```

3.添加菜单项  

```java  
    <item
        android:id="@+id/menu_search"
        android:title="Search"
        android:icon="@android:drawable/ic_menu_search"
        android:showAsAction="collapseActionView|ifRoom"
        android:actionViewClass="android.widget.SearchView"/>

```

4.最终呈现的效果如图  

![image](https://github.com/user-attachments/assets/50734b7a-dc4f-4a15-ae04-3f697fe8c719)

## 美化实现部分

1.覆写getview，该功能能够实现随机生成不同背景颜色的listview，同时我让RGB在一个较为柔和的区间，以免视觉冲击过大  

```java  
   // 创建ListView的后台适配器。
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                this,                             // ListView的上下文
                R.layout.noteslist_item2,        // 指向列表项的XML布局
                cursor,                          // 用于获取项目的光标
                dataColumns,                     // 要显示的光标列
                viewIDs                          // 显示光标列的视图ID
        ){
            @Override
            public View getView(int position, View convertView, ViewGroup parent) {
                // 获取默认的视图
                View view = super.getView(position, convertView, parent);

                // 随机颜色生成器
                Random random = new Random();
                int randomColor1 = Color.rgb(
                        100 + random.nextInt(156),
                        100 + random.nextInt(156),
                        100 + random.nextInt(156)
                );

                // 设置标题和日期背景颜色
                TextView titleView = (TextView) view.findViewById(R.id.textTitel);
                TextView dateView = (TextView) view.findViewById(R.id.textDate);

                titleView.setBackgroundColor(randomColor1);
                dateView.setBackgroundColor(randomColor1);

                titleView.setTextColor(getContrastingColor(randomColor1));
                dateView.setTextColor(getContrastingColor(randomColor1));

                return view;
            }
        };;

        // 设置ListView的适配器为刚刚创建的游标适配器。
        setListAdapter(adapter);
```

2.包括getContrastingColor取对比颜色方法，能够让标题和时间戳随背景随机变化而更加明显  

```java  
   private int getContrastingColor(int backgroundColor) {
        // 提取RGB值
        int red = Color.red(backgroundColor);
        int green = Color.green(backgroundColor);
        int blue = Color.blue(backgroundColor);

        // 计算亮度 (YIQ公式)
        double brightness = (299 * red + 587 * green + 114 * blue) / 1000.0;

        // 如果亮度较高，返回黑色；否则返回白色
        return brightness >= 128 ? Color.BLACK : Color.WHITE;
    }
```

3.最终呈现的效果如图  

![image](https://github.com/user-attachments/assets/6e9929b6-f3fe-4fac-809b-93d039c20963)

## 导出笔记功能实现部分

1.添加菜单项  

```java  
   <item
        android:id="@+id/menu_export"
        android:title="导出"
        android:icon="@android:drawable/ic_menu_save"
        android:showAsAction="ifRoom" />
```

2.在onOptionsItemSelected方法中处理导出菜单项  

```java  
   @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
        case R.id.menu_add:
          /*
           * Launches a new Activity using an Intent. The intent filter for the Activity
           * has to have action ACTION_INSERT. No category is set, so DEFAULT is assumed.
           * In effect, this starts the NoteEditor Activity in NotePad.
           */
           startActivity(new Intent(Intent.ACTION_INSERT, getIntent().getData()));
           return true;
    //-------------------------------------------导出功能case
        case R.id.menu_export:
            // Export all notes
            exportNotes();
            return true;
        case R.id.menu_paste:
          /*
           * Launches a new Activity using an Intent. The intent filter for the Activity
           * has to have action ACTION_PASTE. No category is set, so DEFAULT is assumed.
           * In effect, this starts the NoteEditor Activity in NotePad.
           */
          startActivity(new Intent(Intent.ACTION_PASTE, getIntent().getData()));
          return true;
        default:
            return super.onOptionsItemSelected(item);
        }
    }
```

3.实现exportNotes方法  

```java  
  private void exportNotes() {
        // 获取内容提供者中的笔记数据
        Cursor cursor = getContentResolver().query(
                getIntent().getData(),
                PROJECTION,
                null, null,
                NotePad.Notes.DEFAULT_SORT_ORDER
        );

        if (cursor == null || cursor.getCount() == 0) {
            Log.e(TAG, "No notes available to export.");
            return;
        }

        // 获取当前时间戳用于文件名
        String timestamp = String.valueOf(System.currentTimeMillis());
        File exportDir = new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS), "NotesExport");
        if (!exportDir.exists()) {
            exportDir.mkdirs();
        }

        File exportFile = new File(exportDir, "notes_export_" + timestamp + ".txt");

        try (FileOutputStream fos = new FileOutputStream(exportFile);
             OutputStreamWriter writer = new OutputStreamWriter(fos)) {

            // 遍历光标，写入笔记内容
            while (cursor.moveToNext()) {
                String title = cursor.getString(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
                String modificationDate = cursor.getString(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
                writer.write("Title: " + title + "\n");
                writer.write("Modified on: " + modificationDate + "\n");
                writer.write("===================================\n");

                // 这里我们假设有一个内容字段，可以从数据库中获取笔记的内容
                String content = cursor.getString(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
                writer.write(content + "\n\n");
            }

            Toast.makeText(this, "笔记已成功导出至: " + exportFile.getAbsolutePath(), Toast.LENGTH_LONG).show();
        } catch (IOException e) {
            Log.e(TAG, "Error exporting notes", e);
            Toast.makeText(this, "导出失败，请稍后再试！", Toast.LENGTH_SHORT).show();
        } finally {
            cursor.close();
        }
    }
```

4.修改PROJECTION包括笔记内容  

```java  
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
            NotePad.Notes.COLUMN_NAME_NOTE, // 3 - 假设这里是内容列
    };
```

5.确保权限，在AndroidManifest.xml中添加文件写入权限  

```java  
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

6.最终呈现的效果如图  

![image](https://github.com/user-attachments/assets/a461f038-53c3-4852-9b2d-35345e88d6e9)






