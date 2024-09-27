本页面将介绍败者树，一种为多路归并排序设计的高效数据结构。

## 引入

败者树在多个有序数据流进行合并时，能够迅速确定多个当前元素中的最小（或最大）元素，并在该最值元素被更新时进行高效调整。该结构维护一个完全二叉树，在非叶子节点（内部节点）中存储一轮比较中的 **败者**，将 **胜者** 推进到下一轮比较（即父节点），显著减少了调整过程中出现的元素比较次数，优化了合并过程的效率。

## 结构

-   **叶子节点**：代表对应输入序列的当前元素，直接关联到数据流的输出。
-   **内部节点**：记录其两个子节点（所推进上来的胜者）比较后的败者，即较小（或较大）的元素。
-   **根节点**：在完全二叉树的根节点之上额外添加的父节点，表示所有叶子节点比较后的最终胜者，即当前所有输入序列中的最小（或最大）元素。

???+ note "提示"
    本文中提及被节点储存的元素时，均默认在对应节点中额外保存了该元素 **所在数据流的索引**。在每一轮的最终胜者（最值元素）决出时，可以通过该索引来获取对应数据流中的下一个元素，在下一轮比较中替代本轮胜者。

假设败者为两个元素中较小的一方，各数据流的当前元素分别为 $20,10,30,40$，则败者树的结构如下：

![败者树示例](./images/loser-tree-1.png)

## 过程

败者树涉及的操作包括初始化、构建和调整。

### 初始化

初始化败者树时，将每个叶子节点设置为对应输入数据流的首元素，内部节点则初始化为一个在比较中必定落败的特殊值（如 $+\infty$ 或 $-\infty$）。这使得初次建树时可以方便地进行比较，从而提高构建效率。

### 构建

构建败者树时，从最底层内部节点开始，比较两个子节点各自产生的胜者元素，将产生的败者储存在本节点中，并递归地将胜者推进至上级内部节点，直至根节点，最后形成一棵完整的败者树。

在败者树构建完成后，根节点始终代表当前所有输入序列中的最小（或最大）元素。每当树中的最终胜者退出并更新为新元素时，都需要通过调整操作来更新树的结构，从而确保根节点的性质不变。

### 调整

当根节点的元素被消费（如输出到结果数组），并用其所在数据流中的下一个元素替入对应叶子结点时，需要用调整操作更新树结构。从该叶节点开始，向上逐级调整，确保每一步都重新计算败者信息，以维持树的正确性。

以下是一个调整函数示例，用于阐明败者树如何更新其内部节点以维持整体数据的一致性：

???+ note "示例代码"
    ```cpp
    void adjust(int index) {
      int parent = get_parent(index);  // 获取该节点的父亲节点
      while (parent > 0) {
        // 比较当前节点和父节点的值，选择较小的节点作为胜者，将败者信息存储到父节点
        if (segments[index].empty() ||
            segments[index].front() > segments[tree[parent]].front()) {
          std::swap(index, tree[parent]);
        }
        // 更新父亲节点，继续向上调整
        parent = get_parent(parent)
      }
      // 更新根节点
      tree[0] = index;
    }
    ```

通过以上代码可以看出，败者树在单次调整过程中子节点只需和对应路径的父节点进行比较。该设计减少了数据更新时的比较次数，提高了操作效率，使其成为解决大规模数据排序和归并问题的有效工具。

### 时间复杂度

败者树是一种特殊的完全二叉树，$n$ 个节点的败者树的高度为 $\log n$，节点总数为 $2n-1$，因此构建败者树的时间复杂度为 $O(n)$。

在调整败者树时，由于只需比较和更新对应叶子节点的路径上的节点，无需比较兄弟节点，因此在最坏情况下，单次调整败者树的时间复杂度为 $O(\log n)$。

## 应用和示例

败者树广泛应用于多路归并排序、外部排序和流数据处理，特别是在需要从多个数据源中持续合并数据时。其优势在于：

-   **高效性**：能够迅速确定多个输入序列中的最小（或最大）元素，显著提高排序效率。
-   **快速更新**：在输入序列发生变化时，能够进行快速调整，实时反映最新状态。
-   **易扩展性**：结构简单，易于实现和扩展，适合处理大量数据。

以下是一个简单的败者树示例，用于合并多个有序序列：

假设有四个输入序列，分别为 $\{10,20,30\},\{15,25,35\},\{5,40,45\}$ 和 $\{1,2,3\}$。我们可以使用败者树来合并这些序列，并输出排序后的结果，以下是一个简单的演示动画。

![败者树演示动画](./images/loser-tree-2.apng)

如图所示，通过每一次的败者树的调整和输出，我们可以逐步合并多个有序序列，并输出排序后的结果，直到源序列全部合并完成。

## 实现

??? note "参考代码"
    ```cpp
    --8<-- "docs/ds/code/loser-tree/loser-tree_1.cpp"
    ```

## 参考资料

-   [Wikipedia: k-way merge algorithm](https://en.wikipedia.org/wiki/K-way_merge_algorithm)