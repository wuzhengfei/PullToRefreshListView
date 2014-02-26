package wzf.pulltorefresh;

import java.text.SimpleDateFormat;
import java.util.Date;

import android.content.Context;
import android.util.AttributeSet;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.MotionEvent;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.ViewGroup;
import android.view.animation.Animation;
import android.view.animation.AnimationUtils;
import android.widget.AbsListView;
import android.widget.AbsListView.OnScrollListener;
import android.widget.ImageView;
import android.widget.ListView;
import android.widget.ProgressBar;
import android.widget.TextView;

/**
 * 
 * Datetime   ： 2013-4-18 下午2:07:45
 * author     :  wuzhengfei
 */
public class MyPullRefreshListView extends ListView implements OnScrollListener, OnClickListener {
	private static final String TAG = MyPullRefreshListView.class.getSimpleName();
	private SimpleDateFormat format = new SimpleDateFormat("yyyy年MM月dd日 HH:mm") ;
	
	
	/**
	 * Header移动距离小于PULL_REFRESH_HEIGHT时，不刷新
	 */
	private final int PULL_REFRESH_HEIGHT = 100 ;
	/**
	 * 手拖动list view的距离和head view实际下拉距离之比；
	 */
	private final int RATIO = 2;
	
	
	/*****************************  pull status ***************************************/
	private enum PullStatus{
		/** 释放刷刷新中  */
		RELEASE_To_REFRESH, 
		/** 下拉刷新中   */
		PULL_To_REFRESH, 
		/** 刷新中  */
		REFRESHING, 
		/**刷新或者数据加载完成  */
		DONE, 
		/** 数据加载中  */
		LOADING
	}
	
	
	private LayoutInflater inflater;
	
	/***************************** header and footer *********************************/
	/**
	 * header view，包括了下拉图标，提示信息，进度条等。
	 */
	private View headerView ;
	private TextView headerPullTipsView ;
	private TextView lastUpdatedTimeView ;
	private ImageView headerPullArrowImageView;
	private ProgressBar headerProgressBar ;
	
	private View footerView ;
	private TextView footerMoreView;
	private ProgressBar footerProgressBar ;
	
	
	/******************************* animation ***********************************/
	/** 下拉图标旋转动画 */
	private Animation rotateAnimation ;
	/** 下拉图标再次旋转动画 */
	private Animation reverseRotateAnimation;
	
	private int headContentHeight;
	
	/**
	 * 加载数据的类型，是下拉刷新，还是点击Load More按钮
	 * Datetime   ： 2013-4-18 下午5:02:20
	 * author     :  wuzhengfei
	 */
	private enum RefreshType{
		/**下拉刷新*/
		Refresh, 
		/**加载更多*/
		LoadMore
	}
	private RefreshType refreshType  ;
	
	/**
	 * 当前的状态
	 */
	private PullStatus currentStatus = PullStatus.DONE ;
	/**
	 * 是否可刷新
	 */
	private boolean isRefreshable ;
	/**
	 * 下拉开始时，手接触屏幕时的y轴坐标。ꡣ
	 */
	private int startY ;
	/**
	 * 是否之前已经记录了下拉开始时后的y轴坐标，如果记录了就不在记录，以免反复记录造成换乱。
	 */
	private boolean isRecord = false;
	/**
	 * 是否正在往回拖动视图
	 */
	private boolean isPullBack;
	
	private OnRefreshListener refreshListener;
	
	private int firstItemIndex = 0 ;
	
	/**
	 * 最小拖动的距离，为了防止用户手指触屏误操作，当用户拖动距离过小时，不滑动
	 */
	private int minPullY = 5;
	

	public MyPullRefreshListView(Context context) {
		super(context);
		init(context);
	}

	public MyPullRefreshListView(Context context, AttributeSet attrs) {
		super(context, attrs);
		init(context);
	}
	
	/**
	 * <li>Init Header View
	 * <li>Init Footer View
	 * <li>Init Animation
	 * @param context
	 */
	private void init(Context context){
		inflater = LayoutInflater.from(context);
		initHeaderView();
		initFooterView();
		
		setOnScrollListener(this);
		
		rotateAnimation = AnimationUtils.loadAnimation(context, R.anim.rotate_180);
		reverseRotateAnimation = AnimationUtils.loadAnimation(context, R.anim.rotate_180_reverse);
		rotateAnimation.setFillAfter(true);
		reverseRotateAnimation.setFillAfter(true);
		
		isRefreshable = false;
		
		updateHeaderViewByStatus(currentStatus);
	}
	
	private void initHeaderView(){
		headerView = inflater.inflate(R.layout.listview_header, null);
		headerPullArrowImageView =(ImageView) headerView.findViewById(R.id.listview_header_arrowimage);
		headerProgressBar = (ProgressBar) headerView.findViewById(R.id.listview_header_processbar) ;
		headerPullTipsView = (TextView) headerView.findViewById(R.id.listview_header_tips);
		lastUpdatedTimeView = (TextView) headerView.findViewById(R.id.listview_header_lastupdatetime);
		lastUpdatedTimeView.setText(getResources().getString(R.string.last_update_time, format.format(new Date())));
		
		measureView(headerView);
		headContentHeight = headerView.getMeasuredHeight();
		headerView.invalidate();
		addHeaderView(headerView);
	}
	/**
	 * 初始化Footer View
	 */
	private void initFooterView(){
		footerView = inflater.inflate(R.layout.listview_footer, null);
		footerMoreView = (TextView) footerView.findViewById(R.id.listview_footer_more);
		footerProgressBar = (ProgressBar) footerView.findViewById(R.id.listview_footer_progressbar);
		footerProgressBar.setVisibility(View.GONE);
		measureView(footerView);
		footerView.invalidate();
		addFooterView(footerView);
		
		footerMoreView.setOnClickListener(this);
	}

	@Override
	public void onScroll(AbsListView view, int firstVisibleItem,
			int visibleItemCount, int totalItemCount) {
		firstItemIndex = firstVisibleItem ;
	}
	

	@Override
	public void onScrollStateChanged(AbsListView view, int scrollState) {

	}
	
	
	@Override
	public void onClick(View v) {
		switch (v.getId()) {
		case R.id.listview_footer_more:
			onLoadMore();
			break;

		default:
			break;
		}
	}

	@Override
	public boolean onTouchEvent(MotionEvent ev) {
		if( isRefreshable ){
			switch (ev.getAction()) {
			
			case MotionEvent.ACTION_DOWN : //手按在屏幕上的动作。
				doActionDown(ev);
				break;
				
			case MotionEvent.ACTION_UP:
				doActionUp(ev);
				break;

			case MotionEvent.ACTION_MOVE:
				doActionMove(ev);
				break;
				
			default:
				break;
			}
		}
		return super.onTouchEvent(ev);
	}

	private void doActionDown(MotionEvent ev){
		Log.i(TAG, "doActionDown:  firstItemIndex="+firstItemIndex+"  startY="+ev.getY()+"   isRecord="+isRecord);
		if( firstItemIndex == 0 && !isRecord ){
			startY = (int)ev.getY();
			isRecord = true;
		}
	}
	
	private void doActionUp(MotionEvent ev){
		int currentY = (int)ev.getY();
		int offset = currentY - startY ;
		if( currentStatus != PullStatus.REFRESHING && currentStatus != PullStatus.LOADING &&  currentStatus != PullStatus.DONE){
			if( offset > 0 ){
				if( currentStatus == PullStatus.PULL_To_REFRESH ){
					updateHeaderViewByStatus(PullStatus.DONE);
					
				}else if( currentStatus == PullStatus.RELEASE_To_REFRESH) {
					updateHeaderViewByStatus(PullStatus.REFRESHING);
					onRefresh() ;
				}
			}
		}
		isRecord = false;
		isPullBack = false;
		Log.i(TAG, "doActionUp:  currentStatus="+currentStatus);
	}
	
	private void doActionMove(MotionEvent ev){
		int currentY = (int)ev.getY();
		if( !isRecord && firstItemIndex == 0 ){
			isRecord = true;
			startY = currentY ;
		}
		int offset = currentY - startY ;
		int moveY = (int) (offset / RATIO);
		if( currentStatus != PullStatus.REFRESHING && currentStatus != PullStatus.LOADING && isRecord  ){
			if( offset > 0 ){
				Log.i(TAG, "doActionMove:  offset="+offset+" currentStatus="+currentStatus);
				if( currentStatus == PullStatus.RELEASE_To_REFRESH ){
					setSelection(0);
					if( moveY < PULL_REFRESH_HEIGHT ) {
						isPullBack = true;
						
						updateHeaderViewByStatus(PullStatus.PULL_To_REFRESH);
					}
				}else if ( currentStatus == PullStatus.PULL_To_REFRESH ) {
					setSelection(0);
					if (moveY > PULL_REFRESH_HEIGHT) {
						isPullBack = false;
						updateHeaderViewByStatus(PullStatus.RELEASE_To_REFRESH);
					}
					
				}else if( currentStatus == PullStatus.DONE ){
					if( offset > minPullY ){
						updateHeaderViewByStatus(PullStatus.PULL_To_REFRESH);
					}
				}

				// 更新headView的paddingTop
				if (currentStatus == PullStatus.PULL_To_REFRESH || currentStatus == PullStatus.RELEASE_To_REFRESH) {
					headerView.setPadding(0, - headContentHeight + moveY , 0, 0);
				}
			}
			
		}
	}
	
	/**
	 * 根据状态来跟新UI界面。
	 * @param status : 状态
	 */
	private void updateHeaderViewByStatus(PullStatus status){
		switch (status) {
		case RELEASE_To_REFRESH:
			headerPullArrowImageView.setVisibility(View.VISIBLE);
			headerPullTipsView.setVisibility(View.VISIBLE);
			lastUpdatedTimeView.setVisibility(View.VISIBLE);
			headerProgressBar.setVisibility(View.GONE) ;
			
			headerPullArrowImageView.clearAnimation();
			headerPullArrowImageView.startAnimation(rotateAnimation);
			headerPullTipsView.setText(R.string.release_to_refresh);
			
			break;
		case PULL_To_REFRESH :
			
			headerProgressBar.setVisibility(View.GONE);
			headerPullTipsView.setVisibility(View.VISIBLE);
			lastUpdatedTimeView.setVisibility(View.VISIBLE);
			headerPullArrowImageView.clearAnimation();
			headerPullArrowImageView.setVisibility(View.VISIBLE);
			//如果是先拉到RELEASE_To_REFRESH的状态，后来后拉到PULL_To_REFRESH的状态，即又往回拉了，表示放弃刷新操作。
			if( isPullBack ){
				headerPullArrowImageView.clearAnimation();
				headerPullArrowImageView.startAnimation(reverseRotateAnimation);
				
				isPullBack = false;
			}
			
			headerPullTipsView.setText(R.string.pull_to_refresh);
			break;
		case REFRESHING :
			headerView.setPadding(0, 30 , 0, 0);
			headerProgressBar.setVisibility(View.VISIBLE);
			headerPullTipsView.setVisibility(View.GONE);
			lastUpdatedTimeView.setVisibility(View.VISIBLE);
			headerPullArrowImageView.clearAnimation();
			headerPullArrowImageView.setVisibility(View.GONE);
			break;
		case DONE :
			headerView.setPadding(0, -headContentHeight, 0, 0);
			headerProgressBar.setVisibility(View.GONE);
			
			headerPullArrowImageView.clearAnimation();
			headerPullTipsView.setText(R.string.pull_to_refresh);
			lastUpdatedTimeView.setVisibility(View.VISIBLE);
			break;
		default:
			break;
		}
		Log.i(TAG, "updateHeaderViewByStatus:   状态从 "+currentStatus+"  更新成 "+status);
		currentStatus = status ;
	}
	
	
	public void setOnRefreshListener(OnRefreshListener refreshListener){
		this.refreshListener = refreshListener ;
		this.isRefreshable = true ;
	}
	
	/**
	 * Refresh操作
	 */
	private void onRefresh(){
		if( refreshListener != null){
			refreshType = RefreshType.Refresh ;
			currentStatus = PullStatus.REFRESHING ;
			refreshListener.onRefresh();
		}
	}
	/**
	 * 加载更多操作
	 */
	private void onLoadMore(){
		if( refreshListener != null){
			refreshType = RefreshType.LoadMore ;
			footerMoreView.setVisibility(View.GONE);
			footerProgressBar.setVisibility(View.VISIBLE);
			currentStatus = PullStatus.LOADING ;
			refreshListener.onLoadMore();
		}
	}
	
	/**
	 * Refresh Complete后更新View中的类容。
	 * <li>当下拉Header来刷新，那么需要隐藏Header的信息。
	 * <li>当点击More来刷新，需要显示More，而隐藏ProgressBar
	 */
	public void onRefreshComplete(){
		currentStatus = PullStatus.DONE;
		if( refreshType == RefreshType.Refresh ){
			updateHeaderViewByStatus(currentStatus);
			lastUpdatedTimeView.setText(getResources().getString(R.string.last_update_time, format.format(new Date())));
		}else{
			footerMoreView.setVisibility(View.VISIBLE);
			footerProgressBar.setVisibility(View.GONE);
		}
	}
	

	/**
	 * 计算child的Width和Height
	 * @param child
	 */
	private void measureView(View child) {
		ViewGroup.LayoutParams p = child.getLayoutParams();
		if (p == null) {
			p = new ViewGroup.LayoutParams(ViewGroup.LayoutParams.FILL_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT);
		}

		int childWidthSpec = ViewGroup.getChildMeasureSpec(0, 0 + 0, p.width);
		int lpHeight = p.height;
		int childHeightSpec;
		if (lpHeight > 0) {
			childHeightSpec = MeasureSpec.makeMeasureSpec(lpHeight, MeasureSpec.EXACTLY);
		} else {
			childHeightSpec = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
		}
		child.measure(childWidthSpec, childHeightSpec);
	}

	
	/**
	 * ListView Refresh Listener
	 * Datetime   ： 2013-4-18 上午11:13:12
	 * author     :  wuzhengfei
	 */
	public interface OnRefreshListener {
		/**
		 * Refresh，下拉刷新时执行
		 */
		public void onRefresh();
		/**
		 * Load More，点击家在更多时执行
		 */
		public void onLoadMore();
	}
	
}
