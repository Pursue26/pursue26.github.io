---
title: LeetCode刷题之1410实体解析器
date: 2023-11-23 18:15:39
categories: 
    - LeetCode
    - 哈希表
tags: 
    - LeetCode
    - 哈希表
    - 字符串
abbrlink: 231123181539
---

[HTML 实体解析器](https://leetcode.cn/problems/html-entity-parser/)是一种特殊的解析器，它将 HTML 代码作为输入，并用字符本身替换掉所有这些特殊的字符实体。给你输入字符串 `text` ，请你实现一个 HTML 实体解析器，返回解析器解析后的字符串。

<!--more-->

HTML 里这些特殊字符和它们对应的字符实体包括：
双引号：字符实体为 `&quot;` ，对应的字符是 `"` 。
单引号：字符实体为 `&apos;` ，对应的字符是 `'` 。
与符号：字符实体为 `&amp;` ，对应对的字符是 `&` 。
大于号：字符实体为 `&gt;` ，对应的字符是 `>` 。
小于号：字符实体为 `&lt;` ，对应的字符是 `<` 。
斜线号：字符实体为 `&frasl;` ，对应的字符是 `/` 。

示例：
```
输入：text = "&amp; is an HTML entity but &ambassador; is not."
输出："& is an HTML entity but &ambassador; is not."

输入：text = "and I quote: &quot;...&quot;"
输出："and I quote: \"...\""

输入：text = "&&&"
输出："&&&"
```

## 哈希表+模拟

这个解法，是我今天刷题时写的，主要练习一下使用 `uthash.h` 构建 key 为字符串，value 为字符的哈希表。

解法描述：

1. 首先，我们可以将这六个字符实体存入一个哈希表中，它的 key 为字符实体，value 为对应的字符；
2. 然后，向右遍历给定的文本 `text`，当遍历到的字符是 `&` 时，我们就可以查表判断 `&` 字符之后的一段字符串，是否是这 6 个字符实体中的一个：
    - 如果是，则替换为对应的字符；如果遍历完这一段字符，也没找到，则不替换；
    - 如果中间遇到新的 `&`，则不应替换，并及时停止、继续步骤 2。
3. 遍历完文本 `text` 后，则字符实体替换结束。

```c
// 构建 key 为字符串，value 为字符的哈希表
typedef struct {
    const char *key;
    char val;
    UT_hash_handle hh;
} HashItem;

// 迭代地释放所有哈希节点的空间
void hashFree(HashItem **obj) {
    HashItem *curr = NULL, *tmp = NULL;
    HASH_ITER(hh, *obj, curr, tmp) {
        HASH_DEL(*obj, curr);  
        free(curr);
    }
}

#define MAX_LEN (7)  // strlen("&frasl;")

char *entityParser(char* text) {
    const char *entities[] = {"&quot;", "&apos;", "&amp;", "&gt;", "&lt;", "&frasl;", NULL};
    char entitiesVal[] = {'\"', '\'', '&', '>', '<', '/'};
    HashItem *hashTbl = NULL, *hashNode = NULL;

    for (int i = 0; entities[i]; i++) {
        hashNode = (HashItem *)malloc(sizeof(HashItem));
        hashNode->key = entities[i];
        hashNode->val = entitiesVal[i];
        // 已知添加的key不会重复，这里就不在添加前先查找key是否存在了
        HASH_ADD_STR(hashTbl, key, hashNode);
    }

    int n = strlen(text), ;
    char *ans = (char *)malloc((n + 1) * sizeof(char));

    int idx = 0, i = 0;
    while (text[i] != '\0') {
        if (text[i] != '&') { // 不是字符实体首字符
            ans[idx++] = text[i++];
        } else {  // 是字符实体首字符
            hashNode = NULL;
            char temp[8] = {'\0', '\0', '\0', '\0', '\0', '\0', '\0', '\0'};

            // 遍历接下可能出现字符实体的一段字符串
            for (int j = i; j < fmin(i + MAX_LEN, n); j++) {
                temp[j - i] = text[j];
                HASH_FIND_STR(hashTbl, temp, hashNode);
                if (hashNode) { // 找到则替换，指针后移特定长度
                    ans[idx++] = hashNode->val;
                    i = j + 1;
                    break;
                }
            }
            if (!hashNode) { // 遍历完也没找到
                ans[idx++] = text[i++];
            }
        }
    }

    ans[idx] = '\0';
    hashFree(&hashTbl);

    return ans;
}
```

考虑最坏情况，如字符串 `&&&&&&`。在外循环中，遍历的每个位置都要进入内循环去查表，在内层循环中，每次都是探测到「一段字符串」的**最后位置**，才发现不是字符实体。

- 时间复杂度：`O(kn)`，`k` 表示字符实体的最大长度，哈希表查表为 `O(1)`。
- 空间复杂度：`O(s + 6)`，`s` 表示字符实体的长度和，为哈希表存储的 6 个字符实体的空间。

## 字符串操作+模拟（官方）

```c
typedef struct _entityChar {
    char *entity;
    char character;
} EntityChar;

EntityChar entityList[] = {
    {"&quot;", '\"'},
    {"&apos;", '\''},
    {"&amp;", '&'},
    {"&gt;", '>'},
    {"&lt;", '<'},
    {"&frasl;", '/'},
};

char *entityParser(char *text) {
    int n = strlen(text);
    int cnt = sizeof(entityList) / sizeof(entityList[0]);  // 字符实体的个数

    char *ans = (char *)malloc((n + 1) * sizeof(char));
    char *p = ans;  // 使用指针的方式填充结果

    for (int i = 0; i < n; ) {
        bool isEntity = false;
        if (text[i] == '&') {
            for (int j = 0; j < cnt; j++) {
                char *pKey = entityList[j].entity;
                char *pVal = &entityList[j].character;
                if (strncmp(text + i, pKey, strlen(pKey)) == 0) {
                    strcpy(p, pVal);  // 这样写可以兼容值是字符串的情况
                    p += strlen(pVal);
                    i += strlen(pKey);
                    isEntity = true;
                    break;
                }
            }
        }
        if (!isEntity) {
            *(p++) = text[i++];
        }
    }
    *p = '\0';
    return ans;
}
```

考虑最坏情况，如字符串 `&&&&&&`。在外循环中，遍历的每个位置都要进入内循环去查表，在内层循环中，要比较**每一个长度不同**的字符实体。

- 时间复杂度：`O(sn)`，`s` 为字符实体的长度和。
- 空间复杂度：`O(s + 6)`，变量 `entityList` 的空间。
