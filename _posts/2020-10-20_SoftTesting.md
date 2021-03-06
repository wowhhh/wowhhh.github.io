Line : 保证程序中每一条语句最少执行一次，其覆盖标准无法发现判定中逻辑运算的错误；



Branch：是指选择足够的测试用例，使得运行这些测试用例时，每个**判定的所有可能结果**至少出现一次，也就是每个条件的True和False都出现一次，将整个判断看成一个整体。

​                 但若程序中的判定是有几个条件联合构成时，它未必能发现每个条件的错误；

>判定覆盖比语句覆盖强一些，能发现一些语句覆盖无法发现的问题。但是往往一些判定条件都是由多个逻辑条件组合而成的，进行分支判断时相当于对整个组合的最终结果进行判断，这样就会忽略每个条件的取值情况，导致遗漏部分测试路径。

对比分支覆盖，如果存在组合的条件判断，只关注是不是运行到了判断条件里面的语句，不关注每个条件是不是都配置到了。



Condition条件覆盖：是指选择足够的测试用例，使得运行这些测试用例时，判定中**每个条件的所有可能结果**至少出现一次，但未必能覆盖全部分支；对比上面，这里是取每个条件的可能的结果。

> 它只能保证每个条件都取到了不同结果，但没有考虑到判定结果，因此有时候条件覆盖并不能保证判定覆盖。



条件组合覆盖：

组合覆盖也叫做条件组合覆盖。意思是说我们设计的测试用例应该使得每个判定中的各个条件的各种可能组合都至少出现一次。显然，满足条件组合覆盖的测试用例一定是满足判定覆盖、条件覆盖和判定条件覆盖的。



路径覆盖: 路径覆盖，意思是说我们设计的测试用例可以覆盖程序中所有可能的执行路径。这种覆盖方法可以对程序进行彻底的测试用例覆盖



Fault：程序写错了

Error：有Bug了

Failure：与预期结果不一致。

#### Junit

```java
assertEquals(期望值，返回值);
assertTrue();
    
```

https://wiki.jikexueyuan.com/project/junit/api.html