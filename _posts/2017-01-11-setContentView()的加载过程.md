---
layout: post
title: setContentView()的加载过程
categories: [Android, ]
description: setContentView()的加载过程
keywords: Android, 
---
### setContentView()的加载过程
在我们的项目中，每一个布局都是使用setContentView(R.layout.activity_main);来进行加载布局的，那么，setContentView()加载布局的过程是怎样的呢。 现在我们来跟踪setContentView()的加载流程。 首先我们会看到下面的代码：

```
   /**
     * Set the activity content from a layout resource.  The resource will be
     * inflated, adding all top-level views to the activity.
     *
     * @param layoutResID Resource ID to be inflated.
     *
     * @see #setContentView(android.view.View)
     * @see #setContentView(android.view.View, android.view.ViewGroup.LayoutParams)
     */
    public void setContentView(@LayoutRes int layoutResID) {
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }
```
我们可以看到,我们是通过 getWindow() 获得 setContentView() 的，而 getWindow() 是在 Window 类中。
所以：我们的 setContentView() 实际上是在Window中。但是，Window是一个抽象类，那么他的实现类是什么呢？
Window 的实现类就是 PhoneWindow() ,所以，我们需要去 PhoneWindow() 中寻找我们的 setContentView()实现过程。

```
// This is the view in which the window contents are placed. It is either mDecor itself, or a child of mDecor where the contents go.

//变量申明
ViewGroup mContentParent;

@Override
    public void setContentView(int layoutResID) {
        // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
        // decor, when theme attributes and the like are crystalized. Do not check the feature
        // before this happens.
        if (mContentParent == null) {
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
```
我们可以看到的流程是首先判断mContentParent是不是等于null，是的话就加在installDecor()方法，不是的话再判断是否使用了这个FEATURE_CONTENT_TRANSITIONS的flag，
如果没有的话则mContentParent删除其中所有的view。接下来再判断是否需要通过LayoutInflater.inflate将我们传入的layout放置到mContentParent中。

installDecor()实现过程：
```
//申明变量
// This is the top-level view of the window, containing the window decor.
private DecorView mDecor;

private void installDecor() {
        mForceDecorInstall = false;
        if (mDecor == null) {
            mDecor = generateDecor(-1);
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
            //省略无关代码...
        } else {
            mDecor.setWindow(this);
        }
        if (mContentParent == null) {
            mContentParent = generateLayout(mDecor);

            //省略无关代码...
            } else {
                mTitleView = (TextView) findViewById(R.id.title);
                //设置是否需要标题
                if (mTitleView != null) {
                    if ((getLocalFeatures() & (1 << FEATURE_NO_TITLE)) != 0) {
                        final View titleContainer = findViewById(R.id.title_container);
                        if (titleContainer != null) {
                            titleContainer.setVisibility(View.GONE);
                        } else {
                            mTitleView.setVisibility(View.GONE);
                        }
                        mContentParent.setForeground(null);
                    } else {
                        mTitleView.setText(mTitle);
                    }
                }
            }

            //省略无关代码...

        }
    }
```

首先通过generateDecor(-1)来初始化一下mDecor，这个mDecor是DecorView的对象

 DecorView() 类：
 ```
 protected DecorView generateDecor(int featureId) {

         Context context;
         //没什么鸟用的无关代码，省略...
         return new DecorView(context, featureId, this, getAttributes());
     }
 ```
 generateDecor():
 ```
   //设置当前activity的theme
         TypedArray a = getWindowStyle();

         //省略无关代码...
         //首先通过WindowStyle中设置的各种属性，对Window进行requestFeat
         mIsFloating = a.getBoolean(R.styleable.Window_windowIsFloating, false);
         int flagsToUpdate = (FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR)
                 & (~getForcedWindowFlags());
         if (mIsFloating) {
             setLayout(WRAP_CONTENT, WRAP_CONTENT);
             setFlags(0, flagsToUpdate);
         } else {
             setFlags(FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR, flagsToUpdate);
         }

         //省略无关代码...
         //根据feature来加载对应的xml布局文件
         int layoutResource;
         int features = getLocalFeatures();
         // System.out.println("Features: 0x" + Integer.toHexString(features));
         if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
             layoutResource = R.layout.screen_swipe_dismiss;
         } else if ((features & ((1 << FEATURE_LEFT_ICON) | (1 << FEATURE_RIGHT_ICON))) != 0) {
             if (mIsFloating) {
                 TypedValue res = new TypedValue();
                 getContext().getTheme().resolveAttribute(
                         R.attr.dialogTitleIconsDecorLayout, res, true);
                 layoutResource = res.resourceId;
             } else {
                 layoutResource = R.layout.screen_title_icons;
             }
             // XXX Remove this once action bar supports these features.
             removeFeature(FEATURE_ACTION_BAR);
             // System.out.println("Title Icons!");
         } else if ((features & ((1 << FEATURE_PROGRESS) | (1 << FEATURE_INDETERMINATE_PROGRESS))) != 0
                 && (features & (1 << FEATURE_ACTION_BAR)) == 0) {
             // Special case for a window with only a progress bar (and title).
             // XXX Need to have a no-title version of embedded windows.
             layoutResource = R.layout.screen_progress;
             // System.out.println("Progress!");
         } else if ((features & (1 << FEATURE_CUSTOM_TITLE)) != 0) {
             // Special case for a window with a custom title.
             // If the window is floating, we need a dialog layout
             if (mIsFloating) {
                 TypedValue res = new TypedValue();
                 getContext().getTheme().resolveAttribute(
                         R.attr.dialogCustomTitleDecorLayout, res, true);
                 layoutResource = res.resourceId;
             } else {
                 layoutResource = R.layout.screen_custom_title;
             }
             // XXX Remove this once action bar supports these features.
             removeFeature(FEATURE_ACTION_BAR);
         } else if ((features & (1 << FEATURE_NO_TITLE)) == 0) {
             // If no other features and not embedded, only need a title.
             // If the window is floating, we need a dialog layout
             if (mIsFloating) {
                 TypedValue res = new TypedValue();
                 getContext().getTheme().resolveAttribute(
                         R.attr.dialogTitleDecorLayout, res, true);
                 layoutResource = res.resourceId;
             } else if ((features & (1 << FEATURE_ACTION_BAR)) != 0) {
                 layoutResource = a.getResourceId(
                         R.styleable.Window_windowActionBarFullscreenDecorLayout,
                         R.layout.screen_action_bar);
             } else {
                 layoutResource = R.layout.screen_title;
             }
             // System.out.println("Title!");
         } else if ((features & (1 << FEATURE_ACTION_MODE_OVERLAY)) != 0) {
             layoutResource = R.layout.screen_simple_overlay_action_mode;
         } else {
             // Embedded, so no decoration is needed.
             layoutResource = R.layout.screen_simple;
             // System.out.println("Simple!");
         }


         //将上面获取到的xml布局加载到mDecor对象中
         mDecor.startChanging();
         mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);

         ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
         if (contentParent == null) {
             throw new RuntimeException("Window couldn't find content container view");
         }

         if ((features & (1 << FEATURE_INDETERMINATE_PROGRESS)) != 0) {
             ProgressBar progress = getCircularProgressBar(false);
             if (progress != null) {
                 progress.setIndeterminate(true);
             }
         }

         if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
             registerSwipeCallbacks();
         }

         // Remaining setup -- of background and title -- that only applies
         // to top-level windows.
         if (getContainer() == null) {
             final Drawable background;
             if (mBackgroundResource != 0) {
                 background = getContext().getDrawable(mBackgroundResource);
             } else {
                 background = mBackgroundDrawable;
             }
             mDecor.setWindowBackground(background);

             final Drawable frame;
             if (mFrameResource != 0) {
                 frame = getContext().getDrawable(mFrameResource);
             } else {
                 frame = null;
             }
             mDecor.setWindowFrame(frame);

             mDecor.setElevation(mElevation);
             mDecor.setClipToOutline(mClipToOutline);

             if (mTitle != null) {
                 setTitle(mTitle);
             }

             if (mTitleColor == 0) {
                 mTitleColor = mTextColor;
             }
             setTitleColor(mTitleColor);
         }

         mDecor.finishChanging();

         return contentParent;
     }
 ```
上面这个方法流程我们能看出个大概，首先getWindowStyle在当前的Window的theme中获取我们的Window中定义的属性。然后就根据这些属性的值，对我们的Window进行各种requestFeature,setFlags等等。

还记得我们平时写应用Activity时设置的theme或者feature吗（全屏啥的，NoTitle等）？我们一般是不是通过XML的android:theme属性或者java的requestFeature()方法来设置的呢？譬如：
```
通过java文件设置：
requestWindowFeature(Window.FEATURE_NO_TITLE);

通过xml文件设置：
android:theme="@android:style/Theme.NoTitleBar"
```
其实我们平时 requestWindowFeature() 设置的值就是在这里通过getLocalFeature()获取的；而android:theme属性也是通过这里的getWindowStyle()获取的。
这就是为什么 requestWindowFeature() 需要设置在 setContentView() 之前。

最后通过我们得到了layoutResource，然后将它加载给mDecor对象，这个mDecor对象其实是一个FrameLayout，它的中心思想就是根据theme或者我们在setContentView之前设置的Feature来获取对应的xml布局，然后通过mLayoutInflater转化为view，赋值给mDecor对象，这些布局文件中都包含一个id为content的FrameLayout，最后将其引用返回给mContentParent。

```
//代码清单 xml布局文件，包含id为content的FrameLayout布局
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    android:orientation="vertical">
    <ViewStub android:id="@+id/action_mode_bar_stub"
              android:inflatedId="@+id/action_mode_bar"
              android:layout="@layout/action_mode_bar"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:theme="?attr/actionBarTheme" />
    <FrameLayout
         android:id="@android:id/content"
         android:layout_width="match_parent"
         android:layout_height="match_parent"
         android:foregroundInsidePadding="false"
         android:foregroundGravity="fill_horizontal|top"
         android:foreground="?android:attr/windowContentOverlay" />
</LinearLayout>
```

![https://user-gold-cdn.xitu.io/2017/3/24/a0ab3775fa793ba5fc75759b1d34267c](https://user-gold-cdn.xitu.io/2017/3/24/a0ab3775fa793ba5fc75759b1d34267c)
```
@Override
    public void setContentView(int layoutResID) {

       //我们上面分析的生成系统xml布局的流程
        if (mContentParent == null) {
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {

            //将我们自己写的xml加载到我们上面获取到的里面包含id为content的xml布局中去，并赋值给mContentParent
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
       //省略无关代码...
    }
```

### 总结：
![https://user-gold-cdn.xitu.io/2017/3/24/3576b984f1eab51c460bd1c30a1f2b49](https://user-gold-cdn.xitu.io/2017/3/24/3576b984f1eab51c460bd1c30a1f2b49)
Android中有一个成员叫Window，Window是一个抽象类，提供了绘制窗口的一组通用API，PhoneWindow继承自Window，是Window的具体实现类，PhoneWindow中有一个私有成员DecorView，这个DecorView对象是所有应用Activity页面的根View，DecorView继承自FrameLayout，在内部根据用户设置的Activity的主题（theme）对FrameLayout进行修饰，为用户提供给定Theme下的布局样式。一般情况下，DecorView中包含一个用于显示Activity标题的TitleView和一个用于显示内容的ContentView。
![https://user-gold-cdn.xitu.io/2017/3/24/a68206845af54426aba2ba46dfb53f67](https://user-gold-cdn.xitu.io/2017/3/24/a68206845af54426aba2ba46dfb53f67)
可以看出，DecorView中包含一个Vertical的LinearLayout布局文件，文件中有两个FrameLayout，上面一个FrameLayout用于显示Activity的标题，下面一个FrameLayout用于显示Activity的具体内容，也就是说，我们通过setContentView方法加载的布局文件View将显示在下面这个FrameLayout中。

首先初始化mDecor,即DecorView为FrameLayout的子类。就是我们整个窗口的根视图了。然后，根据theme中的属性值，选择合适的布局，通过infalter.inflater放入到我们的mDecor中。在这些布局中，一般会包含ActionBar，Title，和一个id为content的FrameLayout。最后，我们在Activity中设置的布局，会通过infalter.inflater压入到我们的id为content的FrameLayout中去。

