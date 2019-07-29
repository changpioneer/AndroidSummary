

三个fragment通过replace方法相互切换，从A切换到B时，A会先销毁再创建B，再切换到A时，B先销毁再创建A；

```java
getSupportFragmentManager().beginTransaction().replace(R.id.content, fragmentA).commit();
```

三个fragment通过add方法相互切换，从A切换到B时，A不会变化，创建B；再切换到A时，B也不会变化，然后创建A；其实这三个fragment创建后声明周期就不会变化，他们是叠加在一起的。


```java
public class MainActivity extends FragmentActivity {
​
​
    private FragmentA mFragmentA;
    private FragmentB mFragmentB;
    private FragmentC mFragmentC;
​
​
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
​
​
        mFragmentA = new FragmentA();
        mFragmentB = new FragmentB();
        mFragmentC = new FragmentC();
        FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
        transaction.add(R.id.content, mFragmentA).commit();
    }
​
​
    public void onClick(View view) {
        FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
        switch (view.getId()) {
            case R.id.fragmentA_btn:
                if (!mFragmentA.isAdded()) {
                    transaction.add(R.id.content, mFragmentA);
                }
                transaction.show(mFragmentA);
                transaction.hide(mFragmentB);
                transaction.hide(mFragmentC);
                break;
            case R.id.fragmentB_btn:
                if (!mFragmentB.isAdded()) {
                    transaction.add(R.id.content, mFragmentB);
                }
                transaction.show(mFragmentB);
                transaction.hide(mFragmentA);
                transaction.hide(mFragmentC);
                break;
            case R.id.fragmentC_btn:
                if (!mFragmentC.isAdded()) {
                    transaction.add(R.id.content, mFragmentC);
                }
                transaction.show(mFragmentC);
                transaction.hide(mFragmentB);
                transaction.hide(mFragmentA);
                break;
            default:
                break;
        }
        transaction.commit();
    }
}
```