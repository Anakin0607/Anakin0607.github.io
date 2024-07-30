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
def train(model, device, dataLoader, criterion, optimizer, epoch):
    r"""模型训练函数
    Args:
        model: 要训练的模型
        device: 训练的设备
        dataLoader: 训练集
        criterion: 损失函数
        optimizer: 优化器
        epoch: 当前训练的轮数，给进度条用
    """
    model.train() #将模型设置为训练模式

    tensor = torch.rand(4, 1)
    train_loss = criterion(tensor, tensor) # 随便给loss赋一个初值
    bar = tqdm(dataLoader) # 定义一个进度条
    bar.desc = "Epoch {} train:".format(epoch + 1) # 进度条前面显示的内容
    for wav, label in bar:
        
        # 将数据和标签装载到设备
        wav = wav.to(device)
        label = label.to(device)
        out = model(wav) #用模型计算当前结果

        # 计算损失函数
        loss = criterion(out, label)
        train_loss = loss.data #记录训练loss
        bar.set_postfix(loss = "{:.5f}".format(train_loss)) # 更新进度条后面显示的内容
        optimizer.zero_grad()
        loss.backward()

        # 迭代
        optimizer.step()
    return train_loss

def val(model, device, dataLoader, criterion, epoch):
    r"""验证模型泛化情况
    Args:
        model: 要验证的模型
        device: 验证使用的设备
        dataLoader: 验证数据集
        criterion: 损失函数
        epoch: 当前轮数，给进度条显示使用
    """
    loss_val = []
    model.eval() # 将模型设置为验证模式，其中dropout等层会被关闭，可以加快推理速度
    bar = tqdm(dataLoader)
    bar.set_description("Epoch {} validate ".format(epoch + 1))
    for wav, label in bar:
        # 将数据和标签装载到设备
        wav = wav.to(device)
        label = label.to(device)
        out = model(wav) #用模型计算当前结果

        # 计算损失函数
        loss = criterion(out, label)
        loss_val.append(loss.item())
    
    return np.mean(loss_val)
    
def fit(model, train_data, val_data, device, criterion, optimizer, scheduler=None, epochs = 60):
    r"""拟合数据集
    Args:
        model: 要拟合的模型
        train_data: 训练集的dataLoader
        val_data: 验证集的dataLoader
        criterion: 损失函数
        optimizer: 优化器
        scheduler: 超参数修改管理器
    """
    for e in range(epochs):
        loss_train = train(model, device, train_data, criterion, optimizer, e)
        loss_val = val(model, device, val_data, criterion, e)

        """这里可以根据两个loss评估本轮训练情况，进行一些操作
        如：保存模型，修改超参数，停止训练等
        """
        print("train_loss: {:.5f} val_loss: {:.5f}".format(loss_train, loss_val)) # 输出当前结果
        torch.save(model, 'model.pkl') # 保存模型
        print("model saved!")
```
[进度条设置](https://blog.csdn.net/qq_41554005/article/details/117297861)
# torch利用模型进行推理

# 转onnx

https://mmdeploy.readthedocs.io/zh-cn/latest/tutorial/03_pytorch2onnx.html

# keras中的mel层
```python
def get_melspectrogram_layer(
    input_shape=None,
    n_fft=2048,
    win_length=None,
    hop_length=None,
    window_name=None,
    pad_begin=False,
    pad_end=False,
    sample_rate=22050,
    n_mels=128,
    mel_f_min=0.0,
    mel_f_max=None,
    mel_htk=False,
    mel_norm='slaney',
    return_decibel=False,
    db_amin=1e-5,
    db_ref_value=1.0,
    db_dynamic_range=80.0,
    input_data_format='default',
    output_data_format='default',
    name='melspectrogram',
):
    """A function that returns a melspectrogram layer, which is a `keras.Sequential` model consists of
    `STFT`, `Magnitude`, `ApplyFilterbank(_mel_filterbank)`, and optionally `MagnitudeToDecibel`.

    Args:
        input_shape (None or tuple of integers): input shape of the model. Necessary only if this melspectrogram layer is
            is the first layer of your model (see `keras.model.Sequential()` for more details)
        n_fft (int): number of FFT points in `STFT`
        win_length (int): window length of `STFT`
        hop_length (int): hop length of `STFT`
        window_name (str or None): *Name* of `tf.signal` function that returns a 1D tensor window that is used in analysis.
            Defaults to `hann_window` which uses `tf.signal.hann_window`.
            Window availability depends on Tensorflow version. More details are at `kapre.backend.get_window()`.
        pad_begin (bool): Whether to pad with zeros along time axis (length: win_length - hop_length). Defaults to `False`.
        pad_end (bool): whether to pad the input signal at the end in `STFT`.
        sample_rate (int): sample rate of the input audio
        n_mels (int): number of mel bins in the mel filterbank
        mel_f_min (float): lowest frequency of the mel filterbank
        mel_f_max (float): highest frequency of the mel filterbank
        mel_htk (bool): whether to follow the htk mel filterbank fomula or not
        mel_norm ('slaney' or int): normalization policy of the mel filterbank triangles
        return_decibel (bool): whether to apply decibel scaling at the end
        db_amin (float): noise floor of decibel scaling input. See `MagnitudeToDecibel` for more details.
        db_ref_value (float): reference value of decibel scaling. See `MagnitudeToDecibel` for more details.
        db_dynamic_range (float): dynamic range of the decibel scaling result.
        input_data_format (str): the audio data format of input waveform batch.
            `'channels_last'` if it's `(batch, time, channels)`
            `'channels_first'` if it's `(batch, channels, time)`
            Defaults to the setting of your Keras configuration. (tf.keras.backend.image_data_format())
        output_data_format (str): the data format of output melspectrogram.
            `'channels_last'` if you want `(batch, time, frequency, channels)`
            `'channels_first'` if you want `(batch, channels, time, frequency)`
            Defaults to the setting of your Keras configuration. (tf.keras.backend.image_data_format())
        name (str): name of the returned layer

    Note:
        Melspectrogram is originally developed for speech applications and has been *very* widely used for audio signal
        analysis including music information retrieval. As its mel-axis is a non-linear compression of (linear)
        frequency axis, a melspectrogram can be an efficient choice as an input of a machine learning model.
        We recommend to set `return_decibel=True`.

        **References**:
        `Automatic tagging using deep convolutional neural networks <https://arxiv.org/abs/1606.00298>`_,
        `Deep content-based music recommendation <http://papers.nips.cc/paper/5004-deep-content-based-music-recommen>`_,
        `CNN Architectures for Large-Scale Audio Classification <https://arxiv.org/abs/1609.09430>`_,
        `Multi-label vs. combined single-label sound event detection with deep neural networks <http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.711.74&rep=rep1&type=pdf>`_,
        `Deep Convolutional Neural Networks and Data Augmentation for Environmental Sound Classification <https://arxiv.org/pdf/1608.04363.pdf>`_,
        and way too many speech applications.

    Example:
        ::

            input_shape = (2, 2048)  # stereo signal, audio is channels_first
            melgram = get_melspectrogram_layer(input_shape=input_shape, n_fft=1024, return_decibel=True,
                n_mels=96, input_data_format='channels_first', output_data_format='channels_last')
            model = Sequential()
            model.add(melgram)
            # now the shape is (batch, n_frame=3, n_mels=96, n_ch=2) because output_data_format is 'channels_last'
            # and the dtype is float

    """
    backend.validate_data_format_str(input_data_format)
    backend.validate_data_format_str(output_data_format)

    stft_kwargs = {}
    if input_shape is not None:
        stft_kwargs['input_shape'] = input_shape

    waveform_to_stft = STFT(
        **stft_kwargs,
        n_fft=n_fft,
        win_length=win_length,
        hop_length=hop_length,
        window_name=window_name,
        pad_begin=pad_begin,
        pad_end=pad_end,
        input_data_format=input_data_format,
        output_data_format=output_data_format,
    )

    stft_to_stftm = Magnitude()

    kwargs = {
        'sample_rate': sample_rate,
        'n_freq': n_fft // 2 + 1,
        'n_mels': n_mels,
        'f_min': mel_f_min,
        'f_max': mel_f_max,
        'htk': mel_htk,
        'norm': mel_norm,
    }
    stftm_to_melgram = ApplyFilterbank(
        type='mel', filterbank_kwargs=kwargs, data_format=output_data_format
    )

    layers = [waveform_to_stft, stft_to_stftm, stftm_to_melgram]
    if return_decibel:
        mag_to_decibel = MagnitudeToDecibel(
            ref_value=db_ref_value, amin=db_amin, dynamic_range=db_dynamic_range
        )
        layers.append(mag_to_decibel)

    return Sequential(layers, name=name)
```