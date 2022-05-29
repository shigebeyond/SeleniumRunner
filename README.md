# SeleniumRunner - yaml驱动Selenium测试

## 概述
Selenium是基于浏览器的自动化测试工具，但是要写python代码；

考虑到部分测试伙伴python能力不足，因此扩展Selenium，支持通过yaml配置测试步骤;

本框架让伙伴通过编写yaml, 就可以实现一系列复杂的浏览器操作步骤, 如填充表单/提交表单/上传文件/校验响应/提取变量/打印变量等

## 特性
1. 基于 selenium 的webdriver
2. 使用 selenium-requests 扩展来处理post请求与上传请求
3. 支持通过yaml来配置执行的步骤
每个步骤可以有多个动作，但单个步骤中动作名不能相同（yaml语法要求;
动作代表webdriver上的一种操作，如goto/get/post/upload/submit_form等等;
4. 支持提取器
5. 支持校验器

## todo
1. 支持更多的动作

## 安装依赖
```
pip3 install -r requirements.txt
```

## 步骤配置文件
用于指定步骤, 示例如 step.yml
```
- # 登录
  goto:
    url: http://admin.jym1.com/login
  submit_form:
    account: '18877310999'
    passwd: '123456'
- # 商品列表
  goto:
    url: http://admin.jym1.com/goods/goods_service_list
    # 网页中html的提取变量
    extract_by_xpath:
      goods_id: //table/tbody/tr[1]/td[1] # 第一行第一列
  #  extract_by_css:
  #    goods_id: table>tbody>tr:nth-child(1)>td:nth-child(1) # 第一行第一列
- # 商品详情
  goto:
    url: http://admin.jym1.com/goods/goods_info?id=$goods_id&type=2
- # 新建门店
  goto:
    url: http://admin.jym1.com/store/add_store
  upload: # 上传文件/图片
    url: http://admin.jym1.com/upload/common_upload_img/store_img
    files: # 参数名:文件本地路径
      file: /home/shi/fruit.jpeg
    extract_by_jsonpath:
      img: $.data.url
  # 提交新建门店，不要用 submit_form，ui中太麻烦了太复杂了（如三级地址要逐级动态加载）
  post:
    url: http://admin.jym1.com/store/add_store
    is_ajax: true
    data:
      store_name: teststore-$random_str6
      store_logo_url: '$img'
      store_img_urls: '["$img"]'
      province: 450000
      city: 450100
      district: 450102
      address: testadd
      phone: 1347115$random_int4
      business_day_from: 1
      business_day_to: 1
      work_start_time: 09:00:00 - 20:00:00
      store_type: 0
      licence_url: '$img'
      license_code: 91450100788439413D
      card_true_name: shi
      bank_name: 中国工商银行
      bank_card_num: 6222024100018669328
      store_work_time: '["{\"date_from\":\"1\",\"date_to\":\"1\",\"work_start_time\":\"09:00:00\",\"work_end_time\":\"20:00:00\"}"]'
-
  goto:
    url: http://admin.jym1.com/store/store_list
  sleep: 2
```

## 运行
```
python runner.py 步骤配置文件
```

## 配置详解
支持通过yaml来配置执行的步骤;
每个步骤可以有多个动作，但单个步骤中动作名不能相同（yaml语法要求）;
动作代表webdriver上的一种操作，如goto/get/post/upload/submit_form等等;
下面详细介绍每个动作:

1. sleep: 线程睡眠
```yaml
sleep: 2 # 线程睡眠2秒
```

2. print: 打印, 支持输出变量
```yaml
# 调试打印
print: "总申请数=${dyn_data.total_apply}, 剩余份数=${dyn_data.quantity_remain}"
```

变量格式:
```
$msg 一级变量, 以$为前缀
${data.msg} 多级变量, 用 ${ 与 } 包含
```

3. goto: 浏览器跳转
```yaml
goto:
    url: http://admin.jym1.com/goods/goods_service_list # url,支持写变量
    extract_by_xpath: # 网页中html的提取变量
      goods_id: //table/tbody/tr[1]/td[1] # 第一行第一列
```

4. get: 发get请求, 但无跳转
```yaml
get:
    url: $dyn_data_url # url,支持写变量
    extract_by_eval:
      dyn_data: "json.loads(response.text[16:-1])" # 变量response是响应对象
```

5. post: 发post请求, 但无跳转
```yaml
post:
    url: http://admin.jym1.com/store/add_store # url,支持写变量
    is_ajax: true
    data: # post的参数
      # 参数名:参数值
      store_name: teststore-$random_str6
      store_logo_url: '$img'
```

6. upload: 上传文件
```yaml
upload: # 上传文件/图片
    url: http://admin.jym1.com/upload/common_upload_img/store_img
    files: # 上传的多个文件
      # 参数名:文件本地路径
      file: /home/shi/fruit.jpeg
    extract_by_jsonpath:
      img: $.data.url
```

7. submit_form: 提交表单
是 `input_by_name` 与 `click_by_css('[type=submit]')` 的结合
```yaml
submit_form:
  # 输入框name: 填充的值(支持写变量)
  account: '18877310999'
  passwd: '123456'
```

8. input_by_name: 填充 name 指定的输入框
```yaml
input_by_name:
  # 输入框name: 填充的值(支持写变量)
  account: '18877310999'
```

9. input_by_css: 填充 css selector 指定的输入框
```yaml
input_by_css:
  # 输入框css selector模式: 填充的值(支持写变量)
  '#account': '18877310999'
```

10. input_by_xpath: 填充 xpath 指定的输入框
```yaml
input_by_xpath:
  # 输入框xpath路径: 填充的值(支持写变量)
  "//input[@id='account']": '18877310999'
```

11. click_by_css: 点击 css selector 指定的按钮
```yaml
click_by_css: 'button[type=submit]' # 按钮的css selector模式
```

12. click_by_xpath: 点击 xpath 指定的按钮
```yaml
click_by_xpath: '//button[@type="submit"]' # 按钮的xpath路径
```

## 校验器
只针对 goto/get/post/upload 有发送http请求的动作, 主要是为了校验响应的内容

1. validate_by_xpath
从html的响应中解析 xpath 路径对应的元素的值
```yaml
validate_by_xpath:
  "//div[@id='goods_id']": # 元素的xpath路径
    '>': 0 # 校验符号或函数: 校验的值, 即 id 元素的值>0
  "//div[@id='goods_title']":
    contains: 衬衫 # 即 title 元素的值包含'衬衫'
```

2. validate_by_css
从html的响应中解析 css selector 模式对应的元素的值
```yaml
validate_by_css:
  '#id': # 元素的css selector 模式
    '>': 0 # 校验符号或函数: 校验的值, 即 id 元素的值>0
  '#goods_title':
    contains: 衬衫 # 即 title 元素的值包含'衬衫'
```

3. validate_by_jsonpath
从json响应中解析 多层属性 的值
```yaml
validate_by_jsonpath:
  '$.data.goods_id':
     '>': 0 # 校验符号或函数: 校验的值, 即 id 元素的值>0
  '$.data.goods_title':
    contains: 衬衫 # 即 title 元素的值包含'衬衫'
```

#### 校验符号或函数
1. `=`: 相同
2. `>`: 大于
3. `<`: 小于
4. `>=`: 大于等于
5. `<=`: 小于等于
6. `contains`: 包含子串
7. `startswith`: 以子串开头
8. `endswith`: 以子串结尾
9. `regex_match`: 正则匹配

## 提取器
只针对 goto/get/post/upload 有发送http请求的动作, 主要是为了从响应中提取变量

1. extract_by_xpath
从html的响应中解析 xpath 路径指定的元素的值
```yaml
extract_by_xpath:
  # 变量名: xpath路径
  goods_id: //table/tbody/tr[1]/td[1] # 第一行第一列
```

2. extract_by_css
从html的响应中解析 css selector 模式指定的元素的值
```yaml
extract_by_css:
  # 变量名: css selector 模式
  goods_id: table>tbody>tr:nth-child(1)>td:nth-child(1) # 第一行第一列
```

3. extract_by_jsonpath
从json响应中解析 多层属性 的值
```yaml
extract_by_jsonpath:
  # 变量名: json响应的多层属性
  img: $.data.url
```

4. extract_by_eval
使用 `eval(表达式)` 执行表达式, 并将执行结果记录到变量中
```yaml
extract_by_eval:
    # 变量名: 表达式
    dyn_data: "json.loads(response.text[16:-1])" # 变量response是响应对象
```