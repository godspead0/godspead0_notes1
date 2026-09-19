# P1209 [USACO1.3] 修理牛棚 Barn Repair

- 源文件：`洛谷/P_1209_USACO_1_3_修理牛棚_Barn_Repair.cpp`
- 题目链接：https://www.luogu.com.cn/problem/P1209

```cpp
// P1209 [USACO1.3] 修理牛棚 Barn Repair
// https://www.luogu.com.cn/problem/P1209


// #include<bits/stdc++.h>
// using namespace std;
// #define ll long long
// const int N = 1e6+10;
// ll q[N],ans[N];
// int main(){
//     int m,s,c;
//     cin>>m>>s>>c;
//     if(m>=c){printf("%d",c);return 0;}
//     for(int i = 0;i < c;i ++){
//         cin>>q[i];
//     }
//     sort(q,q+c);
//     ll sum = 0;
//     sum+=q[c-1] - q[0] + m;
//     deque<int>dq;
//     for(int i = 0;i < c-1;i ++){
//         dq.push_back(q[i+1] - q[i]);
//     }
//     sort(dq.begin(),dq.end(),greater<int>());
//     int x = m-1;
//     while(x--){
//         sum-=dq.front();
//         dq.pop_front();
//     }
//     cout<<sum;
//     return 0;
// }
```
