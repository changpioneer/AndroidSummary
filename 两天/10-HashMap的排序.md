
使用比较器

```java
public class HashMapTest {
    public static void main(String[] args) {
        HashMap<Integer, User> map = new HashMap<Integer, User>();
        map.put(1, new User("张三", 26));
        map.put(3, new User("李四", 32));
        map.put(2, new User("王五", 23));
        System.out.println("排序前："+map);
        //TODO 对hashmap排序实现age字段按倒序排列
        HashMap<Integer,User> sortHashMap = sortHashMap(map);
        System.out.println("排序后："+sortHashMap);
        
    }
    //对HashMap排序
    public static HashMap<Integer, User> sortHashMap(HashMap<Integer, User> map) {
        LinkedHashMap<Integer, User> linkedHashMap = new LinkedHashMap<>();
        
        //TODO 将map 转化成Collection
        Set<Entry<Integer,User>> set = map.entrySet();
        
        ArrayList<Entry<Integer,User>> list = new ArrayList<>(set);
        
        Collections.sort(list, new Comparator<Entry<Integer,User>>() {
​
​
            @Override
            public int compare(Entry<Integer, User> o1, Entry<Integer, User> o2) {
                //排序规则
                //前一个对象-后一个对象=正序
                return o2.getValue().getAge()- o1.getValue().getAge(); 
            }
        });
        //TODO 将list的内容添加到linkedHashMap中
        for (int i = 0; i < list.size(); i++) {
            linkedHashMap.put(list.get(i).getKey(), list.get(i).getValue());
        }
        
        return linkedHashMap;
    }
}
```