Twitter 的 news feed 要顯示「你自己 + 你追蹤的人」最新的 10 則推文，按時間倒序排列。

每個用戶的推文是照時間 append 進去的，所以每條 list 內部已經排好：

```
tweets[user1] = [(t=1, id=5), (t=3, id=9), (t=7, id=2)]
tweets[user2] = [(t=2, id=3), (t=5, id=6)]
tweets[user3] = [(t=4, id=1), (t=6, id=8)]
```

問題變成：從 K 條已排好的 list 裡取最新的 10 個，這就是 **Merge K Sorted Lists**。

<details>
<summary>提示</summary>

把每個用戶的「最新一則推文」丟進 max-heap。每次 pop 出最新的，再把那個用戶的前一則推進去，直到拿滿 10 則。

</details>

**資料結構**

```
tweets   map[userId] → [(time, tweetId), ...]   // 每人的推文，按時間遞增
following map[userId] → set of followeeIds
time     int  // 全局計數器，每發一則 +1，用來排序
```

---

## 解法：Merge K Sorted（heap）

TypeScript 沒有內建 heap，改成「每人取最近 10 則、合併後排序取前 10」，結果相同：

```typescript
class Twitter {
    private time = 0;
    private tweets = new Map<number, [number, number][]>(); // userId → [[time, tweetId]]
    private following = new Map<number, Set<number>>();

    postTweet(userId: number, tweetId: number): void {
        if (!this.tweets.has(userId)) this.tweets.set(userId, []);
        this.tweets.get(userId)!.push([this.time++, tweetId]);
    }

    getNewsFeed(userId: number): number[] {
        const users = [userId, ...(this.following.get(userId) ?? [])];
        const all: [number, number][] = [];
        for (const u of users) {
            const tw = this.tweets.get(u) ?? [];
            all.push(...tw.slice(-10)); // 每人最多取最近 10 則
        }
        all.sort((a, b) => b[0] - a[0]);
        return all.slice(0, 10).map(t => t[1]);
    }

    follow(followerId: number, followeeId: number): void {
        if (!this.following.has(followerId)) this.following.set(followerId, new Set());
        this.following.get(followerId)!.add(followeeId);
    }

    unfollow(followerId: number, followeeId: number): void {
        this.following.get(followerId)?.delete(followeeId);
    }
}
```

<details>
<summary>Go 版本</summary>

```go
import "container/heap"

type tweet struct{ time, id int }

type Twitter struct {
    time      int
    tweets    map[int][]tweet
    following map[int]map[int]bool
}

func Constructor() Twitter {
    return Twitter{
        tweets:    make(map[int][]tweet),
        following: make(map[int]map[int]bool),
    }
}

func (t *Twitter) PostTweet(userId int, tweetId int) {
    t.tweets[userId] = append(t.tweets[userId], tweet{t.time, tweetId})
    t.time++
}

// heap entry：(時間戳, tweetId, userId, 在該用戶 list 中的位置)
type entry struct{ ts, id, uid, idx int }
type maxH []entry

func (h maxH) Len() int            { return len(h) }
func (h maxH) Less(i, j int) bool  { return h[i].ts > h[j].ts } // 大的在頂
func (h maxH) Swap(i, j int)       { h[i], h[j] = h[j], h[i] }
func (h *maxH) Push(x any)         { *h = append(*h, x.(entry)) }
func (h *maxH) Pop() any           { old := *h; n := len(old); x := old[n-1]; *h = old[:n-1]; return x }

func (t *Twitter) GetNewsFeed(userId int) []int {
    h := &maxH{}
    heap.Init(h)

    // 把自己 + 所有 followee 的最新一則推進 heap
    users := []int{userId}
    for u := range t.following[userId] {
        users = append(users, u)
    }
    for _, u := range users {
        if tw := t.tweets[u]; len(tw) > 0 {
            i := len(tw) - 1
            heap.Push(h, entry{tw[i].time, tw[i].id, u, i})
        }
    }

    var res []int
    for h.Len() > 0 && len(res) < 10 {
        e := heap.Pop(h).(entry)
        res = append(res, e.id)
        if e.idx > 0 { // 還有更早的推文，補進 heap
            i := e.idx - 1
            tw := t.tweets[e.uid]
            heap.Push(h, entry{tw[i].time, tw[i].id, e.uid, i})
        }
    }
    return res
}

func (t *Twitter) Follow(followerId int, followeeId int) {
    if t.following[followerId] == nil {
        t.following[followerId] = make(map[int]bool)
    }
    t.following[followerId][followeeId] = true
}

func (t *Twitter) Unfollow(followerId int, followeeId int) {
    delete(t.following[followerId], followeeId)
}
```

</details>

---

**核心**

```typescript
// getNewsFeed = merge k sorted lists
// TS 沒有 heap，改成每人取最近 10 則、合併後排序取前 10
all.sort((a, b) => b[0] - a[0]);
```

<details>
<summary>Go 版本</summary>

```go
// getNewsFeed = merge k sorted lists via max-heap
// 每次 pop 最新的，補進同一用戶的前一則
heap.Push(h, entry{tw[i].time, tw[i].id, e.uid, i})
```

</details>

---

**複雜度**

- `postTweet`: O(1)
- `getNewsFeed`: O(K log K + 10 log K)，K = 追蹤人數
- `follow` / `unfollow`: O(1)
- Space: O(N)，N = 所有推文總數
