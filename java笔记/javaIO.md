# File:对文件本身进行操作
	//适配不同系统方便移植的路径分隔符常量
	File.pathSeparator(Windows:;)
	File.separator(Windows:\)
	
	FIle file = new File(路径);
	//创建文件(路径中包含要创建的文件)
	file.createNewFile();
	//删除文件
	file.delete();
	//判断文件是否存在
	file.exists();
	//创建文件夹(路径中包含要创建的文件夹)
	file.mkdir();
	//列出目录中的全部内容
	1.list():列出全部名称，返回一个字符串数组
	2.listFile():列出完整路径，返回一个File对象
	//判断一个给定的路径是否为目录
	file.isDirectory();

# RandomAccessFile:对文件内容进行操作:随机读取固定格式的内容

	//构造
	RandomAccessFile(File/String(路径)，r/w/rw(设置读写模式))
	//关闭
	rdf.close();
	//将内容读取到byte数组中
	rdf.read();
	//从文件中读取整型数据
	rdf.readInt();
	//读取一个字节
	rdf.readByte();
	//设置读指针的位置
	rdf.seek(long pos);
	//将一个字符串写入到文件中
	rdf.writeBytes(String s);
	//将一个整型int写入文件，长度为4位
	rdf.writeInt(int v);
	//指针跳过指定的字节
	rdf.skipBytes(int n);

# 字节流与字符流的基本操作

操作步骤：
1. 使用File打开一个文件  
2. 通过字节流或字符流的子类指定输出的位置  
3. 进行读写操作  
4. 关闭输入输出  

## 字节流：主要操作byte类型数据，以byte数组为准，主要操作类数OutputStream类和InputStream类

* 1.字节输出流OutputStream：**写入文件**：如果操作的是个文件，则可以使用FileOutputStream类，通过向上转型对OutputStream进行实例化，注意写入的都是byte类型

		方法：
		//关闭输出流
		close();
		//刷新缓冲区
		flush();
		//将一个byte数组写入数据流
		write(byte[]);
		//将一个指定范围的byte数组写入数据流
		write(byte[],int off,int len);
		//将一个字节数据写入数据流
		write(int b);

		//追加新内容
		FileOutputStream(file,true);
**\r\n：表示换行**

* 2.字节输入流InputStream：**读取文件：**

		方法：
		//取得输入文件的大小：
		available();
		//关闭输入流
		close();
		//读取内容，以数字方式获取
		read();
		//将内容读取到byte数组中，同时返回读入的个数
		read(byte[]);