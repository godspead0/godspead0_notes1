# P1676 [USACO05FEB] Aggressive cows G

- 源文件：`洛谷/P_1676_USACO_05_FEB_Aggressive_cows_G.cpp`
- 题目链接：https://www.luogu.com.cn/problem/P1676#submit

```cpp
// P1676 [USACO05FEB] Aggressive cows G 
// URL：https://www.luogu.com.cn/problem/P1676#submit

// #include<bits/stdc++.h>
// using namespace std;
// const int N = 1e6;
// int a[N] = {0};
// int n,m;
// bool check(int mid){
//     int place = 0,ant = 1;
//     for(int i = 0;i < n;i++){
//         if(a[i] - a[place]>=mid){
//             ant++;
//             place = i;
//         }
//     }
//     if(ant>=m)return 1;
//     else return 0;
// }
// int main(){
//     cin>>n>>m;
//     for(int i = 0;i < n;i ++){
//         cin>>a[i];
//     }
//     sort(a,a+n);
//     int l = 0,r = a[n-1] - a[0];
//     int ans = 0;
//     while(l<r){
//         int mid = (l+r)>>1;
//         if(check(mid)){
//             ans = mid;
//             l = mid + 1;
//         }
//         else r = mid;
//     }
//     cout<<ans;
//     return 0;
// }
```
