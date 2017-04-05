# Haproxy 文件增刪改查的操作

===

**需求**

HAproxy配置文件操作：

1. 根據用戶輸入輸出對應的backend下的server信息
2. 可新增backend 和 sever信息
3. 可修改backend 和 sever信息
4. 可刪除backend 和 sever信息
5. 操作配置文件前進行備份
6. 添加server信息時，如果ip已經存在則修改;如果backend不存在則創建；若信息與已有信息重複則不操作

**流程圖**

![](http://v3.processon.com/chart_image/58cbee18e4b0701bb3ea9bd6.png)

**說明**

依據現有的haproxy文件去做查詢、新增、修改、刪除等操作。

**1. 查詢**

產生一個空列表 `server_list = []`，接著拼接一個backend的字符串，格式為 `backend www.test.com`，打開 `haproxy.conf` 文件，定義一個 `server_flag = False`，依據拼接出來的字符串逐行去比對，如果有比對到的話，就拉警報器，此時就要去修改 `server_flag = True`，將找到的backend中的server訊息給加入至空列表裡 `server_list.append(line)`，如果沒有比對到的話，就不理它，最後將其結果給打印出來並返回。


**2. 新增**

先抓到使用者輸入的字典形式中的backend，即 `backend = data['backend']`，再來調用查詢函數，來判斷backend是否存在，即 `record_list = search(backend)`，打開源文件 `haproxy.conf` 和 新文件 `haproxy_new.conf`。

情況一:

backend已經存在了，就比對server的訊息，如果不存在就新增。

情況二:

backend不存在，就新增backend，同時也新增server的訊息。

**3. 修改**

使用者可以依據已存在的Backend中的Server紀錄做修改


**4. 刪除**

使用者可以依據已存在的Backend中的Server紀錄做刪除



**代碼**

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

import os

def file_handle(FileName, backend_data, record_list=None, type='search'):
    '''
    將文件操作的類別，分成三種，分別為：search, append, change
    FileName = 'haproxy.conf'
    backend_data = "backend {}".formate(data)"
    record_list = server record
    type = 'search'
    '''
    NewFileName = 'haproxy_new.conf'
    BackFileName = 'haproxy_bak.con'
    if type == 'search':
        server_list = []
        # 打開haproxy.conf，準備開始做判斷
        with open(FileName, 'r') as ha:
        # 替server字符串打上警報，並依據flag做為判斷
            server_flag = False
            for line in ha:
                line = line.strip()
                if line == backend_data:
                    server_flag = True
                    continue
                if server_flag and line.startswith("backend"):
                    break
                if server_flag and line:
                    server_list.append(line)
            for line in server_list:
                print(line)
            return server_list
    elif type == 'append':
        with open(FileName, 'r') as read_file, \
                open(NewFileName, 'w') as write_file:
            for r_line in read_file:
                write_file.write(r_line)

            for new_line in record_list:
                if new_line.startswith('backend'):
                    write_file.write(new_line + '\n')
                else:
                    write_file.write("{}{}\n".format(' '*8, new_line))
        os.rename(FileName, BackFileName)
        os.rename(NewFileName, FileName)
        os.remove(BackFileName)

    elif type == 'change':
        # 讀取原文件，並同時寫入到新文件
        with open(FileName, 'r') as read_file, \
                open(NewFileName, 'w') as write_file:
            server_flag = False
            has_write = False
            for r_line in read_file:
                # 比對原文件的backend 等於 拼接的字符串(已存在的backend)，比對到想要的backend，就拉警報
                if r_line.strip() == backend_data:
                    server_flag = True
                    continue
                # 比對原文件的backend 等於 拼接的字符串(已存在的backend)，就關警報
                if server_flag and r_line.startswith('backend'):
                    server_flag = False
                if not server_flag:
                    write_file.write(r_line)
                # 如果不是，就按照自已的拼接紀錄寫進去新文件
                else:
                    if not has_write:
                        for new_line in record_list:
                            if new_line.startswith('backend'):
                                write_file.write(new_line + '\n')
                            else:
                                write_file.write("{}{}\n".format(' ' * 8, new_line))
                        has_write = True
        os.rename(FileName, BackFileName)
        os.rename(NewFileName, FileName)
        os.remove(BackFileName)

def search(data):
    '''
    主要是讓使用者輸入 Backend 的網址 www.test1.com 進行搜尋
    '''
    # 拼接要抓取的字符串
    backend_data = "backend {}".format(data)
    # 建立一個空的列表，將上面符合server字符串的，都將其加入
    return file_handle('haproxy.conf', backend_data, type='search')

def add(data):
    '''
    使用者可以新增Backend及Server
    輸入的格式為： {'backend':'www.test.com', 'record':{'server':'192.168.1.1', 'weight':'20', 'maxconn':'2000'}}
    '''
    # 抓取使用者輸入的字典形式的key & value
    backend = data['backend']
    # 調用之前的函數，當做查詢backend存不存在
    record_list = search(backend)
    # 拼接目前的record
    # 格式為：{'backend':'www.test.com', 'record':{'server':'192.168.1.1', 'weight':'20', 'maxconn':'2000'}}
    current_record_list = "server {} {} weight {} maxconn {}".format(data['record']['server'],\
                                                                     data['record']['server'],\
                                                                     data['record']['weight'],\
                                                                     data['record']['maxconn'])
    # 拼接字符串
    backend_data = "backend {}".format(backend)

    if not record_list:
        record_list.append(backend_data)
        record_list.append(current_record_list)
        file_handle('haproxy.conf', backend_data, record_list, type='append')
    else:
        #print(record_list)
        record_list.insert(0,backend_data)
        if current_record_list not in record_list:
            record_list.append(current_record_list)
            file_handle('haproxy.conf', backend_data, record_list, type='change')

def fixed(data):
    '''
    使用者可以依據已存在的Backend中的Server紀錄做修改
    輸入格式為：[{'backend':'www.test1.com', 'record':{'server':'2.2.2.2', 'weight':'20', 'maxconn':'2000'}}, {'backend':'www.test1.com', 'record':{'server':'192.168.1.1', 'weight':'30', 'maxconn':'2000'}}]
    '''

    backend = data[0]['backend']
    record_list = search(backend)

    origin_record = "server {} {} weight {} maxconn {}".format(data[0]['record']['server'],\
                                                               data[0]['record']['server'],\
                                                               data[0]['record']['weight'],\
                                                               data[0]['record']['maxconn'])

    new_record = "server {} {} weight {} maxconn {}".format(data[1]['record']['server'],\
                                                            data[1]['record']['server'],\
                                                            data[1]['record']['weight'],\
                                                            data[1]['record']['maxconn'])
    # 拼接字符串
    backend_data = "backend {}".format(backend)

    if not record_list or origin_record not in record_list:
        print('\033[33;1m無此內容\033[0m')
        return
    else:
        record_list.insert(0,backend_data)
        index = record_list.index(origin_record)
        record_list[index] = new_record
        file_handle('haproxy.conf', backend_data, record_list, type='change')

def remove(data):
    '''
    使用者可以依據已存在的Backend中的Server紀錄做刪除
    輸入格式為：{'backend':'www.test1.com', 'record':{'server':'1.1.1.1', 'weight':'20', 'maxconn':'2000'}}
    '''
    backend = data['backend']
    record_list = search(backend)
    # 拼接目前的record
    # 格式為：{'backend':'www.test.com', 'record':{'server':'2.2.2.2', 'weight':'20', 'maxconn':'2000'}}
    current_record_list = "server {} {} weight {} maxconn {}".format(data['record']['server'],\
                                                                     data['record']['server'],\
                                                                     data['record']['weight'],\
                                                                     data['record']['maxconn'])
    # 拼接字符串
    backend_data = "backend {}".format(backend)

    if not record_list or current_record_list not in record_list:
        print('\033[33;1m無此條記錄\033[0m')
        return
    else:
        # 處理record_list
        record_list.insert(0,backend_data)
        record_list.remove(current_record_list)
        file_handle('haproxy.conf', backend_data, record_list, type='change')

def exit(data):
    exit()




if __name__ == '__main__':

    # 提示操作選項
    msg = '''
        1. 查詢
        2. 新增
        3. 修改
        4. 刪除
        5. 登出
    '''

    menu_dict = {
        '1':search,
        '2':add,
        '3':fixed,
        '4':remove,
        '5':exit,
    }

    while True:
        print("操作說明".center(40, '#'))
        print("\033[36;1m" + msg + "\033[0m")
        print("".center(43, '#'))
        choice_number = input("請輸入數字:")
        if choice_number == '0' or choice_number not in menu_dict:
            print("\033[41;30m輸入錯誤，請再選一次\033[0m")
            continue
        if choice_number == '5':
            break

        if choice_number == '1':
            print("輸入格式為：www.test1.com")
        elif choice_number == '3':
            print("輸入格式為：[{'backend':'www.test1.com', 'record':{'server':'1.1.1.1', 'weight':'20', 'maxconn':'2000'}}, {'backend':'www.test1.com', 'record':{'server':'192.168.1.10', 'weight':'30', 'maxconn':'2000'}}]")
        else:
            print("輸入格式為：{'backend':'www.test1.com', 'record':{'server':'1.1.1.1', 'weight':'20', 'maxconn':'2000'}}")

        data = input("請輸入數據:")


        if choice_number != '1':
            data = eval(data)

        menu_dict[choice_number](data)
```

