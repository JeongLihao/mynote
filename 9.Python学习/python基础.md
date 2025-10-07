## python基础

### 一、安装与验证

1. 验证

   ```bash
   python --version
   ```

   ![image-20251007184031470](https://raw.githubusercontent.com/JeongLihao/picgo/main/20251007184031569.png)

2. 基本命令

   | 命令             | 作用       |
   | ---------------- | ---------- |
   | python           | 启动解释器 |
   | exit()           | 退出解释器 |
   | python file.py   | 运行脚本   |
   | pip install 包名 | 安装外部库 |

   ```bash
   pip install requests
   ```

### 二、数据与变量

| 类型    | 名称   | 示例                             | 说明             |
| ------- | ------ | -------------------------------- | ---------------- |
| `int`   | 整数   | a = 10                           | 不带小数点的数字 |
| `float` | 浮点数 | b = 3.14                         | 带小数点的数     |
| `str`   | 字符串 | c = "Winston"                    | 一段文本         |
| `bool`  | 布尔值 | d = True                         | 表示真或假       |
| `list`  | 列表   | `[1, 2, 3]`                      | 一组可修改的数据 |
| `dict`  | 字典   | `{"name": "Winston", "age": 25}` | 键值对形式的数据 |

### 三、运算与逻辑控制

#### 1、算术运算

| 运算符 | 含义 | 示例     | 结果 |
| ------ | ---- | -------- | ---- |
| `+`    | 加法 | `5 + 3`  | 8    |
| `-`    | 减法 | `5 - 3`  | 2    |
| `*`    | 乘法 | `5 * 3`  | 15   |
| `/`    | 除法 | `5 / 2`  | 2.5  |
| `//`   | 整除 | `5 // 2` | 2    |
| `%`    | 取余 | `5 % 2`  | 1    |
| `**`   | 幂次 | `2 ** 3` | 8    |

#### 2、比较运算

| 运算符 | 含义     | 示例     | 结果    |
| ------ | -------- | -------- | ------- |
| `==`   | 等于     | `5 == 5` | `True`  |
| `!=`   | 不等于   | `5 != 3` | `True`  |
| `>`    | 大于     | `7 > 2`  | `True`  |
| `<`    | 小于     | `2 < 5`  | `True`  |
| `>=`   | 大于等于 | `5 >= 5` | `True`  |
| `<=`   | 小于等于 | `4 <= 3` | `False` |

#### 3、逻辑运算

| 运算符 | 含义 | 示例             | 结果    |
| ------ | ---- | ---------------- | ------- |
| `and`  | 并且 | `True and False` | `False` |
| `or`   | 或者 | `True or False`  | `True`  |
| `not`  | 取反 | `not True`       | `False` |

#### 4、条件语句

```python
age = int(input("输入数字"))
if age <= 18:
    print("未成年")
else:
    print("成年")
```

#### 5、循环

1. for循环

   ```python
   for i in range(6):
       print("你好",i)
   ```

   ![image-20251007190606578](https://raw.githubusercontent.com/JeongLihao/picgo/main/20251007190606621.png)

2. while循环

   ```python
   num = 0
   while num < 4:
       print("第",num + 1,"杯")
       num += 1
   ```

   ![image-20251007190850318](https://raw.githubusercontent.com/JeongLihao/picgo/main/20251007190850376.png)

### 四、函数

#### 1、函数基本结构

```python
def 函数名(参数):
    函数体
    return 返回值
```

1. 例如

   ```python
   def make_tea(tea_type):
       print("正在为您准备", tea_type)
       return "一杯热气腾腾的 " + tea_type
   
   # 调用
   result = make_tea("红茶")
   print(result)
   
   # 输出
   正在为您准备 红茶
   一杯热气腾腾的 红茶
   ```

2. 无参

   ```python
   def greet():
       print("下午好，先生。")
   
   # 调用
   greet()
   # 输出
   下午好，先生
   ```

3. 用函数返回结果

   ```python
   def add(a, b):
       return a + b
   
   # 调用
   result = add(3, 5)
   print(result)
   
   
   #输出
   8
   ```

   