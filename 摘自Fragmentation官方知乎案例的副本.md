> 先跟着原作者把案例写一遍,再去放飞自我  


## 1 集成Fragmentation  

```java
// Fragmentation  https://github.com/YoKeyword/Fragmentation
compile 'me.yokeyword:fragmentation:1.3.6'
compile 'me.yokeyword:fragmentation-swipeback:1.3.6'
compile 'me.yokeyword:eventbus-activity-scope:1.1.0'
compile 'org.greenrobot:eventbus:3.0.0'
```  

## 2 在Application中注册Fragmentation  

```java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        // 建议在Application里初始化
        Fragmentation.builder()
                // BUBBLE显示悬浮球 ; SHAKE: 摇一摇唤出 ;  NONE：隐藏
                .stackViewMode(Fragmentation.BUBBLE)
                .debug(BuildConfig.DEBUG)
                .install();
    }
}


<application
        android:name=".MyApplication"
        ... >
</application>
```  

## 3 创建BaseFragment,监听返回键退出      

```java  
public class BaseMainFragment extends SupportFragment {
    protected OnBackToFirstListener _mBackToFirstListener;

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        if (context instanceof OnBackToFirstListener) {
            _mBackToFirstListener = (OnBackToFirstListener) context;
        } else {
            throw new RuntimeException(context.toString()
                    + " must implement OnBackToFirstListener");
        }
    }

    @Override
    public void onDetach() {
        super.onDetach();
        _mBackToFirstListener = null;
    }

    // 对返回键进行监听
    @Override
    public boolean onBackPressedSupport() {
        // 当前栈中的Fragment个数大于1
        if(getChildFragmentManager().getBackStackEntryCount()>1){
            popChild();
        }else{
            // 当这个Fragment是第一个导航页,直接退出
            if(this instanceof ZhihuFirstFragment){
                _mActivity.finish();
            }else{
                // 返回到第一个导航页
                _mBackToFirstListener.OnBackToFirstFragment();
            }
        }
        return true;
    }

    public interface OnBackToFirstListener {
        void OnBackToFirstFragment();
    }
}
```   

## 4 创建4个Fragment,以及他们的子Fragment  

```java
/**
 * 第一个导航页
 *
 * Created by wxyass on 2018/8/17.
 */
public class ZhihuFirstFragment extends BaseMainFragment{

    public static ZhihuFirstFragment newInstance(){
        Bundle args = new Bundle();
        ZhihuFirstFragment fragment = new ZhihuFirstFragment();
        fragment.setArguments(args);
        return fragment;

    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_first,container,false);
        return view;
    }

    @Override
    public void onLazyInitView(@Nullable Bundle savedInstanceState) {
        super.onLazyInitView(savedInstanceState);
        if(findChildFragment(ZhihuFirstFragment.class)==null){
            loadRootFragment(R.id.fl_first_container, FirstHomeFragment.newInstance());
        }
    }
}


// fragment_first.xml  
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout android:id="@+id/fl_first_container"
             xmlns:android="http://schemas.android.com/apk/res/android"
             android:layout_width="match_parent"
             android:layout_height="match_parent">
</FrameLayout>  




/**
 * 第一个导航页的子Fragment
 * 
 * Created by wxyass on 2018/8/17.
 */

public class FirstHomeFragment extends BaseMainFragment{

    public static FirstHomeFragment newInstance(){
        Bundle args = new Bundle();
        FirstHomeFragment fragment = new FirstHomeFragment();
        fragment.setArguments(args);
        return fragment;

    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_first_home,container,false);
        return view;
    }

    @Override
    public void onLazyInitView(@Nullable Bundle savedInstanceState) {
        super.onLazyInitView(savedInstanceState);
        if(findChildFragment(FirstHomeFragment.class)==null){
            // loadRootFragment(R.id.fl_first_container,ZhihuFirstFragment.newInstance());
        }
    }
}


// fragment_first_home.xml  
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout android:id="@+id/fl_first_container"
             xmlns:android="http://schemas.android.com/apk/res/android"
             android:layout_width="match_parent"
             android:background="@color/bg_app"
             android:layout_height="match_parent">
</FrameLayout>
```  

## 5 创建底部导航的自定义BottomBar  

```java
public class BottomBar extends LinearLayout {

}
```  

## 6 为MainActivity布局中的fl_container加载Fragment   

使用loadMultipleRootFragment(...)方法   

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    SupportFragment firstFragment = findFragment(ZhihuFirstFragment.class);
    if (firstFragment == null) {
        mFragments[FIRST] = ZhihuFirstFragment.newInstance();
        mFragments[SECOND] = ZhihuSecondFragment.newInstance();
        mFragments[THIRD] = ZhihuThirdFragment.newInstance();
        mFragments[FOURTH] = ZhihuFourthFragment.newInstance();

        loadMultipleRootFragment(R.id.fl_container, FIRST,
                mFragments[FIRST],
                mFragments[SECOND],
                mFragments[THIRD],
                mFragments[FOURTH]
        );

    } else {
        // 这里库已经做了Fragment恢复,所有不需要额外的处理了, 不会出现重叠问题

        // 这里我们需要拿到mFragments的引用
        mFragments[FIRST] = firstFragment;
        mFragments[SECOND] = findFragment(ZhihuSecondFragment.class);
        mFragments[THIRD] = findFragment(ZhihuThirdFragment.class);
        mFragments[FOURTH] = findFragment(ZhihuFourthFragment.class);
    }

    initView();
}
```  

## 7 为底部BottomBar设置数据关联Fragment  

```java
private void initView() {
    mBottomBar = (BottomBar) findViewById(R.id.bottomBar);

    mBottomBar.addItem(new BottomBarTab(this, R.drawable.ic_home_white_24dp))
            .addItem(new BottomBarTab(this, R.drawable.ic_discover_white_24dp))
            .addItem(new BottomBarTab(this, R.drawable.ic_message_white_24dp))
            .addItem(new BottomBarTab(this, R.drawable.ic_account_circle_white_24dp));

    mBottomBar.setOnTabSelectedListener(new BottomBar.OnTabSelectedListener() {
        @Override
        public void onTabSelected(int position, int prePosition) {
            showHideFragment(mFragments[position],mFragments[prePosition]);
        }

        @Override
        public void onTabUnselected(int position) {

        }

        @Override
        public void onTabReselected(int position) {
            SupportFragment currentFragment = mFragments[position];
            int count = currentFragment.getChildFragmentManager().getBackStackEntryCount();

            // 如果不在该类别Fragment的主页,则回到主页;
            if(count>1){
                if(currentFragment instanceof ZhihuFirstFragment){
                    currentFragment.popToChild(FirstHomeFragment.class,false);
                }else if(currentFragment instanceof ZhihuSecondFragment){
                    currentFragment.popToChild(ViewPagerFragment.class,false);
                }else if(currentFragment instanceof ZhihuThirdFragment){
                    currentFragment.popToChild(ShopFragment.class,false);
                }else if(currentFragment instanceof ZhihuFourthFragment){
                    currentFragment.popToChild(MeFragment.class,false);
                }
                return;

            }


            // 这里推荐使用EventBus来实现 -> 解耦
            if(count == 1){
                // 在FirstPagerFragment中接收, 因为是嵌套的孙子Fragment 所以用EventBus比较方便
                // 主要为了交互: 重选tab 如果列表不在顶部则移动到顶部,如果已经在顶部,则刷新
                EventBusActivityScope.getDefault(MainActivity.this).post(new TabSelectedEvent(position));
            }


        }
    });
}
```  
## 8 为了监听Fragment的返回键,MainActivity需实现OnBackToFirstListener  

```java  
public class MainActivity extends SupportActivity implements BaseMainFragment.OnBackToFirstListener {

	@Override
    public void OnBackToFirstFragment() {
        // 回到第一个导航页
        mBottomBar.setCurrentItem(0);
    }
}
``` 
# 9 ...

# 源码下载   
<https://github.com/wxyass/FragmentationTest>  