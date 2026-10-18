---
id: 355
title: "Design Twitter"
slug: design-twitter
difficulty: Medium
tags: [Hash Table, Linked List, Design, Heap (Priority Queue)]
neetcode150_category: Heap / Priority Queue
blind75_category: null
date_solved: 2026-03-29
languages: [golang, typescript]
time_complexity: O(K log K)
space_complexity: O(N)
"@local": [Heap]
insight: "getNewsFeed = merge k sorted tweet lists via max-heap"
---

# Design Twitter

Design a simplified version of Twitter where users can post tweets, follow/unfollow another user, and is able to see the `10` most recent tweets in the user's news feed.

Implement the `Twitter` class:

*   `Twitter()` Initializes your twitter object.
*   `void postTweet(int userId, int tweetId)` Composes a new tweet with ID `tweetId` by the user `userId`. Each call to this function will be made with a unique `tweetId`.
*   `List<Integer> getNewsFeed(int userId)` Retrieves the `10` most recent tweet IDs in the user's news feed. Each item in the news feed must be posted by users who the user followed or by the user themself. Tweets must be **ordered from most recent to least recent**.
*   `void follow(int followerId, int followeeId)` The user with ID `followerId` started following the user with ID `followeeId`.
*   `void unfollow(int followerId, int followeeId)` The user with ID `followerId` started unfollowing the user with ID `followeeId`.

**Example 1:**

**Input**
\["Twitter", "postTweet", "getNewsFeed", "follow", "postTweet", "getNewsFeed", "unfollow", "getNewsFeed"\]
\[\[\], \[1, 5\], \[1\], \[1, 2\], \[2, 6\], \[1\], \[1, 2\], \[1\]\]
**Output**
\[null, null, \[5\], null, null, \[6, 5\], null, \[5\]\]

**Explanation**
Twitter twitter = new Twitter();
twitter.postTweet(1, 5); // User 1 posts a new tweet (id = 5).
twitter.getNewsFeed(1);  // User 1's news feed should return a list with 1 tweet id -> \[5\]. return \[5\]
twitter.follow(1, 2);    // User 1 follows user 2.
twitter.postTweet(2, 6); // User 2 posts a new tweet (id = 6).
twitter.getNewsFeed(1);  // User 1's news feed should return a list with 2 tweet ids -> \[6, 5\]. Tweet id 6 should precede tweet id 5 because it is posted after tweet id 5.
twitter.unfollow(1, 2);  // User 1 unfollows user 2.
twitter.getNewsFeed(1);  // User 1's news feed should return a list with 1 tweet id -> \[5\], since user 1 is no longer following user 2.

**Constraints:**

*   `1 <= userId, followerId, followeeId <= 500`
*   `0 <= tweetId <= 104`
*   All the tweets have **unique** IDs.
*   At most `3 * 104` calls will be made to `postTweet`, `getNewsFeed`, `follow`, and `unfollow`.
*   A user cannot follow himself.

## Code Template

### Go
```go
type Twitter struct {
    
}


func Constructor() Twitter {
    
}


func (this *Twitter) PostTweet(userId int, tweetId int)  {
    
}


func (this *Twitter) GetNewsFeed(userId int) []int {
    
}


func (this *Twitter) Follow(followerId int, followeeId int)  {
    
}


func (this *Twitter) Unfollow(followerId int, followeeId int)  {
    
}


/**
 * Your Twitter object will be instantiated and called as such:
 * obj := Constructor();
 * obj.PostTweet(userId,tweetId);
 * param_2 := obj.GetNewsFeed(userId);
 * obj.Follow(followerId,followeeId);
 * obj.Unfollow(followerId,followeeId);
 */
```

### TypeScript
```typescript
class Twitter {
    constructor() {
        
    }

    postTweet(userId: number, tweetId: number): void {
        
    }

    getNewsFeed(userId: number): number[] {
        
    }

    follow(followerId: number, followeeId: number): void {
        
    }

    unfollow(followerId: number, followeeId: number): void {
        
    }
}

/**
 * Your Twitter object will be instantiated and called as such:
 * var obj = new Twitter()
 * obj.postTweet(userId,tweetId)
 * var param_2 = obj.getNewsFeed(userId)
 * obj.follow(followerId,followeeId)
 * obj.unfollow(followerId,followeeId)
 */
```
