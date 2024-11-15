# Hashmap的一种遍历

```
ConcurrentHashMap<Long,Long> tidlists = new ConcurrentHashMap<>();
Set<Long> keys = map.keySet();
Iterator<Long> els = keys.iterator();
                while (els.hasNext()) {
                    map.get(key));
                }
```

`HashMap` 是 Java 中常用的数据结构之一，用于存储键值对（key-value pairs）。`HashMap` 提供了平均时间复杂度为 O(1) 的快速查找、插入和删除操作。以下是一些关于 `HashMap` 的基本用法和示例代码。

# 主要方法

1. **创建 `HashMap`**：
   
   ```java
   HashMap<K, V> map = new HashMap<>();
   ```
   
2. **插入键值对**：
   ```java
   map.put(key, value);
   ```

3. **获取值**：
   ```java
   V value = map.get(key);
   ```

4. **删除键值对**：
   ```java
   map.remove(key);
   ```

5. **检查是否存在键**：
   ```java
   boolean containsKey = map.containsKey(key);
   ```

6. **检查是否存在值**：
   ```java
   boolean containsValue = map.containsValue(value);
   ```

7. **获取键值对的数量**：
   ```java
   int size = map.size();
   ```

8. **清空 `HashMap`**：
   ```java
   map.clear();
   ```

9. **遍历 `HashMap`**：
   - **遍历键**：
     ```java
     for (K key : map.keySet()) {
         V value = map.get(key);
         // 处理 key 和 value
     }
     ```
   - **遍历值**：
     ```java
     for (V value : map.values()) {
         // 处理 value
     }
     ```
   - **遍历键值对**：
     ```java
     for (Map.Entry<K, V> entry : map.entrySet()) {
         K key = entry.getKey();
         V value = entry.getValue();
         // 处理 key 和 value
     }
     ```

### 示例代码

以下是一个简单的示例，展示了如何使用 `HashMap` 进行基本操作。

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapExample {
    public static void main(String[] args) {
        // 创建一个 HashMap
        HashMap<String, Integer> map = new HashMap<>();

        // 插入键值对
        map.put("Alice", 25);
        map.put("Bob", 30);
        map.put("Charlie", 35);

        // 获取值
        int age = map.get("Alice");
        System.out.println("Alice's age: " + age);

        // 检查是否存在键
        boolean containsKey = map.containsKey("Bob");
        System.out.println("Contains Bob: " + containsKey);

        // 检查是否存在值
        boolean containsValue = map.containsValue(35);
        System.out.println("Contains age 35: " + containsValue);

        // 获取键值对的数量
        int size = map.size();
        System.out.println("Number of entries: " + size);

        // 遍历键
        System.out.println("Keys:");
        for (String key : map.keySet()) {
            System.out.println(key);
        }

        // 遍历值
        System.out.println("Values:");
        for (Integer value : map.values()) {
            System.out.println(value);
        }

        // 遍历键值对
        System.out.println("Entries:");
        for (Map.Entry<String, Integer> entry : map.entrySet()) {
            String key = entry.getKey();
            Integer value = entry.getValue();
            System.out.println(key + ": " + value);
        }

        // 删除键值对
        map.remove("Charlie");
        System.out.println("After removing Charlie: " + map);

        // 清空 HashMap
        map.clear();
        System.out.println("After clearing: " + map);
    }
}
```

### 输出示例

假设你运行上述代码，输出可能如下：

```
Alice's age: 25
Contains Bob: true
Contains age 35: true
Number of entries: 3
Keys:
Alice
Bob
Charlie
Values:
25
30
35
Entries:
Alice: 25
Bob: 30
Charlie: 35
After removing Charlie: {Alice=25, Bob=30}
After clearing: {}
```

### 解释

1. **创建 `HashMap`**：
   - 使用 `new HashMap<>()` 创建一个 `HashMap` 对象。

2. **插入键值对**：
   - 使用 `put` 方法插入键值对。

3. **获取值**：
   - 使用 `get` 方法根据键获取对应的值。

4. **检查是否存在键**：
   - 使用 `containsKey` 方法检查 `HashMap` 是否包含指定的键。

5. **检查是否存在值**：
   - 使用 `containsValue` 方法检查 `HashMap` 是否包含指定的值。

6. **获取键值对的数量**：
   - 使用 `size` 方法获取 `HashMap` 中键值对的数量。

7. **遍历 `HashMap`**：
   - 使用 `keySet` 方法遍历键。
   - 使用 `values` 方法遍历值。
   - 使用 `entrySet` 方法遍历键值对。

8. **删除键值对**：
   - 使用 `remove` 方法根据键删除键值对。

9. **清空 `HashMap`**：
   - 使用 `clear` 方法清空 `HashMap`。

通过这些方法，你可以灵活地使用 `HashMap` 进行各种操作。