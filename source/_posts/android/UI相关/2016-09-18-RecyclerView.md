---
author: ivyxjc
date: 2016-09-18
title: RecyclerView
category: Android
tags: [android,android_UI,android_RecyclerView]
keywords:
description: 如何使用RecyclerView
toc: true
---

## RecyclerView 设置布局格式
```java
mRecyclerView = (RecyclerView) v.findViewById(R.id.main_rv);
//类似ListView显示用
mRecyclerView.setLayoutManager(new LinearLayoutManager(getActivity()));
//类似GridView显示
mRecyclerView.setLayoutManager(new GridLayoutManager(getActivity(), 3));

```
## RecyclerView 添加适配器 以及设置item点击监听

```java
MainFragment.java

mRecyclerViewAdapter = new RecyclerViewAdapter(getActivity(), mItems, false);
mRecyclerView.setAdapter(mRecyclerViewAdapter)

mRecyclerViewAdapter.setOnItemClickListener(new RecyclerViewAdapter.OnItemClickListenerCustom() {
            @Override
            public void onItemClick(String id, String title) {
                Log.i(LogKeys.MAIN_FRAGMENT, "item is clicked ");
                Intent intent = new Intent(getActivity(), MovieDetailActivity.class);
                intent.putExtra(TransferKeys.MAIN_MOVIE_DETAIL_ID, id);
                intent.putExtra(TransferKeys.MAIN_MOVIE_DETAIL_MOVIE_NAME, title);
                startActivity(intent);
                Log.i(LogKeys.MAIN_FRAGMENT, "startactivity");
            }
        });
```

```java
public class RecyclerViewAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
    private final Context mContext;
    private List<MainItem> mData;
    private OnItemClickListenerCustom mCallback;
    private boolean mIsComingMovie;
    private int mTotalCount;


    public interface OnItemClickListenerCustom {
        void onItemClick(String id,String title);
    }

    public RecyclerViewAdapter_Main(Context context, List<MainItem> data,
                          boolean isComingMovie) {

        this.mContext = context;
        this.mData = data;
        this.mIsComingMovie=isComingMovie;
    }

    public void refresh(ArrayList<MainItem> data){
        mData.addAll(data);
        notifyDataSetChanged();
    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view=LayoutInflater.from(mContext)
                                .inflate(R.layout.item_movie_main,parent,false);
        return new ItemViewHolder(view);
    }

    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
        ((ItemViewHolder)holder).update();
    }

    @Override
    public int getItemCount() {
        return mData.size();
    }

    public void setOnItemClickListener(OnItemClickListenerCustom listener){
        this.mCallback=listener;
    }


    class ItemViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener {
        TextView textTitle;
        MainItem mMainItem;

        public ItemViewHolder(View itemView) {
            super(itemView);
            //初始化控件
            itemView.setOnClickListener(this);
        }

        public void update() {
            //填充控件
        }


        @Override
        public void onClick(View v) {
            if(mCallback!=null){
                mCallback.onItemClick(mMainItem.getMovieId(),mMainItem.getTitle());
            }
        }
    }
}

```

## RecyclerView上滑加载

要使FetchItemTask顺序执行, 否则当下滑触发loadMoreData时,会出现多个线程, 若这几个线程同时
进行,会导致数据重复加载, 这些Task顺序执行, 只需要在doInBackground执行时
检查star<mRecyclerViewAdapter.getStart()
若true, 则说明该线程为重复线程 return null
否则, 执行之后的步骤

```java
MainFragmentBase.java
mRecyclerView.addOnScrollListener(new RecyclerView.OnScrollListener()
{
    int lastVisibleItem;
    @Override
    public void onScrollStateChanged(RecyclerView recyclerView,
                                     int newState) {
        super.onScrollStateChanged(recyclerView, newState);
        if (newState == SCROLL_STATE_IDLE
                && lastVisibleItem + 1 >= mRecyclerViewAdapter.getItemCount()
                && mRecyclerViewAdapter.getItemCount() < mRecyclerViewAdapter.getTotalDataCount()) {
            loadMoreData(mRecyclerViewAdapter.getStart());
        }
    }

    @Override
    public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
        super.onScrolled(recyclerView, dx, dy);
        LinearLayoutManager layoutManager = (LinearLayoutManager) mRecyclerView.getLayoutManager();
        lastVisibleItem = layoutManager.findLastVisibleItemPosition();

    }
});

protected abstract void loadMoreData(int start);


MainFragment.java extentds MainFragmentBase.java



@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    new FetchItemTask(0).executeOnExecutor(AsyncTask.SERIAL_EXECUTOR);
}

@Override
protected void loadMoreData(int start) {
    new FetchItemTask(start).executeOnExecutor(AsyncTask.SERIAL_EXECUTOR);
}

private class FetchItemTask extends AsyncTask<Void, Void, ArrayList<MainItem>> {
    private int start=0;

    public FetchItemTask(int s){
        start=s;
    }


    @Override
    protected ArrayList<MainItem> doInBackground(Void... params) {
        if(start<mRecyclerViewAdapter.getStart()){
            return null
        }
        DoubanFetchrMain df=new DoubanFetchrMain(0,start);
        mRecyclerViewAdapter.setTotalCount(df.getTotalCount());
        return df.fetchItems_SubjectInTheaters();
    }

    @Override
    protected void onPostExecute(ArrayList<MainItem> galleryItems){
        mItems = galleryItems;
        if(mItems==null){
            return;
        }
        mRecyclerViewAdapter.addMoreData(mItems,start);

    }

}
```

## 注意点

点击图片进入电影详细页面时, 如果在详细页面图片未加载时返回, 应用会crash.<br>
应该是此时ImageView已经被销毁了, 所以就造成NullPointerException.

官方说已经解决这个问题, 但是下面这写法还是会crash.

```java
Glide.with(getActivity())
     .load(url)
     .diskCacheStrategy(DiskCacheStrategy.ALL)
     .centerCrop()
     .into(imageMovie);
```

更改为

```java
onCreate(...){
mRequestManager=Glide.with(getActivity());
}

...
mRequestManager
           .load(url)
           .diskCacheStrategy(DiskCacheStrategy.ALL)
           .centerCrop()
           .into(imageMovie);
```

这样就不会出问题.
