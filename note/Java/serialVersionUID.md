实现序列化接口后，为什么要添加这个字段？

​	不添加**serialVersionUID** 这个字段，对象的序列化和反序列化动作之间，如果修改了类的字段，反序列化一定失败

​	添加后，对象的序列化和反序列化动作之间，如果只是添加了字段（不能删除原有的字段），反序列化不报错