# CS144 - Lab 1

做了 lab1，最开始有很多 bug，感觉 cs144 确实有很多地方很难搞，太几把多规则了，而且这才 lab1，属实有点想不做了，但 lab1 还是写吧，看后面能写多少写多少，过程中主要是感觉因为一些无关紧要的代码而消磨了很多时间，感觉不是很值当。最后也是参考了别人的才写出来了 。

---

这里是需要我们为传输过来的字节流进行排序然后推送给我们在 lab0 中实现的内存字节流推送，这里有很多比较刁钻的地方，比如说字节流超出了需要怎么处理，和缓存的字节流重叠了需要怎么处理，然而这些手册上并没有明确指出，这让我感觉很受不了。

直接讲思路，我们有一个缓冲区，当我们接下来推送的字符串的索引下标不是对应的我们当前需要的索引下标的时候，我们需要将他缓存起来，并且如果当这个字符串的起始索引加上长度如果超出了我们的 capacity，还需要对他进行截取，总之就是，截取就完了，只需要按照 capacity 可以容纳的最大长度进行截取就可以了，如果一开始的字符串下标索引就超出了，就可以抛弃了。

其实这些都没啥难的，因为都是些条件判断，字符串截取而已，重点是字符串如果有重叠，我们应该如何处理。

首先给出整体的框架，也就是我们的 Reassembler 类

```cpp
class Reassembler
{
public:
	// 这个是构造函数
  explicit Reassembler( ByteStream&& output )
    : output_( std::move( output ) ), capacity_( writer().available_capacity() )
  {}
  void insert( uint64_t first_index, std::string data, bool is_last_substring );

  // 获取 Reassembler 中当前存储的字节数。
  uint64_t bytes_pending() const;

  // reader
  Reader& reader() { return output_.reader(); }
  const Reader& reader() const { return output_.reader(); }

  // 获取只读的 writer
  const Writer& writer() const { return output_.writer(); }
  // 工具函数，获取缓冲区剩下的容量
  uint64_t available_capacity() const { return writer().available_capacity(); };
  // 获取接下来需要的 index
  uint64_t next_byte_index() const { return writer().bytes_pushed(); };
  // 我们自己要去实现，合并字符串
  void merge_substrings();

private:
  ByteStream output_;
  uint64_t capacity_;
  uint64_t last_byte_index_ = -1;
  std::multimap<uint64_t, std::string> pending_data_ {};
};
```

为了操作简单，不去处理在插入的时候重叠怎么办，直接在插入缓冲区之后，在进行重叠合并，所以每次插入数据都需要对哈希表进行遍历操作，然后这里给出我们的 merge_substrings 的实现：

```cpp
void Reassembler::merge_substrings()
{
  if ( pending_data_.size() <= 1 )
    return;

  auto it = pending_data_.begin();
  auto next = std::next( it );

  // 遍历合并。
  while ( next != pending_data_.end() ) {
    // 如果当前子串和下一个子串有重叠
    if ( it->first + it->second.length() >= next->first ) {

      // 如果当前子串完全包含下一个子串，直接删除下一个子串
      if ( it->first + it->second.length() >= next->first + next->second.length() ) {
        next = pending_data_.erase( next );
        continue;
      }

      // 计算重叠部分的字节索引，将重叠部分的子串合并到当前子串中
      uint64_t overlap_index = it->first + it->second.length() - next->first;
      it->second += next->second.substr( overlap_index );

      // 删除下一个子串
      next = pending_data_.erase( next );
    } else {
      it ++;
      next ++;
    }
  }
}
```

这里其实最开始感觉这种合并子串很麻烦，我就直接用的 char 来实现的传输，但是后面一个测试样例始终过不去，哈哈，受不了了。

对于任何字符串，我们都需要及时的进行删除，一定不要忘了，否则会导致我们的计数出错，对于计数，我的建议是不要使用一个成员来记录，因为我最开始就是这么做的，后面各种各样改动就导致了这个很难进行修改，所以最好是通过遍历哈希表来计算所有缓存数据的长度。

```cpp
uint64_t Reassembler::bytes_pending() const
{
  uint64_t count = 0;
  for ( auto it = pending_data_.begin(); it != pending_data_.end(); it ++ ) {
    count += it->second.length();
  }
  return count;
}
```

最后就是 insert，看着很复杂，其实就是一些条件判断：

```cpp
void Reassembler::insert( uint64_t first_index, std::string data, bool is_last_substring )
{
  if ( data.empty() && is_last_substring && first_index == next_byte_index() ) {
    output_.writer().close();
    return;
  }

  // 如果是最后一个子串，更新最后一个字节的位置
  if ( is_last_substring ) {
    last_byte_index_ = first_index + data.length() - 1;
  }

  // 如果当前子串可以插入
  if ( first_index < next_byte_index() + available_capacity() && first_index + data.length() > next_byte_index() ) {
    uint64_t insert_key = std::max( first_index, next_byte_index() );

    pending_data_.insert( std::make_pair(
      insert_key,
      data.substr( insert_key - first_index,
                   std::min( data.length(), next_byte_index() + available_capacity() - insert_key ) ) ) );
    // 每次插入都需要进行合并
    merge_substrings();

    if ( pending_data_.begin()->first == next_byte_index() ) {
      output_.writer().push( pending_data_.begin()->second );

      if ( pending_data_.begin()->first + pending_data_.begin()->second.length() - 1 == last_byte_index_ ) {
        output_.writer().close();
      }

      // 删除已经处理完的子串
      pending_data_.erase( pending_data_.begin() );
    }
  }
}
```

在最后

```bash
root@r-linux:/home/rinai/project/CS144# cmake --build build --target check1
Test project /home/rinai/project/CS144/build
      Start  1: compile with bug-checkers
 1/17 Test  #1: compile with bug-checkers ........   Passed    5.64 sec
      Start  3: byte_stream_basics
 2/17 Test  #3: byte_stream_basics ...............   Passed    0.01 sec
      Start  4: byte_stream_capacity
 3/17 Test  #4: byte_stream_capacity .............   Passed    0.01 sec
      Start  5: byte_stream_one_write
 4/17 Test  #5: byte_stream_one_write ............   Passed    0.01 sec
      Start  6: byte_stream_two_writes
 5/17 Test  #6: byte_stream_two_writes ...........   Passed    0.01 sec
      Start  7: byte_stream_many_writes
 6/17 Test  #7: byte_stream_many_writes ..........   Passed    0.04 sec
      Start  8: byte_stream_stress_test
 7/17 Test  #8: byte_stream_stress_test ..........   Passed    0.02 sec
      Start  9: reassembler_single
 8/17 Test  #9: reassembler_single ...............   Passed    0.01 sec
      Start 10: reassembler_cap
 9/17 Test #10: reassembler_cap ..................   Passed    0.01 sec
      Start 11: reassembler_seq
10/17 Test #11: reassembler_seq ..................   Passed    0.02 sec
      Start 12: reassembler_dup
11/17 Test #12: reassembler_dup ..................   Passed    0.03 sec
      Start 13: reassembler_holes
12/17 Test #13: reassembler_holes ................   Passed    0.01 sec
      Start 14: reassembler_overlapping
13/17 Test #14: reassembler_overlapping ..........   Passed    0.01 sec
      Start 15: reassembler_win
14/17 Test #15: reassembler_win ..................   Passed    0.30 sec
      Start 37: compile with optimization
15/17 Test #37: compile with optimization ........   Passed    2.76 sec
      Start 38: byte_stream_speed_test
             ByteStream throughput: 2.42 Gbit/s
16/17 Test #38: byte_stream_speed_test ...........   Passed    0.10 sec
      Start 39: reassembler_speed_test
             Reassembler throughput: 7.73 Gbit/s
17/17 Test #39: reassembler_speed_test ...........   Passed    0.14 sec

100% tests passed, 0 tests failed out of 17

Total Test time (real) =   9.15 sec
Built target check1
```

感觉自己可能写不下去这个实验，这个 lab1 就花了两天时间byd，真的相似了，走一步算一步吧。