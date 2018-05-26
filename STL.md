# STL

### 1、vector

#### 定义一个vector迭代器

```c++
vector<类型>::iterator iter;
```

#### 获取vector中元素的个数

```c
size()
```

##### 获取一维vector中元素的个数

```
v.size();
```

##### 获取二维vector中的行数

```
v.size();
```

##### 获取二维vector中的列数

```
v[0].size();
```

#### 获取vector的起始位置

```c
begin()
```

#### 获取vector的末尾的下一个位置

```c
end()
```

#### 通过下标获取vector的元素

```
v[i]
```

#### 添加元素

```c
push_back()
```

注意，如果还没有使用`push_back()` 添加元素到达`v[i][j]`的位置，那么就不能够使用`v[i][j]`。但是如果在定义vector容器的时候，指明了容器中元素的个数，那么就可以使用下标添加（因为在指明容器中元素的个数的时候，实际上会自动赋予默认值，所以本质上不是添加而是修改容器中元素的值）

#### 遍历vector

##### 方法一（推荐）

```c++
for (int i = 0; i < findNums.size(); ++i) { // 其中findNums是一个vector
    cout<<findNums[i]<<endl;
}
```

##### 方法二

```c++
for (std::vector<int>::iterator i = findNums.begin(); i != findNums.end(); ++i) {
    cout<<*i<<endl;
}
```

### 2、集合（set）

集合里面的元素都不重复，且默认是**从小到大排好了序**

### 定义一个set迭代器

```c++
set<类型>::iterator iter
```

#### 插入元素

```c
insert() // 可以往里面插入元素。如果元素重复了，就不会被插入
```

#### 获取集合中第几个元素

```c++
set<int> s;
set<int>::iterator iter = s.begin(); // 注意，前面的set<int>不能漏了
int i = 2;
while (i > 0) {
    iter++;
    i--;
}
cout<<*iter; // 此时得到的是集合中第3个元素
```

**不能直接使用`s[2]`来获取第三个元素。**

### 3、find()

#### 用法

```c++
std::find(first, end, value);
```

##### 对迭代器进行查找

```c++
vector<int> nums;
vector<int>::iterator iter;
/* some codes */
iter = find(nums.begin(), nums.end(), 一个需要查找的数字); // find返回一个迭代器
```

如果没有找到指出的对象，就会返回`nums.end()`的值，要是找到了就返回一个指着找到的对象的iterator

##### 对数组进行查找

```c++
int IntValue[5] = {1,2,3,4,5}; 
int* Result; 

Result = find(IntValue, IntValue + 4, 8); 
if (Result != IntValue + 4)  { 
    cout<<"Result = "<<*Result<<endl; 
}
else { 
    cout <<"No corret result"<<endl; 
}
```

### 4、队列（queue）

由于仅需要取队首和队尾元素的操作，因此queue队列容器并**不提供任何类型的迭代器**。

#### 创建队列

```c++
queue<类型> q; 
```

使用默认的**双端队列**为底层容器创建了一个空的queue队列对象q

#### 入队

```c++
q.push(3);
```

#### 出队

```c++
q.pop(); // 此函数并不返回任何值
```

#### 取队首元素

```c++
q.front(); // 返回队首元素，并不会让它出队
```

#### 取队尾元素

```c++
q.back(); // 返回队尾元素，并不会让它出队
```

#### 判断队列是否为空

```c++
q.empty();
```

#### 队列长度

```c++
q.size();
```

### 5、哈希（map）

map内部是**按照key的顺序排序**的，不是按照插入的顺序排序的

#### 创建哈希表

```c++
std::map<int, bool> m; // 不需要给出哈希表的大小，并且可以直接使用[]来操作哈希表（即时开始没有先插入元素）
```

#### 插入元素

```c++
m[key] = num;
```

#### 查找key

```c++
map<int, string>::iterator iter = m.find(key);
```

用find函数来定位数据出现位置，它返回的一个迭代器，当数据出现时，它返回数据所在位置的迭代器。

如果map中没有要查找的数据，它返回的迭代器等于end函数返回的迭代器。

```
m.count(key);
```

因为map中的key是不允许重复的，所以一个key只能出现一次，这说明count的返回值就只能是0或1了。

#### 获得键

例如获得哈希表第一个元素的键：

```c++
m.begin()->first;
```

#### 获得值

```c++
m.begin()->second;
```

#### 无序哈希表

举个例子：

```c++
unordered_map<string, int> um;
```

### 6、string类型

#### 排序

```c++
sort(s.begin(), s.end());
```

#### 判断子串

例如，判断`str1`是否是`str2`的子串：

```c++
if (str2.find(str1) ！= -1) {
    return true;
}
else {
    return false;
}
```

#### 是用auto类型来接收迭代器类型

```c++
auto iter = unique(str1.begin(), str1.end());
```

#### 添加字符到字符串的末尾

添加字符到字符串末尾一定用push_back()函数，切不可直接用“+”。

```c++
string temp;
temp.push_back('a');
```

注意，如果使用`push_back`往字符串中添加整型，那么会添加空的字符进去。（也就是添加不进去）。而且，不能使用`push_back`来添加字符串。

### 7、求并集

```c++
/*
Given two arrays, write a function to compute their intersection.

Example:
Given nums1 = [1, 2, 2, 1], nums2 = [2, 2], return [2].

Note:
Each element in the result must be unique.
The result can be in any order.
*/
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        sort(nums1.begin(), nums1.end());
        sort(nums2.begin(), nums2.end());

        vector<int> v3;

        set_intersection(nums1.begin(), nums1.end(), nums2.begin(), nums2.end(), back_inserter(v3));

        vector<int>::iterator iter = unique(v3.begin(), v3.end());
        v3.erase(iter,v3.end()); // 删除后面的那些重复元素

        return v3;
    }
};
```

使用`set_intersection`之前需要对容器（不一定是集合<`set`>）中的元素进行排序（从小到大）。

### 8、unique

unique()函数是一个**去重**函数，STL中unique的函数unique的功能是去除**相邻**的重复元素(只保留一个),还有一个容易忽视的特性是**它并不真正把重复的元素删除**。

unique(num,mun+n)**返回的是num去重后的尾地址** 。之所以说不真正把重复的元素删除，其实是，该函数把重复的元素移到后面去了，然后依然保存到了原数组中，然后**返回去重后最后一个元素的地址。**

因为unique去除的是相邻的重复元素，所以**一般用之前都会要排一下序**。

### 9、pair

 pair 是 一种模版类型。每个pair 可以存储两个值。这两种值无限制。也可以将自己写的struct的对象放进去。

例如：

```c++
pair<string, int> p;
pair<int, int> p;
pair<double, int> p;
```

#### 应用

如果一个函数有两个返回值的话，如果是相同类型，就可以用数组返回，如果是不同类型，就可以自己写个struct，但为了方便就可以使用 c++  自带的pair ，返回一个pair，其中带有两个值。

如果有三个属性的话，其实也是可以用的pair 的 ，极端的写法 `pair <int, pair<int, int> >`写法极端。（后边的两个` > > `要有空格，否则就会是`>>`位移运算符）

#### 获取pair的值

 每个pair 都有两个**属性值**  first  和second

```c++
cout<<p1.first<<p1.second;
```

 注意**是属性值而不是方法**（即不是`p1.first()和p1.second()`）。

### 10、reverse()

```c++
reverse(begin, end) // 其中的begin和end是迭代器
```

reverse()会将区间[begin, end)内的元素全部逆序（反转容器内指定范围中的元素）

### 11、清空容器

```c++
容器.clear();
```

### 12、栈（stack）

栈**没有迭代器**。

### 13、is_sorted

```c++
bool is_sorted(ForwardIterator first, ForwardIterator last);
```

测试范围内的元素是否已经有序。

### 14、sort自定义排序

```c++
/**
 * Definition for an interval.
 * struct Interval {
 *     int start;
 *     int end;
 *     Interval() : start(0), end(0) {}
 *     Interval(int s, int e) : start(s), end(e) {}
 * };
 */
class Solution {
public:
    vector<Interval> merge(vector<Interval>& intervals) {
        sort(intervals.begin(),intervals.end(),  compare_Interval); 
    }
  
    static int compare_Interval(Interval val1, Interval val2){
        return val1.start < val2.start;  
    }
};
```

例如，`[[1,4],[0,0]]`排序后，变为：`[[0,0],[1,4]]`



























































