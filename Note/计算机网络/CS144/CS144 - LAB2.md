# CS144 - Lab 2

比起 lab1 一堆没说清楚的规则，lab2 就友好了很多，我们只需要去判断一些字段，然后调用我们在 lab1 里面实现的类方法就行了

---

首先我们需要需要做一些准备工作，把 wrap 和 unwrap 给准备好，这里的 unwrap 我感觉其实也不是非常清晰，还是看了测试点才知道她是想要我们干啥，至于 wrap 一行代码就可以秒杀了，他是希望我们根据零点和给定的 64 为无符号整数转换为 无符号 32 位的数字并且需要包装在 wrap32 的类里面

```cpp
Wrap32 Wrap32::wrap( uint64_t n, Wrap32 zero_point )
{
  return Wrap32( ( n + zero_point.raw_value_ + ( 1ULL << 32 ) ) % ( 1ULL << 32 ) );
}
```

而对于 unwrap，就需要麻烦一点，给定一个零点和一个 checkpoint，需要我们把调用这个方法的对象对应的值给转换成 64 位整型，就相当于是 wrap 的逆操作。

给定的 checkpoint 是因为我们逆运算出来的值可能有多个，比如 1 可以对应 `1 + (1ULL << 32) * n ` 所以我们给定了一个 checkpoint，我们只需要尽可能地接近这个 checkpoint 就行了。

此时根据与 checkpoint 的接近情况，也会出现三种取值，具体如下：

```cpp
uint64_t Wrap32::unwrap( Wrap32 zero_point, uint64_t checkpoint ) const
{
  // 零点值和当前值的差值。
  uint64_t offset = ( ( 1ULL << 32 ) + ( raw_value_ - zero_point.raw_value_ ) ) % ( 1ULL << 32 );
  // 这里是计算去除低 32 位的偏移量然后只取前面的部分。
  uint64_t base = ( checkpoint / ( 1ULL << 32 ) ) * ( 1ULL << 32 );

  // 取值可能有三种情况
  uint64_t src[3] = { base + offset, base - ( 1ULL << 32 ) + offset, base + ( 1ULL << 32 ) + offset };

  uint64_t closest = src[0];
  uint64_t min_diff = ( src[0] > checkpoint ) ? ( src[0] - checkpoint ) : ( checkpoint - src[0] );
  for ( int i = 1; i < 3; i++ ) {
    uint64_t diff = ( src[i] > checkpoint ) ? ( src[i] - checkpoint ) : ( checkpoint - src[i] );
    // 如果差值更小，则更新最近值
    if ( diff < min_diff ) {
      min_diff = diff;
      closest = src[i];
    }
  }
  return closest;
}
```

---

那么对于 wrap 的准备工作已经做完了，此时你可以输入 `cmake --build build --target check2` 验证一下对于 wrap 的测试样例是否全部通过了，然后我们可以着手编写 send 和 receive 了。

这个 receive 针对的是从客户端接受的数据进行处理，而 send 代表我们需要回复 ack 表示确认接收到，并发送需要的下一个字节索引位置。针对于我们的 receiver 类，我们也需要编写一些字段表示一些状态：

```cpp
class TCPReceiver
{
public:
  // Construct with given Reassembler
  explicit TCPReceiver( Reassembler&& reassembler ) : reassembler_( std::move( reassembler ) ) {}

  /*
   * The TCPReceiver receives TCPSenderMessages, inserting their payload into the Reassembler
   * at the correct stream index.
   */
  void receive( TCPSenderMessage message );

  // The TCPReceiver sends TCPReceiverMessages to the peer's TCPSender.
  TCPReceiverMessage send() const;

  // Access the output (only Reader is accessible non-const)
  const Reassembler& reassembler() const { return reassembler_; }
  Reader& reader() { return reassembler_.reader(); }
  const Reader& reader() const { return reassembler_.reader(); }
  const Writer& writer() const { return reassembler_.writer(); }

private:
  Reassembler reassembler_;
  // 新增的字段，表示是否被初始化
  Wrap32 initial_seqno_ { 0 };
  bool is_initialized_ { false };
};
```

然后我们可以开始着手去编写 send 方法，因为我觉得先写 send 的话，更有助于我们去写 receive，而且 send 的判断逻辑更简单，我们首先需要判断是否已经初始化，如果已经初始化了，我们由于最开始发送方会先发送 synchronize 同步信号，所以我们还需要给当前的序列号加上一位 offset，对于关闭的连接，我们还需要加一位 fin 的信号。

然后这里我们的窗口大小不能超过 2^16，因为测试样例是这么告诉我的🤮

```cpp
TCPReceiverMessage TCPReceiver::send() const
{
  TCPReceiverMessage message;

  uint64_t next_seqno = reassembler_.next_byte_index();
  if ( is_initialized_ ) {
    // 计算 ackno 时需要考虑 SYN 占用一个序列号
    uint64_t offset = 1;
    // 如果连接已经关闭，还需要一个 FIN
    if ( reassembler_.writer().is_closed() ) {
      offset += 1;
    }
    message.ackno = Wrap32::wrap( next_seqno + offset, initial_seqno_ );
  }

  // 面向测试样例编程，窗口大小不能超过 UINT16_MAX
  uint64_t window_size = reassembler_.available_capacity();
  message.window_size = min( window_size, static_cast<uint64_t>( UINT16_MAX ) );

  // 如果发生错误，设置 RST 标志
  message.RST = reassembler_.reader().has_error();

  return message;
}
```

最后我们可以通过 send 清楚的知道整个发送流程是如何工作的，如果这边 receive 接收到了 syn 同步信号，那么就说明我们需要初始化信息，然后接下来我们根据已经插入的数据计算应该插入的位置，别忘了，我们最开始有一位 SYN 并没有被插入到数据中，所以需要 -1，然后判断 FIN 标志位，正常结束就行了。

```cpp
void TCPReceiver::receive( TCPSenderMessage message )
{
  // 处理 RST 标志，这里是传输出现错误了
  if ( message.RST ) {
    reassembler_.reader().set_error();
    return;
  }

  // 处理 SYN 标志
  if ( message.SYN ) {
    if ( !is_initialized_ ) {
      initial_seqno_ = message.seqno; // 保存初始序列号
      is_initialized_ = true;
    }
  }

  // 如果连接尚未初始化，则不处理数据
  if ( !is_initialized_ ) {
    return;
  }

  // 计算数据应该插入的索引位置
  uint64_t _index = 0;
  if ( message.SYN ) {
    _index = 0;
  } else {
    // 因为还有一个 SYN 多余的，所以需要减去1
    _index = message.seqno.unwrap( initial_seqno_, reassembler_.next_byte_index() ) - 1;
  }

  // 处理数据段
  if ( !message.payload.empty() ) {
    reassembler_.insert( _index, message.payload, message.FIN );
  } else if ( message.FIN ) {
    // 如果 FIN 标志并且没有数据
    reassembler_.insert( _index, "", true );
  }
}
```

总的来说，这个实验还是比较简单的，下面给出最终的测试结果：

```bash
Test project /home/rinai/project/CS144/build
      Start  1: compile with bug-checkers
 1/29 Test  #1: compile with bug-checkers ........   Passed    0.04 sec
      Start  3: byte_stream_basics
 2/29 Test  #3: byte_stream_basics ...............   Passed    0.03 sec
      Start  4: byte_stream_capacity
 3/29 Test  #4: byte_stream_capacity .............   Passed    0.03 sec
      Start  5: byte_stream_one_write
 4/29 Test  #5: byte_stream_one_write ............   Passed    0.03 sec
      Start  6: byte_stream_two_writes
 5/29 Test  #6: byte_stream_two_writes ...........   Passed    0.03 sec
      Start  7: byte_stream_many_writes
 6/29 Test  #7: byte_stream_many_writes ..........   Passed    0.13 sec
      Start  8: byte_stream_stress_test
 7/29 Test  #8: byte_stream_stress_test ..........   Passed    0.08 sec
      Start  9: reassembler_single
 8/29 Test  #9: reassembler_single ...............   Passed    0.05 sec
      Start 10: reassembler_cap
 9/29 Test #10: reassembler_cap ..................   Passed    0.04 sec
      Start 11: reassembler_seq
10/29 Test #11: reassembler_seq ..................   Passed    0.04 sec
      Start 12: reassembler_dup
11/29 Test #12: reassembler_dup ..................   Passed    0.15 sec
      Start 13: reassembler_holes
12/29 Test #13: reassembler_holes ................   Passed    0.06 sec
      Start 14: reassembler_overlapping
13/29 Test #14: reassembler_overlapping ..........   Passed    0.03 sec
      Start 15: reassembler_win
14/29 Test #15: reassembler_win ..................   Passed    0.49 sec
      Start 16: wrapping_integers_cmp
15/29 Test #16: wrapping_integers_cmp ............   Passed    0.03 sec
      Start 17: wrapping_integers_wrap
16/29 Test #17: wrapping_integers_wrap ...........   Passed    0.02 sec
      Start 18: wrapping_integers_unwrap
17/29 Test #18: wrapping_integers_unwrap .........   Passed    0.02 sec
      Start 19: wrapping_integers_roundtrip
18/29 Test #19: wrapping_integers_roundtrip ......   Passed    0.77 sec
      Start 20: wrapping_integers_extra
19/29 Test #20: wrapping_integers_extra ..........   Passed    0.17 sec
      Start 21: recv_connect
20/29 Test #21: recv_connect .....................   Passed    0.03 sec
      Start 22: recv_transmit
21/29 Test #22: recv_transmit ....................   Passed    0.32 sec
      Start 23: recv_window
22/29 Test #23: recv_window ......................   Passed    0.03 sec
      Start 24: recv_reorder
23/29 Test #24: recv_reorder .....................   Passed    0.03 sec
      Start 25: recv_reorder_more
24/29 Test #25: recv_reorder_more ................   Passed    1.58 sec
      Start 26: recv_close
25/29 Test #26: recv_close .......................   Passed    0.03 sec
      Start 27: recv_special
26/29 Test #27: recv_special .....................   Passed    0.04 sec
      Start 37: compile with optimization
27/29 Test #37: compile with optimization ........   Passed    0.03 sec
      Start 38: byte_stream_speed_test
28/29 Test #38: byte_stream_speed_test ...........   Passed    0.15 sec
      Start 39: reassembler_speed_test
29/29 Test #39: reassembler_speed_test ...........   Passed    0.35 sec

100% tests passed, 0 tests failed out of 29

Total Test time (real) =   4.90 sec
```



