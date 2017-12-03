### 裝飾器

===

**定義**

`本質是函數，用來裝飾其他函數，主要就是為了幫其他函數添加附加功能`

**原則**

1. 不能修改被裝飾的函數的源代碼
2. 不能修改被裝飾的函數的調用方式


簡單示範一下，什麼是 `裝飾器`

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import time

# 寫一個裝飾器
def timmer(func):
    def warpper(*args, **kwargs):
        start_time = time.time()
        func()
        stop_time = time.time()
        print('the func run time is {}'.format(stop_time - start_time))
    return warpper

@timmer # 調用裝飾器
def hello():
    time.sleep(3)
    print('in the hello')

hello()

---------------執行結果---------------

in the hello
the func run time is 3.004323959350586

Process finished with exit code 0
```

**實現裝飾器知識須知**

1. 函數就是 `變量`
2. 高階函數
  * 把一個函數名當做實際(形式)參數傳給另一個函數， `在不修改被裝飾函數源代碼的情況下，為它加上新的功能`。
  * 返回值中包含函數名， `不修改函數的調用方式`
  * 返回值可以是『字符串、數字、列表、元組』
3. 巢狀(嵌套)函數


`高階函數 + 巢狀(嵌套)函數 => 裝飾器`


#### 函數就是變量

下面就用代碼來解釋一下 `函數即變量` 是什麼意思！

觀察一下這個代碼，能不能正常執行？

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

def foo():
    print('in the foo')
    bar()
    
foo()

---------------執行結果---------------

Traceback (most recent call last):
  File "/Python/project/path/decorator1.py", line 9, in <module>
    foo()
  File "/Python/project/path/decorator1.py", line 7, in foo
    bar()
NameError: name 'bar' is not defined
in the foo

Process finished with exit code 1
```

嗯！發現出現了一個錯誤 `NameError: name 'bar' is not defined` ， 這是因為 `bar` 這個函數沒有被定義，所以才會報錯，那要怎麼解決這問題呢！？其實很簡單，就在寫一個函數叫 `bar` 就行了，那我們來寫寫看吧！

* **寫法一: 把bar函數定義在foo函數之前，看看能不能正常執行？** 

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

def bar():
    print('in the bar')

def foo():
    print('in the foo')
    bar()

foo()

---------------執行結果---------------

in the foo
in the bar

Process finished with exit code 0
```

* **寫法二: 把bar函數定義在foo函數之後，看看能不能正常執行？**

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

def foo():
    print('in the foo')
    bar()
    
def bar():
    print('in the bar')

foo()

---------------執行結果---------------

in the foo
in the bar

Process finished with exit code 0
```

* **寫法三: 把bar函數寫在foo函數調用之後**

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

def foo():
    print('in the foo')
    bar()
    
foo()

def bar():
    print('in the bar')
    
---------------執行結果---------------

in the foo
Traceback (most recent call last):
  File "/Python/project/path/decorator1.py", line 9, in <module>
    foo()
  File "/Python/project/path/decorator1.py", line 7, in foo
    bar()
NameError: name 'bar' is not defined

Process finished with exit code 1
```

耶~！怎麼第三個寫法會報錯？

#### 解說概念：

![](http://images2015.cnblogs.com/blog/1070619/201705/1070619-20170507160036914-625785340.png)

**寫法一圖解說明**

![](http://images2015.cnblogs.com/blog/1070619/201705/1070619-20170508081600066-1043876410.png)


**寫法二圖解說明**

![](http://images2015.cnblogs.com/blog/1070619/201705/1070619-20170508081617254-650893584.png)


**寫法三圖解說明**

![](http://images2015.cnblogs.com/blog/1070619/201705/1070619-20170508081642191-1258797184.png)



#### 總結

寫法一跟寫法二，都是在調用函數之前，就已經先定義函數好，所以都能正常執行，而寫法三會報錯，就是因為bar函數，是在調用函數之後，所以還沒有被定義，才無法正常執行，因此也証明了 `函數即變量`。




