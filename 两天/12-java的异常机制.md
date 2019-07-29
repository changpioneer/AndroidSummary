
```java
public class ExceptionTest {
    public static void main(String[] args) {
        System.out.println(test());//2 3 3 
    }


    public static int test() {
        try {
            int i = 1 / 0;
            //上面报异常，return不会执行
            return getNum1();
        } catch (Exception e) {
            int a = getNum2();//2
            return a;//有finally时会将结果寄存起来
        } finally {
            int b = getNum3();//3
            return b;//3
        }
    }


    public static int getNum1() {
        System.out.println(1);
        return 1;
    }


    public static int getNum2() {
        System.out.println(2);
        return 2;
    }


    public static int getNum3() {
        System.out.println(3);
        return 3;
    }
}
```