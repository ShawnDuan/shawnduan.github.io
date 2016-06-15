---
layout: post
title:  "Nested Fragment Management"
date:   2016-03-30 21:30:36 -0700
comments: true
categories: [Android, Fragment]
header-img: "img/nested-bg.jpg"
---

Since Android 4.2 (API 17), [Nested Fragment](http://developer.android.com/about/versions/android-4.2.html#NestedFragments) becames available, which is also available in Android Support Library.
A handy method is provided as `getChildFragmentManager()`, which will return a private FragmentManager for placing and managing Fragments inside of this Fragment.

This gives us an option to maintain multiple Fragments inside one Fragments, and organize them nicely. For example, we can create a MainFragment in MainActivity, then place sub Fragments inside MainFragment.

**MainActivity**

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setContentView(R.layout.main);
   mFragmentManager = getSupportFragmentManager();
   if (savedInstanceState == null) {
       setFragment(new SignInFragment());
   }
}
```

**main.xml**

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
   android:id="@+id/main_frag_container"
   android:layout_width="match_parent"
   android:layout_height="match_parent" />
```

**IFragmentStackHolder.java**

```java
public interface IFragmentStackHolder {
    /**
     * @param containerId   in which container you want to replace your new fragment to.
     * @param frag          the new fragment used to replace
     * @param eltrans       a list of hints for cool transitions
     */
    void replaceFragment(int containerId, Fragment frag, ArrayList<Pair<View, String>> eltrans);
}
```

**MainFragment.java**

```java
public class MainFragment extends Fragment implements IFragmentStackHolder {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        super.onCreateView(inflater, container, savedInstanceState);
        mFragView = inflater.inflate(R.layout.fragment_main, container, false);
        mFragmentManager = getChildFragmentManager();
        mFragmentManager.addOnBackStackChangedListener(new FragmentManager.OnBackStackChangedListener() {
            @Override
            public void onBackStackChanged() {
                Log.d(TAG, "onBackStackChanged()");
            }
        });
        return mFragView;
    }
    
    // set the first inner fragment in mainFragment
    private void setFragment(Fragment frag) {
        FragmentTransaction transaction = mFragmentManager.beginTransaction();
        // clear the back stack
        mFragmentManager.popBackStack(null, FragmentManager.POP_BACK_STACK_INCLUSIVE);
        
        String backStackName = frag.getBackStackName();
        transaction.replace(R.id.inner_frag_container, frag, backStackName);
        transaction.addToBackStack(backStackName);
        transaction.commit();
    }
    
    @Override
    public void replaceFragment(int containerId, Fragment frag, ArrayList<Pair<View, String>> eltrans) {
        FragmentTransaction transaction = mFragmentManager.beginTransaction();
        
        String backStackName = frag.getBackStackName();
        transaction.replace(containerId, frag, backStackName);
        transaction.addToBackStack(backStackName);
        transaction.setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN);
        if (eltrans != null) {
            for (Pair<View, String> et : eltrans) {
                transaction.addSharedElement(et.first, et.second);
            }
        }
        transaction.commit();
    }
}
```

**fragment_main.xml**

```xml
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools" 
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent" 
    android:layout_height="match_parent"
    android:fitsSystemWindows="true" 
    tools:openDrawer="start">

    <include layout="@layout/content_frags" />

    <android.support.design.widget.NavigationView android:id="@+id/nav_view"
        android:layout_width="wrap_content" 
        android:layout_height="match_parent"
        android:layout_gravity="start" 
        android:fitsSystemWindows="true"
        app:headerLayout="@layout/nav_header_main" 
        app:menu="@menu/activity_main_drawer" />

</android.support.v4.widget.DrawerLayout>
```

**content_frags.xml**

```xml
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/inner_frag_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    tools:showIn="@layout/app_bar_main" />
```

**Notice that, two container IDs are very important, but easily confused.**

* `"@+id/main_frag_container"` in `main.xml`
	* This one is from MainActivity, which contains MainFragment. Here is how you refer to the MainFragment from MainActivity:  `mainActivity.getSupportFragmentManager().findFragmentById(R.id.main_frag_container);`
    
* `"@+id/inner_frag_container"` in `content_frags.xml`
	* This container is from MainFragment, which contains inner Fragments. If you are trying to manage inner fragments, you should `add/replace` in `R.id.inner_frag_container`, like what is done in `setFragment()` and `replaceFragment()` of `MainFragment.java`

From any given inner Fragment, or any given view of the inner Fragment, we are able to call `replaceFragment()` from `IFragmentStackHolder`, like this:

```java
if (getParentFragment() instanceof IFragmentStackHolder) {
    Fragment newFrag = new SomeInnerFragment();
    IFragmentStackHolder fsh = (IFragmentStackHolder) getParentFragment();
    fsh.replaceFragment(R.id.inner_frag_container, tagApplyFragment, null);
}
```

> **Note:** The hierarchy is maintained like this: 
`MainActivity --> MainFragment --> InnerFragment`, Where `InnerFragment` can be either a normal single Fragment or another Fragment-container, by implementing `IFragmentStackHolder`.

## Conclusion
Activity transitions could be expensive sometime. Thus, many teams are believing in the philosophy that you should consider using Fragments whenever it's possible, instead of using Activity. Activity or Fragment? To discuss this dillema is not the intent in this airtical. However, once you decide to follow this "Fragment-ism" philosophy, you should keep your fragment backstack hierarchical and organized -- using nested fragments and `getChildFragmentManager()` could be a nice choice.
