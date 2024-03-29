# base64

参考：

[为什么要使用base64编码-知乎](https://www.zhihu.com/question/36306744)

[廖雪峰](https://www.liaoxuefeng.com/wiki/897692888725344/949441536192576)

---

[TOC]



## 为什么使用

在计算机中，字节共有256（$$2^8$$）个组合，对应ascii码，而ascii码的128～255之间的值是不可见字符

> 不可见字符：即控制字符，如回车、空格、警告等，可以用转义符合表示

由于不同的设备对不可见字符的处理会有不同，不利于传输，所以就先把数据统统变成可见字符，这样出错的可能性就大降低了

而Base64使用的64个字符都是可见字符，在大多数机器上的行为是一样的

## 怎么使用

原理：将 [0, 255] （$$2^8$$）的字符转移到 [0, 63]（$$2^6$$），保证他们都在可见字符范围内

1. 准备一个len = 64的列表，标准base64只有64个字符（英文大小写、数字和‘+’、‘/’）以及用作后缀的‘=’

2. 对二进制比特流重新划分：取6，8的公因数，把3个8位字节（3 * 8=24）转化为4个6位的字节（4 * 6=24），之后在6位的前面补两个0，形成8位一个字节的形式。 
   相当于取6个比特为一组，计算它的ascii值，得到一个字符，这个字符肯定是可见字符，再取6个比特，计算...，直到最后

   - 比如：

     转换前 10101101,10111010,01110110

     转换后 00101011, 00011011 ,00101001 ,00110110

     十进制 43 27 41 54

     对应64码中的值 r b p 2

     即编码后的Base64值为 rbp2

3. 如果要编码的二进制数据不是3的倍数，Base64用`\x00`字节在末尾补足后，再在编码的末尾加上1个或2个`=`号，表示补了多少字节，解码的时候，会自动去掉。

```python
import base64
base64.b64encode('binary\x00string')
# 'YmluYXJ5AHN0cmluZw=='
base64.b64decode('YmluYXJ5AHN0cmluZw==')
# 'binary\x00string'
```

## 运用到URL的问题

1. 由于标准的Base64编码后可能出现字符`+`和`/`，在URL中就不能直接作为参数，所以又有一种"url safe"的base64编码，把字符`+`和`/`分别变成`-`和`_`

   ```python
   base64.b64encode('i\xb7\x1d\xfb\xef\xff')
   # 'abcd++//'
   base64.urlsafe_b64encode('i\xb7\x1d\xfb\xef\xff')
   # 'abcd--__'
   base64.urlsafe_b64decode('abcd--__')
   # 'i\xb7\x1d\xfb\xef\xff'
   ```

2. `=`字符也可能出现在Base64编码中，但`=`用在URL、Cookie里面会造成歧义，所以很多Base64编码后会直接把`=`去掉 去掉`=`后怎么解码呢？因为Base64编码的长度永远是4的倍数，因此解码时，加上`=`把Base64字符串的长度变为4的倍数，就可以了

   ```python
   def b64decode_self(str):     
     return base64.b64decode(str+'='*(4-len(str)%4))
   ```

## 总结

Base64编码会把3字节的二进制数据编码为4字节的文本数据，长度增加1/3

base64编码后的字符串一定能被4整除（不算用作后缀的等号）

使用base64，是为了方便把含有不可见字符串的信息用可见字符串表示出来，以便文本数据可以在邮件正文、网页等直接显示

Base64是一种通过查表的编码方法，不能用于加密，即使使用自定义的编码表也不行

Base64适用于小段内容的编码，比如数字证书签名、Cookie的内容等