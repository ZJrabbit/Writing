# 滑动均值滤波的matlab实现和Java实现 #

最近时间在研究滤波算法，目的是为了更好的识别音频数据。因为有些音频数据里面的杂波太多，很难识别，所以需要先对其进行过滤，才能解析识别。为此，我先在matlab上做了仿真.采用的很多滤波算法，最后选择了对我这个效果最好的，滑动均值滤波。
## 什么是滑动均值滤波 ##

滑动平均滤波就是把连续取得的N个采样值看成一个队列，队列的长度固定为N，每次采样得到一个新数据放到队尾，并丢掉原来队首的一次数据，把队列中的N个数据进行平均运算，就可以获得新的滤波结果
## 具体的matlab代码
```matlab
clear
clc
load boxinfo.mat  %载入音频数据
T = data;
figure(1)
plot(T,'-*')
title('原始数据')
hold on;
%% 
%滑动平滑滤波
L = length(T);
N=0;  % 窗口大下
k = 0;
m =0 ;
for i = 1:L
    m = m+1;
    if i+N-1 > L
        break
    else
        for j = i:N+i-1
            k = k+1;
            W(k) = T(j) ;
        end
        T1(m) = mean(W);
        k = 0;
    end
end
plot(T1,'r-o')
grid
legend('原始数据','滤波之后')

```
### 滤波前后对比图
![滤波前后对比图](http://i.imgur.com/ISs0Ous.png)
### 简单分析一下
经过滑动滤波后，波形变得平滑很多。
### 改写成Java
matlab用来做验证，下面是具体的改写成Java代码后的均值滤波
```Java
/*
 * 滑动均值滤波 作者：ZJsnowman 时间：2015/9/10
 */
public class TestMovingAverageFilter {
    private static final int WINDOWS = 10;
    private static short[] temp = null; // 只声明暂时不初始化,用来记录最后得不到均值处理的点
    private static short[] bufout = null;

    // 均值滤波方法，输入一个buf数组，返回一个buf1数组，两者下表不一样，所以定义不同的下表，buf的下表为i，buf1的下表为buf1Sub.
    // 同理，临时的winArray数组下表为winArraySub
    public static short[] fiter(short[] buf) {
        int bufoutSub = 0;
        int winArraySub = 0;
        short[] winArray = new short[WINDOWS];

        if (temp == null) {
            bufout = new short[buf.length - WINDOWS + 1];
            for (int i = 0; i < buf.length; i++) {
                if ((i + WINDOWS) > buf.length) {
                    break;
                } else {
                    for (int j = i; j < (WINDOWS + i); j++) {
                        winArray[winArraySub] = buf[j];
                        winArraySub = winArraySub + 1;
                    }
                   
                    bufout[bufoutSub] = mean(winArray);
                    bufoutSub = bufoutSub + 1;
                    winArraySub = 0;
                }
            }
            temp = new short[WINDOWS - 1];
            System.arraycopy(buf, buf.length - WINDOWS + 1, temp, 0,
                    WINDOWS - 1);
            return bufout;
        } else {
            short[] bufadd = new short[buf.length + temp.length];
            bufout = new short[bufadd.length - WINDOWS + 1];
            System.arraycopy(temp, 0, bufadd, 0, temp.length);
            System.arraycopy(buf, 0, bufadd, temp.length, buf.length); // 将temp和buf拼接到一块
            for (int i = 0; i < bufadd.length; i++) {
                if ((i + WINDOWS) > bufadd.length)
                    break;
                else {
                    for (int j = i; j < (WINDOWS + i); j++) {
                        winArray[winArraySub] = bufadd[j];
                        winArraySub = winArraySub + 1;
                    }
                    bufout[bufoutSub] = mean(winArray);
                    bufoutSub = bufoutSub + 1;
                    winArraySub = 0;
                    System.arraycopy(bufadd, bufadd.length - WINDOWS + 1, temp,
                            0, WINDOWS - 1);
                }
            }
            return bufout;
        }

    }

    public static short mean(short[] array) {
        long sum = 0;
        for (int i = 0; i < array.length; i++) {
            sum += array[i];
        }
        return (short) (sum / array.length);
    }    
}
```
