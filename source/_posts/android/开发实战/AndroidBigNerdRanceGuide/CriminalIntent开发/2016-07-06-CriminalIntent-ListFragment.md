---
layout: post
title: CriminalIntent程序中ListFragment相关内容
category: Android
tags: [android,android_adapter]
keywords:
description:
---

## 定制adapter

adapter负责：
1. 创建必要的视图对象
2. 用模型层数据填充视图对性爱那个
3. 将准备好的视图对象返回给ListView


```java
...
内部类
private class CrimeAdapter extends ArrayAdapter<Crime>{
        public CrimeAdapter(ArrayList<Crime> crimes){
            super(getActivity(),0,crimes);
        }

        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            if(convertView==null){
                convertView=getActivity()
                        .getLayoutInflater()
                        .inflate(R.layout.list_item_crime,null);
            }

            Crime c=(Crime)(getListAdapter()).getItem(position);

            TextView titleTextView=(TextView)convertView.findViewById(R.id.list_item_crime_title_TextView);
            TextView dateTextView=(TextView)convertView.findViewById(R.id.list_item_crime_date_TextView);
            CheckBox solvedCheckBox=(CheckBox)convertView.findViewById(R.id.list_item_crime_isSolved_CheckBox);

            solvedCheckBox.setChecked(c.isSolved());
            return convertView;
        }
    }

...
```
注： 由于CheckBox默认是可聚焦的，也就是点击列表向会被解读为切换CheckBox的状态，因而无法触发`onListItemClick()`方法。解决方法：

```xml
<CheckBox
        android:id="@+id/list_item_crime_isSolved_CheckBox"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:gravity="center"
    android:focusable="false"/>
```
