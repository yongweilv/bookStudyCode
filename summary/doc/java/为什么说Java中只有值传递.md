#URL
https://blog.csdn.net/bjweimengshu/article/details/79799485

#回答
* 所以，值传递和引用传递的区别并不是传递的内容。而是实参到底有没有被复制一份给形参。在判断实参内容有没有受影响的时候，要看传的的是什么，如果你传递的是个地址，那么就看这个地址的变化会不会有影响，而不是看地址指向的对象的变化。就像钥匙和房子的关系。
* 所以说，Java中其实还是值传递的，只不过对于对象参数，值的内容是对象的引用。

无论是值传递还是引用传递，其实都是一种求值策略(Evaluation strategy)。在求值策略中，还有一种叫做按共享传递(call by sharing)。其实Java中的参数传递严格意义上说应该是按共享传递。
按共享传递，是指在调用函数时，传递给函数的是实参的地址的拷贝（如果实参在栈中，则直接拷贝该值）。在函数内部对参数进行操作时，需要先拷贝的地址寻找到具体的值，再进行操作。如果该值在栈中，那么因为是直接拷贝的值，所以函数内部对参数进行操作不会对外部变量产生影响。如果原来拷贝的是原值在堆中的地址，那么需要先根据该地址找到堆中对应的位置，再进行操作。因为传递的是地址的拷贝所以函数内对值的操作对外部变量是可见的。
简单点说，Java中的传递，是值传递，而这个值，实际上是对象的引用。