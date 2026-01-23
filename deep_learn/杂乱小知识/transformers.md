 **`Dataset` 对象**。

代码中通过 `from datasets import load_dataset` 引入了这个库，并使用它来加载预训练数据。

### 1. 数据解读

- **类型**：`datasets.arrow_dataset.Dataset`
    
- **`features: ['text']`**：表示这个数据集只有一列，列名叫 `text`（存放原始文本）。
    
- **`num_rows: 1413103`**：表示数据集中一共有 **1,413,103** 条样本（行）。
    

### 2. 常用方法 (Methods)

这个对象非常强大，它的操作逻辑类似于 Pandas DataFrame，但针对大规模数据做了内存优化（基于 Arrow 格式，支持内存映射，不占满内存）。

以下是你在处理大模型数据时最常用的方法：

#### A. 查看与访问数据

- **索引访问**：像列表一样访问单行。
    
    Python
    
    ```
    print(ds[0])  # 输出第一行：{'text': '...内容...'}
    ```
    
    _这也是你的 `PretrainDataset` 类中 `__getitem__` 使用的方式_。
    
- **切片访问**：
    
    Python
    
    ```
    print(ds[:5]) # 输出前5行：{'text': ['...', '...', ...]}
    ```
    
- **列访问**：
    
    Python
    
    ```
    texts = ds['text'] # 获取所有文本组成的列表
    ```
    

#### B. 数据处理 (最核心)

- **`.map(function)`**：**这是最重要的函数**。用于批量处理每一行数据，例如进行分词（Tokenization）。它支持多进程加速 (`num_proc=8`)。
    
    Python
    
    ```
    # 示例：把文本变成 token IDs
    def process_func(example):
        return tokenizer(example['text'])
    
    ds = ds.map(process_func, num_proc=8)
    ```
    
- **`.filter(function)`**：按条件过滤数据。
    
    Python
    
    ```
    # 示例：去掉太短的句子
    ds = ds.filter(lambda x: len(x['text']) > 10)
    ```
    
- **`.shuffle(seed=42)`**：打乱数据顺序（训练前常用）。
    

#### C. 数据集分割与选择

- **`.train_test_split(test_size=0.1)`**：一键切分训练集和验证集。
    
- **`.select(indices)`**：选择特定的行（例如调试时只取前 1000 行跑通流程）。
    
    Python
    
    ```
    ds_small = ds.select(range(1000))
    ```
    

#### D. 保存与加载

- **`.save_to_disk('path')`**：处理完（如分词后）的数据可以存到硬盘，下次直接读，不用重新处理。
    
- **`.to_json('file.jsonl')`** / **`.to_csv('file.csv')`**：导出为常见格式。
    

### 总结

在你的 `dataset/lm_dataset.py` 代码中，`PretrainDataset` 类把这个巨大的 `Dataset` 对象赋值给了 `self.samples`。训练时，`DataLoader` 会根据索引不断从这个对象里取出一行行文本 (`sample['text']`)，然后喂给 Tokenizer 处理。