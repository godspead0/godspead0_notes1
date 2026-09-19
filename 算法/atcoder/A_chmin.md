# A_chmin

- 源文件：`atcoder/A_chmin.cpp`

```cpp
#include<bits/stdc++.h>
using namespace std;
int main(){
    int a,x,n;
    cin>>n>>x;
    for(int i = 0;i < n;i ++){
        cin>>a;
        if(a < x){
            x = a;
            cout<<1<<endl;
        }else{
            cout<<0<<endl;
        }
    }
    return 0;
}
```
