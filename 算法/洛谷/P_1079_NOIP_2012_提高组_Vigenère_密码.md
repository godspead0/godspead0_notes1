# P1079 [NOIP 2012 提高组] Vigenère 密码

- 源文件：`洛谷/P_1079_NOIP_2012_提高组_Vigenère_密码.cpp`
- 题目链接：https://www.luogu.com.cn/problem/P1079

```cpp
// P1079 [NOIP 2012 提高组] Vigenère 密码
// https://www.luogu.com.cn/problem/P1079

// 我来发个代码稍短一点的题解。
// 铺垫一个关于ASCII码的小知识：字母'A'的ASCII码是41H（0100 0001B），字母'a'的ASCII码是61H（0110 0001B），字母'A'与'a'的二进制后5位是相同的，所以无论是大写字母还是小写字母x，x &31（1 1111B）的值就是x在字母表里的顺序。
// 开始题解：本题的目的就是问你如何去篡位，例如秘钥k里面的字符'C'代表的就是明文向后篡2位得到密文，反过来已知密文，'C'代表的就是向前篡2位，这样'Y'就得到了'W'。
// 而篡位是有边界的，就是如果篡过了头，需要再回来，A之后就是Z了。所以需要判断一下密文的和篡位之间的大小，如果篡位大，需要补26.
// 参考代码：

// #include <iostream>
// using namespace std;
// int main() {
// 	string k,c;
// 	cin>>k>>c;
// 	for (int i=0;i<c.length();i++) {
// 		int t=(k[i%k.length()]&31)-1;
// 		c[i]=(c[i]&31)-t>0?c[i]-t:c[i]-t+26;
// 	}
// 	cout<<c<<endl;
// 	return 0;
// }
```
