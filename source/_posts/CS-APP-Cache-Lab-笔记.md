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

首先对 `32 x 32`，`64 x 64` 类型的矩阵进行 `8 x 8` 分块，对不是 8 的位数的 `61 x 67` 矩阵，进行额外处理。

如此得到的分数大概在 14 分，`64 x 64` 矩阵的分数为 0。