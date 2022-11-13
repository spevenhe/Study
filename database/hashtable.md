# hashtable

Two types of data structures in database:

→ Hash Tables

→ Trees


## design key
![image](https://user-images.githubusercontent.com/42630862/201528397-a40e2924-183f-4d5a-84fc-926ed35187dc.png)


# hashtabe desgin

![image](https://user-images.githubusercontent.com/42630862/201528612-1ad5ac54-3f6f-4857-8df9-491eeac34f98.png)


# static hashtable schema
## linear probe hashing (mostly for joins)
![image](https://user-images.githubusercontent.com/42630862/201528974-15547962-d353-490e-96c4-954d01756059.png)

![image](https://user-images.githubusercontent.com/42630862/201528986-f24663d5-0e66-4505-bec7-371f60d8f2aa.png)
![image](https://user-images.githubusercontent.com/42630862/201529128-9ce41dea-f8e6-442d-b7d0-de57ea8eddd8.png)

![image](https://user-images.githubusercontent.com/42630862/201529094-5ac22b85-9fcc-4392-ac5c-ee426fda8c83.png)

![image](https://user-images.githubusercontent.com/42630862/201529206-ed93898d-0aeb-468b-a5a8-8112fbcc904b.png)

## RO B I N H O O D H A S H I N G
Variant of linear probe hashing that steals slots
from "rich" keys and give them to "poor" keys.

→ Each key tracks the number of positions they are from
where its optimal position in the table.

→ On insert, a key takes the slot of another key if the first
key is farther away from its optimal position than the
second key

![image](https://user-images.githubusercontent.com/42630862/201529514-79125cf1-5e52-40a2-9d77-8a53f102b4ba.png)

![image](https://user-images.githubusercontent.com/42630862/201529491-8bac7906-2386-4a74-9e0d-78e233475d9a.png)
![image](https://user-images.githubusercontent.com/42630862/201529505-05bbb6d5-fc41-45cb-90c4-05ca1dafa859.png)

## C U C K O O H A S H I N G
Use multiple hash tables with different hash
function seeds.

→ On insert, check every table and pick anyone that has a
free slot.

→ If no table has a free slot, evict the element from one of
them and then re-hash it find a new location.

Look-ups and deletions are always O(1) because
only one location per hash table is checked.

Best open-source implementation:https://github.com/efficient/libcuckoo
![image](https://user-images.githubusercontent.com/42630862/201529913-e1829728-16fc-4ab2-9237-7c228370163e.png)
![image](https://user-images.githubusercontent.com/42630862/201529920-b8563721-bc9c-43ae-9ac9-dbd5e36905a6.png)
![image](https://user-images.githubusercontent.com/42630862/201529953-0f65c8c5-3319-4d62-965e-ffb21d366ab7.png)
![image](https://user-images.githubusercontent.com/42630862/201529963-6179e3ac-65b1-46d6-b3ed-3059af874e71.png)

The previous hash tables require the DBMS to
know the number of elements it wants to store.

→ Otherwise, it must rebuild the table if it needs to
grow/shrink in size.

**对于不唯一的key来说，可能要在更高层做去做抽象，获取rowid，等，这部分属于数据库其他层的事情，不应在hashtable 内做**

# dynamic hashtable schema

## C H A I N E D  H A S H I N G (indexes)
Maintain a linked list of buckets for each slot in
the hash table.
Resolve collisions by placing all elements with the
same hash key into the same bucket.

→ To determine whether an element is present, hash to its
bucket and scan for it.

→ Insertions and deletions are generalizations of lookups.


![image](https://user-images.githubusercontent.com/42630862/201530280-312033c5-75ff-43a1-9f95-f9f394122ea5.png)

**在某些程度上 类似 linear hashing 对于not unique key 的处理**

## E X T E N D I B L  E H A S H I N G

Chained-hashing approach where we split buckets
instead of letting the linked list grow forever.
Multiple slot locations can point to the same
bucket chain.

Reshuffle bucket entries on split and increase the
number of bits to examine.

→ Data movement is localized to just the split chain.

**hash 是通过global来做，global 会随着bit 增加而large**
![image](https://user-images.githubusercontent.com/42630862/201530825-d246ee12-8f01-4161-ace0-f5ceb6c9e2af.png)

![image](https://user-images.githubusercontent.com/42630862/201530853-0b6ad7aa-f3f2-4f58-a5ad-78b4f2544bec.png)
**此时hash(c) 冲突了，global 指针就改成使用3个bit**
![image](https://user-images.githubusercontent.com/42630862/201530962-6f3ff3c4-064f-4bdc-98bc-893dd0edf32d.png)

**rehash**

![image](https://user-images.githubusercontent.com/42630862/201530968-63b8783d-558a-4afb-aff0-142f02435325.png)


## L I N E A R H A S H I N G （mostly）
The hash table maintains a pointer that tracks the
next bucket to split.

→ When any bucket overflows, split the bucket at the
pointer location.
Use multiple hashes to find the right bucket for a
given key.
Can use different overflow criterion:

→ Space Utilization

→ Average Length of Overflow Chains

![image](https://user-images.githubusercontent.com/42630862/201531722-74022379-2d00-4350-b6dd-3c5e6ef27843.png)

![image](https://user-images.githubusercontent.com/42630862/201531739-ee695de6-f4f1-4c1e-aded-f6c19074be99.png)
**移动spill pointer**
![image](https://user-images.githubusercontent.com/42630862/201531745-79a3da53-70c0-4350-8440-bfc0c4db76e2.png)
因为在spilt pointer 前，知道被spilt 了， 使用 hash2 函数
![image](https://user-images.githubusercontent.com/42630862/201531767-1c5ba234-5116-4fd2-b15a-c7db49b2de84.png)

![image](https://user-images.githubusercontent.com/42630862/201531804-da1fa482-bfdf-46f5-a64f-29edf218ddcc.png)




