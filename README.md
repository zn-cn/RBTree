# 红黑树

红黑树是平衡二叉查找树的一种。为了深入理解红黑树，我们需要从二叉查找树开始讲起。

## BST

二叉查找树（Binary Search Tree，简称BST）是一棵二叉树，它的左子节点的值比父节点的值要小，右节点的值要比父节点的值大。它的高度决定了它的查找效率。

在理想的情况下，二叉查找树增删查改的时间复杂度为O(logN)（其中N为节点数），最坏的情况下为O(N)。当它的高度为logN+1时，我们就说二叉查找树是平衡的。
![BST](https://tech.meituan.com/img/redblack-tree/tree-all.png)

### 查找

```
T  key = a search key
Node root = point to the root of a BST

while(true){
    if(root==null){
        break;
    }
    if(root.value.equals(key)){
        return root;
    }
    else if(key.compareTo(root.value)<0){
        root = root.left;
    }
    else{
        root = root.right;
    }
}
return null;

```

从程序中可以看出，当BST查找的时候，先与当前节点进行比较：

- 如果相等的话就返回当前节点；
- 如果少于当前节点则继续查找当前节点的左节点；
- 如果大于当前节点则继续查找当前节点的右节点。

直到当前节点指针为空或者查找到对应的节点，程序查找结束。

### 插入

```
Node node = create a new node with specify value
Node root = point the root node of a BST
Node parent = null;

//find the parent node to append the new node
while(true){
   if(root==null)break;
   parent = root;
   if(node.value.compareTo(root.value)<=0){
      root = root.left;  
   }else{
      root = root.right;
   } 
}
if(parent!=null){
   if(node.value.compareTo(parent.value)<=0){//append to left
      parent.left = node;
   }else{//append to right
      parent.right = node;
   }
}

```

插入操作先通过循环查找到待插入的节点的父节点，和查找父节点的逻辑一样，都是比大小，小的往左，大的往右。找到父节点后，对比父节点，小的就插入到父节点的左节点，大就插入到父节点的右节点上。

### 删除

删除操作的步骤如下：

1. 查找到要删除的节点。
2. 如果待删除的节点是叶子节点，则直接删除。
3. 如果待删除的节点不是叶子节点，则先找到待删除节点的中序遍历的后继节点，用该后继节点的值替换待删除的节点的值，然后删除后继节点。

![BST remove](https://tech.meituan.com/img/redblack-tree/bst-tree-remove.png)

### BST存在的问题

BST存在的主要问题是，数在插入的时候会导致树倾斜，不同的插入顺序会导致树的高度不一样，而树的高度直接的影响了树的查找效率。理想的高度是logN，最坏的情况是所有的节点都在一条斜线上，这样的树的高度为N。

## 红黑树

基于BST存在的问题，一种新的树——平衡二叉查找树(Balanced BST)产生了。平衡树在插入和删除的时候，会通过旋转操作将高度保持在logN。其中两款具有代表性的平衡树分别为AVL树和红黑树。AVL树由于实现比较复杂，而且插入和删除性能差，在实际环境下的应用不如红黑树。

**红黑树（Red-Black Tree，以下简称RBTree）**的实际应用非常广泛，比如Linux内核中的完全公平调度器、高精度计时器、ext3文件系统等等，各种语言的函数库如Java的TreeMap和TreeSet，C++ STL的map、multimap、multiset等。

RBTree也是函数式语言中最常用的持久数据结构之一，在计算几何中也有重要作用。值得一提的是，Java 8中HashMap的实现也因为用RBTree取代链表，性能有所提升。

### 定义

RBTree的定义如下:

1. 任何一个节点都有颜色，黑色或者红色
2. 根节点是黑色的
3. 所有叶子都是黑色（叶子是NIL节点）
4. 父子节点之间不能出现两个连续的红节点（每个红色节点必须有两个黑色的子节点）
5. 从任一节点到其每个叶子的所有[简单路径](https://zh.wikipedia.org/wiki/%E9%81%93%E8%B7%AF_(%E5%9B%BE%E8%AE%BA))都包含相同数目的黑色节点。

数据结构表示如下：

```
class  Node<T>{
   public  T value;
   public   Node<T> parent;
   public   boolean isRed;
   public   Node<T> left;
   public   Node<T> right;
}

```

RBTree在理论上还是一棵BST树，但是它在对BST的插入和删除操作时会维持树的平衡，即保证树的高度在[logN,logN+1]（理论上，极端的情况下可以出现RBTree的高度达到2*logN，但实际上很难遇到）。这样RBTree的查找时间复杂度始终保持在O(logN)从而接近于理想的BST。RBTree的删除和插入操作的时间复杂度也是O(logN)。RBTree的查找操作就是BST的查找操作。

图例：

![An example of a red-black tree](https://upload.wikimedia.org/wikipedia/commons/thumb/6/66/Red-black_tree_example.svg/450px-Red-black_tree_example.svg.png)

### 旋转

Rotate分为left-rotate（左旋）和right-rotate（右旋）

**注：网上可能有两种版本的旋转，请注意，以下全文均采用此处定义的旋转**

#### 旋转规则：

假设当前节点为 node

##### 左旋：（node即为右图中的P)

node节点的父节点变为node的右子树，node的右节点变为node的右子树的左子树，node节点的祖父节点的child节点（可能左可能右）变为node节点的右子树

##### 右旋：（node即为左图中的Q)

node节点的父节点变为node的左子树，node的左节点变为node的左子树的右子树，node节点的祖父节点的child节点（可能左可能右）变为node节点的左子树

![Tree rotation.png](https://upload.wikimedia.org/wikipedia/commons/2/23/Tree_rotation.png)

### 查找

RBTree的查找操作和BST的查找操作是一样的。请参考BST的查找操作代码。



插入删除前言：

在红黑树上进行插入操作和删除操作会导致不再匹配红黑树的性质。恢复红黑树的性质需要少量![{\displaystyle {\text{O}}(\log n)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/67697a0b44080bbf967c00d60bf4aac79f9ce385)的颜色变更（实际是非常快速的）和不超过三次[树旋转](https://zh.wikipedia.org/wiki/%E6%A0%91%E6%97%8B%E8%BD%AC)（对于插入操作是两次）。虽然插入和删除很复杂，但操作时间仍可以保持为![{\displaystyle {\text{O}}(\log n)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/67697a0b44080bbf967c00d60bf4aac79f9ce385)次。

### 插入

约定：新插入的节点初始都是**红色的** 。

#### 无须修复：

+ 新节点C位于树的**根**上，没有父节点

  操作：初始化根节点即可

+ 新节点的父节点B是黑色

  操作：直接插入即可，易得红黑树所有性质满足，因为新插入的节点为红色的，且与原树不冲突


#### 需要修复：

注：以下均采用父节点为祖父节点的左节点条件，若为右节点，则只需做镜像操作即可

循环条件：需要修复的节点的父节点的颜色为RED

##### case 1

父节点B和叔父节点C二者都是红色

操作：将父节点和叔叔节点与祖父节点的颜色互换，即维持了局部的颜色**符合RBTree定义的第四条和第五条。下图中，操作完成后A节点变成了新的修复节点。**如果A节点的父节点不是黑色的，则继续做修复操作（将A当做新加入的节点）。（没有父节点的话，可以在循环外面加一句root->color = BLACK保证颜色正确）**

![插入修复case 1](https://tech.meituan.com/img/redblack-tree/insert-case1.png)

##### case 2

父节点B是红色而叔父节点U是黑色(NIL节点也是黑色的)，新节点C是其父节点B的左子节点，而父节点B又是其父节点A的左子节点

操作：将祖父节点A节点进行右旋操作，并且和父节点B互换颜色。通过该修复操作RBTRee的高度和颜色都符合红黑树的定义。

![插入修复case 2](https://tech.meituan.com/img/redblack-tree/insert-case2.png)

##### case 3

父节点B是红色而叔父节点U是黑色(NIL节点也是黑色的)，新节点C是其父节点B的右子节点(即一家三代不在一条线上)

操作：将父节点B节点进行左旋，这样就从case 3转换成case 2了，然后针对case 2进行操作处理就行了。case 2操作做了一个右旋操作和颜色互换来达到目的。

![插入修复case 3](https://tech.meituan.com/img/redblack-tree/insert-case3.png)

### 删除

删除过程：

+ 如果是叶子节点或者只有一个子节点就直接删除；

  删除的图示：

  ![img](https://user-gold-cdn.xitu.io/2018/4/7/162a00075de35231?w=350&h=135&f=png&s=5375)

+ 如果有左右节点都有，会用右节点的最小节点（记为T）顶替要删除节点(记为N)的位置，即将N的value替换为T的value，之后删除T；

+ 删除后，如果删除的节点的颜色为黑色就需要做删除修复操作，删除修复的主要思想就是从兄弟节点上**借调黑色的节点**过来，如果兄弟节点没有黑节点可以借调的话，就只能往上追溯，将每一级的黑节点数减去一个，使得整棵树符合红黑树的定义。

+ 删除修复操作在遇到被调整的节点是红色节点或者到达root节点时，修复操作完毕，**修复之后要将被调整的节点颜色变为黑色（主要防止以下case 2中父节点为红色的）**。

删除修复操作分为四种情况(删除黑节点后)：

**注：待调整的节点的初始节点为删除节点的子节点（优先非空子节点），以下删除修复情况只讨论待调整的节点为左节点的情况，若为右节点，则只需做相应的镜像操作即可。**

1. 待调整的节点的兄弟节点是红色的节点；
2. 待调整的节点的兄弟节点是黑色的节点，且兄弟节点的子节点都是黑色的；
3. 待调整的节点的兄弟节点是黑色的节点，且兄弟节点的左子节点是红色的，右节点是黑色的；
4. 待调整的节点的兄弟节点是黑色的节点，且右子节点是是红色的；



注：以下图示中待删除均修改为待调整

#### case 1

情况：待调整的B的兄弟节点C是红色节点

操作：交换此兄弟节点和父节点的颜色，再对待调整的节点的父节点A进行左旋

解释：由于兄弟节点是红色节点，无法借调黑节点，所以需要将兄弟节点提升到父节点，由于兄弟节点是红色的，所以兄弟节点的子节点是黑色的，这样就可以从它的子节点借调黑节点了

![删除情况1](https://tech.meituan.com/img/redblack-tree/remove-case1.png)

#### case 2

情况：待调整的节点B，兄弟节点C，及C的两个儿子节点的颜色都是黑色的

操作：将兄弟节点颜色变为红色，同时将待调整的节点的父节点变为新的待调整的节点继续向上调整

解释：当将兄弟节点也变红之后，达到了局部的平衡了（由于原来计算定义的第五条的时候就是多了一个黑色节点的数量），但是对于祖父节点不一定满足条件，所以继续上溯

![删除情况2](https://tech.meituan.com/img/redblack-tree/remove-case2.png)

#### case 3

情况：待调整的节点B的兄弟节点C是黑色，C的左儿子是红色，C的右儿子是黑色

操作：交换兄弟节点的左儿子和兄弟节点的颜色，再对兄弟节点进行右旋

解释：case 3的删除操作是一个中间步骤，目的是转换为case 4状态

![删除情况3](https://tech.meituan.com/img/redblack-tree/remove-case3.png)

#### case 4

情况：待调整的B和它的兄弟节点D是黑色的，D的右儿子是红色的

解决：交换兄弟节点D和父节点A的颜色（防止父节点A为红色），再对父节点进行左旋即可

解释：修复完成，整棵树还是符合红黑树的定义的，因为黑色节点的个数没有改变。

![删除情况4](https://tech.meituan.com/img/redblack-tree/remove-case4.png)

### RBTree的C语言实现

https://github.com/tofar/RBTree/blob/master/RBTree.c

## 总结

红黑树通过引入颜色的概念，通过颜色这个约束条件的使用来保持树的高度平衡。作为平衡二叉查找树，旋转是一个必不可少的操作。通过旋转可以降低树的高度，在红黑树里面还可以转换颜色。

红黑树里面的插入和删除的操作比较难理解，这时要注意记住一点：操作之前红黑树是平衡的，颜色是符合定义的。在操作的时候就需要向兄弟节点、父节点、侄子节点借调和互换颜色，要达到这个目的，就需要不断的进行旋转。所以红黑树的插入删除操作需要不停的旋转，一旦借调了别的节点，删除和插入的节点就会达到局部的平衡（局部符合红黑树的定义），但是被借调的节点就不会平衡了，这时就需要以被借调的节点为起点继续进行调整，直到整棵树都是平衡的。在整个修复的过程中，插入修复具体的分为3种情况，删除修复分为4种情况。

整个红黑树的查找，插入和删除都是O(logN)的，原因就是整个红黑树的高度是logN，查找从根到叶，走过的路径是树的高度，删除和插入操作是从叶到根的，所以经过的路径都是logN。



文章来源：

[红黑树深入剖析及Java实现](https://tech.meituan.com/redblack-tree.html) 

[红黑树-wikipedia](https://zh.wikipedia.org/zh-hans/%E7%BA%A2%E9%BB%91%E6%A0%91) 

演示参照：

[红黑树插入删除过程](https://www.jianshu.com/p/ad5d65e7ce62) 

或者 [红黑树从头至尾插入和删除结点的全程演示图](https://blog.csdn.net/v_JULY_v/article/details/6284050) 

