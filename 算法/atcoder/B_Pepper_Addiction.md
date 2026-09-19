# B_Pepper_Addiction

- 源文件：`atcoder/B_Pepper_Addiction.cpp`

```cpp
#include<bits/stdc++.h>
using namespace std;
#define N 100005
int a[N];
int main(){
    int n,m,p,b;
    int sum = 0;
    cin>>n>>m;
    for(int i = 1;i <= m;i ++){
        cin>>a[i];
    }
    for(int i = 0;i < n;i ++){
        cin>>p>>b;
        if(b<=a[p]){
            sum += b;
            a[p] -= b;
        }else{
            sum += a[p];
            a[p] = 0;
        }
    }
    cout<<sum<<endl;
    return 0;
}
```
