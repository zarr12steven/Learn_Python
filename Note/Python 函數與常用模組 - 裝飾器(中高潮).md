### 裝飾器 - 中高潮

那如果想要在 `裝飾器` 裡傳參數的話，那應該要怎麼做呢？還記得先前提到的[函數的非固定參數](http://www.cnblogs.com/zarr12steven/p/6538632.html)，其實很簡單

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-


import time

def timer(func):
    def deco(*args, **kwargs):
        start_time = time.time()
        func(*args, **kwargs)
        stop_time = time.time()
        print("in the func run time is {}".format(stop_time - start_time))
    return deco

@timer
def foo():
    time.sleep(3)
    print('in the foo')

@timer
def bar(name, age):
    print("bar:", name, age)

foo()
bar("Tony Stark", 36)

---------------執行結果---------------

in the foo
in the func run time is 3.0010311603546143
bar: Tony Stark 36
in the func run time is 4.291534423828125e-05

Process finished with exit code 0
```

因為 `@timer => bar = timer(bar)` 、 `func = bar` ，其實 `timer(bar) = return deco` ，所以要修改 `def deco(*args, **kwargs)`，而 `func() = bar()`，也請修改成 `func(*args, **kwargs)` ，這樣就可以傳參數了，如此一來，就不管有沒有傳參數進來，全都可以正常去調用 `裝飾器`。