# P1182 数列分段 Section II

- 源文件：`洛谷/P_1182_数列分段_Section_II.cpp`
- 题目链接：https://www.luogu.com.cn/problem/P1182#submit

```cpp
// P1182 数列分段 Section II
// https://www.luogu.com.cn/problem/P1182#submit

// #include<bits/stdc++.h>
// using namespace std;
// const int N = 1e6+10;
// int a[N],n,m,l = 0,r = 0,ans = 0;
// bool check(int x){
//     int sum = 0,flag = 0;
//     for(int i = 0;i < n;i ++){
//         if(sum + a[i] <= x)sum+=a[i];
//         else sum = a[i],flag++;
//     }
//     return flag>=m;
// }
// int main(){
//     cin>>n>>m;
//     for(int i = 0;i < n;i ++){
//         cin>>a[i];
//         l = max(l,a[i]);
//         r+=a[i];
//     }
//     while(l<=r){
//         int mid = l+r+1>>1;
//         if(check(mid)){
//             ans = mid;
//             l = mid + 1;
//         }else{
//             r = mid - 1;
//         }
//     }
//     cout<<l;
//     return 0;
// }
```
