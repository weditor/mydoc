# Python yield用法

## 返回一个generator

这是最普通的用法, 使用yield的函数可以用于for循环, 每次返回一个值。  
见[PEP 255 -- Simple Generators](https://www.python.org/dev/peps/pep-0255/)

```python
def myrange():
    m = 0
    while m < 10:
        yield "hello%d" % m
        m += 1


if __name__ == '__main__':
    for item in myrange():
        print(item)
```

上面myrange实际上实现了类似于python的range函数，myrange()用在for循环时，并不会一次性产生包含10个数的列表，
而是每次for循环过来取得时候，就从上一次yield停下来的地方继续执行，直到执行到下一个yield返回点，然后返回那个数。  
yield类似于return，但是return会导致函数结束，但是yield返回一个返回值之后，函数并不会结束，下次执行会接着执行。  
输出如下
```
hello0
hello1
hello2
hello3
hello4
hello5
hello6
hello7
hello8
hello9
```
更加具体的查看，可以使用python内置的next函数手动调用。
```python
def myrange():
    m = 0
    while m < 10:
        yield "hello%d" % m
        m += 1


if __name__ == '__main__':
    mr = myrange()
    print(next(mr))
    print(next(mr))
    print(next(mr))
    print(next(mr))
```
执行输出如下：
```
hello0
hello1
hello2
hello3
```

基于这种generator，可以节省一次性产生所有数据的内存，当需要遍历一个具有10亿数据的列表时，节省的内存就很可观了。  
而且，在某些情况下需要实现一个包含无穷元素的列表，使用generator很容易实现，但是使用列表就无法实现了

## 产生一个协程

协程可以理解为一种另类的、更轻量的线程，可以进行通信，在没有数据的时候一直阻塞，接收到数据的时候就触发执行, 但是协程并不是操作系统级别的概念。  
实现此功能的关键点在于yield除了上面返回一个值之外，实际上还可以接收值.  
官方文档见[PEP 342 -- Coroutines via Enhanced Generators](https://www.python.org/dev/peps/pep-0342/)

示例代码
```python
def consumer():
    while True:
        a = yield
        print('consumer recive [%s]' % a)

if __name__ == '__main__':
    c = consumer()
    next(c)
    print('sending a.')
    c.send('a')
    print('sending b.')
    c.send('b')
    c.close()
```
consumer协程的while循环一直在接收值a。  
程序开始创建了一个consumer协程，注意此时c是一个generator类型,并不是一个function类型, generator具有如下接口
```python
# 使generator执行至下一个yield阻塞点
generator.__next__()
# 向generator发送一个值
generator.send()
# 使generator在当前yield阻塞点抛出一个异常
generator.throw(type[, value[, traceback]])
# 停止一个generator, 不可再进行next和send操作。
generator.close()
```
然后调用c.send会发送一个值过去，此时consumer中接收到之后，触发继续执行，直到下一个yield阻塞。  
输出如下
```
sending a.
consumer recive [a]
sending b.
consumer recive [b]
```

注意：
* generator创建之后必须先调用一次next，因为此时generator并没有开始执行，调用了next之后，generator才开始执行并阻塞在第一个yield语句，此时才准备好接收值。
* generator使用完之后调用close关闭，之后不可调用send，否则会throw一个StopIteration异常

