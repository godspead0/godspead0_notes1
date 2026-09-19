# U514471 奶龙列阵（easy version）

- 源文件：`洛谷/U_514471_奶龙列阵_easy_version.cpp`
- 题目链接：https://www.luogu.com.cn/problem/U514471

```cpp
// U514471 奶龙列阵（easy version）
// https://www.luogu.com.cn/problem/U514471

// #include<bits/stdc++.h>
// using namespace std;
// int main(){
//     int n,a,b,c,aa,bb,cc;
//     cin>>n>>a>>b>>c;
//     aa = a,bb = b,cc = c;
//     long long res1 = 0,res2 = 0;
//     for(int i = 1;i <= n;i ++){
//         if(i%2==1){
//             if(a>=i){
//                 a -= i;
//             }
//             else{
//                 if(a+c>=i){
//                     c -= (i-a);
//                     a = 0;
//                 }
//                 else{
//                     res1 = i-1;
//                     break;
//                 }
//             }
//         }else{
//             if(b>=i){
//                 b -= i;
//             }
//             else{
//                 if(b+c>=i){
//                     c -= (i-b);
//                     b = 0;
//                 }
//                 else{
//                     res1 = i-1;
//                     break;
//                 }
//             }
//         }
//     }
//     a = aa,b = bb,c = cc;
//     for(int i = 1;i <= n;i ++){
//         if(i%2==0){
//             if(a>=i){
//                 a -= i;
//             }
//             else{
//                 if(a+c>=i){
//                     c -= (i-a);
//                     a = 0;
//                 }
//                 else{
//                     res2 = i-1;
//                     break;
//                 }
//             }
//         }else{
//             if(b>=i){
//                 b -= i;
//             }
//             else{
//                 if(b+c>=i){
//                     c -= (i-b);
//                     b = 0;
//                 }
//                 else{
//                     res2 = i-1;
//                     break;
//                 }
//             }
//         }
//     }
//     cout<<max(res1,res2);   
//     return 0;
// }
```
