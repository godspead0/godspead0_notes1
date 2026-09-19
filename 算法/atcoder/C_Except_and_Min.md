# C_Except_and_Min

- 源文件：`atcoder/C_Except_and_Min.cpp`

```cpp
#include<bits/stdc++.h>
using namespace std;
int main(){
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int N, Q;
    cin >> N >> Q;
    vector<int> A(N+1); // 1-based indexing
    for(int i=1; i<=N; ++i){
        cin >> A[i];
    }
    // Create vector of (value, index) pairs and sort by value
    vector<pair<int, int>> min_candidates;
    for(int i=1; i<=N; ++i){
        min_candidates.emplace_back(A[i], i);
    }
    sort(min_candidates.begin(), min_candidates.end());
    // Take only the first 10 elements (since K <=5, 10 is enough)
    if(min_candidates.size() > 10){
        min_candidates.resize(10);
    }
    // Process queries
    while(Q--){
        int K;
        cin >> K;
        unordered_set<int> excluded;
        for(int j=0; j<K; ++j){
            int b;
            cin >> b;
            excluded.insert(b);
        }
        // Find the first candidate not in excluded set
        for(auto &[val, idx] : min_candidates){
            if(!excluded.count(idx)){
                cout << val << '\n';
                break;
            }
        }
    }
    return 0;
}
```
