This summary covers the internal mechanics of hash table array resizing, growth factors, bitwise filters, and memory optimizations as explained by Arpit Bhayani.

### 1. Foundations: The Load Factor & Need for Resizing

The **Load Factor ($\alpha$)** quantifies how full a hash table is. It dictates the ratio between the occupied space and total capacity:

$$\alpha = \frac{\text{Number of occupied keys }(N)}{\text{Total available slots }(M)}$$

- **The Problem:** As $\alpha$ increases, performance degrades exponentially [[03:00](http://www.youtube.com/watch?v=zt1E0akArqQ&t=180)]. It becomes harder to find empty slots (in Open Addressing) or bucket chains grow longer (in Chaining), converting constant-time lookups $O(1)$ into linear paths $O(N)$.
    
- **The Solution:** To maintain predictable $O(1)$ latency, the underlying array must change size when it hits a critical threshold [[03:09](http://www.youtube.com/watch?v=zt1E0akArqQ&t=189)].
    
- **The Threshold:** A typical production standard is to trigger growth when **$\alpha = 0.5$** (the table is 50% full) [[03:58](http://www.youtube.com/watch?v=zt1E0akArqQ&t=238)].
    

### 2. The Internal Mechanics of Resizing

Resizing is structurally simple but computationally heavy [[04:21](http://www.youtube.com/watch?v=zt1E0akArqQ&t=261)]. It consists of three primary stages:

1. **Allocation:** The Operating System allocates a completely new, contiguous memory block of size $M_{\text{new}}$ [[04:42](http://www.youtube.com/watch?v=zt1E0akArqQ&t=282)].
    
2. **Rehashing & Copying:** The elements cannot simply be copied directly (e.g., via C's standard `realloc`) because the mapping function depends heavily on array capacity ($M$) [[05:15](http://www.youtube.com/watch?v=zt1E0akArqQ&t=315)]. Every single key must be passed through the hash function again using the new modulo constraints and placed into its newly recalculated position [[06:23](http://www.youtube.com/watch?v=zt1E0akArqQ&t=383)].
    
3. **Deallocation:** The memory block containing the old array is freed [[04:49](http://www.youtube.com/watch?v=zt1E0akArqQ&t=289)].
    

The overall time complexity required to perform this relocation is directly proportional to the number of keys currently held by the structure [[05:59](http://www.youtube.com/watch?v=zt1E0akArqQ&t=359)].

### 3. Emphasising Your Doubt: Aggressive Resizing vs. Power-of-Two Doubling

#### 💡 Your Doubt: Why can't we grow the array size by only 1 slot at a time?

If we try to conserve memory by using an aggressive resize strategy (incrementing the array capacity by exactly $+1$ every time a new element arrives), the operations accumulate destructively [[08:23](http://www.youtube.com/watch?v=zt1E0akArqQ&t=503)].

**Visualizing the $+1$ Strategy Costs:**

- **Insert 1st Element:** Allocate size 1 $\rightarrow$ Insert element. (**1 Op**) [[09:01](http://www.youtube.com/watch?v=zt1E0akArqQ&t=541)]
    
- **Insert 2nd Element:** Allocate size 2 $\rightarrow$ Copy 1 element $\rightarrow$ Insert new element. (**2 Ops**) [[09:08](http://www.youtube.com/watch?v=zt1E0akArqQ&t=548)]
    
- **Insert 3rd Element:** Allocate size 3 $\rightarrow$ Copy 2 elements $\rightarrow$ Insert new element. (**3 Ops**) [[09:27](http://www.youtube.com/watch?v=zt1E0akArqQ&t=567)]
    
- **Insert $N$-th Element:** Allocate size $N$ $\rightarrow$ Copy $(N-1)$ elements $\rightarrow$ Insert new element. (**$N$ Ops**) [[10:03](http://www.youtube.com/watch?v=zt1E0akArqQ&t=603)]
    

Summing the operations for inserting $N$ total elements yields an arithmetic progression:

$$\text{Total Operations} = 1 + 2 + 3 + 4 + \dots + N = \frac{N(N + 1)}{2} = \mathbf{O(N^2)}$$

An aggressive $+1$ strategy drops lookup/insertion capability down to a catastrophic **quadratic time complexity** [[10:58](http://www.youtube.com/watch?v=zt1E0akArqQ&t=658)].

#### The Proof Behind Doubling ($O(1)$ Amortized)

When the capacity is doubled instead, the cost of copying elements is spread across a massive sequence of rapid operations [[11:17](http://www.youtube.com/watch?v=zt1E0akArqQ&t=677)].

Assuming we double our table size from $N$ to $2N$ right when it fills completely [[12:14](http://www.youtube.com/watch?v=zt1E0akArqQ&t=734)]:

```
[ Array Size: N ]  --> (Left half filled with N/2 keys, Right half is completely empty)
```

1. For the next $\frac{N}{2} - 1$ elements inserted, the table has empty slots available. They insert cleanly in **$O(1)$ constant time** per insertion [[14:38](http://www.youtube.com/watch?v=zt1E0akArqQ&t=878)].
    
2. When the final element arrives to fill the table completely, a resize triggers [[15:18](http://www.youtube.com/watch?v=zt1E0akArqQ&t=918)]:
    
    - Allocate a brand-new space of size $2N$ (**$2N$ operations**) [[15:23](http://www.youtube.com/watch?v=zt1E0akArqQ&t=923)].
        
    - Rehash and copy all elements over (**$N$ operations**) [[15:32](http://www.youtube.com/watch?v=zt1E0akArqQ&t=932)].
        

**Mathematical Summary of the Resizing Window:**

$$\text{Total Ops} = \underbrace{\left(\frac{N}{2} - 1\right)}_{\text{Easy insertions}} + \underbrace{1}_{\text{Final key}} + \underbrace{2N}_{\text{Allocation}} + \underbrace{N}_{\text{Rehash/Copy}} = \frac{7N}{2} - 1 = \mathbf{O(N)}$$

Dividing the total work ($O(N)$) by the number of elements processed shows that the work per element is tightly bounded to a constant. This technique provides an **amortized constant time $O(1)$** performance profile [[16:43](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1003)].

### 4. Mathematical Optimization: Why Array Sizes are Powers of Two ($2^n$)

In traditional mapping, a key's index is calculated using a modulo constraint:

$$\text{Index} = \text{Hash}(K) \pmod M$$

The CPU handles modulo operations using long division or signed division, which are highly expensive instruction cycles [[18:28](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1108)].

However, if the array size $M$ is guaranteed to be a **Power of Two ($2^n$)**, the division operations can be replaced with a lightning-fast **Bitwise AND ($\&$) filter** using $(M - 1)$ [[19:30](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1170)]:

$$\text{Hash}(K) \pmod M \equiv \text{Hash}(K) \ \& \ (M - 1)$$

#### Bitwise Execution Example ($M = 4$)

When $M = 4$ ($2^2$), $(M - 1) = 3$. In binary notation, $3$ is expressed as `0011`.

|**Operation**|**Standard Modulo**|**Bitwise Representation**|**Result**|
|---|---|---|---|
|**Key = 1**|$1 \pmod 4 = \mathbf{1}$|`0001 & 0011`|`0001` ($\mathbf{1}$) [[20:06](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1206)]|
|**Key = 2**|$2 \pmod 4 = \mathbf{2}$|`0010 & 0011`|`0010` ($\mathbf{2}$) [[20:32](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1232)]|
|**Key = 3**|$3 \pmod 4 = \mathbf{3}$|`0011 & 0011`|`0011` ($\mathbf{3}$) [[20:38](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1238)]|
|**Key = 4**|$4 \pmod 4 = \mathbf{0}$|`0100 & 0011`|`0000` ($\mathbf{0}$) [[20:51](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1251)]|
|**Key = 5**|$5 \pmod 4 = \mathbf{1}$|`0101 & 0011`|`0001` ($\mathbf{1}$) [[21:06](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1266)]|

**Why this works:** Any number matching $2^k - 1$ (like 3, 7, 15, 32) turns into a binary sequence where lower bits are all `1`s and higher bits are all `0`s [[22:00](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1320)]. The bitwise AND operator isolates the lower bit values perfectly, creating a natural cycling mechanism without invoking standard CPU division [[22:25](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1345)].

### 5. Memory Management: Scaling Down (Shrinking)

If elements are heavily deleted from a hash table, leaving it unmanaged creates bloated, dead memory structures [[23:32](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1412)]. To combat this, tables implement shrinking steps.

- **Why we can't shrink at $\alpha = 0.5$ or $\alpha = 0.25$:** If a table with 16 slots shrinks down to 8 slots when it hits 4 elements ($\alpha = 0.25$), its new load factor immediately skyrockets to $4 / 8 = 0.5$ [[26:19](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1579)]. Adding just one more single item triggers an immediate, expensive upscale resize back to 16 slots [[26:26](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1586)]. This creates a volatile loop called **thrashing**, where sequential `insert` and `delete` calls waste processing cycles constantly resizing the system back and forth [[27:13](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1633)].
    
- **The Solution:** To provide safe engineering headroom, tables typically shrink when the load factor drops significantly low, specifically around **$\alpha = \frac{1}{8}$ ($0.125$)** [[27:01](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1621)]. This ensures that once the table shrinks in half, it still has enough open slots left over to handle a long sequence of future insertions without needing to resize immediately [[27:33](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1653)].
    

### Memory Refresh Cheat Sheet

|**Property**|**Aggressive Growth (+1)**|**Amortized Doubling (×2)**|
|---|---|---|
|**Time Complexity**|$O(N^2)$ (Quadratic / Highly Latent) [[10:58](http://www.youtube.com/watch?v=zt1E0akArqQ&t=658)]|Amortized $O(1)$ (Constant / Fast) [[17:02](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1022)]|
|**Compute Operation**|Arithmetic Modulo ($\%$) (Slow) [[18:28](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1108)]|Bitwise AND ($\&$) filter (Extremely Fast) [[21:45](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1305)]|
|**Growth Point**|Immediately when an item arrives [[08:40](http://www.youtube.com/watch?v=zt1E0akArqQ&t=520)]|When Load Factor hits $\alpha = 0.5$ [[24:01](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1441)]|
|**Shrink Point**|Immediately when an item leaves|When Load Factor drops to $\alpha = 0.125$ [[28:43](http://www.youtube.com/watch?v=zt1E0akArqQ&t=1723)]|

URL Reference: [https://www.youtube.com/watch?v=zt1E0akArqQ](https://www.youtube.com/watch?v=zt1E0akArqQ)