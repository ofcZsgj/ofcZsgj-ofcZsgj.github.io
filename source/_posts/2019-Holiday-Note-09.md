---
title: Holiday Note_09
date: 2019-07-21 20:06:44
categories: C
tags: [Note,C,DataStructure]
---

# Holiday Note_09

**二叉树**，递归递归递归。纸上画了一遍递归遍历二叉树的函数调用全过程😵。值得注意的是，拷贝二叉树时，需要使用malloc给每一个结点分配内存，否则函数结束栈区内存销毁，拷贝的数据均会消失。

<!-- more -->

## 二叉树的结构定义
~~~
struct BinaryTreeNode {
	char ch;//简单定义的一个数据类型，不够规范。
	struct BinaryTreeNode* left;
	struct BinaryTreeNode* right;
};
~~~


## 遍历

以中序遍历为例，思路为：要想进行中序遍历，先要遍历根节点的左子树，再遍历右子树，最后是根。要想遍历根节点的左子树，要先遍历根结点的左子树的左子树，再遍历根结点的左子树的右子树，再是根节点的左子树。。。循环反复，结点为空返回。
~~~
void recursion(struct BinaryTreeNode* root) {

	//返回条件
	if (NULL == root) {
		return;
	}

	//此为先序遍历打印
	printf("%c ", root->ch);
	recursion(root->left);
	recursion(root->right);

	//此为中序遍历打印
	recursion(root->left);
	printf("%c ", root->ch);
	recursion(root->right);

	//此为后序遍历打印
	recursion(root->left);
	recursion(root->right);
	printf("%c ", root->ch);
}
~~~

## 求二叉树的叶子数

叶子即没有左子树和右子树的结点。和遍历整颗二叉树的思路相同，只不过多了一个判断条件对叶子数进行记录。
~~~
void getTreeLeafNumber(struct BinaryTreeNode* root, int* leafNum) {

	if (NULL == root) {
		return;
	}

	if (root->left == NULL && root->right == NULL) {
		//*和++运算等级一样，因此注意加括号
		(*leafNum)++;
	}

	getTreeLeafNumber(root->left, leafNum);
	getTreeLeafNumber(root->right, leafNum);

}
~~~

## 二叉树高度

高度简单理解为一颗树向下延申的最高层数，根节点记为一层，有子节点增加一层以此类推。

思路为：要想知道这颗二叉树的高度，就要知道左子树和右子树的高度分别为多少，对其比较取最大值。每一个结点都是一棵树，理解这个就明白了。
~~~
int getBinaryHeight(struct BinaryTreeNode* root) {

	if (NULL == root) {
		return 0;
	}

	int leftHeight = getBinaryHeight(root->left);
	int rightHeight = getBinaryHeight(root->right);

	int height = leftHeight > rightHeight ? leftHeight + 1 : rightHeight + 1;

	return height;

}
~~~

## 二叉树的拷贝

注意，需要**对每一个拷贝的结点分配空间**，因此对应分配空间必定需要**释放空间**。

如果不分配空间的话，拷贝的只是一块不再是原先存放拷贝内容的地址了，这是**内存的四区模型**知识点，切记。
~~~
struct BinaryTreeNode* copyTree(struct BinaryTreeNode* root) {

	if (NULL == root) {
		return NULL;
	}

	struct BinaryTreeNode* left = copyTree(root->left);
	struct BinaryTreeNode* right = copyTree(root->right);

	struct BinaryTreeNode* node = malloc(sizeof(struct BinaryTreeNode));

	node->left = left;
	node->right = right;
	node->ch = root->ch;

	return node;

}
~~~

PS:准备换电脑了💻