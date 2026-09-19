# P1055 [NOIP 2008 普及组] ISBN 号码

- 源文件：`洛谷/P_1055_NOIP_2008_普及组_ISBN_号码.cpp`
- 题目链接：https://www.luogu.com.cn/problem/P1055#submit

```cpp
// P1055 [NOIP 2008 普及组] ISBN 号码
// https://www.luogu.com.cn/problem/P1055#submit

// #include<bits/stdc++.h>
// using namespace std;
// int main(){
//     char s[100] = {0};
//     int a[100] = {0};
//     int j = 0;
//     cin>>s;
//     for(int i = 0;s[i] != '\0';i++){
//         if(s[i] >= '0' && s[i] <= '9'){
//             a[j] = s[i] - '0';
//             j++;
//         }
//     }
//     long long sum = 0;
//     for(int i = 0;i < 9;i++){//这里千万不能写j-1不然会错
//         a[i]*=i+1;
//         sum+=a[i];
//     }
//     int det = sum%11;
//     if(det == a[j-1]){
//         cout<<"Right"<<endl;
//     }else{
//         s[12] = det+'0';
//         cout<<s;
//     }
//     return 0;
// }//这段代码是对的但是没有处理X我懒



// #include <bits/stdc++.h>
// using namespace std;
// int main() {
//     char s[100] = {0};  
//     int a[100] = {0};  
//     int j = 0;          
//     cin >> s;
//     for (int i = 0; s[i] != '\0'; i++) {
//         if (s[i] >= '0' && s[i] <= '9') {
//             a[j] = s[i] - '0';
//             j++;
//         }
//     }
//     long long sum = 0;
//     for (int i = 0; i < 9; i++) {  
//         sum += a[i] * (i + 1);
//     }
//     int checkCode = sum % 11;
//     // 校验码为 10 时，应为 'X'
//     if (checkCode == 10) {
//         if (s[12] == 'X') {
//             cout << "Right" << endl;
//         } else {
//             s[12] = 'X';  
//             cout << s << endl;
//         }
//     } else {
//         if (checkCode == (s[12] - '0')) {
//             cout << "Right" << endl;
//         } else {
//             s[12] = checkCode + '0';  // 修正校验码
//             cout << s << endl;
//         }
//     }
//     return 0;
// }
```
