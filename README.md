区块链完全可以说是人类智慧的结晶，它的诞生是人类科技文明发展到一定程度的结果展现。区块链的功能得以实现要有赖于加解密技术的发展，而后者又来源于数论和抽象代数几百年的发展，因此要把握区块链的技术思路，不了解其加解密原理，那你就不可能掌握区块链的技术精髓，所以我们庖丁解牛，一点点的研究和解析相应的算法设计思路，首先需要看的是来自抽象代数的有限群概念。

在抽象代数中，”群“其实是一系列元素的集合，并且在这些元素上定义了两种运算，分别为“+”和“*”， 如果你没有接触过抽象代数，你很容易把这两种运算跟我们熟悉的四则运算中的加法和乘法混淆。一组由有限个元素构成的集合只要能满足以下几个性质就可以形成“群”，
1，任意两个集合中的元素a,b , 如果他们进行“+” 和"\*"运算后，所得结果依然属于该集合，也就是a"+"b, 和a"\*"b也属于该集合，这个性质叫封闭性。
2，存在一个特定元素，我们称之为0，它与任何元素进行”+“运算后，所得结果就是元素本身，也就是 a "+" 0 = a
3， 存在一个特定元素，我们称之为1，它与任何元素进行”\*"运算后，所得结果就是元素本身，也就是 a "\*" 1 = a
4, 如果a是集合中的元素，那么集合中存在另一个元素b, 使得 a "+" b = 0, 我们把满足条件的b用符号-a表示
5，如果a是集合中的元素，那么集合存在另一个元素c,使得a "\*" c = 1, 我们把满足条件的c称为1/a

我们看具体例子，集合F={0, 1, 2, 3, 4}, 如果我们把定义中的两种运算理解为普通的加法和乘法，我们可以看到F无法满足有限群的要求，因为 5 = 1+4 不属于该集合，同时6 = 2 \* 3 也属于该集合。但是我们只要把两种运算稍微做个改变就可以，集合中元素的个数有5个，如果我们做完普通的加法和乘法运算后，将结果再对5进行求余运算，例如 5 % 5 = 0 属于集合F, 6 % 5 = 1属于集合F，于是定义中的两种操作只要先对元素进行普通的加法和乘法运算后再将结果在元素个数的基础上求余即可。

由此我们可以构造特定的有限群F(p) = {0, 1, 2, ...., p - 1}其中p是素数。前面例子中的群就是F(5),接下来我们使用python代码来定义有限群的元素：
```
class FieldElemet:
    def __init__(self, num, prime):
        # prime 对应群的大小，也就是元素的数量, num就是元素对应数值
        if num >= prime or num < 0:
            # 群元素必须是大于等于0的整数
            error = f'field element shound be integer in range 0 to {prime-1}'
            raise ValueError(error)
        self.num = num
        self.prime = prime

    def __repr__(self):
        return f'FieldElement with value:{self.num} and order:{self.prime}'

    def __eq__(self, other):
        if other is None:
            return False
        return self.num == other.num and self.prime == other.prime 
       

a = FieldElemet(7, 23)
b = FieldElemet(6, 23)
print(a == b)
print(a == a)
```
上面代码运行后会输出False 和 True。接下来我们要实现两种有限群操作，也就是加法和乘法，他们实际上就是普通四则运算后再针对群的元素个数进行求余，因此我们要增加__add__的实现：
```
  def __add__(self, other):
        if self.prime != other.prime:
            raise TypeError('there are two different field')
        num = (self.num + other.num) % self.prime
        #新建生成一个对象
        return self.__class__(num, self.prime)
    def __sub__(self, other):
        if self.prime != other.prime:
            raise TypeError('there are two different fields')
        num = (self. num - other.num) % self.prime
        return self.__class__(num, self.prime)
     
     def __mul__(self, other):
        if self.prime != other.prime:
            raise TypeError('there are two different fields')
        num = (self.num * other.num) % self.prime
        return self.__class__(num, self.prime)
    def __pow__(self, num):
        if num < 0:
            raise ValueError(f"exponent should >=0 but value is {num}")
        num = pow(self.num, num, self.prime)
        return self.__class__(num, self.prime)
```
上面代码同时定义了两种运算，分别是加法和减法，减法其实就是加法的逆运算，相应的运算都可以在普通的四则运算上针对群元素个数进行求余操作，我们看看上面代码运行的结果：
```
a = FieldElemet(7, 23)
b = FieldElemet(9, 23)
print(a - b)
print(a * b)

c = FieldElemet(3, 13)
d = FieldElemet(1, 13)
print(c ** 3 == d)
```
代码运行结果为：
FieldElement with value:21 and order:23
FieldElement with value:17 and order:23
True
我们检验一下， 30 = 21 + 9， 7 = 30 % 23，由此减法实现是正确的，同时 63 = 7 \* 9，针对23求余后正好是17，对于指数操作而言，我们同样是在普通的指数运算后再将结果求余，因此3^3的结果是27，然后对13求余所得结果是1，因此它与元素1在求余的基础上是相等的，接下来我们要处理比较烧脑的除法操作，针对有限群的除法需要探究一些数论逻辑，在数论中有关费马小定理为：
```
n^(p-1) % p = 1
```
例如针对群F(13)，我们随机取一个元素例如5，然后验证一下费马小定理：
```
print(pow(4, 12, 13))
```
上面代码运行后结果为1，因此从实践上看定理成立，实际上给定任意整数n,我们把群里面的每个元素都乘以n，然后求余，所得结果还是原来群中的元素，假设我们选定n为5:
```
#F(13)的群元素
n = 5
field_before = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
field_after = []
for i in field_before:
    field_after.append((i * n) % 13)
print("field_after: ", field_after)
```
上面代码运行后输出结果为：
field_after:  [0, 5, 10, 2, 7, 12, 4, 9, 1, 6, 11, 3, 8]
从结果看，输出的数值都属于群F(13)只不过是元素的位置发生了改变而已，对群而言，元素的位置没有任何意义，只要两个群中元素的数量相同，元素能一一对应，两个群就相同，于是就有：
{1, 2, 3, ..., p -1} = {1 \*n, 2 \*n ..., (p-1)\*n}
由此我们把左边元素全部相乘然后对p求余就等于右边元素全部相乘后对p求余：
1\*2\*3...\*(p-1)  % q= n\*(2n)\*(3n)...\*(p-1)n %q
于是等式左边就是
(p-1)! % q, 
右边就是 (p-1)!\*(n^(p-1))
左右两步把(p-1)!消掉就有：
(n^(p-1)) % q = 1

为了利用上面的结论，我们可以把除法操作改为乘法操作 a / b 就等价与 a \* b^(-1), 在群的定义中，任何一个元素a, 都存在另一个元素a\^(-1), 使得两两者进行“\*"操作后结果为1，但根据费马小定理，对任何群里面的元素a，我们都有a^(p-1) % q  = a \* a^(p-2)= 1,于是我们有a^(-1)=a^(p-2)。于是我们计算除法a/b 时，其实就是计算乘法 a \* b^(p-2) % q，于是我们实现群元素除法如下：
```
    def __truediv__(self, other):
        if self.prime != other.prime:
            raise TypeError('there are two different fields')
        num = (self.num * pow(other.num, self.prime - 2, self.prime)) % self.prime
        return self.__class__(num, self.prime)
```
我们验证一下代码逻辑：
```
e = FieldElemet(12, 23)
f = FieldElemet(20, 23)
print( e / f)
```
上面代码输出为：
FieldElement with value:19 and order:23
我们检验一下，20\*19 % 23 = 12，由此可见我们实现的除法没有问题。由于我们可以计算a^(-1)，那么 a\^(-3) = a\^(-3) \* 1 = a\^(-3) \* a\^(p-1) = a\^(p-4),也就是说当我们针对群成员进行指数运算时，指数可以取负数，于是__pow__函数的修改如下：
```
 def __pow__(self, num):
        num = num % self.prime
        num = pow(self.num, num, self.prime)
        return self.__class__(num, self.prime)
```

有限群是区块链加解密技术的基础，区块链使用了所谓的椭圆曲线进行数据加密，而椭圆曲线的加密功能又得依赖于有限群概念，因此掌握它才能掌握后续的区块链加解密算法，
