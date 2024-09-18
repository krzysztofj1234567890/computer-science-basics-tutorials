
# Tree questions

## Given the root of a binary tree, flatten the tree into a "linked list."

## Given the root of a binary tree, invert the tree, and return its root.
## Given the root of a binary tree, return its maximum depth.
## Given the root of a binary tree and an integer targetSum, return true if the tree has a root-to-leaf path such that adding up all the values along the path equals targetSum.
## Given the root of a binary tree, return the pre-order traversal of its nodes' values.
## Given the roots of two binary trees root and subRoot, return true if there is a subtree of root with the same structure and node values of subRoot and false otherwise.

## Binary tree in-order traversal
```
public class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        Stack<TreeNode> stack = new Stack<>();
        TreeNode curr = root;
        while (curr != null || !stack.isEmpty()) {
            while (curr != null) {
                stack.push(curr);
                curr = curr.left;
            }
            curr = stack.pop();
            res.add(curr.val);
            curr = curr.right;
        }
        return res;
    }
}
```

Time complexity: O(n)

Space complexity: O(n)

## Symmetric tree

Given a binary tree, check whether it is a mirror of itself.


Iterative Approach:

Algorithm for checking whether a binary tree is a mirror of itself using an iterative approach and a stack:

Create a stack and push the root node onto it twice.
While the stack is not empty, repeat the following steps:
    a. Pop two nodes from the stack, say node1 and node2.
    b. If both node1 and node2 are null, continue to the next iteration.
    c. If one of the nodes is null and the other is not, return false as it is not a mirror.
    d. If both nodes are not null, compare their values. If they are not equal, return false.
    e. Push the left child of node1 and the right child of node2 onto the stack.
    f. Push the right child of node1 and the left child of node2 onto the stack.
If the loop completes successfully without returning false, return true as it is a mirror.

```
// Java program to check if a given Binary Tree is symmetric
// or not
import java.util.*;

// A Binary Tree Node
class Node {
  int key;
  Node left, right;
  // Constructor
  Node(int item)
  {
    key = item;
    left = right = null;
  }
}

public class GFG 
{
  
  // Returns true if a tree is symmetric i.e. mirror image
  // of itself
  static boolean isSymmetric(Node root)
  {
    
    // If the root is null, then the binary tree is
    // symmetric.
    if (root == null) {
      return true;
    }
    
    // Create a stack to store the left and right
    // subtrees
    // of the root.
    Stack<Node> stack = new Stack<>();
    stack.push(root.left);
    stack.push(root.right);

    // Continue the loop until the stack is empty.
    while (!stack.empty()) {
      // Pop the left and right subtrees from the
      // stack.
      Node node1 = stack.pop();
      Node node2 = stack.pop();

      // If both nodes are null, continue the loop.
      if (node1 == null && node2 == null) {
        continue;
      }

      // If one of the nodes is null, the binary tree
      // is not symmetric.
      if (node1 == null || node2 == null) {
        return false;
      }

      // If the values of the nodes are not equal, the
      // binary tree is not symmetric.
      if (node1.key != node2.key) {
        return false;
      }

      // Push the left and right subtrees of the left
      // and right nodes onto the stack in the
      // opposite order.
      stack.push(node1.left);
      stack.push(node2.right);
      stack.push(node1.right);
      stack.push(node2.left);
    }

    // If the loop completes, the binary tree is
    // symmetric.
    return true;
  }

  // Driver code
  public static void main(String[] args)
  {
    // Let us construct the Tree shown in the above
    // figure
    Node root = new Node(1);
    root.left = new Node(2);
    root.right = new Node(2);
    root.left.left = new Node(3);
    root.left.right = new Node(4);
    root.right.left = new Node(4);
    root.right.right = new Node(3);

    if (isSymmetric(root))
      System.out.println("Symmetric");
    else
      System.out.println("Not symmetric");
  }
}
```

## Maximum depth of binary tree

Given the root of a binary tree, return its max depth where max depth is the number of nodes along the longest path from the root node down to the farthest leaf node.

```
var maxDepth = function(root) {
    if(!root) return 0
    return 1 + Math.max(depth(root.left), depth(root.right))
  }
```

## Convert sorted array to binary search tree

Given an array where elements are sorted in ascending order, convert it to a height balanced BST

The mid value of the given sorted array would represent the root of one possible BST.

```
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        
       return helper(nums, 0, nums.length - 1);
    }
    
    private TreeNode helper(int[] nums, int low, int high) {
        if(low > high) {
            return null;
        }
        
        int mid = low + (high - low)/2;
        //center val of sorted array as the root of the bst
        TreeNode head = new TreeNode(nums[mid]);
        
        //left of the mid value should go to the left of this root node to satisfy bst
        head.left = helper(nums, low, mid - 1);
        //right of the mid value should go to the right of this root node to satisfy bst
        head.right = helper(nums, mid + 1, high);
        return head;
        
    }
    
    //T O(log n) S O(n) recursion stack space
}
```

Complexity Analysis:

Since we perform binary search on the array elements, splitting the input size by half through each recursion, therefore the time complexity that would be incurred from the aforementioned algorithm would be same as that incurred in binary search, that of T O(log n).

Space complexity due to recursion stack space would incur in the worst case of S O(n).

## Invert binary tree

An inverted form of a Binary Tree is another Binary Tree with left and right children of all non-leaf nodes interchanged.

```
// Iterative Function to invert given binary Tree using stack
void invertBinaryTree(TreeNode root) 
{
    // base case: if tree is empty
    if (root is null) 
        return
    // create an empty stack and push root node
    stack S
    S.push(root)
    // iterate until the stack is not empty
    while (S is not empty)
    {
        // pop top node from stack
        TreeNode curr = S.top()
        S.pop()
        // swap left child with right child
        swap(curr.left, curr.right)
        // push right child of popped node to the stack
        if (curr.right)
            S.push(curr.right)
        // push left child of popped node to the stack
        if (curr.left)
            S.push(curr.left)
    }
}
```

For each node in the tree, we are performing push() and pop() operation only once. Total no of stack operations = 2n 

Time complexity: O(n), where n is the total number of nodes.

Space Complexity: O(n)

## Minimum depth of binary tree

The minimum depth of a binary tree is the number of nodes from the root node to the nearest leaf node.

```
public class Solution {
    public int minDepth(TreeNode root) {
        if(root == null) return 0;
        int left = minDepth(root.left);
        int right = minDepth(root.right);
        return (left == 0 || right == 0) ? left + right + 1: Math.min(left,right) + 1;
       
    }
}
```

## Sum of left leaves

Given a Binary Tree, find the sum of all left leaves in it.

The idea is to traverse the tree, starting from root. For every node, check if its left subtree is a leaf. If it is, then add it to the result. 

```
// Java program to find sum of all left leaves
class Node 
{
    int data;
    Node left, right;
  
    Node(int item) 
    {
        data = item;
        left = right = null;
    }
}
  
class BinaryTree 
{
    Node root;
  
    // A utility function to check if a given node is leaf or not
    boolean isLeaf(Node node) 
    {
        if (node == null)
            return false;
        if (node.left == null && node.right == null)
            return true;
        return false;
    }
  
     // This function returns sum of all left leaves in a given
     // binary tree
    int leftLeavesSum(Node node) 
    {
        // Initialize result
        int res = 0;
  
        // Update result if root is not NULL
        if (node != null) 
        {
            // If left of root is NULL, then add key of
            // left child
            if (isLeaf(node.left))
                res += node.left.data;
            else // Else recur for left child of root
                res += leftLeavesSum(node.left);
  
            // Recur for right child of root and update res
            res += leftLeavesSum(node.right);
        }
  
        // return result
        return res;
    }
  
    // Driver program
    public static void main(String args[]) 
    {
        BinaryTree tree = new BinaryTree();
        tree.root = new Node(20);
        tree.root.left = new Node(9);
        tree.root.right = new Node(49);
        tree.root.left.right = new Node(12);
        tree.root.left.left = new Node(5);
        tree.root.right.left = new Node(23);
        tree.root.right.right = new Node(52);
        tree.root.left.right.right = new Node(12);
        tree.root.right.right.left = new Node(50);
  
        System.out.println("The sum of leaves is " + 
                                       tree.leftLeavesSum(tree.root));
    }
}
```

# Graph questions

# Arrays questions

# String questions

## Remove Vowels from a String 

```

import java.util.Scanner; 
  
public class Practice { 
  
    public static void main(String[] args) 
    { 
        Scanner sc = new Scanner(System.in); 
        String s = sc.nextLine(); 
        for (int i = 0; i < s.length(); i++) { 
            if (s.charAt(i) == 'a' || s.charAt(i) == 'e'
                || s.charAt(i) == 'i' || s.charAt(i) == 'o'
                || s.charAt(i) == 'u' || s.charAt(i) == 'A'
                || s.charAt(i) == 'E' || s.charAt(i) == 'I'
                || s.charAt(i) == 'O'
                || s.charAt(i) == 'U') { 
                continue; 
            } 
            else { 
                System.out.print(s.charAt(i)); 
            } 
        } 
    } 
}

```

## First Unique Character in a String

If you have read about hashmaps then, you would know that the lookup time of a key is constant.

We could use the hashmap to store the frequency of each character of the string and that could be done in a single pass of the string.

In another pass of the string, we may look for the first character with value in the map equal to 1. 

```
int firstUniqChar(String s) {
    Create a Hashmap freq
    n = s.length()
    // build hash map : character and how often it appears
    for (int i = 0 to i < n) {
        c = s[i]
        freq[c] = freq[c] + 1
    }   
    // find the index
    for (int i = 0 to i < n) {
        if (freq[s[i]] == 1) 
            return i
    }
    return -1
}
```

## Reverse a string in Java

```
import java.io.*;
import java.util.Scanner;

class GFG {
    public static void main (String[] args) {
      
        String str= "Geeks", nstr="";
        char ch;
      
      System.out.print("Original word: ");
      System.out.println("Geeks"); //Example word
      
      for (int i=0; i<str.length(); i++)
      {
        ch= str.charAt(i); //extracts each character
        nstr= ch+nstr; //adds each character in front of the existing string
      }
      System.out.println("Reversed word: "+ nstr);
    }
}
```

## Valid Anagram

An anagram of a word is basically another word that uses the same letters with the same frequency, just in a different order.

```
class Solution {
    public boolean isAnagram(String S, String T) {
        int len = S.length();
        int[] fMap = new int[123];
        if (T.length() != len) return false;
        for (int i = 0; i < len; i++)
            fMap[S.codePointAt(i)]++;
        for (int i = 0; i < len; i++)
            if (--fMap[T.codePointAt(i)] < 0) return false;
        return true;
    }
}
```

## Valid Palindrome

A palindrome is a string that's written the same forward as backward.

```
var isPalindrome = function(s) {

    let leftIdx = 0;
    let rightIdx = s.length - 1;

    while(leftIdx < rightIdx){
        if (s[leftIdx] !== s[rightIdx]) return false

        rightIdx --
        leftIdx ++ 

    }
    return true
};
```


## Valid Parentheses

Declare a character stack (say temp).
Now traverse the string exp. 
    If the current character is a starting bracket ( ‘(‘ or ‘{‘  or ‘[‘ ) then push it to stack.
    If the current character is a closing bracket ( ‘)’ or ‘}’ or ‘]’ ) then pop from the stack and if the popped character is the matching starting bracket then fine.
    Else brackets are Not Balanced.
After complete traversal, if some starting brackets are left in the stack then the expression is Not balanced, else Balanced.

```
import java.util.*;

public class BalancedBrackets {

    // function to check if brackets are balanced
    static boolean areBracketsBalanced(String expr)
    {
        // Using ArrayDeque is faster than using Stack class
        Deque<Character> stack
            = new ArrayDeque<Character>();

        // Traversing the Expression
        for (int i = 0; i < expr.length(); i++) {
            char x = expr.charAt(i);

            if (x == '(' || x == '[' || x == '{') {
                // Push the element in the stack
                stack.push(x);
                continue;
            }

            // If current character is not opening
            // bracket, then it must be closing. So stack
            // cannot be empty at this point.
            if (stack.isEmpty())
                return false;
            char check;
            switch (x) {
            case ')':
                check = stack.pop();
                if (check == '{' || check == '[')
                    return false;
                break;

            case '}':
                check = stack.pop();
                if (check == '(' || check == '[')
                    return false;
                break;

            case ']':
                check = stack.pop();
                if (check == '(' || check == '{')
                    return false;
                break;
            }
        }

        // Check Empty Stack
        return (stack.isEmpty());
    }

    // Driver code
    public static void main(String[] args)
    {
        String expr = "([{}])";

        // Function call
        if (areBracketsBalanced(expr))
            System.out.println("Balanced ");
        else
            System.out.println("Not Balanced ");
    }
}
```

## Roman to Integer

```
class Solution:
def romanToInt(self, s):
    roman = {'M': 1000,'D': 500 ,'C': 100,'L': 50,'X': 10,'V': 5,'I': 1}
    z = 0
    for i in range(0, len(s) - 1):
        if roman[s[i]] < roman[s[i+1]]:
            z -= roman[s[i]]
        else:
            z += roman[s[i]]
    return z + roman[s[-1]]

```

## Longest Substring Without Repeating Characters

## Remove Element

Given an integer array nums and an integer val, remove all occurrences of val in nums in-place. The order of the elements may be changed. Then return the number of elements in nums which are not equal to val.

Consider the number of elements in nums which are not equal to val be k, to get accepted, you need to do the following things:

Change the array nums such that the first k elements of nums contain the elements which are not equal to val. The remaining elements of nums are not important as well as the size of nums.
Return k.

```
class Solution {
    public int removeElement(int[] nums, int val) {
        int result = 0 ;
        int lastNotVal = nums.length-1 ;
        for ( int i=0; i<nums.length; i++ ) {
            if ( nums[i] == val ) {
                nums[i] = 0 ;
                // find not val 
                while( lastNotVal >=0 ) {
                    if ( nums[lastNotVal] != val ) {
                        nums[i] = nums[lastNotVal] ;
                        lastNotVal -- ;
                        break ;
                    }
                    lastNotVal -- ;
                }
            } else {
                result ++ ;
            }
        }
        return result ;
    }
}
```

## Remove Duplicates from Sorted Array

Given an integer array nums sorted in non-decreasing order, remove the duplicates in-place such that each unique element appears only once. The relative order of the elements should be kept the same. Then return the number of unique elements in nums.

```
class Solution {
    public int removeDuplicates(int[] nums) {
        if ( nums.length == 0 ) return 0 ;
        int indexToInsert = 1 ;
        for( int i=1; i< nums.length; i++ ) {
            if( nums[i-1] != nums[i]) {
                nums[indexToInsert++] = nums[i] ;
 //               System.out.println( nums[i] ) ;
            }
        }
        return indexToInsert ;
    }
}
```

## Remove Duplicates from Sorted Array II


Given an integer array nums sorted in non-decreasing order, remove some duplicates in-place such that each unique element appears at most twice. The relative order of the elements should be kept the same.

Since it is impossible to change the length of the array in some languages, you must instead have the result be placed in the first part of the array nums. More formally, if there are k elements after removing the duplicates, then the first k elements of nums should hold the final result. It does not matter what you leave beyond the first k elements.

Return k after placing the final result in the first k slots of nums.

```
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums.length <=2 ) return nums.length ;
        int result = nums.length ;

        int lastIndex = 2 ;
        for ( int i = 2; i<nums.length; i++ ) {
            if ( nums[lastIndex-2] == nums[lastIndex-1] && nums[lastIndex-1] == nums[i] ) {
                result -- ;
            } else {
                nums[lastIndex] = nums[i] ;
                lastIndex ++ ;
            }
        }
        return result ;
    }
}
```

## Rotate Array

Given an integer array nums, rotate the array to the right by k steps, where k is non-negative.

```
class Solution {
    public void rotate(int[] nums, int k) {
        if ( nums.length <= 1 ) return ;
        if ( k<0 ) return ;

        if ( k > nums.length) k = k % nums.length ;

        int[] result = new int[nums.length] ;
        for( int i = k; i<nums.length; i++ ) {
            result[i] = nums[i-k] ;
        }
        for( int i = 0; i<k; i++ ) {
            result[i] = nums[nums.length-k+i] ;
        }
        for ( int i=0; i<nums.length; i++ ) {
            nums[i]= result[i] ;
        }
    }
}
```

## Best Time to Buy and Sell Stock II

You are given an integer array prices where prices[i] is the price of a given stock on the ith day.

On each day, you may decide to buy and/or sell the stock. You can only hold at most one share of the stock at any time. However, you can buy it then immediately sell it on the same day.

Find and return the maximum profit you can achieve.

```
class Solution {
    public int maxProfit(int[] prices) {
        if (prices.length<2) return 0 ;

        int i = 0 ;
        int lo = prices[0] ;
        int hi = prices[0] ;
        int profit = 0 ;
        int n = prices.length ;

        while (i<n-1) {
            // where to buy
            while ( i<(n-1) && prices[i] >= prices[i+1] ) {
                i++ ;
            }
            lo = prices[i] ;

            // where to sell
            while ( i<(n-1) && prices[i] <= prices[i+1] ) {
                i++ ;
            }
            hi = prices[i] ;

            profit += hi-lo ;
        }

        return profit ;
    }
}

```

## Jump Game

You are given an integer array nums. You are initially positioned at the array's first index, and each element in the array represents your maximum jump length at that position.

Return true if you can reach the last index, or false otherwise.

```
class Solution {
    public boolean canJump(int[] nums) {

        int goal_post = nums.length - 1;

        for (int i = nums.length - 1; i >= 0; i--) {
            int jump_distance = i + nums[i];
            if (jump_distance >= goal_post) {
                goal_post = i;
            }
        }

        return (goal_post == 0) ? true : false;
    }
}
```

## H-Index

Given an array of integers citations where citations[i] is the number of citations a researcher received for their ith paper, return the researcher's h-index.

According to the definition of h-index on Wikipedia: The h-index is defined as the maximum value of h such that the given researcher has published at least h papers that have each been cited at least h times.

```
import java.lang.* ;

class Solution {
    public int hIndex(int[] citations) {
        Arrays.sort( citations );
        if ( citations.length == 0 ) return 0;
        if ( citations.length == 1 && citations[0] == 0 ) return Math.min( citations[0], 1 );
        int result = citations[citations.length-1] ;
        for( int i=citations.length-1; i >=0 ; i-- ) {
            if ( citations[i] < (citations.length-i) ) {
                result = citations.length-i-1 ;
                break ;
            }
        }
        return Math.min( result, citations.length) ;
    }
}
```

or

```
class Solution {

    public int hIndex(int[] citations) {

        // Sorting an int[] in reverse in Java is annoying

        // We first sort normally then reverse the array

        Arrays.sort(citations);

        for (int i = 0; i < citations.length/2; i++) {

            int tmp = citations[i];

            citations[i] = citations[citations.length-1-i];

            citations[citations.length-1-i] = tmp;

        }


        int h = 0;

        while (h < citations.length && citations[h] >= h+1) {

            h++;

        }

        return h;

    }

}
```

# References

https://igotanoffer.com/blogs/tech/amazon-software-development-engineer-interview