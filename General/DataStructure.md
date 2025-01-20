
# Data structure

## 線性資料結構
元素以線性順序排列，通常有明確的前後關係。
+ Array: 固定大小的連續內存區域，按索引存取，支持隨機訪問。
+ LinkedList: 節點通過指針連接，分為單向鏈表、雙向鏈表、環形鏈表。
+ Stack: 後進先出（LIFO）結構，僅允許從一端（頂部）操作。
+ Queue: 先進先出（FIFO）結構，操作在頭尾兩端進行。
+ Deque: 雙端佇列，可以從兩端插入或刪除元素

## 集合類資料結構
元素無序存儲，通常不允許重複。
+ Set: 不允許重複的集合，可用於判斷存在性和去重。
+ Map: 鍵值對（key-value）結構，用於快速查找。
+ HashTable: 基於哈希函數的鍵值對結構，支持高效的插入和查找。

## 層次資料結構
+ Tree: 包含值與一些子節點的址
+ Binary Tree: 每個節點最多有兩個子節點的樹結構。
+ Binary Search Tree (BST):每個節點的左子樹小於根，右子樹大於根。
+ Heap: 完全二叉樹，分為最大堆（根節點最大）和最小堆（根節點最小）。
+ B Tree: 平衡多叉樹，適合磁碟存取，常用於數據庫索引。
+ B+ Tree: 改進版 B 樹，葉節點構成鏈表，適合範圍查詢。

## 圖（Graph）結構
+ Adjacency Matrix: 用矩陣表示節點之間的連接，適合密集圖。
+ Weighted Graph: 邊有權重的圖，用於最短路徑問題。