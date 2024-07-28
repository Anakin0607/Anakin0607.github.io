# 用torch重写keras
https://juejin.cn/post/6863822972709732366

# torch中自定义层
[定义模型](https://blog.csdn.net/qq_27825451/article/details/90550890)

[定义层](https://blog.csdn.net/qq_27825451/article/details/90705328)

# torch中自定义加载数据方式
Dataset负责逐条读取数据

Dataloader负责将Dataset中的数据组合

https://blog.csdn.net/zyw2002/article/details/128175177

# torch训练模型
一般用下面的模板就行
```python
criterion = nn.CrossEntropyLoss() #选择损失函数
optimizer = torch.optim.Adam(model.parameters(), lr=0.001, weight_decay=0.01) # 选择优化器
scheduler = torch.optim.lr_scheduler.StepLR(optimizer = optimizer, step_size=10, gamma=0.1) # 逐轮下降学习率

for e in range(epochs):
        for i, (wav, label) in enumerate(dataLoader):
            # 将数据和标签装载到设备
            wav = wav.to(device)
            label = label.to(device)
            out = model(wav) #用模型计算当前结果

            # 计算损失函数
            loss = criterion(out, label)
            optimizer.zero_grad()
            loss.backward()
            
            # 迭代
            optimizer.step()
        scheduler.step() #降低学习率
        torch.save(model, 'model.pkl') # 保存模型
        print("model saved!")
```

# torch利用模型进行推理

# 转onnx

https://mmdeploy.readthedocs.io/zh-cn/latest/tutorial/03_pytorch2onnx.html