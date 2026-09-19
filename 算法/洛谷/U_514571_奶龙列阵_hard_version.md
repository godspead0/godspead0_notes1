# U514571 奶龙列阵（hard version）

- 源文件：`洛谷/U_514571_奶龙列阵_hard_version.cpp`
- 题目链接：https://www.luogu.com.cn/problem/U514571

```cpp
// U514571 奶龙列阵（hard version）
// https://www.luogu.com.cn/problem/U514571

// #include<bits/stdc++.h>
// using namespace std;
// #define ll long long 
// int a,b,c;
// bool check(int x){
//     int n = (x + 1) / 2, m = x - n;
//     if (n * (4 * n - 2) / 2 > a + c) {
//         return false;
//     }
//     if (m * (4 * m + 2) / 2 > b + c) {
//         return false;
//     }
//     int i = max(0, n * (4 * n - 2) / 2 - a);
//     int j = max(0, m * (4 * m + 2) / 2 - b);
//     return i + j <= c;
// }
// int main(){
//     ios::sync_with_stdio(0);cin.tie(0);cout.tie(0);
//     int t = 0;
//     cin>>t;
//     while(t--){
//         cin>>a>>b>>c;
//         ll l1 = 0;
//         ll r1 = 40000;
//         while(l1<r1){
//             ll mid = l1+r1+1>>1;
//             if(check(mid)){
//                 l1 = mid;
//             }else{
//                 r1 = mid-1;
//             }
//         }
//         swap(a,b);
//         ll l2 = 0;
//         ll r2 = 40000;
//         while(l2<r2){
//             ll mid = l2+r2+1>>1;
//             if(check(mid)){
//                 l2 = mid;
//             }else{
//                 r2 = mid-1;
//             }
//         }
//         cout<<max(l1,l2)<<"\n";
//     }
//     return 0;
// }
```
