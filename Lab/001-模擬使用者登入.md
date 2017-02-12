# 模擬使用者登入

===

**需求**

1. 輸入使用者帳號及密碼
2. 認証成功後，顯示歡迎訊息
3. 輸入三次錯誤後，鎖定該帳戶

**流程圖**

![](https://www.processon.com/chart_image/58a000a9e4b07ac8fc94b87a.png)

**說明**

當使用者登入時，會先檢查黑名單，如果使用者存在於黑名單，就鎖定帳號，無法登入，如果不存在於黑名單裡，就可以輸入帳號及密碼，並做帳號密碼的驗証，如果輸入錯誤三次，就鎖定帳號，輸入正確，就顯示歡迎登入的訊息。

**代碼**

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
# Author: Daniel Lin

import getpass

_username = 'tony'
_password = 'ironman'

count = 0

with open('block_user.txt', 'r') as f:
    if f.read() == _username:
        print('帳戶已被鎖定，請聯繫系統管理員')
        exit()

while count <= 2:
    username = input("Username:")
    password = getpass.getpass("Password:")

    if _username == username and _password == password:
        print("Weclome {name} Login ".format(name=username))
        break
    else:
        print("Invalid Username or Password")
        count += 1
        print("登入失敗第%s次" % count)

    if count == 3:
        with open('block_user.txt', 'w+') as f:
            f.write('%s' % username)
            print('帳戶已被鎖定，請聯繫系統管理員')
```
