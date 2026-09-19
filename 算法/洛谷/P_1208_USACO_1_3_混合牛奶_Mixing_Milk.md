# P1208 [USACO1.3] 混合牛奶 Mixing Milk

- 源文件：`洛谷/P_1208_USACO_1_3_混合牛奶_Mixing_Milk.cpp`
- 题目链接：https://www.luogu.com.cn/problem/P1208#submit

```cpp
// P1208 [USACO1.3] 混合牛奶 Mixing Milk
// https://www.luogu.com.cn/problem/P1208#submit

// #include<bits/stdc++.h>
// using namespace std;
// #define ll long long
// const int N = 1e6+10;
// struct milk{
//     int price,amount;
// }q[N];
// bool cmp(milk a,milk b){
//     return a.price<b.price;
// }
// int main(){
//     ll n,m;
//     cin>>n>>m;
//     for(int i=1;i<=m;i++){
//         cin>>q[i].price>>q[i].amount;
//     }
//     sort(q+1,q+m+1,cmp);
//     ll ans = 0;
//     for(int i = 1;i <= m;i ++){
//         if(n>=q[i].amount){
//             ans += q[i].price*q[i].amount;
//             n -= q[i].amount;
//         }
//         else{
//             ans += q[i].price*n;
//             break;
//         }
//     }
//     cout<<ans;
//     return 0;
// }
```
