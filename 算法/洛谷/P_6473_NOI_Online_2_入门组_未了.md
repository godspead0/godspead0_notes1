# P6473 [NOI Online #2 入门组] 未了

- 源文件：`洛谷/P_6473_NOI_Online_2_入门组_未了.cpp`
- 题目链接：https://www.luogu.com.cn/problem/P6473#submit

```cpp
// P6473 [NOI Online #2 入门组] 未了
// https://www.luogu.com.cn/problem/P6473#submit

// #include<bits/stdc++.h>
// using namespace std;
// const int N = 1e6+10;
// double a[N],b[N];
// int main(){
//     long long n;
// 	double l,v,q;
//     cin>>n>>l>>v;
//     for(int i = 1;i <= n;i ++){
//         cin>>a[i];
//     }
// 	sort(a+1,a+1+n,greater<int>());
// 	b[0] = l/v;
// 	for(int i = 1;i <= n;i ++){
// 		b[i]=double(a[i])/v+b[i-1];
// 	}
//     cin>>q;
//     while(q--){
// 		double t;
// 		cin>>t;
// 		if(b[n]<=t){
// 			printf("-1\n");
// 			continue;
// 		}
// 		int l = 0,r = n,ans = 0;
// 		while(l<=r){
// 			int mid = (l+r)>>1;
// 			if(b[mid]>t){
// 				ans = mid;
// 				r = mid-1;
// 			}
// 			else{
// 				l = mid+1;
// 			}
// 		}
// 		printf("%d\n",ans);
// 	}
//     return 0;
// }
```
