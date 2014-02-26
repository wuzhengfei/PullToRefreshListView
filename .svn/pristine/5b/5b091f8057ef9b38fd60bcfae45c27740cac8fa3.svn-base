package wzf.pulltorefresh;

import java.util.LinkedList;

import wzf.pulltorefresh.MyPullRefreshListView.OnRefreshListener;
import android.app.Activity;
import android.os.AsyncTask;
import android.os.Bundle;

/**
 * 
 * Datetime   ： 2013-4-18 下午2:10:07
 * author     :  wuzhengfei
 */
public class MyPullRefreshActivity extends Activity {
	private LinkedList<String> datas ;
	private ListViewAdapter adapter;
	private MyPullRefreshListView listView;

	/** Called when the activity is first created. */
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);
		
		datas = new LinkedList<String>();
		for (int i = 0; i < 4; i++) {
			datas.add(" data : line " + (i + 1));
		}
		
		
		listView = (MyPullRefreshListView) findViewById(R.id.listView);
		
		adapter = new ListViewAdapter(this, datas);
		listView.setAdapter(adapter);
		
		listView.setOnRefreshListener(new OnRefreshListener() {
			@Override
			public void onRefresh() {
				boolean refresh = true;
				Object[] params = { refresh };
				AsyncTask<Object, Void, LinkedList<String>> task = new AsyncTaskImpl();
				task.execute(params);
			}

			@Override
			public void onLoadMore() {
				boolean refresh = false;
				Object[] params = { refresh };
				AsyncTask<Object, Void, LinkedList<String>> task = new AsyncTaskImpl();
				task.execute(params);
				
			}
		}) ;

	}
		
	
	private class AsyncTaskImpl extends AsyncTask<Object, Void, LinkedList<String>>{

		@Override
		protected LinkedList<String> doInBackground(Object... params) {
			try {
				Thread.sleep(1000);
			} catch (Exception e) {
				e.printStackTrace();
			}
			Boolean isRefresh = (Boolean)params[0];
			if( isRefresh != null && isRefresh ){
				datas.addFirst("刷新后的内容 " + (datas.size()+1));
				
			}else{
				datas.addLast("加载更多数据 " + (datas.size()+1)) ;
			}
			return datas;
		}

		@Override
		protected void onPostExecute(LinkedList<String> result) {
			adapter.notifyDataSetChanged();
			listView.onRefreshComplete();
		}

		@Override
		protected void onProgressUpdate(Void... values) {
			super.onProgressUpdate(values);
		}
		
	}
}