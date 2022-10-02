---
layout: post
title:  Leetcode 解題編輯器裡的貼心
date:   2022-10-1 06:00:50 +0800
categories: leetcode
tag: [程式, 技巧, 心得]
---

這故事告訴我們：開始 coding 前，記得先把規則跟條件仔細地看清楚 ... XD

在 leetcode 裡可以選擇各種語言來解題，我自己練了很久都沒發現一件事\
除了 Problem 裡基本的 Description, Examples, Constraints, Hits 以外\
連選擇的語言旁邊都有一個小小的 icon: 「i」 來做資訊提供

![infomation](/assets/images/leetcode/p1.png)

---

這是 C 語言的 infomation
![c](/assets/images/leetcode/p2.png)

由於在 C 語言裡面，沒有像其他現代高階語言預設就內建\
許多常用的資料結構與對應的操作功能(e.g. HashTable, sort, find...)

害我以為在 leetcode 用 C 解題只能很悲催地自己慢慢刻那些資料結構與演算法\
導致嘗試用 C 解題的時候遇到很大的挫敗

到現在才發現原來 leetcode 很貼心地幫你內建了 `uthash.h` 請安心地用吧 XDDD

---

其他常用語言的 infomation:

javascript 內建 `lodash.js`
![javascript](/assets/images/leetcode/p3.png)

python 可自行 import 所需的 lib
![python](/assets/images/leetcode/p4.png)

ruby 也提供官方內建的各種常用資料結構與演算法
![ruby](/assets/images/leetcode/p5.png)
