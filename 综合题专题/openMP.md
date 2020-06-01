# openMP

## 常用制导参数

|         参数         |                       作用                       |
| :------------------: | :----------------------------------------------: |
|    `num_threads`     |               指定并行域内的线程数               |
|      `private`       | 指定一个或多个变量在每个线程中都有自己的私有副本 |
|    `firstprivate`    |            指定变量私有，并拷贝初始值            |
|    `lastprivate`     | 指定变量私有，初始值未知，最后结果拷贝进入主线程 |
|       `shared`       |     指定一个或多个变量为多个线程间的共享变量     |
| `omp_get_thread_num` |                  获得当前线程号                  |
|      `schedule`      |    指定for任务分担中的任务分配调度类型，可取     |



### reductin()

将每个线程计算得到的最终结果进行合并

```c++
#pragma omp parallel for num_threads(4), reduction(+:sum)
for(int i=0; i<maxn; i++){
	if(a[i] & 1) sum += a[i];
}
```



### 调度方式

静态调度`static`：每次哪些循环由哪个线程执行是固定的。由于每个线程的任务是固定的，但是可能有的循环任务执行快，有的慢，不能达到最优

动态调度`dynamic`：根据线程的执行快慢，已经完成任务的线程会自动请求新的任务或者任务块，每次领取的任务块是固定的

启发式调度`guided`：每个线程分配的任务是先大后小，指数下降。当有大量任务需要循环时，刚开始为线程分配大量任务，最后任务不多时，给每个线程少量任务，可以达到线程任务均衡



## 实例

### 数组a,b相加

```c++
#include <iostream>
#include <cstdio>
#include <omp.h>
#include <vector>
#include <time.h>
using namespace std;
const int maxn = 1e7;
int a[maxn], b[maxn]; 
int main()
{
	srand(time(0));
	for (int i = 0; i < maxn; ++i) {
		a[i] = rand() % 100 + 1;
		b[i] = rand() % 100 + 1;
	}
	double start = omp_get_wtime();
	std::vector<int>sum(maxn), sum1(maxn);
	for (int i = 0; i < maxn; ++i) {
		sum[i] =  a[i] + b[i];
	}
	double end = omp_get_wtime();
	printf("串行运行时间为%.6f\n", (end - start) / (CLOCKS_PER_SEC));;
	
    start = omp_get_wtime();
#pragma omp parallel for num_threads(4)
	for (int i = 0; i < maxn; ++i) {
		sum1[i] = a[i] + b[i];
	}
	end = omp_get_wtime();
	printf("4核并行运行时间为%.6f\n", (end - start) / (CLOCKS_PER_SEC));
    
	if (sum == sum1) {
		printf("并行结果无误\n");
	}
	system("pause");
}
```



### 矩阵乘法

```c++
#include <bits/stdc++.h>
#include <omp.h>
const int MAXN = 500;//矩阵大小
int a[MAXN][MAXN], b[MAXN][MAXN], ans1[MAXN][MAXN],ans2[MAXN][MAXN];
void init()
{ 
	for (int i = 0; i < MAXN; ++i)
	{
		for (int j = 0; j < MAXN; ++j)
		{
			a[i][j] = rand() % 100;
			b[i][j] = rand() % 100;
		}
	}
}

bool check()
{
	for (int i = 0; i < MAXN; ++i)
	{
		for (int j = 0; j < MAXN; ++j)
		{
			if (a[i][j] != b[i][j]) {
				return false;
			}
		}
	}
	return true;
}


int main()
{
	srand(time(0));
	omp_set_num_threads(8);
    
	//串行
	double start = omp_get_wtime();//获取起始时间
	for (int i = 0; i < MAXN; ++i)
	{
		for (int j = 0; j < MAXN; ++j)
		{
			int temp = 0;
			for (int k = 0; k < MAXN; ++k)
			{
				temp += a[i][k] * b[k][j];
			}
			ans1[i][j] = temp;
		}
	}
	double end = omp_get_wtime();
	printf("%.6f\n", end - start);

	//并行
	for (int t = 2; t <= 16; t*=2) {
		start = omp_get_wtime();//获取起始时间
		int temp = 0;
#pragma omp parallel for num_threads(t),reduction(+:temp)
		for (int i = 0; i < MAXN; ++i)
		{
			for (int j = 0; j < MAXN; ++j)
			{
				for (int k = 0; k < MAXN; ++k)
				{
					temp += a[i][k] * b[k][j];
				}
				ans2[i][j] = temp;
			}
		}
		end = omp_get_wtime();
		printf("%d线程并行时间%.6f\n", t, end - start);
		if (check()) {
			printf("结果正确\n");
		}
		else {
			printf("结果错误");
		}
	}
	system("PAUSE");
}
```

