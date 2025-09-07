Got it 🚀 — let’s tie this directly to **DP-300 exam style** and keep it short + crisp:

---

### **Sharding Types (DP-300 context)**

1. **Range-based sharding**

   * Data split by **value ranges** (e.g., IDs 1–1M → Shard A, 1M–2M → Shard B).
   * ✅ Easy to understand & query by range.
   * ❌ Risk of **hot spots** (one shard gets overloaded if new data always lands there).

---

2. **Hash-based sharding**

   * Data split by applying a **hash function** to shard key (e.g., `UserID % 4 → Shard #`).
   * ✅ Balances load evenly → avoids hot spots.
   * ❌ Range queries (`give me all users between 1000–2000`) become **hard/expensive**, since data is scattered.

---

3. **Lookup (Directory) sharding**

   * Uses a **mapping table** (directory) that tells where each record lives.
   * ✅ Very flexible → shard location doesn’t depend only on ID/range/hash.
   * ❌ Extra lookup step → performance overhead, directory must stay consistent.

---

👉 **DP-300 exam lens:**

* Range = good for sequential/natural splits but risk of uneven load.
* Hash = good for uniform distribution, bad for ranges.
* Lookup = flexible but adds indirection.

---
