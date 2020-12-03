---
title: FFT-guide
date: 2020-07-30 18:48:05
tags: FFT入门
mathjax: true 
toc: true 
---
<!-- toc -->
# FFT入门
## 预备知识

1. 信号 $s=A\cos(\omega t + \phi)$，其中$A$为信号的幅值，$\phi$为信号的相位。傅里叶变换就是要找到  $\omega$ ~$A$， 以及$\omega$ ~$\phi$  的关系。（一般我们处理数据大多使用$\omega$ ~$A$的关系）。![时域频域图](./FFT-guide/时域频域图.jpeg)
2. 所有的信号都可以由若干个不同幅值、相位、频率的余弦信号叠加而成即$s=A_1\cos(\omega_1+\phi_1)+A_2\cos(\omega_2+\phi_2)+A_3\cos(\omega_3+\phi_3)+...$   如下图多个频率的余弦逼近门函数
![逼近方波](./FFT-guide/正弦逼近方波.gif)

3. 傅里叶变换公式：
   $$ \mathscr{F}(\omega)=\mathscr{F}[f(t)]=\int_{-\infty}^{+\infty}f(t)e^{-j\omega t} \mathcal{d}t$$ 
   傅里叶逆变换
   $$\mathscr{f}(\omega)=\mathscr{F}^{-1}[f(\omega)]=\frac{1}{2\pi}\int_{-\infty}^{+\infty}\mathscr{F}(\omega)e^{j\omega t} \mathcal{d}\omega$$
   注：$\mathscr{F}[f(t)]$称之为对 $f(t)$的傅里叶变换

4. 这里需要简单介绍一下单位冲击函数$\delta(t)$，在图形上用一个长度为1的箭头表示如下图所示![Alt](FFT-guide/deltaFunc.png)
单位冲击函数具有如下性质
$$\int_{-\infty}^{+\infty} \delta(t)dt=\int_{0^+}^{0^-}\delta(t)dt=1
$$

5. cos的傅里叶变换为两个幅值为$\pi$冲击函数 $\delta(\omega)$
   $$\mathscr{F}[\cos(\omega_1t)]=\pi[\delta(\omega-\omega_1)+\delta(\omega+\omega_1)]
   $$
   称$\cos(\omega_1 t) \Leftrightarrow\pi[\delta(\omega-\omega_1)+\delta(\omega+\omega_1)]$为傅里叶变换对。
   ![cos的傅里叶变换](FFT-guide/cosineF.png)
在图形上是两个幅值为$\pi$频率$\omega=\pm\omega_1$的冲激函数 所以在乘上$\frac{1}{\pi}$后可以得到原来信号的幅值，由此我们找到了$\omega$ ~$A$的关系
 
6. 实际处理的数据不同于上面的连续信号，这里要引入一个采样率$Fs$即一秒钟对信号进行多少次采样。例如对于$\cos(\omega_1 t)$ 来说，以采样率$Fs=1000$得到的信号就是一个这样的序列：[$\cos(0.001\omega_1),\cos(0.002\omega_1),\cos(0.003\omega_1),...$]。
7. 由于实际处理的信号都是一个个离散的数值，我们对这些信号的处理称之为离散傅里叶变换DTF，在利用傅里叶变换的对称性优化后得到快速傅里叶变换即我们常用的FFT。(记住我们处理的是离散的数值序列就行。)

## 代码实操
这里用到的两个库: 
- `numpy` 包含常用的数学函数，我们要用的就是与fft相关的函数。
- `matplotlib` 常用的画图库，语法与`MATLAB`类似。


```python
import numpy as np
import matplotlib.pyplot as plt

Fs = 100 #每秒采样率
 
t = np.linspace(0,2,2*Fs) #2秒钟，每秒100个点 这句话是把[0,2]分成200个等间距序列
signal = np.cos(2*np.pi*1*t) 
plt.scatter(t,signal)#plt.plot(t,signal) 画曲线用plot 画离散点用scatter
plt.show()
```
当我们将t和signal的作为坐标值得出下图
![余弦离散点](FFT-guide/余弦离散点.png)
为了便于观看通常我们使用`plt.plot()`将相邻的点连接起来便会看到较为光滑的曲线。 
![余弦连线图](FFT-guide/余弦连线图.png)
采样率$Fs$越大，采集的点就会越密集。这里补充一个知识点：叫做奈奎斯特采样定理。简单的说就是我们可以采集到的频率范围是$[-\frac{Fs}{2},\frac{Fs}{2}]$

```python
import numpy as np
import matplotlib.pyplot as plt

Fs = 1000 #每秒采样率
 
t = np.linspace(0,2,2*Fs) #2秒钟，每秒1000个点
signal =2 * np.cos(2*np.pi*1*t) + 0.2*np.sin(2*np.pi*30*t)
plt.plot(t, signal)
plt.show()
```
这段程序中先确定了变量$t = [0,0.001,0.002,...,1.999,2]$
这里我们生成一个幅值为 2  频率为 1Hz 的余弦与一个幅值为  0.2  频率为  30Hz  的正弦相加$Signal = 2\cos(2\pi t)+0.2\sin(2\pi\times30t)$根据t的不同取值得到得到一个与之对应的信号序列。使用`plt.plot(t, signal)`画图后可以得到下图。
![低频余弦加高频正弦](FFT-guide/低频余弦加高频正弦.png)

接下来我们对这个信号进行傅里叶变换，具体实现如下。其中`np.fft.fftfreq(n, d=1/Fs)`是把计算的值换算到常用的以Hz为单位的坐标轴，范围就是之前提到的$[-\frac{Fs}{2},\frac{Fs}{2}]$。傅里叶变换得到的频率信号是[0, 0.1, ... , 499.9, 500,-500,-499.9, ..., -0.1]。`np.fft.fftshift(freq)`的作用是把频率从小到大重新排列。`F_disp`也是同理。
```python
F = np.fft.fft(signal)
n = signal.size
freq =  np.fft.fftshift( np.fft.fftfreq(n, d=1/Fs))
F_disp =  np.fft.fftshift(F)
plt.subplot(211), plt.plot(freq, (2*np.abs( F_disp )/n)) 
plt.subplot(212), plt.plot(freq, (np.angle( F_disp )/np.pi*180))
plt.show()
```
在二维平面上我们使用幅值和相位来描述这个信号。(`np.abs(a)`，若a是复数，返回a的模，否则就是a的绝对值。)通过`plt`画图得到下图，代表了幅值和相位随频率的变化。`(2*np.abs( F_disp )/n)`对应的就是预备知识中的第五点，连续函数乘上$\frac{1}{\pi}$， 离散序列乘上$\frac{1}{\frac{n}{2}}$也就是$\frac{2}{n}$。($2\pi$对应$n$， $\pi$对应$\frac{n}{2}$)
![fft结果](FFT-guide/fft_out.png)
对幅值和相位不是很敏感的同学可以绘制3D图来理解FFT的结果，这里我们设置x轴为频率，y轴为实部，z轴为虚部。
![3D_FFT](FFT-guide/3D_fft.gif)
绘制3D图的程序如下
```python
ax = plt.axes(projection='3d')
ax.set_zlim(-2,2)
ax.set_ylim(-2,2)
ax.set_xlim(-50,50)
ax.plot3D(freq  ,2*F_disp.imag/n, 2*F_disp.real/n)
plt.show()
```
由于我们的采样率$Fs = 1000$,这就意味着我们的的频率范围是$[-500,500]$。为了方便观察，我们将只选取频率在[-40, 40]的范围进行观察。

```python
freq_band = np.logical_and(freq>-40, freq<40)
plt.plot(freq[freq_band], (2*np.abs( F_disp[freq_band])/n)) 
```
![freq_band信号](FFT-guide/fft_out_band.png)
虽然点击对话框上的放大镜可以实现放大观察，但是`freq_band = np.logical_and(freq>-40, freq<40)`将[-40,40]之间的频率信号保留下来，为我们接下来的滤波打下基础。
![bandFreq_UI](FFT-guide/bandFreq_UI.png)
现实处理的信号也与我们之前处理的$Singal$类似，我们希望得到的信号通常是频率较低幅值较大，对应$2\cos(2\pi t)$，夹杂的噪声通常是频率较高幅值较小,对应$0.2\sin(2\pi\times30t)$
接下来我们将30Hz的正弦波滤去还原一个平滑的余弦波。
 - `np.logical_or(freq<-5, freq>5)`选取频率小于-5和大于5的区域，
 - `F_disp[freq_band_filter] = 0`将这些区域的数值置零
 - 绘制频谱发现30Hz处的尖峰已经滤除
  ![filter_freq](FFT-guide/filter_freq.png)
 - 由于我们刚刚处理的是平移过的频谱，因此在傅里叶逆变换还原信号前需要使用`np.fft.fftshift(F_disp)`进行频谱平移，再使用`np.fft.ifft()`还原信号
  ![filter_sig](FFT-guide/filter_sig.png)
- 滤除高频噪声后就能得到我们想要的低频信号。
```python
freq_band_filter = np.logical_or(freq<-5, freq>5)
F_disp[freq_band_filter] = 0
plt.plot(freq[freq_band], (2*np.abs( F_disp[freq_band])/n)) 
filter_sig = np.fft.ifft(np.fft.fftshift(F_disp))
plt.plot(t,filter_sig)
plt.show()
```
## 心电信号处理
### 数据库简介 Wrist PPG During Exercise


数据库包含了在步行，跑步和骑自行车过程中记录的腕部 PPGs。同时使用加速度计和陀螺仪进行运动估计，为消除运动对PPG干扰提供多种选择。并记录了胸部心电图作为运动心率的参考标准。

[心率数据 https://physionet.org/content/wrist/1.0.0/](https://physionet.org/content/wrist/1.0.0/)

Delaram Jarchi and Alexander J. Casson. Description of a Database Containing Wrist PPG Signals Recorded during Physical Exercise with Both Accelerometer and Gyroscope Measures of Motion. Data 2017, 2(1), 1; doi:10.3390/data2010001

Goldberger, A., Amaral, L., Glass, L., Hausdorff, J., Ivanov, P. C., Mark, R., ... & Stanley, H. E. (2000). PhysioBank, PhysioToolkit, and PhysioNet: Components of a new research resource for complex physiologic signals. Circulation [Online]. 101 (23), pp. e215–e220.

### ECG信号处理

``` python
import numpy as np
import matplotlib.pyplot as plt
import wfdb
```
心率数据数据都是使用 `WFDB` 格式保存的,这里我们需要加载wfdb的库。

``` python 
    path = ".\PPG\s2_walk"
    sampFrom = 0
    sampLen = 4000
    sampTo = sampFrom+sampLen

    record = wfdb.rdrecord(path, channels=[0,1],sampfrom=sampFrom, sampto=sampTo)
    annotation = wfdb.rdann(path, "atr", sampfrom=sampFrom, sampto=sampTo)
    wfdb.plot_wfdb(record=record,annotation=annotation,title="ecg",time_units="seconds")
    print("anno aver heart rate:",len(annotation.sample)*Fs/sampLen*60)
```
可以得到以下图形，上半部分为ECG数据，R峰处用红点标记（红点使用数据库自带的标记标出的）。下半部分图是同一时刻对应点PPG数据 。
![wfdb_plot](./FFT-guide/心率原始数据绘制.png)

接下来读取ECG信号波形做傅里叶变换，绘制频谱
```python
   signals, fields = wfdb.rdsamp(path, channels=[0,1], sampfrom=sampFrom, sampto=sampTo)
   ecg = signals[:,0]
   ppg = signals[:,1]
   Fs = fields['fs'] # 读取采样率
   n = ecg.size #采样点个数
   F_sig = np.fft.fft(ecg)#对ecg做傅里叶变换
   freq = np.fft.fftfreq(n, d=1/Fs)#折合到频域Hz
   plt.figure()
   plt.plot(freq[freq>0],2*np.abs(F_sig[freq>0])/n)#这里只绘制频率大于0的部分
```
![ecg频谱](FFT-guide/ECG频谱.png)

可以看到频率大多集中在低频部分，观察放大R峰的图形可以发现一个尖峰的频率在15-20Hz，因此我们将频谱中频率小于15Hz的剔除，再求逆变换。
```python
   filter_F = F_sig[:].copy()#先复制一份频域数据
   filter_F[np.abs(freq)<15] = 0 #高通滤波 低于15Hz 的都是0  ，观察图像可知 R峰的频率在15 - 20 Hz
   filter_sig = np.fft.ifft(filter_F).real #傅里叶逆变换取实部
   plt.figure()
   plt.plot(filter_sig)
```
![高通滤波后](FFT-guide/高通滤波后.png)
观察滤波后的图形可以看出R峰在图中已经十分明显。
```python
    winSize = 25 #设置一个窗口大小 
    thre = 5.5 * np.mean(np.abs(filter_sig))#对滤波后的数据取绝对值再求平均 乘上一个系数作为 选择R峰点的一个门限
    sq_sig = filter_sig.copy().reshape(int(filter_sig.size/winSize),winSize)#将滤波后的数据 以winSize为一排 重新排列  这里将其变换成 25 * 160的矩阵
    peakList = [] #先设置一个R峰列表 所有的R峰数据都存进去
    for i in range(len(sq_sig)):#对每一行数据进行处理
        norm = np.abs(sq_sig[i]) #对每一行数据取模
        aver = np.mean(norm) #每一行数据的模求平均
        win_max = np.max(sq_sig[i])#每一行数据的最大值
        if np.abs(win_max/aver) < 2.5 or win_max < thre: 
            sq_sig[i][:] = 0 #如果最大值/平均值<2.5 或者最大值小于门限值 则该行清零
        else:
            localIndex = sq_sig[i].tolist().index(win_max)#找到最大值在该行的索引
            peakList.append(localIndex + winSize * i)#求出最大值在整个信号上的索引
    new_sig = sq_sig.flatten() #将矩阵重整为一维数据


    sortPeakList =[peakList[0]]#离得太近的最大值取较大的哪一个 存入这个列表中
    for i in range(1,len(peakList)):#第一个不用处理
        front = peakList[i]
        behind = sortPeakList[-1]#列表中的最后一个值
        if  front- behind < 20:#当下一个值与整理后的的值太近时
            if ecg[front] >ecg[behind]: #删去小的 存入大的
                sortPeakList.pop(-1)
                sortPeakList.append(front)
        else: #如果距离足够远则直接存入
            sortPeakList.append(front)
    # print(len(sortPeakList))
    yPeak = [ecg[x] for x in sortPeakList]#根据刚刚整理过的R峰横坐标从数据中找出纵坐标y       
    plt.figure()
    print("calc aver heart rate:",len(sortPeakList)*Fs/sampLen*60)
    plt.subplot(212)
    plt.plot(ecg)#在ecg上画出找到的R峰
    plt.scatter(sortPeakList, yPeak, color='red')

    plt.subplot(211)
    plt.plot(filter_sig,color="g")#绿色为15Hz高通滤波后数据
    plt.plot(new_sig,color = "r")#红色是找R峰处理后的数据
```
![求出R峰](FFT-guide/自己标记R峰.png)
可以看到自己求得的R峰与最初数据库的值几乎一致，实现了对ECG信号找R峰求心率的目的。