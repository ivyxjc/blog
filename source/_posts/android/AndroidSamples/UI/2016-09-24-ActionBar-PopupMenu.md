---
author: ivyxjc
date: 2016-09-24
title: ActionBar PopuMenu
category: Android
tags: [android,android_UI,android_fragment]
keywords:
description: popupmenu可以非常方便得实现在指定view下弹出一个菜单,实现类似ActionBar中的效果.
---

## PopupMenu
popupmenu可以非常方便得实现在指定view下弹出一个菜单,实现类似ActionBar中的效果.

```java
public void showPopupMenu(View view){
        final PopupAdapter adapter = (PopupAdapter) getListAdapter();
        final String item = (String) view.getTag();

        PopupMenu popup = new PopupMenu(getActivity(), view);

        popup.getMenuInflater().inflate(R.menu.popup, popup.getMenu());

        popup.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(MenuItem menuItem) {
                switch (menuItem.getItemId()){
                    case R.id.menu_remove:
                        adapter.remove(item);
                        return true;
                }
                return false;
            }
        });
        popup.show();
    }
```
