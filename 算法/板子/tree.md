# tree

- 源文件：`板子/tree.cpp`

```cpp
#include <iostream>
using namespace std;
struct TreeNode{
    int val;
    TreeNode* left;
    TreeNode* right;
};
//先序遍历
void preorder(TreeNode* root){
    if(root == nullptr){
        return;
    }
    cout << root->val << endl;
    preorder(root->left);
    preorder(root->right);
}
//中序遍历
void inorder(TreeNode* root){
    if(root == nullptr){
        return;
    }
    inorder(root->left);
    cout << root->val << endl;
    inorder(root->right);
}
//后序遍历
void postorder(TreeNode* root){
    if(root == nullptr){
        return;
    }
    postorder(root->left);
    postorder(root->right);
    cout << root->val << endl;
}
//插入节点
void insertNode(TreeNode* root, int val){
    if(root == nullptr){
        root = new TreeNode();
        root->val = val;
        return;
    }
    if(val < root->val){
        insertNode(root->left, val);
    }
    else{
        insertNode(root->right, val);
    }
}
//删除节点
void deleteNode(TreeNode* root){
    if(root == nullptr){
        return;
    }
    
}
//更新节点
void updateNode(TreeNode* root, int val){
    if(root == nullptr){
        return;
    }
    if(val < root->val){
        updateNode(root->left, val);
    }
    else if(val > root->val){
        updateNode(root->right, val);
    }
    else{
        root->val = val;
    }
}
//搜索节点
void searchNode(TreeNode* root, int val){
    if(root == nullptr){
        return;
    }
    if(val < root->val){
        searchNode(root->left, val);
    }
    else if(val > root->val){
        searchNode(root->right, val);
    }
    else{
        cout << "find" << endl;
    }
}
int main(){
    return 0;
}
```
