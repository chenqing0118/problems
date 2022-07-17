# cpp union 关键字

联合类型，可以有多个数据成员，但同一时间，只有一个数据成员可以被赋值，有相同的地址。

分配给一个union对象的存储空间至少要能容纳它的最大的数据成员

联合类型可以有构造和析构函数，但是不能继承也不能作为基类。所以，union中不能含有虚函数