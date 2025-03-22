# Palindrome Linked List

For a markdown version of this document, please visit [https://github.com/prestonfitz/palindrome-linked-list-tutorial](https://github.com/prestonfitz/palindrome-linked-list-tutorial)

## The Problem

Given the `head` of a singly linked list, return `true` if it is a palindrome or `false` otherwise.

## Some Terminology

#### Palindromes

A palindrome is a pattern in a word or number such that it is the same forwards and backwards. For example, the word racecar backwards is racecar. Or the number 12321 backwards is 12321.

#### Linked Lists

A linked list is a data structure composed of nodes, each of which contains a value and a reference (or link) to the next node in the sequence. Each node is usually an instance of a class with the value named `val` and the link named `next`, as it points to the next node. We know we have hit the end of the list when the `next` value is set to `None` or `null`. The first node in the list is called the `head` Using this structure, we are able to store data in a way that is very similar to using an array.

_That's great,_ you may say, _but why would I want to do that when I could just use an array?_

That is a good question. It has to do with how computers store memory.

While we don't see it, computers have to physically store memory. When we allocate an array, the computer sets aside a block of memory to be used for that array. Regardless if that memory is used, that memory is set aside exclusively for the array and cannot be used for other tasks. Linked lists, on the other hand, do not have to be stored in blocks like arrays do. Each node can be stored in a separate, smaller block. The `next` value then tells the computer where the next node is stored. This frees up some of the memory implications of using an array.

In addition, linked lists can have other performance implications. In an array, when an item is inserted or deleted, the computer has to shift the other items in the array so that they remain contiguous in memory. With a linked list, nodes do not have to be reordered. Instead, we just change which node the `next` value is referrencing, which can result in a much faster modification.

## Approaching the Problem

### Approach 1: Converting to Array

As humans, it is really easy to see if a word or number is a palindrome. We begin by looking at the first character and then at the last character to see if they match. We do the same for the second character and the second to last character and so on until we have checked all of the characters to see if they match. Unfortunately, because we are working with a singly linked list, we can't see the ending value, only the `head`. If only there were a way to see the end.

One way that we could do that is by pulling all of the values out of the linked list and storing them in a different data structure. We could store them in a string, but what is a string other than an array of characters? Because it is more intutive to access the elements of an array than a string, let's try converting the list to an array. We can do this using the following code:

```
  arr_check = []
  while head:
      arr_check.append(head.val)
      head = head.next
``` 

What we are doing here is appending each value to the array in the order that it is linked. We are then setting the variable head to be the next node in the list until it gets set to `None`. We will go over the reason for overwriting the variable later.

The next thing we need to check is if the numbers in our array represent a palindrome. We can do that using the following code:

```
  for i in range(0, math.floor(len(arr_check)/2)):
      if arr_check[i] != arr_check[len(arr_check) - 1 -i]:
          return False

  return True
```

What we are doing here is looking at the value of a given index `i` and the relative value from the back side of the array. If those values are the same, there is still a possibility of a palindrome, and we can move on to the next characters. If at any point they don't match, we know we don't have a palindrome and we can just return `False`, thus ending the loop. We only iterate through half of the array, because after that point we start seeing values that we have already seen. We don't worry about a middle value here, because middle values don't need to be mirrored. If all of the letters match, then we know we have a palindrome and we can return `True`.

Here is the full code:

 ```
class Solution:
    def isPalindrome(self, head: Optional[ListNode]) -> bool:
        arr_check = []
        while head:
            arr_check.append(head.val)
            head = head.next

        for i in range(0, math.floor(len(arr_check)/2)):
            if arr_check[i] != arr_check[len(arr_check) - 1 -i]:
                return False

        return True
``` 

#### The strengths of this approach

This approach is relatively simple and fairly easy to understand. Arrays are basic data structures, and it is easy to visualize what is going on. It is also a concise solution that is relatively fast to write.

#### The weaknesses of this approach

We are back on the topic of memory. Every variable that a program uses has to be stored in memory. A computer can remember a lot of things, but its memory is finite. If it runs out of memory, a computer cannot function properly. As a computer gets closer to its memory limit, it usually slows down. You may have experienced this if you have ever left a lot of tabs open in Google Chrome.

Because of these limitations, it pays to be efficient with memory. An example of doing this is how we kept overwriting `head` rather than saving each node to a new variable. Writing to a new variable every time would result in an additional memory cost every time we iterated over a node, though it would admittedly be a pretty small cost in this particular case. While that is a good example of being memory efficient, you may have noticed another variable that we created: `arr_check`. This array is a problem because the array grows in proportion to the length of the linked list; we are now effectively storing the linked list twice. While this isn't a problem with a small linked list, it could become a problem if our list had millions or billions of nodes.

### Approach 2: The Toroise and the Hare

The goal in our next approach is to determine if the linked list is a palindrome without using variables that grow in relation to the length of the linked list.

We know that we can determine if we have a palindrome by comparing one half of the list to the other, so it is a good idea to start by seeing if we can split the list in half. Our first step is to find the middle of the list. We can do this by taking inspiration from the [tortoise and the hare](https://en.wikipedia.org/wiki/Cycle_detection#Floyd's_tortoise_and_hare):

```
  # Step 1: Find the middle of the linked list
  slow = head
  fast = head.next if head else None
  while fast and fast.next:
      slow = slow.next
      fast = fast.next.next
```

As with the array example above, we are beginning by iterating over the array. However, in this case, we keep track of two nodes at once. `slow` is our tortoise. `fast` is our hare. Our hare moves twice as fast as our tortoise. This means that by the time our hare has reached the end, our tortoise will be half way through (because 100% / 2 = 50% or half). This gives us the middle point of the array, which will be important for what we do next.

Our next step is to split the linked list in half. Splitting the list in half is relatively easy. We know the middle point is `slow`, so everything after that is the half we want. Reversing the list is a bit trickier, because we can't just go to the end and work backwards (though if you want to, I suggest looking at [doubly linked lists](https://www.geeksforgeeks.org/doubly-linked-list/)). So how do we do it? The following code may be useful:

```
  # Step 2: Reverse the second half of the linked list
  prev = None
  current = slow.next
  while current:
      temp = current.next
      current.next = prev
      prev = current
      current = temp
```

We start by storing the previous node and `current.next` in a temporary variables. It is particularly important to store `current.next` because the next step is to change the value of `current.next` to be `prev`. In other words, the node that was before the current node, is now the current node's `next` value. We are reversing the list! We then take our `temp` value (the old `next` value) and use it as our new `current` value and use our old `current` value as our new `prev` value. We do this until the second have of the list has been reversed into `prev`.

_But wait!_ you may say, _`prev` is growing in length with the length of the linked list! We specifically_ didn't _want that to happen!_

You'd be correct, we _don't_ want that to happen. However, there is one caveat that makes this okay in this case. Every time that we pass a node to `prev`, we are not actually duplicating the node. We are just changing the node's `next` value. So, the amount of data that is stored doesn't change, we just change which node points to which. This does, however, come with a downside. Because we are changing where the nodes are pointing, it does mean that the original list no longer in tact. If we needed the original list for something later, we would need to reconstruct it. However, in this case, we do not need the list for any additional tasks, so we are fine to change it.

The final step is to compare the two halves of the list to see if we have a palindrome. We can do this using the following code:

 ```
  # Step 3: Compare the first half and the reversed second half
  while prev:
      if prev.val != head.val:
          return False
      prev = prev.next
      head = head.next

  return True
``` 

This code is similar to what we did with the array. We compare the corresponding values in each list. If they do not match, we return `False`. If we survive the loop, we can return `True`. Unlike the array approach, we can compare the same index in each list here, because we reversed the back half of the list.

Here is the full code:

```
class Solution:
    def isPalindrome(self, head: Optional[ListNode]) -> bool:
        # Step 1: Find the middle of the linked list
        slow = head
        fast = head.next if head else None
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

        # Step 2: Reverse the second half of the linked list
        prev = None
        current = slow.next
        while current:
            temp = current.next
            current.next = prev
            prev = current
            current = temp

        # Step 3: Compare the first half and the reversed second half
        while prev:
            if prev.val != head.val:
                return False
            prev = prev.next
            head = head.next

        return True
```

#### The strengths of this approach

This approach is more memory efficient. In tests that I ran, this approach tended to be around 12% more efficient, which can add up, especially as the list grows longer.

#### The weaknesses of this approach

There are more steps for the computer to process in this approach. This means that the program can take a little bit longer to run. In addition, this code is more complex, which can make it harder to write and maintain.

### When to use which approach

Both approaches are valuable in different situations. If you know that you are only going be working with relatively small lists, the array approach is quick to set up and will move through the lists quickly. However, if memory is a concern, like if you are working with larger lists, then the second approach will be worth the time to set up.

# AI Appendix

#### Prompt 1

Given that head is a singly linked list, can you rewrite this to use constant space complexity and linear time?

```
class Solution:
def isPalindrome(self, head: Optional[ListNode]) -> bool:
arr_check = []
while head:
arr_check.append(head.val)
head = head.next

for i in range(0, math.floor(len(arr_check)/2)):
if arr_check[i] != arr_check[len(arr_check) - 1 -i]:
return False

return True

```

#### Answer 1

To rewrite the code to check if a singly linked list is a palindrome using constant space complexity and linear time, you can follow these steps:

Find the Middle of the List: Use the two-pointer technique with a slow and fast pointer to find the middle of the linked list.

Reverse the Second Half: Reverse the second half of the linked list.

Compare the Halves: Compare the first half and the reversed second half to check if they are the same.

Here's how you can implement it:

```
class Solution:
def isPalindrome(self, head: Optional[ListNode]) -> bool: # Step 1: Find the middle of the linked list
slow = head
fast = head.next if head else None
while fast and fast.next:
slow = slow.next
fast = fast.next.next

        # Step 2: Reverse the second half of the linked list
        prev = None
        current = slow.next
        while current:
            temp = current.next
            current.next = prev
            prev = current
            current = temp

        # Step 3: Compare the first half and the reversed second half
        while prev:
            if prev.val != head.val:
                return False
            prev = prev.next
            head = head.next

        return True
```

#### Prompt 2

Give me an explanation for what a linked list is based off of this structure:

```

# class ListNode:

# def **init**(self, val=0, next=None):

# self.val = val

# self.next = next

```

Target the explanation to a relatively new programmer who is familiar with object oriented programming. Please be concise.

#### Answer 2

A linked list is a data structure composed of nodes, each of which contains a value and a reference (or "link") to the next node in the sequence. Here's how it works based on the given structure:

Node Creation: Each node is an instance of the ListNode class. It has two attributes: val to store the node's value and next to point to the next node in the list.

List Structure: The first node is called the head of the list. Each subsequent node is linked to the next one through the next attribute, forming a chain.

Traversal: You can traverse the list by starting at the head and following the next pointers until you reach the end, where next is typically None.

This structure allows for efficient insertion and deletion of nodes at any position in the list, as only the affected nodes need to be updated.
