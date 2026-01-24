# Checksums and error control
Chen Luo · Michael J. Carey    19 Jul 2019

## 目录

- [Abstract](#abstract)


## 5.1 Fletcher Checksum

Fletcher 校验和（Fletcher checksum）［6］［11］最初是为 OSI 通信模型的传输层（第 4 层）而设计的。该方法在本质上是一种“和的和（sum of sums）”的计算技术，所有加法运算均在模 255 的意义下进行。（但需要注意的是，255 并不是一个素数！）因此，我们计算如下过程。

$$
\begin{align*}
s1 &= (s1 + d_i) \bmod 255 \\
s2 &= (s2 + s1) \bmod 255
\end{align*}
$$

如果校验和位于消息末尾（这是通常的情况），则将两个校验字节设置为
$B_1 = s1 - s2$ 和 $B\_2 = -2s\_1 + s\_2$，
从而使**包括这两个校验字节在内的整体校验和为零**。与许多校验和方法不同，对正确传输的检测方式略有差异：当 $s1 = 0 \text{ or } s2 = 0$ 中任意一个成立时，结果即被认为是正确的；只有在 **两个累加和均非零** 的情况下，才会报告发生错误。

> 注：n1_s1=(s1+B1)=2s1-s2,n1_s2=(s2+n1_s1)=2s1 ==> n2_s1=(n1_s1+B2)=(2s1-s2-(-2s1+s2))=0,n2_s2=(n1_s2+n2_s1)=(2s1+0)=2s1

> 注："包括这两个校验字节在内的整体校验和为零"表示包含校验数据在内最终 s1 或 s2 任意一个为 0。

如果校验和字节位于一个长度为 $L$ 个八位字节（octet）的消息中的第 $n$ 和第 $n+1$ 个位置（消息位置编号为0 ... $L-1$），则

$$
\begin{align*}
b_n &= (L - n) \times s1 - s2 \\
b_{n+1} &= s2 - (L - n + 1) \times s1
\end{align*}
$$

> 注：上面公式最终计算结果 s2 = 0，s2 是“带位置权重”的和。

下面给出了一个 Java 代码片段，用于根据包含 (nChar) 个字符的数组 `c[]` 计算校验和。需要注意的是，这些字符应当限制为 **ASCII 字符**，或者至少其取值应 **< 256**。其中使用的 `while` 循环在功能上等价于**带余除法（division with remainder）**，但在本场景下通常只需进行极少次数的修正，因此这种实现方式在性能上更为高效。

```java
int s1 = 0, s2 = 0; // initialise checksums
for (int i = 0; i < nChars; i++) // scan the characters
  {
  s1 += c[i]; // add in the character
  while (s1 >= 255) // reduce modulo 255
    s1 -= 255;
  s2 += s1; // get the sum of sums
  while (s2 >= 255) // modulo 255
    s2 -= 255;
  }
```

Fletcher 校验和被认为其差错检测能力几乎与下文所描述的 CRC-16 校验和同样强大，能够检测到：

* 所有单比特错误，
* 所有双比特错误，
* 长度不超过 16 的突发错误中除 0.000019% 之外的全部错误，以及长度更长的突发错误中除 0.0015% 之外的全部错误。


## References

[Checksums and error control](https://pages.cs.wisc.edu/~remzi/OSTEP/Citations/checksums-03.pdf)