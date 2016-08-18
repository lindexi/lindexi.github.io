# win10 uwp 网络编程


 我们来弄简单协议
 
 首先第一层是传输层，这一层是Head+Data
 
 其中Head就是有传输的头，加上Data长度
 
 Head=Head+length
 
 length是Data长度
 
 上面传输的头，Head=Head+length 中的第二个Head，包含
 传输者id，当前传输是传输的消息最后一段还是中间
 
 传输的最后一段还是中间指的是在上一层，有很长的数据，被拆为多个Data发送，
 这时就需要标注接下来几条消息要合并为一条