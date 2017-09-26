---
layout: post
title: 从二分图探讨如何解决排课问题
date: 2017-09-26 20:41:39.000000000 +09:00
---
# 从二分图探讨如何解决排课问题
## 二分图
>二分图是一种图论问题，探讨相邻节点的染色问题

```CPP
#include <vector>
using namespace std;

const int MAX_N = 4;
vector<int> G[MAX_N];
int V = MAX_N;
int color[MAX_N] = {0};

//用于建立图，内部代码为举例说明使用
void InitGraph(void) {
	for (int i = 0; i < MAX_N; i++) {
		for (int j = 0; j < MAX_N; j++) {
			if (i != j) G[i].push_back(j);
		}
	}
}
//图的深层遍历，以使各个顶点染上不同颜色
bool dfs(int v, int c) {
	color[v] = c; // all v get c
	for (size_t i = 0; i < G[v].size(); i++) {
		// 如果下一个顶点的颜色和当前顶点相同则不再继续
		if (color[ G[v][i] ] == c) return false;
		// 如果下一顶点的颜色未配置，则向下进行染色
		if (color[ G[v][i] ] == 0 && !dfs(G[v][i], -c)) return false;
	}
	return true;
}
void solve(void) {
	InitGraph();
	//以每一个顶点为开始进行dfs染色
	for (int i = 0; i < V; i++) {
		if (color[i] == 0) {
			if (!dfs(i, 1)) {
				printf("No\n");
				return;
			}
		}
	}
	printf("Yes\n");
}

```
上部分的代码，用于染1和-1两种颜色，0为不染色。是最基础的染色算法。但是我想如果出现多种颜色可以用于染色，那么代码应该如何改变呢？
之前的颜色是通过赋值1、-1、1、-1、1...以此循环，这相当与形成了一个闭环。
当出现多色时，就需要让多个颜色出现环路，那么显然易见，使用%对N取模即可生成0~N-1的闭环。

## 多色图
```CPP
#include <vector>
using namespace std;

const int MAX_N = 3;
const int MAX_C = 3;
vector<int> G[MAX_N]; // Graph connect data
vector<int> color(MAX_N, -1); // 将各个顶点的颜色初始化为-1，未使用
int V = MAX_N;

void InitGraph(void) {
	for (int i = 0; i < MAX_N; ++i) {
		for (int j = 0; j < MAX_N; ++j)
		{
			if (i != j) G[i].push_back(j);
		}
	}
}
bool dfs(int v, int c) {
	//改变，对c取MAX_Color的模。
	color[v] = c % MAX_C;
	for (size_t i = 0; i < G[v].size(); ++i) {
		if (color[ G[v][i] ] == c) return false;
		// 改变，下一个图的颜色为c+1，但是我认为最优化的是通过不同颜色的使用次数，
		// 动态改变优先级。
		if (color[ G[v][i] ] == -1 && !dfs(G[v][i], c+1)) return false; 
	}
	return true;
}

void solve(void) {
	//与上一代码相同
}
```
既然可以解决图的多色不相邻问题，那么是否还能解决其他问题呢？
刚巧，我正好遇到了一个有关自动安排考场的问题，需求是将考生的考试科目输入，生成最有效的考场安排。
将问题抽象化，单个考生的所有科目相当于一个[完全图](https://baike.baidu.com/item/%E5%AE%8C%E5%85%A8%E5%9B%BE/10073908?fr=aladdin),每个顶点相连就是说需要每个顶点的颜色不同，那么需要的是将每个完全图拼合成一个完整的图，对所有的顶点进行染色，可以求得考试时间的最少安排。

## 安排课表解决方案

```CPP
#include <algorithm>
#include <vector>
using namespace std;

const int MAX_S = 3;
const int MAX_OBJ = 6;
static int MAX_T = 4; //TODO
typedef pair<int, int> P;

vector<int> stu[MAX_S]; //= { {1,2,3},{2,4,6},{3,4,5} };
vector<P> stu_G[MAX_S];
vector<int> G[MAX_OBJ];
int objects[MAX_OBJ];

//假设学生考试科目数据
void getStu(void) {
	stu[0] = { 1,2,3 };
	stu[1] = { 2,4,6 };
	stu[2] = { 3,4,5 };
}

//建立完全图，需要相当高的复杂度
void InitStu() {
	getStu();
	for (int i = 0; i < MAX_S; ++i) {
		int size = stu[i].size();
		for (int x = 0; x < MAX_S; x++) {
			for (int y = x; y < MAX_S; y++) {
				if (x != y) stu_G[i].push_back(P(stu[i][x], stu[i][y]));
			}
		}
	}
}
//通过学生的完全图构建考试图
void InitGraph(void) {
	InitStu();
	for (int i = 0; i < MAX_S; ++i) {
		for (int j = 0; j < stu_G[i].size(); ++j) {
			P &p = stu_G[i][j];
			G[p.first-1].push_back(p.second-1);
		}
	}
}

bool dfs(int i, int c) {
	objects[i] = c % MAX_T;
	for (int j = 0; j < G[i].size(); ++j) {
		if (objects[ G[i][j] ] == c) return false;
		if (objects[ G[i][j] ] == -1 && !dfs(G[i][j], c + 1)) return false;
	}
	return true;
}

bool solve(int i) {
	MAX_T = i;
	InitGraph();
	fill(objects, objects + MAX_OBJ, -1);

	for (int i = 0; i < MAX_OBJ; i++) {
		if (objects[i] == -1) {
			if (!dfs(i, 0)) {
				printf("No\n");
				return false;
			}
		}
	}
	printf("Yes\n");
	return true;
}

int main()
{
	for (int z = 1; z < MAX_OBJ; z++) {
		if(!solve(z)) continue;

		for (int i = 0; i < MAX_T; ++i) {
			printf("%c\t", 'A' + i);
			for (int j = 0; j < MAX_OBJ; ++j) {
				if (objects[j] == i) printf("%d ", j);
			}
			printf("\n");
		}
	}
	system("pause");
    return 0;
}
```
以下为结果
```result
No
No
No
Yes
A       0 4 5
B       1
C       2
D       3
Yes
A       0
B       1
C       2
D       3
E       4 5
```
