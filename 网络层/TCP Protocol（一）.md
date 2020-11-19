[

TCP RFC](https://tools.ietf.org/html/rfc793)

1. To provide this service on top of unreliable internet communication system requires facilities following:
   1. Basic Data Transfer
      1. The TCP protocol can transfer a continuous  stream of octets in each direction. 
   2. Reliability
      1. Use ACK mechanism to do this 
   3.  Flow Control : splide window algorithm 
      1. TCP provide a means for receiver to govern the amount of data that sent by the sender. This achieved by returning a “window” with every ACK indicating a range of acceptable number of data after perceding data had been received successfully.
   4. Multiplexing
      1. this was done by socket manchanism,  concatenated with the network and a host addresses of the internet communication layer. 
   5. Connections
      1. 由于必须在不可靠的主机和 在不可靠的互联网通讯系统上握手 基于时钟的序列号的机制被用来避免 连接的错误初始化。
   6. Precedency and security



The sequence number of the first octet of data in a segment is transmitted with that segment and is called the segment number.



## TCP 重传 

Transmission is made reliable via the use of sequence numbers and acknowledgments.  Conceptually, each octet of data is assigned sequence number.  The sequence number of the first octet of data in a segment is transmitted with that segment and is called the segmen sequence number.  Segments also carry an acknowledgment number which is the sequence number of the next expected data octet of transmissions in the reverse direction.  When the TCP transmits a segment containing data, it puts a copy on a retransmission queue an  starts a timer; when the acknowledgment for that data is received, the segment is deleted from the queue.  If the acknowledgment is not  received before the timer runs out, the segment is retransmitted.

## TCP 断开

 The clearing of a connection also involves the exchange of segments, in this case carrying the FIN control flag.



## TCP Format

TCP segments are sent as internet datagrams.  The Internet Protocol header carries several information fields, including the source and destination host addresses,**so，the TCP packet / header doesn’t have the source and destination host address**

```
  TCP Header Format


    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |          Source Port          |       Destination Port        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        Sequence Number                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Acknowledgment Number                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Data |           |U|A|P|R|S|F|                               |
   | Offset| Reserved  |R|C|S|S|Y|I|            Window             |
   |       |           |G|K|H|T|N|N|                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Checksum            |         Urgent Pointer        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             data                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

1. Source Port:  16 bits

2. Destination Port:  16 bits

3. Sequence Number:  32 bits

       The sequence number of the first data octet in this segment (except
       when SYN is present). If SYN is present the sequence number is the
       initial sequence number (ISN) and the first data octet is ISN+1.  

4. Acknowledge number: 32 bits

   1. ```
       If the ACK control bit is set this field contains the value of the
          next sequence number the sender of the segment is expecting to
          receive.  Once a connection is established this is always sent.
      ```

5. Data offsets 5 bits

   1. ```
      The number of 32 bit words in the TCP Header.  This indicates where
          the data begins.  The TCP header (even one including options) is an
          integral number of 32 bits long.
      ```

6. Control Bits:  6 bits (from left to right):

   ```
   
       URG:  Urgent Pointer field significant
       ACK:  Acknowledgment field significant
       PSH:  Push Function
       RST:  Reset the connection
       SYN:  Synchronize sequence numbers
       FIN:  No more data from sender
   ```

7. Window 16 bits

   1. ```
         The number of data octets beginning with the one indicated in the
          acknowledgment field which the sender of this segment is willing to
          accept.
      ```

8. Checksums 16 bits
9. 

## TCP 状态

1. LISTEN - represents waiting for a connection request from any remote
       TCP and port.
       

2. SYN-SENT - represents waiting for a matching connection request
   after having sent a connection request.

3. SYN-RECEIVED - represents waiting for a confirming connection
   request acknowledgment after having both received and sent a
   connection request.

4. ESTABLISHED - represents an open connection, data received can be
   delivered to the user.  The normal state for the data transfer phase
   of the connection.

5. FIN-WAIT-1 - represents waiting for a connection termination request
   from the remote TCP, or an acknowledgment of the connection
   termination request previously sent.

6. FIN-WAIT-2 - represents waiting for a connection termination request
   from the remote TCP.

7. CLOSE-WAIT - represents waiting for a connection termination request
   from the local user.

8. CLOSING - represents waiting for a connection termination request
   acknowledgment from the remote TCP.

9. LAST-ACK - represents waiting for an acknowledgment of the
   connection termination request previously sent to the remote TCP
   (which includes an acknowledgment of its connection termination
   request).
10. TIME-WAIT : 表示等待足够的时间来确定 远程TCP收到其连接的确认 终止请求。Representing waiting for longer enough time to ensure remote TCP connection received  the acknowledgement of the terimination request.



```
                              +---------+ ---------\      active OPEN
                              |  CLOSED |            \    -----------
                              +---------+<---------\   \   create TCB
                                |     ^              \   \  snd SYN
                   passive OPEN |     |   CLOSE        \   \
                   ------------ |     | ----------       \   \
                    create TCB  |     | delete TCB         \   \
                                V     |                      \   \
                              +---------+            CLOSE    |    \
                              |  LISTEN |          ---------- |     |
                              +---------+          delete TCB |     |
                   rcv SYN      |     |     SEND              |     |
                  -----------   |     |    -------            |     V
 +---------+      snd SYN,ACK  /       \   snd SYN          +---------+
 |         |<-----------------           ------------------>|         |
 |   SYN   |                    rcv SYN                     |   SYN   |
 |   RCVD  |<-----------------------------------------------|   SENT  |
 |         |                    snd ACK                     |         |
 |         |------------------           -------------------|         |
 +---------+   rcv ACK of SYN  \       /  rcv SYN,ACK       +---------+
   |           --------------   |     |   -----------
   |                  x         |     |     snd ACK
   |                            V     V
   |  CLOSE                   +---------+
   | -------                  |  ESTAB  |
   | snd FIN                  +---------+
   |                   CLOSE    |     |    rcv FIN
   V                  -------   |     |    -------
 +---------+          snd FIN  /       \   snd ACK          +---------+
 |  FIN    |<-----------------           ------------------>|  CLOSE  |
 | WAIT-1  |------------------                              |   WAIT  |
 +---------+          rcv FIN  \                            +---------+
   | rcv ACK of FIN   -------   |                            CLOSE  |
   | --------------   snd ACK   |                           ------- |
   V        x                   V                           snd FIN V
 +---------+                  +---------+                   +---------+
 |FINWAIT-2|                  | CLOSING |                   | LAST-ACK|
 +---------+                  +---------+                   +---------+
   |                rcv ACK of FIN |                 rcv ACK of FIN |
   |  rcv FIN       -------------- |    Timeout=2MSL -------------- |
   |  -------              x       V    ------------        x       V
    \ snd ACK                 +---------+delete TCB         +---------+
     ------------------------>|TIME WAIT|------------------>| CLOSED  |
                              +---------+                   +---------+

                      TCP Connection State Diagram
                               Figure 6.
```

## Sequence Number

A fundmental of this design is that evey octet of data sent by over TCP connection have a sequenec number.

socket 交换 ISN(initial sequence number)

    1) A --> B  SYN my sequence number is X
    2) A <-- B  ACK your sequence number is X
    3) A <-- B  SYN my sequence number is Y
    4) A --> B  ACK your sequence number is Y
三次握手的实质其实事交换 sequence number

A three way handshake is neccessary because sequence numbers are not tied to a global clock in the network, and TCPs may have different mechanism to choose the ISN’s，The receiver of the first SYN has no way of knowing the ISN whether the segment was an old delayed one or not, unless it remeber the last sequence number used on the connection(which not always possible), and so it must ask the sender to verify this SYN.  

## Create a connection

一个典型的三次握手

```
      TCP A                                                TCP B

  1.  CLOSED                                               LISTEN

  2.  SYN-SENT    --> <SEQ=100><CTL=SYN>               --> SYN-RECEIVED

  3.  ESTABLISHED <-- <SEQ=300><ACK=101><CTL=SYN,ACK>  <-- SYN-RECEIVED

  4.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK>       --> ESTABLISHED

  5.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK><DATA> --> ESTABLISHED
```

 In line 2 of figure 7, TCP A begins by sending a SYN segment  indicating that it will use sequence numbers starting with sequence number 100.

In line 3, TCP B sends a SYN and acknowledges the SYN it received from TCP A.  Note that the acknowledgment field indicates TCP B is now expecting to hear sequence 101, acknowledging the SYN which occupied sequence 100.

At line 4, TCP A responds with an empty segment containing an ACK for TCP B's SYN;

in line 5, TCP A sends some data.  Note that the sequence number of the segment in line 5 is the same as in line 4 because the ACK does not occupy sequence number space (if it did, we would wind up ACKing ACK's!).

**The principle reason for using three step handshake is to prevent old duplicated connections initiations from causing confusion.**

 