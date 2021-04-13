[783. 二叉搜索树节点最小距离](https://leetcode-cn.com/problems/minimum-distance-between-bst-nodes/)

```java
/**
* 遍历获取，排序，相减
**/
class Solution {
    List<Integer> values = new ArrayList<>();
    public int minDiffInBST(TreeNode root) {
        dfs(root);
        Collections.sort(values);
        int minValue = Integer.MAX_VALUE;
        for(int i = 1;i<values.size();i++)
        {
            minValue = Math.min(minValue,values.get(i)-values.get(i-1));
        }
        return minValue;
    }
    public void dfs(TreeNode node)
    {
        if(node == null) return;
        values.add(node.val);
        dfs(node.left);
        dfs(node.right);
    }
}
```

