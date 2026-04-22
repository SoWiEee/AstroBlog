---
title: PP code 0422
published: 2026-04-22
description: ''
image: ''
tags: [CS]
category: ''
draft: false 
lang: ''
---

## Thread Index

- 編譯：`g++ -fopenmp for.c -o for`
- 執行：`.\for 4 20`（開 4 個 thread，跑 20 次迴圈）

```c
#include <omp.h>
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <Windows.h>

int main (int argc, char *argv[])
{
    // ex: ./for 4 20 (表示開 4 個執行緒，跑 20 次迴圈)
    assert(argc == 3);
    
    omp_set_num_threads(atoi(argv[1]));
    int n = atoi(argv[2]);
    
    printf("# of proc = %d\n", omp_get_num_procs());
    printf("# of loop iterations = %d\n", n);
    
    // 開啟平行迴圈區塊
    #pragma omp parallel for
    for(int i = 0; i < n; i ++) {
        // 印出：哪個 thread、負責哪個 loop index、在系統哪個 core 上跑
        printf("thread %d runs index %d in core %lu.\n",
               omp_get_thread_num(), i, GetCurrentProcessorNumber());
    }
    
    // 程式執行到這裡之前，所有執行緒會自動等待彼此 (自動同步)
    return 0;
}
```


## Parallel Linear Search using OpenMP

- $n$ 改成電腦的核心數
- 編譯：`g++ -fopenmp linear_search.c -o linear_search`
- 執行：`.\linear_search`

```c
#include <stdio.h>
#include <stdlib.h>
#include <omp.h>

void linear_search_parallel(int data[], int key, int num, int tid)
{
    int i;
    for (i = 0; i < num; i++) {
        if (data[i] == key) {
            printf("[Parallel] Key found. Position = %d by thread %d \n", tid * num + i, tid);
        }
    }
}

void linear_search_sequential(int data[], int key, int num)
{
    int i;
    for (i = 0; i < num; i++) {
        if (data[i] == key) {
            printf("[Sequential] Key found. Position = %d\n", i);
        }
    }
}

int main()
{
    // 將資料量放大到 1 億，否則搜尋時間太短無法比較出差異
    int maxn = 100000000; 
    int key = 98765432; // 設定一個靠近陣列尾端的值
    int *a = (int*)malloc(maxn * sizeof(int));
    int i, num, step, tid;
    int n = 12; // 執行緒數量
    double start_time, seq_time, par_time;

    if (a == NULL) {
        printf("Memory allocation failed!\n");
        return -1;
    }

    // 初始化陣列
    for (int k = 0; k < maxn; k++) {
        a[k] = k;
    }

    printf("Array size: %d, Target Key: %d, Threads: %d\n\n", maxn, key, n);

    // 測量循序線性搜尋的時間
    start_time = omp_get_wtime();
    linear_search_sequential(a, key, maxn);
    seq_time = omp_get_wtime() - start_time;
    printf("Linear (Sequential) Time : %f seconds\n\n", seq_time);

    // 測量平行線性搜尋的時間
    start_time = omp_get_wtime();
    #pragma omp parallel num_threads(n) private(i, num, step, tid)
    {
        tid = omp_get_thread_num();
        num = maxn / n;           // 每個 thread 負責的資料量
        step = tid * num;         // 每個 thread 的起始索引
        
        // 將對應區塊的記憶體位址傳入函式中
        linear_search_parallel(&a[step], key, num, tid);
    }
    par_time = omp_get_wtime() - start_time;
    printf("Parallel Time            : %f seconds\n\n", par_time);

    // Speedup
    if (par_time > 0.0) {
        double ratio = seq_time / par_time;
        printf("Linear / Parallel Ratio (Speedup): %f x\n", ratio);
    }

    free(a);
    return 0;
}
```

