# string

- string不可变，需要变成数组
  str[0] = 's'  错误

# 二叉树

- 递归：

  (1)  test1返回None，等价于在最后加了return（最后没有返回值==return None），上面的return 1只是在最后一层的返回并不是函数最后的返回

  ​      test2返回1，最后一层返回的1会在递归回溯的时候层层保存并返回，最终的结果

  ```python
  def test1(n):
      if n == 1:
          return 1
      test(n-1)
      # return 
      
  def test2(n):
      if n == 1:
          return 1
      return test(n-1)
    
  def test3(n):
      if n == 1:
          return 1
      test(n-1)
      print(n)
      # return 

​	（2）test3的输出结果2，3，4。。。10，每一层递归结束后会继续运行test()下面的代码

-  迭代，**统一迭代法**

28. ==二叉搜索树的最近公共祖先== 递归算法中只遍历一条边

31. ```
    # 二叉搜索树剪枝
    class Solution:
        def trimBST(self, root: TreeNode, low: int, high: int) -> TreeNode:
            '''
            确认递归函数参数以及返回值：返回更新后剪枝后的当前root节点
            '''
            # Base Case
            if not root: return None
    
            # 单层递归逻辑
            if root.val < low:
            		# 直接抛弃**
                # 若当前root节点小于左界：只考虑其右子树，用于替代更新后的其本身，抛弃其左子树整体
                return self.trimBST(root.right, low, high)
            
            if high < root.val:
                # 若当前root节点大于右界：只考虑其左子树，用于替代更新后的其本身，抛弃其右子树整体
                return self.trimBST(root.left, low, high)
    
            if low <= root.val <= high:
                root.left = self.trimBST(root.left, low, high)
                root.right = self.trimBST(root.right, low, high)
                # 返回更新后的剪枝过的当前节点root
                return root
    ```

32. 
