## @ConditionalOnProperty

1. 简介

**Spring Boot**通过**@ConditionalOnProperty**来控制**Configuration**是否生效

2. 使用方法

- 通过其两个属性**name**以及**havingValue**来实现的，

- 其中**name**用来从**application.properties**中读取某个属性值,

  - 如果为空，则返回false
  - 如果不为空，则将该值与havingValue指定的值进行比较，如果一样则返回true;否则返回false。

- 如果返回值为false，则该configuration不生效；为true则生效。

