想像一棵家族樹，每個成員手上有一個「請勿打擾」牌。掛牌規則：你掛之前要確認，你的父母、祖父母、所有上一輩都沒掛；你的子女、孫子、所有下一輩也沒掛。兄弟姊妹掛不掛沒差。

```
        A
       / \
      B   C
     / \
    D   E
```

A 掛牌 → B 想掛失敗（A 是祖先）→ A 拿下 → B 掛牌成功 → D 想掛失敗（B 是祖先擋住）→ A 想掛也失敗（B 是後代擋住）。

要設計一個 `Node` 類別，提供 `is_locked / lock / unlock` 三個方法，每個都要 O(h)。h 是樹高。

---

**解題引導**

用上面那棵樹想。

**Step 1：is_locked 要 O(1)，怎麼存？**

*The state has to live on the node, not be computed.*

<span class="spoiler">每個節點掛一個 boolean flag，is_locked 直接讀。 / Store a boolean on each node, is_locked just reads it.</span>

**Step 2：lock 時怎麼知道祖先有沒有人鎖？**

*Walk somewhere. Where to?*

<span class="spoiler">每個節點加一個 parent pointer，從自己往 root 走，沿路看每個祖先的 flag。最壞 O(h)，剛好在預算內。 / Add a parent pointer, walk from self up to root checking each ancestor's flag. Worst case O(h), within budget.</span>

**Step 3：lock 時怎麼知道後代有沒有人鎖？naive 怎麼做？瓶頸在哪？**

*DFS down — straightforward but expensive.*

<span class="spoiler">naive：DFS 整個子樹找 flag 為 true 的節點。最壞 O(n)，超出 O(h) 預算。 / Naive: DFS the subtree to find any locked node. Worst case O(n), busts the O(h) budget.</span>

**Step 4：能不能讓「我子樹裡有沒有人鎖」變成 O(1) 查詢？**

*Don't compute. Cache.*

<span class="spoiler">每個節點多存一個 counter：「我子樹裡有幾個鎖住的後代」。lock 時走 parent chain 把每個祖先的 counter +1，unlock 時 -1。查詢就 O(1)。維護那趟走訪是 O(h)，剛好在預算內。 / Each node holds a counter: how many locked descendants in my subtree. On lock, walk up the parent chain incrementing each ancestor; on unlock, decrement. Query becomes O(1). The maintenance walk is O(h), within budget.</span>

**Step 5：unlock 需不需要再檢查祖先和後代？**

*Think about what must already be true.*

<span class="spoiler">不用。Invariant：被鎖的節點，它的祖先和後代必定都沒鎖（否則當初鎖不上）。所以 X 還鎖著時，整條祖先鏈和子樹都乾淨。unlock 只要看 this.locked 是 true 就能解。 / No. Invariant: a locked node never has a locked ancestor or descendant — otherwise it couldn't have been locked. So while X is locked, its ancestor chain and subtree are clean. unlock only needs to verify this.locked is true.</span>

想完再往下看 code。

---

## 解法一：暴力 DFS

```typescript
class Node {
  locked = false;
  parent: Node | null = null;
  left: Node | null = null;
  right: Node | null = null;

  is_locked(): boolean { return this.locked; }

  lock(): boolean {
    if (this.locked) return false;
    // 走祖先鏈
    for (let cur = this.parent; cur; cur = cur.parent) {
      if (cur.locked) return false;
    }
    // DFS 整個子樹找鎖
    if (this.hasLockedDescendant()) return false;
    this.locked = true;
    return true;
  }

  unlock(): boolean {
    if (!this.locked) return false;
    this.locked = false;
    return true;
  }

  private hasLockedDescendant(): boolean {
    if (this.locked) return true;
    return (this.left?.hasLockedDescendant() ?? false)
        || (this.right?.hasLockedDescendant() ?? false);
  }
}
```

複雜度：

- `is_locked`: O(1) ✓
- `lock`: O(h) 走祖先 + **O(n) 掃整棵子樹** ✗
- `unlock`: O(1)（unlock 不用查，後面 invariant 那段會解釋）

**慢在後代檢查那趟 DFS**。祖先鏈最多走 $O(h)$，失敗也可能很快（碰到第一個鎖著的祖先就 return），真正貴的是前面都乾淨的時候：要把整棵子樹翻完才敢說「後代沒人鎖」，靠近 root 的節點一次就是 $O(n)$。$N = 10^4$、連續操作 $10^4$ 次，總共 $10^8$ 步，照 $10^7$ 步 ≈ 1 秒算約 10 秒。題目給的預算是每個操作 $O(h)$，平衡樹 $h \approx 14$，差了三個數量級。

---

## 解法二：Counter 快取

瓶頸在「每次都要掃整棵子樹找鎖」。如果**每個節點預先存好「我子樹裡有幾個鎖」**，這個檢查就是讀一個 int，O(1) 結束。

維護成本：lock / unlock 時，從自己往 root 走，沿路每個祖先 ±1。一趟 O(h)。

```typescript
class Node {
  locked = false;
  lockedDescendants = 0;  // 子樹裡鎖住的節點數，不含自己
  parent: Node | null = null;
  left: Node | null = null;
  right: Node | null = null;

  is_locked(): boolean {
    return this.locked;
  }

  lock(): boolean {
    if (this.locked) return false;
    if (this.lockedDescendants > 0) return false;  // O(1)，靠 counter

    // 祖先鏈檢查 O(h)
    for (let cur = this.parent; cur; cur = cur.parent) {
      if (cur.locked) return false;
    }

    // 鎖定 + 通知所有祖先 +1，又一趟 O(h)
    this.locked = true;
    for (let cur = this.parent; cur; cur = cur.parent) {
      cur.lockedDescendants++;
    }
    return true;
  }

  unlock(): boolean {
    if (!this.locked) return false;
    // 不用查祖先 / 後代 — invariant 保證它們都沒鎖
    this.locked = false;
    for (let cur = this.parent; cur; cur = cur.parent) {
      cur.lockedDescendants--;
    }
    return true;
  }
}
```

複雜度：

- `is_locked`: O(1)
- `lock`: O(h) — 兩趟祖先走訪
- `unlock`: O(h) — 一趟祖先走訪
- 額外空間：每個節點多一個 boolean + 一個 int，整體 O(n)

平衡樹 $N = 10^6$、$h \approx 20$，每次操作大約 40 步。

<details>
<summary>Go 版本</summary>

```go
type Node struct {
    locked            bool
    lockedDescendants int
    Parent            *Node
    Left              *Node
    Right             *Node
}

func (n *Node) IsLocked() bool {
    return n.locked
}

func (n *Node) Lock() bool {
    if n.locked || n.lockedDescendants > 0 {
        return false
    }
    // 祖先鏈檢查
    for cur := n.Parent; cur != nil; cur = cur.Parent {
        if cur.locked {
            return false
        }
    }
    // 鎖定 + 通知祖先
    n.locked = true
    for cur := n.Parent; cur != nil; cur = cur.Parent {
        cur.lockedDescendants++
    }
    return true
}

func (n *Node) Unlock() bool {
    if !n.locked {
        return false
    }
    n.locked = false
    for cur := n.Parent; cur != nil; cur = cur.Parent {
        cur.lockedDescendants--
    }
    return true
}
```

</details>

---

**走一遍**

```
        A
       / \
      B   C
     / \
    D   E
```

初始：所有節點 `locked=false`, `lockedDescendants=0`。

```
B.lock():
  B.locked? false ✓
  B.lockedDescendants > 0? 0 > 0 = false ✓
  祖先鏈：A.locked? false ✓
  → B.locked = true
  → A.lockedDescendants++  (=1)
  return true

D.lock():
  D.locked? false ✓
  D.lockedDescendants > 0? 0 > 0 = false ✓
  祖先鏈：B.locked? true → return false ✗

A.lock():
  A.locked? false ✓
  A.lockedDescendants > 0? 1 > 0 = true → return false ✗
  （counter 直接擋下，沒去掃子樹）

B.unlock():
  B.locked? true ✓
  → B.locked = false
  → A.lockedDescendants--  (=0)
  return true

A.lock():
  A.locked? false ✓
  A.lockedDescendants > 0? 0 > 0 = false ✓
  祖先：parent is null ✓
  → A.locked = true
  return true
```

`A.lock()` 第一次失敗那行，`lockedDescendants=1` 直接 O(1) 回絕，不必 DFS 走 B、D、E。

---

**為什麼 unlock 不用檢查祖先和後代？**

直覺：「我都還鎖著，怎麼可能突然冒出一個鎖住的祖先？」

形式化：

- 我（X）能成功鎖，當下保證祖先沒鎖、後代沒鎖。
- X 鎖上之後，沿著祖先鏈每個節點的 counter 都 +1。
- 之後若某個祖先 A 想 lock，A 讀 `A.lockedDescendants ≥ 1`，直接 return false。
- 後代 Y 想 lock，Y 走祖先鏈時看到 X.locked=true，直接 return false。

所以「X 鎖著」這個狀態是穩定的：除非 X 自己 unlock，否則 X 的祖先和後代都動不了。unlock 不必重複驗證。

---

**Overthinking — 拿掉單執行緒假設會壞掉哪？**

題目特別強調「單執行緒，不需要實際的鎖」，是把 race condition 拿掉了。多執行緒下三個 hazard 會跑出來。

**Hazard 1：Counter 更新非 atomic**

兩條執行緒並行 lock：Tx 想鎖 X、Ty 想鎖 Y，X 和 Y 是兄弟、共同祖先 P。（執行緒名字取自它要做的事，沒有先後含義；誰先跑哪一步由排程器決定，下面的時間軸只是可能的穿插之一。）

```
時間 ↓
Tx: read P.lockedDescendants → 0
Ty: read P.lockedDescendants → 0       ← 都讀到 0
Tx: write P.lockedDescendants = 0+1 = 1
Ty: write P.lockedDescendants = 0+1 = 1 ← 該是 2，丟了一次更新
```

`P.lockedDescendants` 應該 = 2（X、Y 都鎖了），實際 = 1。當下還沒出事：1 仍然大於 0，X、Y 還鎖著的期間，想鎖 P 照樣被擋。錯誤是延後發生的：X、Y 先後 unlock，counter 被減兩次，1 − 2 = −1，此刻明明沒有任何後代上鎖，counter 卻停在 −1。之後任何一個後代 Z 再上鎖，P 的 counter 從 −1 變 0，Z 明明鎖著，P.lock() 檢查 `0 > 0` 不成立照樣通過：後代鎖著、祖先也鎖上，invariant 被打破。

**修**：把讀、加、寫合成拆不開的一步。Go 的做法是換欄位型別，`sync/atomic` 套件：

```go
type Node struct {
    locked            bool
    lockedDescendants atomic.Int64  // 原本是 int
    Parent            *Node
    Left              *Node
    Right             *Node
}

// 檢查後代，原本是 n.lockedDescendants > 0
if n.lockedDescendants.Load() > 0 {
    return false
}

// 通知祖先，原本是 cur.lockedDescendants++
for cur := n.Parent; cur != nil; cur = cur.Parent {
    cur.lockedDescendants.Add(1)  // 讀加寫在硬體層一條指令完成，插不了隊
}
```

Java 對應的是 `AtomicInteger` 的 `incrementAndGet()`。

**Hazard 2：Check-then-act 的空檔**

Tx 想鎖 X，Ty 想鎖 X 的後代 Y：

```
時間 ↓
Tx: X.lockedDescendants > 0 ?         → 0, 通過
Ty: 對 Y 跑完整個 lock，成功
Ty: 沿祖先鏈 increment, X.lockedDescendants = 1
Tx: X.locked = true                    ← 依據的是過期的檢查結果，違反 invariant
```

最後狀態：X 鎖著 ＋ X 的後代 Y 也鎖著。invariant（一個節點上鎖時，祖先與後代都不得同時鎖著）被打破，這個共存本身就是事故。這個情境裡 counter 是對的：X、Y 上鎖時各自沿祖先鏈 +1 過，X.unlock() 減回一次之後，祖先的 counter = 1，正確反映 Y 還鎖著。Hazard 2 的傷害是那段祖先與後代同時鎖著的違規窗口，不是 counter。

**修**：在 lock 給一道鎖。從檢查到動手這一整段做完之前，別的執行緒進不來：

```go
type Node struct {
    // 欄位同解法二，多一把整棵樹共用的鎖
    mu *sync.Mutex
}

func (n *Node) Lock() bool {
    n.mu.Lock()          // 進門：同一時間只有一條執行緒能往下走
    defer n.mu.Unlock()  // 出門自動還
    // 以下跟單執行緒版一字不差：
    // 檢查自己 → 檢查 counter → 走祖先鏈 → 設 locked → 通知祖先
    if n.locked || n.lockedDescendants > 0 {
        return false
    }
    for cur := n.Parent; cur != nil; cur = cur.Parent {
        if cur.locked {
            return false
        }
    }
    n.locked = true
    for cur := n.Parent; cur != nil; cur = cur.Parent {
        cur.lockedDescendants++
    }
    return true
}
```

Tx 進了門，Ty 只能在門外等 Tx 整段做完，中間插不進別的執行緒。進階做法是從 root 往下抓 read lock、對 X 抓 write lock，並行度更高，但那是另一個等級的工程。

**Hazard 3：死鎖**

死鎖的成因是不同執行緒抓鎖的順序不一致。

```
樹：
    R
    |
    A
    |
    B
```

假設實作 A 是「自下而上抓鎖」、實作 B 是「自上而下抓鎖」。執行緒 Ta 照實作 A 跑、Tb 照實作 B 跑：

```
時間 ↓
Ta（自下往上）: 抓到 A
Tb（自上往下）: 抓到 R
Ta: 等 R （Tb 佔著）
Tb: 等 A （Ta 佔著）
死鎖。
```

**修**：規定全域順序，例如「**所有執行緒永遠用 pre-order id 從小到大抓**」或「**永遠由 root 往下抓**」。順序一致就不會循環等待。

**用 Go 重現 Hazard 1**

Go 是文章裡唯一開得了執行緒的語言，可以拿解法二的 Node（還沒加任何保護的版本）直接重現。兩條 goroutine 各自反覆鎖、解 P 底下的兩個兄弟節點；每次 lock 對 P.lockedDescendants +1、unlock −1，全部結束後 counter 應該回到 0：

```go
func main() {
    P := &Node{}
    X := &Node{Parent: P}
    Y := &Node{Parent: P}

    var wg sync.WaitGroup
    for _, n := range []*Node{X, Y} {
        wg.Add(1)
        go func(n *Node) {
            defer wg.Done()
            for i := 0; i < 100000; i++ {
                n.Lock()   // 嘗試上鎖。這個場景必定成功：兄弟互不擋、沒人鎖 P
                n.Unlock() // 馬上解鎖，下一輪再鎖
            }
        }(n)
    }
    wg.Wait()

    fmt.Println(P.lockedDescendants)  // 該是 0；實際幾乎不是 0，每次跑都不同
}
```

X 跟 Y 是兄弟，互不阻擋，所以每輪 Lock 都成功，P 的 counter 被兩條 goroutine 各加減十萬次。`go run -race` 會把 `cur.lockedDescendants++` 那行標成 data race；不開 race detector 直接跑，印出來的數字每次不同，就是排程器每次給出不同穿插的具體證據。收尾時**沒有任何節點鎖著，counter 卻不是 0**：這就是 Hazard 1，之後誰想鎖 P，判斷就建立在這個錯的數字上。

修法就是 Hazard 2 那把 mutex。一把鎖，三個 hazard 一起消失。counter 的加減都發生在鎖裡面，讀加寫不會被拆開，Hazard 1 沒了。檢查跟動手也都在鎖裡面，中間沒有空檔，Hazard 2 沒了。整個系統就這一把鎖，不會有兩把鎖互等，Hazard 3 也沒了。

代價是所有操作排隊。樹很大、並行量很高的時候，這把鎖就是瓶頸。

那只把 counter 換成 `atomic.Int64` 呢？只救得了 Hazard 1。Hazard 2 的每一步就算各自 atomic，步驟跟步驟之間還是會被插隊。要保護的是整段流程，不是單一變數。

**面試講到哪就夠**

面試 5 分鐘的 follow-up，**列出這三個 hazard + 各自的修法**就達標 — 面試官想看你**知道有 race**。再深入到「怎麼設計 fine-grained 鎖協議」是另一場 system design 對話了。

---

## 解法比較表

| 解法 | Time (lock) | Space | $h=14$（$N=10^4$ 平衡樹） | 備註 |
|---|---|---|---|---|
| 暴力（DFS 掃後代） | $O(n + h)$ | $O(1)$ 額外 | 約 $10^4$ ops/call ≈ 1ms | 違反 $O(h)$ 預算 |
| Counter（本解） | $O(h)$ | $O(n)$ 額外 | 約 28 ops/call ≈ 2.8μs | 通過 |

差距約 350 倍。樹越深、節點越多差距越大。

---

## 結論

**用空間換時間**：每節點多一個 counter，把 O(n) 後代掃描變成 O(1) 查詢，維護成本剛好分攤到 O(h) 那條 parent chain 上。Invariant 保證 unlock 不用重複驗證。
