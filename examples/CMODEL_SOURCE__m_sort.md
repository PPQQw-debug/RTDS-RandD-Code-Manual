# 示例：CMODEL_SOURCE 排序模型说明

## 1. 场景

该模型展示了在 C 模型中对输入量进行排序并输出名次的典型写法，适合给新员工理解 RAM/CODE 分段与辅助函数组织方式。

## 2. 代码示例（排序函数）

```c
void heap_sortxxx(int n)
{
    /* top-down heap sort */
    int i;
    int hn = 0;

    for (i = 1; i <= n; i++)
        insertxxx(&hn, arrayIndex[i]);

    for (i = hn; i >= 1; i--)
        arrayIndex2[i] = extractxxx(&hn);
}
```

来源文件：CMODEL_SOURCE/_m_sort.c

说明：

- 该函数通过堆结构完成索引排序，不直接搬移原始数值。  
- 排序结果写入 arrayIndex2，后续逻辑可据此映射排名。

风险：

- 若将循环下标改为从 0 开始，需要同步改动 upheap/downheap 的父子节点计算。  
- 若改变数组长度，必须统一更新初始化与边界逻辑。

## 3. 代码示例（仿真步执行）

```c
arrayItem[1].dValue = ma;
arrayItem[2].dValue = mb;
arrayItem[3].dValue = mc;

for (i = 1; i < 4; i++)
{
    // index initialization
    arrayIndex[i] = i;
    arrayIndex2[i] = i;
}

heap_sortxxx(3);

for (i = 1; i < 4; i++)
{
    arrayItem[arrayIndex2[i]].nRanking = i - 1;
}
```

来源文件：CMODEL_SOURCE/_m_sort.c

说明：

- 该片段位于 CODE 区，按仿真步周期执行。  
- 输入 ma/mb/mc 每步刷新后重新排序并输出排名。

相关头文件：CMODEL_SOURCE/_m_sort.h

## 4. 新人建议

1. 先看 _m_sort.h 的 INPUTS/OUTPUTS，再看 _m_sort.c 的 CODE 区。  
2. 修改排序逻辑前先补边界测试：相等值、负值、极大值。

## 变更记录

- 2026-06-03：初版建立。
