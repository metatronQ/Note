### REcyclerView
1.布局:xml + item.xml  
2.适配器adapter.java  

	public class adapter extends Recycler.Adapter<adapter.ViewHolder>{

		private class ViewHolder extends RecyclerVIew.ViewHolder{
		public ViewHolder(View view){//绑定View}}

		public adapter（List<实体类> 实体类集合）{}
		
		public ViewHolder onCreateViewHolder(ViewGroup parent,int viewType){//处理VIew的逻辑}

		public void onBindVIewHolder(ViewHolder holder,int position){//为view赋值}

		public int getItemCount(){return 集合.size();}	
	}
  
3.使用RecyclerView  

* 绑定RecyclerVIew	RecyclerVIew recycler = (RecyclerView)findVIewById(R.id.recycler_view);
* 选择布局方式并实例化：LinearLayoutMananger、GridLayoutManager、StaggeredGridlayoutManager
* 将布局传入：	recycler.serLayoutManager(layoutManager);
* 实例化适配器并传入实例集合；
* 传入适配器：	recycler.setAdapter(adapter);




















