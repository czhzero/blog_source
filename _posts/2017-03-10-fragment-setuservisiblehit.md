---
title: Fragment的setUserVisibleHint详解
date: 2017-03-10 11:06:12
categories: Android
tags: setUserVisibleHint
---



Android应用开发过程中，ViewPager同时加载多个fragment，以实现多tab页面快速切换, 但是fragment初始化时若加载的内容较多，就可能导致整个应用启动速度缓慢，影响用户体验。
为了提高用户体验，我们会使用一些懒加载方案，实现加载延迟。这时我们会用到getUserVisibleHint()与setUserVisibleHint()这两个方法。

<!-- more -->

```
/**
*
* @param isVisibleToUser true if this fragment's UI is currently visible to the user (default),
*                        false if it is not.
*/
public void setUserVisibleHint(boolean isVisibleToUser) {
   if (!mUserVisibleHint && isVisibleToUser && mState < STARTED) {
       mFragmentManager.performPendingDeferredStart(this);
   }
   mUserVisibleHint = isVisibleToUser;
   mDeferStart = !isVisibleToUser;
}

/**
 * @return The current value of the user-visible hint on this fragment.
 * @see #setUserVisibleHint(boolean)
 */
public boolean getUserVisibleHint() {
    return mUserVisibleHint;
}

```

从上述源码注释我们可以看出，当fragment被用户可见时，setUserVisibleHint()会调用且传入true值，当fragment不被用户可见时，setUserVisibleHint()则得到false值。而在传统的fragment生命周期里也看不到这个函数。

![这里写图片描述](http://img.blog.csdn.net/20160428203327414)

那么，问题来了，

> 1. fragment是如何知道自己时候用户可见？
> 2. setUserVisibleHint() 在上图所示fragment的生命周期的什么位置?

先说结论,

> 1. viewpager监听切换tab事件，tab切换一次，执行一次setUserVisibleHint()方法
> 2. setUserVisibleHint() 在 上图所示fragment所有生命周期之前，无论viewpager是在activity哪个生命周期里初始化。
> 3. activity生命周期 和 fragment生命周期 时序并不是按序来的，也就是说fragment的oncreate方法时序并不一定在activity的oncreate方法之后。

具体原因，我们从应用场景开始一点一点的分析。

### Theme 1.  我们的应用场景

```
public class MainActivity extends FragmentActivity {

    private ViewPager viewPager;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        viewPager = (ViewPager) findViewById(R.id.viewpager);
        vpOrder.setAdapter(new MainFragmentPagerAdapter(getSupportFragmentManager()));
        vpOrder.setOffscreenPageLimit(5);
        vpOrder.setCurrentItem(0);

    }
｝

```

### Theme 2. ViewPager ,FragmentPagerAdapter 

- `/frameworks/base/core/java/com/android/internal/widget/ViewPager.java`


```
//每次切换ViewPager的Tab时调用的方法
void populate(int newCurrentItem) {

        mAdapter.startUpdate(this);

        //......
        addNewItem(mCurItem, curIndex);
        // mCurItem 为当前可见Fragment
        // 调用setUserVisibleHint(true)
        mAdapter.setPrimaryItem(this, mCurItem, curItem != null ? curItem.object : null); 

        mAdapter.finishUpdate(this);

        //.....

 }


ItemInfo addNewItem(int position, int index) {
        ItemInfo ii = new ItemInfo();
        ii.position = position;
        //初始化fragment, 调用setUserVisibleHint(false)
        ii.object = mAdapter.instantiateItem(this, position);
        ii.widthFactor = mAdapter.getPageWidth(position);
        if (index < 0 || index >= mItems.size()) {
            mItems.add(ii);
        } else {
            mItems.add(index, ii);
        }
        return ii;
}

```

- `/frameworks/support/v4/java/android/support/v4/app/FragmentPagerAdapter.java`
 

```
	@Override
public Object instantiateItem(ViewGroup container, int position) {
    if (mCurTransaction == null) {
        mCurTransaction = mFragmentManager.beginTransaction();
    }

    final long itemId = getItemId(position);

    // Do we already have this fragment?
    String name = makeFragmentName(container.getId(), itemId);
    Fragment fragment = mFragmentManager.findFragmentByTag(name);
    if (fragment != null) {
        if (DEBUG) Log.v(TAG, "Attaching item #" + itemId + ": f=" + fragment);
        mCurTransaction.attach(fragment);
    } else {
        fragment = getItem(position);
        if (DEBUG) Log.v(TAG, "Adding item #" + itemId + ": f=" + fragment);
        //将fragment添加到FragmentManager里面
        mCurTransaction.add(container.getId(), fragment,
                makeFragmentName(container.getId(), itemId));
    }
    if (fragment != mCurrentPrimaryItem) {
        fragment.setMenuVisibility(false);
        //我们要找的方法在这里
        fragment.setUserVisibleHint(false); 
    }

    return fragment;
}

@Override
public void setPrimaryItem(ViewGroup container, int position, Object object) {
    Fragment fragment = (Fragment)object;
    if (fragment != mCurrentPrimaryItem) {
        if (mCurrentPrimaryItem != null) {
            mCurrentPrimaryItem.setMenuVisibility(false);
            mCurrentPrimaryItem.setUserVisibleHint(false);
        }
        if (fragment != null) {
	        //我们要找的方法在这里
            fragment.setMenuVisibility(true);
            fragment.setUserVisibleHint(true);
        }
        mCurrentPrimaryItem = fragment;
    }
}



```

### Theme 3. Activity  , FragmentManager


- `/frameworks/base/core/java/android/app/Activity.java`


```
protected void onCreate(@Nullable Bundle savedInstanceState) {
    if (DEBUG_LIFECYCLE) Slog.v(TAG, "onCreate " + this + ": " + savedInstanceState);
    if (mLastNonConfigurationInstances != null) {
        mFragments.restoreLoaderNonConfig(mLastNonConfigurationInstances.loaders);
    }
    if (mActivityInfo.parentActivityName != null) {
        if (mActionBar == null) {
            mEnableDefaultActionBarUp = true;
        } else {
            mActionBar.setDefaultDisplayHomeAsUpEnabled(true);
        }
    }
    if (savedInstanceState != null) {
        Parcelable p = savedInstanceState.getParcelable(FRAGMENTS_TAG);
        mFragments.restoreAllState(p, mLastNonConfigurationInstances != null
                ? mLastNonConfigurationInstances.fragments : null);
    }
    //分发fragment的onCreate()事件
    mFragments.dispatchCreate(); 
    getApplication().dispatchActivityCreated(this, savedInstanceState);
    if (mVoiceInteractor != null) {
        mVoiceInteractor.attachActivity(this);
    }
    mCalled = true;
}

```

- `/frameworks/support/v4/java/android/support/v4/app/FragmentManager.java`

```

//分发onCreate事件函数
public void dispatchCreate() {
    mStateSaved = false;
    moveToState(Fragment.CREATED, false);
}

//moveToState 重载 1
void moveToState(int newState, boolean always) {
    moveToState(newState, 0, 0, always);
}

//moveToState 重载 2
void moveToState(int newState, int transit, int transitStyle, boolean always) {
    if (mHost == null && newState != Fragment.INITIALIZING) {
        throw new IllegalStateException("No host");
    }

    if (!always && mCurState == newState) {
        return;
    }

    mCurState = newState;


    //若mActive为null,就算Activtiy里面调用了dispatchOnCreate()也不会执行Fragment
    //的OnAttach和onCreate等方法。

	//只有mActive非null,即addFragment()执行后，才会真正进入到生命周期。
	//而根据FragmentPagerAdapter可知，只有当viewpager调用setAdapter方法，才会添加fragment到FramentManager。

    //执行setAdapter的时候，会调用setUserVisibleHint()方法，并且，只有当setAdapter方法执行完之后，才会进入到Fragment到生命周期，因此setUserVisibleHint()方法在所有生命周期之前被调用。
    

    if (mActive != null) {
        boolean loadersRunning = false;
        for (int i=0; i<mActive.size(); i++) {
            Fragment f = mActive.get(i);
            if (f != null) {
                moveToState(f, newState, transit, transitStyle, false);
                if (f.mLoaderManager != null) {
                    loadersRunning |= f.mLoaderManager.hasRunningLoaders();
                }
            }
        }

        if (!loadersRunning) {
            startPendingDeferredFragments();
        }

        if (mNeedMenuInvalidate && mHost != null && mCurState == Fragment.RESUMED) {
            mHost.onSupportInvalidateOptionsMenu();
            mNeedMenuInvalidate = false;
        }
    }
}

//moveToState 重载 3
void moveToState(Fragment f, int newState, int transit, int transitionStyle,
         
	     //...

         f.onAttach();

         //...

         f.performOnCreate();
          
         //其他生命周期
 }


//添加fragment到FragmentManager
public void addFragment(Fragment fragment, boolean moveToStateNow) {
    if (mAdded == null) {
        mAdded = new ArrayList<Fragment>();
    }
    if (DEBUG) Log.v(TAG, "add: " + fragment);

    //激活fragment
    makeActive(fragment);

    if (!fragment.mDetached) {
        if (mAdded.contains(fragment)) {
            throw new IllegalStateException("Fragment already added: " + fragment);
        }
        mAdded.add(fragment);
        fragment.mAdded = true;
        fragment.mRemoving = false;
        if (fragment.mHasMenu && fragment.mMenuVisible) {
            mNeedMenuInvalidate = true;
        }
        if (moveToStateNow) {
            moveToState(fragment);
        }
    }
}


void makeActive(Fragment f) {
    if (f.mIndex >= 0) {
        return;
    }
    
    if (mAvailIndices == null || mAvailIndices.size() <= 0) {
        if (mActive == null) {
        	//激活Fragment
            mActive = new ArrayList<Fragment>();	
        }
        f.setIndex(mActive.size(), mParent);
        mActive.add(f);
        
    } else {
        f.setIndex(mAvailIndices.remove(mAvailIndices.size()-1), mParent);
        mActive.set(f.mIndex, f);
    }
    if (DEBUG) Log.v(TAG, "Allocated fragment index " + f);
}

```