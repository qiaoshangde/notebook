


# iter（）迭代器
当遇到一个可迭代对象（列表，元组，字典，字符串,DataLoader等）都可以使用 iter() 方法获取迭代器，


```python
my_list = [1, 2, 3, 4, 5]
iter（my_list）  #把my_list变成一个可迭代对象


print(next(iterator))  # 输出：1
print(next(iterator))  # 输出：2
#在第一次调用next方法时，用它会从迭代器中获取下一个值。在第一次调用时，它会返回列表的第一个元素，第二次调用时返回列表的第二个元素，以此类推。当迭代器中没有更多元素时，next() 函数会抛出 StopIteration [异常]。
for i in iterator:
    print(i)  # 输出：3 4 5
    ##前面已经用过next方法两次，for 循环会自动为每个元素调用 next() 函数。当我们使用 for 循环遍历迭代器时，一旦所有元素都被遍历完，迭代器就会结束。
注意

#如果我们对可迭代对象使用 iter() 函数多次，会得到同一个迭代器对象。在再次对可迭代对象调用 iter() 时并不会重新创建新的迭代器对象。因此，迭代器对象是共享的。举个例子：
my_list = [1, 2, 3, 4, 5]
iterator1 = iter(my_list)
iterator2 = iter(my_list)  # iterator1 和 iterator2 是同一个迭代器对象
print(next(iterator1))  # 输出：1
print(next(iterator2))  # 输出：2
print(next(iterator1))  # 输出：3
# iterator1 和 iterator2，它们都是对同一个可迭代对象 my_list 的引用。因此，改变其中一个迭代器的状态会影响另一个迭代器的状态


#以下可以看成一个简单的迭代器
class MyIterator:
    def __init__(self, data):
        self.data = data
        self.index = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.index < len(self.data):
            result = self.data[self.index]
            self.index += 1
            return result
        else:
            raise StopIteration  # 没有更多元素时抛出异常

data = [1, 2, 3]
my_iter = MyIterator(data)
for item in my_iter:
    print(item)
```


**`DataLoader` 本身是一个可迭代对象，所以我们可以使用 `iter()` 将其转换为迭代器**


# tensor可以   img.shape 也可以img.size（）查看tenor的大小



# 修改python包导入的搜索路径


```python
sys.path.append(os.path.join(ROOT_DIR, 'models'))
```


- **`sys.path`** 是一个 Python 列表，包含了 Python 查找模块时会搜索的路径。当你使用 `import` 语句导入模块时，Python 会按照 `sys.path` 中的路径顺序来查找模块。
- 默认情况下，`sys.path` 包含了一些标准路径，比如当前工作目录、Python 库路径等。你可以向其中添加新的路径，这样 Python 就能找到这些路径下的模块。
- ### **`os.path.join(ROOT_DIR, 'models')`**

- **`os.path.join()`** 是 Python 中的一个函数，用来智能地拼接路径。
    
    - 它根据操作系统自动选择正确的路径分隔符（在 Linux 和 macOS 中是 `/`，在 Windows 中是 `\`）。
    - `os.path.join(ROOT_DIR, 'models')` 会将 `ROOT_DIR` 和 `'models'` 拼接成一个完整的路径。例如，如果 `ROOT_DIR` 是 `/home/user/project`，那么这个操作会得到 `/home/user/project/models`。
- **`ROOT_DIR`**：通常是一个指向项目根目录的路径变量。它可能通过一些其他方式在代码中定义。例如，`ROOT_DIR` 可以是当前脚本所在的路径，或者是项目的根目录。
    
    - 如果 `ROOT_DIR` 是 `/home/user/project`，那么 `os.path.join(ROOT_DIR, 'models')` 会返回 `/home/user/project/models`，即 `models` 目录的路径。

**`sys.path.append()`**
- **`sys.path.append()`** 是将路径添加到 `sys.path` 列表的末尾。
    - 使用 `sys.path.append()` 后，Python 会在搜索模块时也会检查这个新加入的路径。
    - 如果你希望 Python 能找到 `models` 目录下的模块，你就需要将 `models` 路径添加到 `sys.path` 中。



**将 `ROOT_DIR/models` 目录的路径添加到 Python 的模块搜索路径 `sys.path` 中。这样，当你在代码中 `import` 模块时，Python 会在 `ROOT_DIR/models` 目录中查找这些模块。**

使用方法：
```python
BASE_DIR = os.path.dirname(os.path.abspath(__file__))###获取当前文件夹和文件的路径，然后去文件夹路径  
ROOT_DIR = BASE_DIR  
sys.path.append(os.path.join(ROOT_DIR, 'models'))   ##用于 修改 Python 模块的搜索路径，以便 Python 能够找到并导入 models 目录下的模块或包。  
##将 ROOT_DIR/models 目录的路径添加到 Python 的模块搜索路径 sys.path 中。这样，当你在代码中 import 模块时，Python 会在 ROOT_DIR/models 目录中查找这些模块。
```




# 训练进度显示tqdm

`tqdm` 的基本用法非常简单，只需要将一个可迭代对象传递给 `tqdm()` 函数，它就会自动显示一个进度条。

```python
from tqdm import tqdm
import time

# 创建一个简单的循环
for i in tqdm(range(100)):
    time.sleep(0.1)  # 模拟一些耗时操作

#输出
100%|██████████| 100/100 [00:10<00:00,  9.99it/s]

```

tqdm还有一些参数如：desc="Training"和total，前这显示在训练前有一串字符，表示在进行什么迭代，total时迭代的总数，显示在{ }/{ }中，当不指定时，为前面可迭代对象的长度


