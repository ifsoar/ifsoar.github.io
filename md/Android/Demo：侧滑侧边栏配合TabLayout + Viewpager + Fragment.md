gradle
```
compile 'com.android.support:design:26.0.0-alpha1'
```

布局
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.soar.sccount.view.main.MainActivity">

    <android.support.v4.widget.DrawerLayout
        android:id="@+id/drawer_main_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <!-- 正文 -->
        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:orientation="vertical">
                <!-- 标题栏 -->
                <RelativeLayout
                    android:layout_width="match_parent"
                    android:layout_height="50dp"
                    android:background="@color/colorPrimaryDark">

                    <ImageView
                        android:id="@+id/btn_main_menu"
                        android:layout_width="50dp"
                        android:layout_height="50dp"
                        android:padding="10dp"
                        android:src="@mipmap/btn_main_menu" />

                    <ImageView
                        android:id="@+id/btn_main_search"
                        android:layout_width="120dp"
                        android:layout_height="50dp"
                        android:layout_alignParentRight="true"
                        android:padding="10dp"
                        android:src="@mipmap/btn_main_search" />
                </RelativeLayout>

                <android.support.design.widget.TabLayout
                    android:id="@+id/tablayout_main_layout"
                    android:layout_width="match_parent"
                    android:layout_height="40dp"
                    app:tabMode="scrollable">

                </android.support.design.widget.TabLayout>
                <!-- 正文viewpager -->
                <android.support.v4.view.ViewPager
                    android:id="@+id/viewpager_main_layout"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"/>

            </LinearLayout>

        </RelativeLayout>

        <!-- 侧边栏 -->
        <LinearLayout
            android:id="@+id/side_main_layout"
            android:layout_width="240dp"
            android:layout_height="match_parent"
            android:layout_gravity="start"
            android:background="@color/colorPrimaryDark"
            android:orientation="vertical">

        </LinearLayout>
    </android.support.v4.widget.DrawerLayout>
</RelativeLayout>
```
1、侧边栏
```java
//打开
drawerMainLayout.openDrawer(Gravity.START);  
//关闭
drawerMainLayout.closeDrawer(Gravity.START);  
//判断
if (drawerMainLayout.isDrawerOpen(Gravity.START)) {
	drawerMainLayout.closeDrawer(Gravity.START);
	return true;
}
```


  

2、Viewpager
```java
viewPagerAdapter = new MainViewPagerAdapter(getSupportFragmentManager());

viewpagerMainLayout.setAdapter(viewPagerAdapter);

viewpagerMainLayout.addOnPageChangeListener(this);
```

  

ViewPager.OnPageChangeListener
```java
@Override

public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

  

}

  

@Override

public void onPageSelected(int position) {

tablayoutMainLayout.getTabAt(position).select();

}

  

@Override

public void onPageScrollStateChanged(int state) {

  

}
```
MainViewPagerAdapter
```java
public class MainViewPagerAdapter extends FragmentPagerAdapter {

private List fragmentList;

  

public MainViewPagerAdapter(FragmentManager fm) {

super(fm);

fragmentList = new ArrayList<>();

}

  

public void updateFragmentList(List fragmentList) {

this.fragmentList = fragmentList;

notifyDataSetChanged();

}

  

@Override

public Fragment getItem(int position) {

return fragmentList.get(position);

}

  

@Override

public int getCount() {

return fragmentList.size();

}

}
```
  

```java
//设置显示页
viewpagerMainLayout.setCurrentItem(0);

//获取选中页
viewpagerMainLayout.getCurrentItem()

```
  

  

3、TabLayout

```java
tablayoutMainLayout.addOnTabSelectedListener(this);
```

 
TabLayout.OnTabSelectedListener

```java
@Override
public void onTabSelected(TabLayout.Tab tab) {
int position = (int) tab.getTag();
viewpagerMainLayout.setCurrentItem(position);
}
  
@Override
public void onTabUnselected(TabLayout.Tab tab) {
  
}
  
@Override
public void onTabReselected(TabLayout.Tab tab) {
  
}
```

  

跳转到指定
```java
tablayoutMainLayout.getTabAt(position).select();
```

  

动态添加tab
```java
public void fillTabList(TabLayout layout, List list) {

for (int i = 0; i < list.size(); i++) {

PO_Group group = list.get(i);

TabLayout.Tab tab = layout.newTab();

tab.setText(group.getGroup());

tab.setTag(i);

layout.addTab(tab);

}

}
```