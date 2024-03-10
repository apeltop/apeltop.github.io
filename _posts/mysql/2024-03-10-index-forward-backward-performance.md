---
title: MySQL 왜 Forward Scan 이 Backward Scan 보다 빠를까? 
author: sunshine@ptokos.com
categories: [mysql]
---


## 목표
MySQL 에서 **forward scan** 이 **backward scan** 보다 빠른 이유에 대해 알아보자.

### 상황
```sql
CREATE TABLE test_table (
    id INT AUTO_INCREMENT PRIMARY KEY,
    data VARCHAR(100),
    created_at DATETIME
);

INSERT INTO test_table (data, created_at) VALUES ('Data 1', '2024-01-01 12:00:00');
INSERT INTO test_table (data, created_at) VALUES ('Data 2', '2024-01-02 12:00:00');
INSERT INTO test_table (data, created_at) VALUES ('Data 3', '2024-01-03 12:00:00');
...

```

위와 같은 테이블이 있다고 가정하자.

```sql
SELECT * FROM test_table ORDER BY ASC id LIMIT 100000000, 1;
SELECT * FROM test_table ORDER BY id DESC LIMIT 100000000, 1;
```

이 때 위 쿼리의 실행 결과는 첫 번재 쿼리가 더 빠른데 이유를 알아보자.

## Index
index 는 데이터베이스에서 데이터를 빠르게 조회하기 위해 사용한다.
index 가 있으면 insert 와 update 시 성능이 떨어지지만 select 시 성능이 빨라진다.

MySQL 에서 인덱스 알고리즘은 **B-Tree** 와 **Hash** 가 있다.

### Hash
간단한 Hash 알고리즘부터 알아보자.

> 칼럼의 값을 해싱하여 인덱싱하는 방법이다.

#### 특징
매우 빠른 검색을 제공한다. 그러나 해싱을 하다보니 **값이 변형**이 될 경우 인덱스 사용이 **불가능**하다.
그러기에 **<** 와 같은 비교 연산자를 사용할 수 없다. **ORDER BY** 도 인덱스 사용이 불가능하다.
그래서 주로 **key-value** 형태에 사용된다.

### B-Tree, B+Tree
B-Tree 하면 자료구조에서 배우듯 **Binary Tree** 를 떠올리는 것이 일반적이다.
그렇지만 MySQL 에서는 **Balanced Tree** 를 의미한다. 자식 노드를 2개 이상 가질 수 있기 때문이다.

자세히 살펴보면 B-Tree 를 응용한 **B+Tree** 를 사용한다.
B-Tree 에서는 Node 가 Key 값과 Data 를 가지고 있지만 B+Tree 에서 **Data 는 Leaf Node** 에서만 가지고 있다.

Root 도 아니고 Leaf 가 아닌 Node 를 **Internal Node** 또는 **Non-Leaf Node** 라고 한다.

또 다른 B+Tree 의 특징은 같은 레벨인 Node 들이 **Double Linked List 로 연결**되어 있다는 점이다.

![B+Tree](https://upload.wikimedia.org/wikipedia/commons/thumb/3/37/Bplustree.png/800px-Bplustree.png)

출처 [위키피디아 B+Tree](https://en.wikipedia.org/wiki/B%2B_tree)

## Page
MySQL 에서는 **Page** 는 B-Tree 에서 **Node** 와 같다.

![B+Tree Structure](https://i0.wp.com/jcole.us/blog/files/innodb/20130109/50dpi/B_Tree_Structure.png)

출처 [B+Tree index structures in InnoDB](https://blog.jcole.us/2013/01/10/btree-index-structures-in-innodb/)

각 노드마다 Page 번호로 구성이 되어있는 것을 볼 수 있다.
위에서 B-Tree 를 설명하였던 것 같이 Leaf Node 만 데이터를 가지고 있고 나머지는 Key 값만 가지고 있는 것을 볼 수 있다.

Leaf Node 를 왼쪽에서 오른쪽으로 읽으면 데이터가 정렬된 데이터를 읽을 수 있다.

중요한 점은 Leaf Node 의 **Data 는 index 의 key 값**이다. 그러므로 key 이외에 다른 데이터를 가져오려면 데이터 파일에 접근해서 가져와야 한다.

위와 같은 구조 때문에 index 가 테이블에 데이터 insert, update 시 성능이 떨어지는 것이다.
Leaf Node 가 꽉 차게 되면 Split 이 발생하게 되고 이때 데이터를 재배치하게 된다.
작업 범위를 생각해보면 꽉 차게 된 Leaf Node, 상위 Internal Node 까지 작업 범위가 된다.

위 B+Tree Structure 이미지에서 보면 각 레벨의 노드는 Double linked list 로 연결되어 있다.
그러기에 오름차순과 오름차순의 속도 차이는 없어야할 것으로 보인다. 그러나 **실제로는 차이가 있다**.
이 부분을 알기 위해 더 Page 의 더 작은 단위인 page directory 와 page directory slot 에 대해 알아보자.

추가적으로 여기서 기억하면 좋을 것 하나는 인덱스 키의 크기이다.
인덱스 페이지는 Key, 자식 주소를 가지고 있기 때문에 인덱스 키가 클수록 페이지에 들어갈 수 있는 레코드 수가 줄어든다.
innodb_page_size 의 기본 값이 16KB 이다. 

index key 의 크기가 작을 수록 한 페이지에 들어갈 수 있는 레코드가 많기 때문에 더 빠르게 데이터를 찾을 가능성이 높아진다.
한 페이지 안에서 조회가 끝날 경우 다음 페이지로 넘어가지 않아도 되기 때문이다.


### Page Directory
Page Directory 는 **Directory Slot** 정보를 가지고 있다.
이와 같은 구조가 필요한 이유는 데이터가 들어있는 page 를 찾아도 page 에 레코드가 많아질 경우 레코드를 하나씩 찾아야 하기 때문에 느려진다.
이를 개선하기 위해 Slot 으로 나누어 불필요한 레코드 검색을 건너뛸 수 있게 하였다.
Slot 은 8개의 레코드를 넘지 않도록 한다. 넘을 경우 4개 레코드로 나누어진다.

Page Directory 는 언제나 infimum 과 supremum 을 가지고 있다.
infimum 과 supremum 은 system records 이다. 이 둘은 index page 에서 고정된 위치에 있기에 byte offset 으로 직접 접근이 가능하다.

![Page Directory](https://i0.wp.com/jcole.us/blog/files/innodb/20130114/50dpi/INDEX_Page_Directory.png)

출처 [Efficiently traversing InnoDB B+Trees with the page directory](https://blog.jcole.us/2013/01/14/efficiently-traversing-innodb-btrees-with-the-page-directory/)


아래 첨부한 B+Tree Page Directory Structure 를 보면 상단에 있는 한 행으로 되어있는 것이 slot 목록이다.
데이터를 찾을 때 이 slot 목록을 참조하여 데이터가 있는 page 를 효율적으로 찾을 수 있다.

밑에 각 레코드들이 연결되어있는데 **K** 는 **Key** 값이고 **O** 는 **Owned** 를 의미한다.
가지고 있는 레코드 수를 의미한다. 이 값은 slot 만 가질 수 있다. Infimum 의 N_Owned 가 1인 이유는 자기 자신을 가지고 있기 때문이다.

Page 안에 레코드는 **Single linked list** 로 연결되어 있다. 그렇기에 Backward index scan 일 때 느리다. 
다시 말해 같은 레벨의 page 는 double linked list 로 연결되어 있지만 page 안에 레코드는 single linked list 로 연결되어 있다.

Kakao Tech 블로그에서 matt.lee 의 MySQL Ascending index vs Descending index 글을 보면 이러한 차이로 약 **30% 정도의 차이**가 난다고 한다.   

![Page Directory Structure](https://i0.wp.com/jcole.us/blog/files/innodb/20130114/50dpi/B_Tree_Page_Directory_Structure.png)

출처 [Efficiently traversing InnoDB B+Trees with the page directory](https://blog.jcole.us/2013/01/14/efficiently-traversing-innodb-btrees-with-the-page-directory/)


## MySQL 코드로 확인하기
[mysql-server](https://github.com/mysql/mysql-server) 를 clone 받아서 코드를 확인해보자.

**storagee/innobaase/btr/btr0pcur.cc** 파일을 열어보자.
그리고 2개의 함수를 비교해보자. 2개의 함수는 페이지를 이동하는 함수이다.
- move_to_next_page
- move_backward_from_page

**move_to_next_page** 함수를 중요 부분만 가져와보자.
1. next_block 을 잠금하고 가져온다.
2. next_page 를 가져온다.
3. 현재 page 를 release (lock 해제) 한다.
4. next_block 을 cursor 가 가리키는 page 로 설정한다.

> 함수 인자인 mtr 은 Mini-Transaction 이다. Mini-Transaction 은 InnoDB 에서 사용하는 트랜잭션의 작은 단위이다.

```cpp
void btr_pcur_t::move_to_next_page(mtr_t *mtr) {
  ...

  auto next_block =
      btr_block_get(page_id_t(block->page.id.space(), next_page_no),
                    block->page.size, mode, UT_LOCATION_HERE, index, mtr);

  auto next_page = buf_block_get_frame(next_block);

  btr_leaf_page_release(get_block(), mode, mtr);

  page_cur_set_before_first(next_block, get_page_cur());
  ...
}
```

[page_cur_set_before_first](https://dev.mysql.com/doc/dev/mysql-server/latest//page0cur_8ic.html#a98ba4279d173afa32612674e8ab09b09) 의 경우 공식 문서를 참고해보면
> Sets the cursor object to point before the first user record on the page.

user record 는 다음 레코드를 가르킨다. 

**move_backward_from_page** 함수를 중요 부분만 가져와보자.

forward 와 달리 왼쪽으로 이동해야 하기에 left_block 과 prev 를 사용해서 이동한다.

가장 눈에 띄는 차이점은 처음 3줄이다.

트랜잭션을 저장하고 commit 한 후 다시 트랜잭션을 시작한다.
또한 restore_position 을 통해 복구 작업을 한다.
이러한 작업을 하는 이유는 페이지 작업이 Forward index scan 에 적합한 구조이기 때문이다. 

```cpp
void btr_pcur_t::move_backward_from_page(mtr_t *mtr) {
  ...
  store_position(mtr);
  mtr_commit(mtr);
  mtr_start(mtr);

  restore_position(latch_mode2, mtr, UT_LOCATION_HERE);

  if (!get_btr_cur()->index->table->is_intrinsic()) {
    buf_block_t *prev_block;

    if (prev_page_no == FIL_NULL) {
      ;
    } else if (is_before_first_on_page()) {
      prev_block = get_btr_cur()->left_block;

      btr_leaf_page_release(get_block(), old_latch_mode, mtr);

      page_cur_set_after_last(prev_block, get_page_cur());
    } else {
      prev_block = get_btr_cur()->left_block;

      btr_leaf_page_release(prev_block, old_latch_mode, mtr);
    }
  }
  ...
}
```

페이지를 이동하는 것이 아니라 레코드 단위로 코드를 보자.
**storage/innobase/include/page0page.ic** 파일을 열어보자.

Forward scan 의 경우 정말 간단하게 다음 offset 을 가져와서 정보를 반환한다.
```cpp
static inline const rec_t *page_rec_get_next_low(
    const rec_t *rec, /*!< in: pointer to record */
    ulint comp)       /*!< in: nonzero=compact page layout */
{
  ...

  page = page_align(rec);
  offs = rec_get_next_offs(rec, comp);
  
  ...

  return (page + offs);
}
```

> rec 인자 변수명은 record 를 의미한다.

Backward scan 의 경우 **while 문**을 통해 해당 레코드를 찾는다.
한 Page Slot 당 최대 8개 레코드이기 때문에 많은 수는 아닐 수 있지만 Forward scan 보다는 느리다.

```cpp
/** Gets the pointer to the previous record.
 @return pointer to previous record */
static inline const rec_t *page_rec_get_prev_const(
    const rec_t *rec) /*!< in: pointer to record, must not be page
                      infimum */
{
  ...
  slot = page_dir_get_nth_slot(page, slot_no - 1);
  rec2 = page_dir_slot_get_rec(slot);

  ...
  while (rec != rec2) {
    prev_rec = rec2;
    rec2 = page_rec_get_next_low(rec2, false);
  }
  
  ...

  return (prev_rec);
}
```

#### 참고 문서
- [10.3.9 Comparison of B-Tree and Hash Indexes](https://dev.mysql.com/doc/refman/8.3/en/index-btree-hash.html)
- [17.6.2.2 The Physical Structure of an InnoDB Index](https://dev.mysql.com/doc/refman/8.3/en/innodb-physical-structure.html)
- [B+Tree index structures in InnoDB](https://blog.jcole.us/2013/01/10/btree-index-structures-in-innodb/)
- [Efficiently traversing InnoDB B+Trees with the page directory](https://blog.jcole.us/2013/01/14/efficiently-traversing-innodb-btrees-with-the-page-directory/)
- [The physical structure of InnoDB index pages](https://blog.jcole.us/2013/01/07/the-physical-structure-of-innodb-index-pages/)
- [MySQL Ascending index vs Descending index](https://tech.kakao.com/2018/06/19/mysql-ascending-index-vs-descending-index/)
- [Overview of INDEX page structure](https://blog.jcole.us/category/mysql/page/5/)

