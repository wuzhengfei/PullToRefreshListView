package wzf.pulltorefresh;

import java.util.LinkedList;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.TextView;

/**
 *
 * Datetime   ： 2013-4-18 上午11:03:25
 * author     :  wuzhengfei
 */
public class ListViewAdapter extends BaseAdapter {
	private LinkedList<String> datas ;
	private LayoutInflater inflater ;
	
	
	public ListViewAdapter(Context context, LinkedList<String> datas) {
		super();
		this.datas = datas;
		inflater = LayoutInflater.from(context);
	}

	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		ViewHolder holder = null;
		if( convertView == null ){
			convertView = inflater.inflate(R.layout.listview_item, null);
			TextView textView = (TextView) convertView.findViewById(R.id.listview_item);
			
			holder = new ViewHolder();
			holder.textView = textView ;
			convertView.setTag(holder);
		}else{
			holder = (ViewHolder)convertView.getTag();
		}
		holder.textView.setText(datas.get(position));
		return convertView;
	}
	
	@Override
	public long getItemId(int position) {
		return position;
	}
	
	@Override
	public Object getItem(int position) {
		return datas.get(position);
	}
	
	@Override
	public int getCount() {
		return datas.size();
	}
	
	private class ViewHolder{
		TextView textView;
	}

}
