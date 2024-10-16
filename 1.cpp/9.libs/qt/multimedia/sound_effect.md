`QSoundEffect` 提供以较低延迟的方式播放未压缩的音频文件（通常是 `.wav` 文件），适用于“反馈”类型的声音。

#### 用例

播放一次 `1.wav` 文件。

```cpp
auto ef = QSoundEffect{};
ef.setSource(QUrl::fromLocalFile("1.wav"));
ef.setLoopCount(1);
ef.setVolume(0.5);
ef.play();
```

- `setSource()`，加载音频文件。

- `setLoopCount()`，循环播放的次数，可以使用 `QSoundEffect::Infinite` 表示无限循环播放。

- `setVolume()`，设置音量大小，从 0~1 线性缩放。

- `play()`，开始播放。可以使用 `stop()` 停止播放。

#### 指定设备

qt6 开始，可以使用 `setAudioDevice()` 指定输出音频设备。
