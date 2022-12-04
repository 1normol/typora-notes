# Netty





## Netty组件分析

- Channel：理解为数据流转通道
- msg：简单来说，msg就是流转的数据，数据最开始输入时ByteBuf，经过pipeline的加工，转换成其他的对象，最后输出又变为ByteBuf对象
- handler：数据处理的处理工序
  - handler可以有多个，合在一起就是pipeline，pipeline负责发布事件，如读取，读取完成，传播给每一个handler,handler通过重写对应事件的方法来进行对自己感兴趣的事件处理。
  - handler分为Inbound和Outbound两类
- eventLoop：负责管理处理数据
  - 一个eventLoop可以管理多个channel的io操作，通过绑定关联将channel关联到eventLoop
  - 