---
title: 'CS: APP - Cache Lab 笔记'
date: 2018-09-07 11:20:19
categories:
	- 学习笔记
tags:
	- CSAPP
	- C
	- 学习笔记
---

本次 Lab 通过模拟高速缓存、优化程序性能来帮助理解高速缓存。

## Part A

第一部分即是通过程序模拟高速缓存，使用 LRU（Least-recently used）替换策略，即最后一次使用时间距离现在最远的被替换。

程序正确性检验：

```bash
$ make clean
$ make
$ ./test-csim
```

### 解析命令行

使用  `getopt` 函数来解析命令行。

<!-- more -->

```c
/**
 * 处理命令行参数。
 * 没有处理参数不足的情况
 * optstring = "s:E:b:t:vh?"
 */
void parseCommand(int argc, char **argv, char *optstring) {
	int opt = getopt(argc, argv, optstring);
	while(opt != -1) {
		switch(opt) {
			case 'h':
				globalArgs.h = 1;
				break;
			case 'v':
				globalArgs.v = 1;
				break;
			case 's':
				globalArgs.s = *optarg - '0';
				break;
			case 'E':
				globalArgs.E = *optarg - '0';
				break;
			case 'b':
				globalArgs.b = *optarg - '0';
				break;
			case 't':
				globalArgs.t = optarg;
				break;
			default:
				printf("unknown option, some error occurred");
				exit(1);
		}
		
		opt = getopt(argc, argv, optstring);
	}
}
```

使用结构来存储变量：

```c
struct globalArgs_t {
	int s;
	int E;
	int b;
	int v;
	int h;
	char *t;
	
} globalArgs;
```

### 分配、回收高速缓存

使用 `malloc` 与 `free` 函数。 

```
void init_cache() {
	int s = globalArgs.s;
	int S = 1 << s;
	int E = globalArgs.E;
	group = (cache_line_t *) malloc(sizeof(cache_line_t) * S * E);
	if (group == NULL) {
		printf("cannot allocate memory\n");
		exit(1);
	}
	for (int i = 0, size = S * E; i < size; i++) {
		group[i].valid = 0;
		group[i].tag_bit = 0;
		group[i].used = 0;
	}
}

void free_cache() {

	free(group);
	group = NULL;
}
```

使用的结构体为：

```c
typedef struct {
	int valid;
	int tag_bit;
	unsigned int used;
} cache_line_t;

cache_line_t *group;
```

### 读取指令

从文件中读取指令，最先一行可能会读取两次，所以需要额外

```c
#define BUFFER_SIZE 1024


	char strbuf[BUFFER_SIZE];
	FILE *fp;
	
	if ((fp = fopen(globalArgs.t, "r")) == NULL) {
		printf("cannot open file: %s\n", globalArgs.t);
		return -1;
	}
	
	while(!feof(fp)) {
		fgets(strbuf, BUFFER_SIZE, fp);
		if (strbuf[strlen(strbuf) - 1] == '\n') {
			strbuf[strlen(strbuf) - 1] = '\0';
		}
		if (!feof(fp)) {
			//printf("%s\n", strbuf);
			 parse_line(strbuf);
		}		

	}
```

### 处理指令

其实 `Load`、`Save`、`Modify` 三个指令几乎相同，只有 `Modify` 是要取两次，而且第二次必定 `hit`。

```c
void parse_line(char * strbuf) {
	if (strbuf[0] == 'I') {
		return;
	}
	char *pEnd;
	long int addr, size = 0;
	addr = strtol(strbuf + 3, &pEnd, 16);
	size = strtol(pEnd + 1, NULL, 16);
	
	int S = (addr >> globalArgs.b) & ((1 << globalArgs.s) - 1);
	int tag_bit = (addr >> (globalArgs.b + globalArgs.s));
	int B = addr & ((1 << globalArgs.b) - 1);
	//printf("S: 0x%8x, tag_bit: %8x, B: %8d, size: %8d\t\t", S, tag_bit, B,(int)size);
	char * desc = "";
	desc = parse_load(S, tag_bit, B, size);

	if (strbuf[1] == 'M') {
		hit++;
		printf("%s %s hit\n", strbuf, desc);
	} else {
		printf("%s %s\n", strbuf, desc);
	}

}
```

`parse_load` 函数：

```c
#define IDX(S, i, E) ((S) * (E) + (i))


int hit = 0;
int miss = 0;
int evict = 0;
int counter = 1;
char * parse_load(int S, int tag_bit, int B, int size) {
	int E = globalArgs.E;
    // 命中即退出
	for (int i = 0; i < E; i++) {
		if (group[IDX(S, i, E)].valid && group[IDX(S, i, E)].tag_bit == tag_bit) {
			hit++;
			group[IDX(S, i, E)].used = counter++;
			return "hit";
		}
	}
    // 否则必定 miss
	miss++;
    // 尚有 valid = 0 的缓存行
	for (int i = 0; i < E; i++) {
		
		if (group[IDX(S, i, E)].valid == 0) {
			group[IDX(S, i, E)].valid = 1;
			group[IDX(S, i, E)].tag_bit = tag_bit;
			group[IDX(S, i, E)].used = counter++;
			return "miss";
		}
	}
    // 没有的话，必 eviction
	evict++;
	int min_idx = -1;
	int min_val = counter;
	for (int i = 0; i < E; i++) {
		if (group[IDX(S, i, E)].used < min_val) {
			min_idx = i;
			min_val = group[IDX(S, i, E)].used;
		}
	}
	if (min_idx != -1) {
		group[IDX(S,min_idx, E)].tag_bit = tag_bit;
		group[IDX(S,min_idx, E)].used = counter++;
	}
	return "miss evict";
}
```

## Part B

优化矩阵转置函数，以减少缓存不命中数。

查看分数：

```bash
$ ./driver.py
```

首先对 `32 x 32`，`64 x 64` 类型的矩阵进行 `8 x 8` 分块，对不是 8 的位数的 `61 x 67` 矩阵，进行额外处理。

如此得到的分数大概在 14 分，`64 x 64` 矩阵的分数为 0。

对 `32 x 32` 矩阵，每一行都有 `4 * 32 = 128 byte`。使用的是 `(s=5, E=1, b=5)` 的缓存结构，所以我们能用 8 个缓存行。（$2^{s +b} / 128 = 2^{3} = 8$）

```c
	int i, j, tmp;
	int stride = 8, a, b, tmp1, tmp2, tmp3, tmp4, rm, rn;
	
	
	// M 與 N 不相等，或者 M 不是 8 的整數倍  
	if ((M != N) || (M & 7) || (N & 7)) {
	    //int dm = M & (~7);
	    rm = M & 7;
	    
	    //int dn = N & (~7);
	    rn = N & 7;
	    
	    for (i = N - rn; i < N; i++) {
	    	for (j = 0; j < M - rm; j++) {
	    		tmp = A[i][j];
	    		B[j][i] = tmp;
	    	}
	    }
	    for (i = 0; i < N; i++) {
	    	for (j = M - rm; j < M; j++) {
	    		tmp = A[i][j];
	    		B[j][i] = tmp;
	    	}
	    }
	    
	    M = M - rm;
	    N = N -rn;
	}
	for (i = 0; i < N; i+=stride) {
		for (j = 0; j < M; j+=stride) {      
			for (a = i; a<i+stride;a++) {  
				tmp = A[a][j];
				tmp1 = A[a][j + 1];
				tmp2 = A[a][j + 2];
				rm = A[a][j + 3];
				rn = A[a][j + 4];
				b = A[a][j + 5];
				tmp3 = A[a][j + 6];
				tmp4 = A[a][j + 7];
				        
					         
				B[j][a] = tmp;
				B[j+1][a] = tmp1;
				B[j+2][a] = tmp2;
				B[j+3][a] = rm;
				B[j+4][a] = rn;
				B[j+5][a] = b;
				B[j+6][a] = tmp3;
				B[j+7][a] = tmp4;        
				
			}
		}
	}
```

进一步优化。主要是 `64 x 64` 矩阵优化。

通过以下指令查看缓存命中情况：

``` bash
$ ./test-tsim -M 64 -N 64

$ ./csim-ref -v -s 5 -E 1 -b 5 -t trace.f0
```

其中 `trace.f0` 表示的是 `registerFunctions` 函数中第一个被注册的函数的缓存使用情况。

分析得知，`64 x 64` 矩阵只能使用 4 个缓存行（$2^{s + b} / (4 * 64) = 2^2 = 4$）。所以使用 `8 x 8` 分块会导致大量的 `miss eviction`。所以，改为 `4 x 4` 分块，对 `64 x 64` 进行特别处理。

```c
	int i, j, tmp;
	int stride = 8, a, b, tmp1, tmp2, tmp3, tmp4, rm, rn;

	if (M == 64 && N == 64) {
		for (i = 0; i < 64; i += 8) {
			for (j = 0; j < 64; j += 8) {
                
				tmp = A[i][j];
				tmp1 = A[i][j+1];
				tmp2 = A[i][j+2];
				stride = A[i][j+3];
				
				rm = A[i+1][j];
				rn = A[i+1][j+1];
				b = A[i+1][j+2];
				M = A[i+1][j+3];
				
				tmp3 = A[i+2][j];
				tmp4 = A[i+2][j+1];
				a = A[i+2][j+2];
				N = A[i+2][j+3];
				
				B[j][i] = tmp;
				B[j][i+1] = rm;
				B[j][i+2] = tmp3;

				
				tmp = A[i+3][j+1];
				rm = A[i+3][j+2];
				tmp3 = A[i+3][j+3];
				B[j][i+3] = A[i+3][j];
				
				B[j+1][i] = tmp1;
				B[j+1][i+1] = rn;
				B[j+1][i+2] = tmp4;
				B[j+1][i+3] = tmp;
				
				B[j+2][i] = tmp2;
				B[j+2][i+1] = b;
				B[j+2][i+2] = a;
				B[j+2][i+3] = rm;
				
				B[j+3][i] = stride;
				B[j+3][i+1] = M;
				B[j+3][i+2] = N;
				B[j+3][i+3] = tmp3;

			}
		}
		return;
	}
```

这样得到的 `miss` 数为 `1603`。达到了瓶颈。但满分要求的是 `1300` 及以下，所以应该还有什么奇技淫巧可以用。

先看图。对一个 `8 x 8` 的分块：

A 矩阵，左上角坐标 `(i, j)`。

```
0	0	0	0	0_	0_	0_	0_ 
1	1	1	1	1_	1_	1_	1_
2	2	2	2	2_	2_	2_	2_
3	3	3	3	3_	3_	3_	3_
------------------------------
------------------------------
------------------------------
------------------------------
```

转置到 B 矩阵时，不求一步到位，先变成下面的样子：

B 矩阵，左上角坐标 `(j, i)`。

```
0	1	2	3	0_	1_	2_	3_ 
0	1	2	3	0_	1_	2_	3_ 
0	1	2	3	0_	1_	2_	3_ 
0	1	2	3	0_	1_	2_	3_ 
------------------------------
------------------------------
------------------------------
------------------------------
```

原本应该到 B 矩阵**左下**的部分被分配到了 **B 的右上**。这样我们可以最大化利用 4 个缓存行而不用担心 `miss eviction`。

下一步是把 **A 的左下**转置到 **B 的右上**，同时把 **B 的右上** 移动到 **B 的左下**。这样我们还是可以只用处理矩阵中的 4 行，充分利用缓存。

最后一步是把 A 的右下转置到 B 的右下，没有复杂操作，直接转换就好。

代码如下：

```c
// 第一步
	for (a = i; a < i+4; a++) {
		tmp1 = A[a][j];
		tmp2 = A[a][j+1];
		tmp3 = A[a][j+2];
		tmp4 = A[a][j+3];
		
		tmp = A[a][j+4];
		rm = A[a][j+5];
		rn = A[a][j+6];
		stride = A[a][j+7];
					
					
		B[j][a] = tmp1;
		B[j][a+4] = tmp;
		B[j+1][a] = tmp2;
		B[j+1][a+4] = rm;
		B[j+2][a] = tmp3;
		B[j+2][a+4] = rn;
		B[j+3][a] = tmp4;
		B[j+3][a+4] = stride;
	}
// 第二步			
	for (a = j; a < j+4; a++) {
		tmp = B[a][i+4];
		rm = B[a][i+5];
		rn = B[a][i+6];
		stride = B[a][i+7];
		
		tmp1 = A[i+4][a];
		tmp2 = A[i+5][a];
		tmp3 = A[i+6][a];
		tmp4 = A[i+7][a];
		
		B[a][i+4] = tmp1;
		B[a][i+5] = tmp2;
		B[a][i+6] = tmp3;
		B[a][i+7] = tmp4;
					
		B[a+4][i] = tmp;
		B[a+4][i+1] = rm;
		B[a+4][i+2] = rn;
		B[a+4][i+3] = stride;
	}

// 第三步			
	for (a = j+4; a < j + 8; a++) {
		tmp1 = A[i+4][a];
		tmp2 = A[i+5][a];
		tmp3 = A[i+6][a];
		tmp4 = A[i+7][a];
		
		B[a][i+4] = tmp1;
		B[a][i+5] = tmp2;
		B[a][i+6] = tmp3;
		B[a][i+7] = tmp4;
	}
```

这样，第一步有 8 个 miss，第二步有 8 个 miss，第三步有 4 个 miss，一共 $(8+8+4) * ((64 * 64)/(8*8)) = 1280$ 个 miss。实际只有 1219 个 miss。

## 小结

总得来说这个 lab 并不难，但是优化起来比较花时间。

最后的奇技淫巧是从网上找来的，自己只想到 `miss = 1603` 的版本，而且 `61 x 67` 的矩阵也没有做到满分，说明还要努力！