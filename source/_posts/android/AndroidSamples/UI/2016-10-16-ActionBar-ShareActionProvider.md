---
author: ivyxjc
date: 2016-10-16
title: ActionBar ShareActionProvider
category: Android
tags: [android,android_UI,android_fragment]
keywords:
description: ShareActionProvider 可以非常方便地提供分享功能.
toc: true
---


## 添加share按钮

添加share按钮的主要步骤:
1. 在ActionBar中添加share按钮
2. 从item中获取ShareActionProvider
```java
ShareActionProvider`<br> `mShareActionProvider=(ShareActionProvider) MenuItemCompat.getActionProvider(shareItem);
```
3. 向`ShareActionProvider`中添加`itent`


```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:support="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/share_item"
        android:title="@string/menu_share"

        support:actionProviderClass="android.support.v7.widget.ShareActionProvider"
        support:showAsAction="always"
        />
</menu>
```

```java
/MainActivity.java

@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.main_menu,menu);

    MenuItem shareItem=menu.findItem(R.id.share_item);

    mShareActionProvider=(ShareActionProvider) MenuItemCompat.getActionProvider(shareItem);

    int currentViewPagerItem=((ViewPager)findViewById(R.id.viewPager)).getCurrentItem();
    setShareItem(currentViewPagerItem);
    return super.onCreateOptionsMenu(menu);
}


public void setShareItem(int i){
    if(mShareActionProvider!=null){
        ContentItem item=mItems.get(i);

        Intent intent=item.getShareIntent(this);

        mShareActionProvider.setShareIntent(intent);
    }
  }

```


```java
public Intent getShareIntent(Context context){
    Intent intent=new Intent(Intent.ACTION_SEND);

    switch (contentType) {
        case CONTENT_TYPE_IMAGE:
            intent.setType("image/jpg");
            intent.putExtra(Intent.EXTRA_STREAM, getContentUri());

            break;

        case CONTENT_TYPE_TEXT:
            intent.setType("text/plain");
            intent.putExtra(Intent.EXTRA_TEXT, context.getString(contentResourceId));
            break;
    }

    return intent;
}
```

```java
\ContentItem
public Uri getContentUri(){
    if(!TextUtils.isEmpty(contentAssetFilePath)){
        return Uri.parse(ShareProvider.CONTENT_URI+contentAssetFilePath);
    }else{
        return null;
    }
}
```

## 注意点

### 确保ShareActionProvider和所在的ViewPager的`CurrentItem()`对应

很容易被`onCreateOptionsMenu()`中下列代码迷惑, 以为ShareActionProvider已经和`CurrentItem()`对应了.

```java
int currentViewPagerItem=((ViewPager)findViewById(R.id.viewPager)).getCurrentItem();
setShareItem(currentViewPagerItem);
```

事实上并没有, `onCreateOptionsMenu`方法只会在初始ActionBar时调用, 且也不会设置监听. 所以ShareActionProvider总是设置在了第一个item的intent中.

所以需要添加以下代码, 确保切换页面后, ShareActionProvider和item仍是正确对应的.

```java
mViewPager.setOnPageChangeListener(new ViewPager.OnPageChangeListener() {
      @Override
      public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
      }
      @Override
      public void onPageSelected(int position) {
          setShareItem(position);
      }
      @Override
      public void onPageScrollStateChanged(int state) {
      }
    });
```


### 如何获取图片uri


获取`/res/drawable`中图片的uri可以使用以下方法:

```java
Uri imageUri = Uri.parse(ContentResolver.SCHEME_ANDROID_RESOURCE +
 "://" + getResources().getResourcePackageName(R.drawable.ic_launcher)
 + '/' + getResources().getResourceTypeName(R.drawable.ic_launcher) + '/'
 +  getResources().getResourceEntryName(R.drawable.ic_launcher) );
 ```

 等价于

 ```
Uri uri = Uri.parse("android.resource://your.package.here/drawable/image_name");
 ```

如果不正确,可以调用第一个方法,再使用Log自行查看准确的字符串.
