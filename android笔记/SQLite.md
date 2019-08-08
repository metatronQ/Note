## SqLite  
### SQLiteOpenHelper  {
抽象方法  

* onCreate()//创建  
* onUpggrate()//更新 

		public void onUppgrate(SQLiteDatabase db,int oldVersion,int newVersion){
		//删表语句
		db.execSQl("drop table if exists 表名")；
		} //当newVersion大于构造版本号时执行 
 
构造方法：  
传入四个参数：Context、数据库名、Cursor（null）、版本号  
###}  

###实例方法：  

#### 创建：  
都返回一个SQLiteDatabase对象  

* getReadableDatabase()//满时只读  
* getWriteableDatabase()//满时异常  

建表语句：参考<https://www.runoob.com/sqlite/sqlite-intro.html>

	create table 表名（
		列名称1 数据类型 （primary key 设置主键）（autoincrement 自增），
		列名称2 数据类型，
		列名称3 数据类型
	）

### (SQLiteDatabase).   
#### 添加数据：
**insert("表名"，null，ContentValues);**  
**ContentValues**:承载数据的对象，用其 **put（"列名称"，数据）**方法进行数据的组装(.clear()清除数据)  

#### 更新数据  
**update("表名",ContentValues,"约束项（列名称 = ？）"，"该列下的某个具体数据(new String[] {"数据"})")**  

#### 删除数据  
**delete("表名","约束项(列名称 > ?)","该列下的某个具体数据")**  

#### 查询数据：返回可供提取数据的Cursor对象  
**query(table,columns, selection, selectionArgs, groupBy, having, orderBy)**  

>db.query(tableName,columns,selection,selectionArgs,null4,null5,null6);  

>select columns(列) from tableName where selection（列 = ?)= selectionArgs(行) group by null4 having null5 order by null6  

**取出数据:**
  
	Cursor cursor = db.query(tableName,columns,selection,selectionArgs,null4,null5,null6);
	if(cursor.moveToFirst()){//表的起始是第一行
		do{
			String name = cursor.getString(cursor.getColumnIndex("某列"))；
			//以此类推//
		}while（cursor。moveToNext()）;
	}