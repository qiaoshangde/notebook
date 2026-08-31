要把单卡代码改造成适配 A100 集群的 DDP 代码，必须严格执行以下 6 个步骤。漏掉任何一步都可能导致训练慢、显存爆炸或模型无法收敛。

---

### 第一步：启动方式 (Shell)

不要再用 `python script.py`，必须改用 **`torchrun`** 指令，它会自动分配环境变量。

```bash
CUDA_VISIBLE_DEVICES=0,3,4,5,6,7 torchrun --nproc_per_node=6 train_dpo.py --batch_size 128 --num_workers 8 --epochs 2
```


```bash
CUDA_VISIBLE_DEVICES=0,3,4,5,6,7 torchrun --nproc_per_node=6 2-full_sft.py \
--batch_size 128 \
--num_workers 8 \
--epochs 5
```



```bash
# nproc_per_node = 你的显卡数量 (例如 8)
torchrun --nproc_per_node=8 train.py --args...
```

---

### 第二步：初始化环境 (Init)

在代码最开头（`main` 函数内），必须完成“认领身份”和“绑定设备”。



```Python
# 1. 初始化通信后端
dist.init_process_group(backend="nccl")

# 2. 获取当前进程负责的显卡编号 (0-7)
local_rank = int(os.environ["LOCAL_RANK"])

# 3. 【关键】强制绑定设备
# 这一步如果不写，所有进程都会去抢 GPU 0，导致 OOM
torch.cuda.set_device(local_rank)
device = torch.device(f"cuda:{local_rank}")

# 4. 设置随机种子
# 必须加上 local_rank，保证每张卡的 Dropout/数据增强 不一样
setup_seed(42 + local_rank)
```

---

### 第三步：模型包装 (Model Wrapper)

给模型穿上 DDP 的“外衣”，让它具备自动同步梯度的能力。



```Python
model = MyModel().to(device)

# 【可选】如果是 Llama 类模型，忽略旋转位置编码的 Buffer，防止报错和浪费带宽
model._ddp_params_and_buffers_to_ignore = {"freqs_cos", "freqs_sin"}

# 【关键】包装模型
# device_ids 必须指定，否则会发生多进程资源冲突
model = DistributedDataParallel(model, device_ids=[local_rank])
```

---

### 第四步：数据切分 (Sampler)

确保 8 张卡读到的数据是不重样的，且合起来是完整的数据集。



```Python
# 1. 定义采样器
train_sampler = DistributedSampler(train_dataset)

# 2. 放入 DataLoader
# 注意：shuffle=False (因为 Sampler 已经负责乱序了)
train_loader = DataLoader(dataset, batch_sampler=..., num_workers=..., pin_memory=True)
```

---

### 第五步：训练循环 (Training Loop)

这部分有两个细节如果不注意，训练效果会大打折扣。



```Python
for epoch in range(epochs):
    # 1. 【关键】打乱数据顺序
    # 如果不写这行，每个 Epoch 的数据顺序都会一模一样
    train_sampler.set_epoch(epoch)
    
    for batch in train_loader:
        # ... 正常训练代码 ...
        
        # 2. 日志记录 (WandB / Print)
        # 只允许主进程说话，防止 8 个进程同时刷屏
        if local_rank == 0:
            wandb.log({...})
            print(f"Loss: {loss}")
```

---

### 第六步：模型保存 (Saving)

保存时要“剥壳”，只保存模型本体。



```Python
if local_rank == 0:
    # model.module 才是真正的模型，model 只是 DDP 的壳子
    # 如果不取 module，下次单卡加载时 key 会多出 "module." 前缀导致报错
    state_dict = model.module.state_dict()
    torch.save(state_dict, "save.pth")
```

---

### 总结图解

| **步骤**  | **关键词**              | **作用**  | **后果如果不做**       |
| ------- | -------------------- | ------- | ---------------- |
| **启动**  | `torchrun`           | 自动管理多进程 | 无法启动并行           |
| **初始化** | `set_device`         | 隔离显存    | **GPU 0 显存爆炸**   |
| **模型**  | `DDP(device_ids)`    | 同步梯度    | 模型无法收敛，参数不更新     |
| **数据**  | `DistributedSampler` | 切分数据    | 每张卡都跑全量数据，毫无加速   |
| **循环**  | `set_epoch`          | 随机洗牌    | **模型过拟合**，死记硬背顺序 |
| **保存**  | `model.module`       | 剥离外壳    | 权重文件无法被单卡加载      |

这份清单涵盖了您代码中所有涉及 DDP 的关键修改点。照着这个检查一遍，训练就稳了。