##   Python functools

* partial method

```python
import functools
def add(a):
    print(a + 1)
add(2) #输出3
newAdd = functools.partial(add,3)
newAdd() #输出4
```

* wrap method

```python
import functools
def note(func):
    "note function"
    @functools.wraps(func)
    def wrapper():
        "wrapper function"
        print('note something')
        return func()
    return wrapper
 
@note
def test():
    "test function"
    print('I am test')
 
test()
print(test.__doc__)
```

###  函数式编程（略）

允许把函数本身作为参数传入另一个函数，还允许返回一个函数！

* **map()**

  `map(func,seq)`函数接收两个参数，一个是函数，一个是`Iterable(可迭代对象，序列)`，**`map`将传入的函数func()依次作用到序列seq的每个元素**，并把结果作为新的`Iterator(迭代器)`返回，之后可转为lis或其他类型t输出。

* **reduce()**

   `reduce(func,seq)`把一个函数作用在一个序列`[x1, x2, x3, ...]`上，这个函数必须接收两个参数，`reduce()`把结果继续和序列的下一个元素做累积计算

* **filter()**

  `filter(func,seq)`函数用于**过滤序列，**调用一个布尔函数func()来迭代遍历每个seq中的元素，返回一个使func返回值为True的元素的序列

  和`map()`类似，`filter()`也接收一个函数和一个序列。和`map()`不同的是，`filter()`把传入的函数依次作用于每个元素，然后**根据返回值是`True`还是`False`决定保留还是丢弃该元素。**

  筛选回文   回数是指从左向右读和从右向左读都是一样的数

* **sorted()**

  排序的核心是比较两个元素的大小。如果是数字，我们可以直接比较，但如果是字符串或者两个dict呢？直接比较数学上的大小是没有意义的，因此，比较的过程必须通过函数抽象出来。

  `sorted(seq,key=func)`函数也是一个高阶函数，它可以接收一个`key`函数来实现自定义的排序，例如按绝对值大小排序

* lambda

* 返回函数

  ```python
  def lazy_sum(*args):
   4     def sum():
   5     #闭包   sum可以引用外部函数lazy_sum的参数和局部变量
   6         ax = 0
   7         for n in args:
   8             ax = ax+n
   9         return ax
  10     return sum
  11     
  12 f = lazy_sum(1, 3, 5, 7, 9)  #不需要立即求和，而是在后面的代码中根据需要计算
  13 #返回的函数并没有立刻执行，而是直到调用了f()才执行
  14 print(f())
  ```

* Decorator

  ```python
  def log(func):
      def printInfo(*arge, **kw):   #*不定长参数  **关键字参数
          print('call %s()' % func.__name__) #在调用func之前打印一些信息
          return func(*arge, **kw)
      return printInfo
  
  #log是一个装饰器，接收函数返回函数，借助@语法把装饰器置于函数的定义外
  @log    #相当于执行了  now = log(now)
  def now():
      print('2019-4-20')
  
  #现在同名now()指向了新的函数，调用now()返log()中的printInfo()函数
  print(now())
  ```



##  3.  Python  `concurrent.futures`

* `ThreadPoolExecutor`

  * `map`

    ```python
    URLS = ['http://httpbin.org', 'http://example.com/', 'https://api.github.com/']
    def load_url(url):
        return requests.get(url)
    with ThreadPoolExecutor(max_workers=3) as executor:
        for url, data in zip(URLS, executor.map(load_url, URLS)):
            print('%r page status_code %s' % (url, data.status_code))
    #'http://httpbin.org' page status_code 200
    #'http://example.com/' page status_code 200
    #'https://api.github.com/' page status_code 200
    ```

  * `as_complete`

    ```python
    def load_url(url):
        return url, requests.get(url).status_code
    with ThreadPoolExecutor(max_workers=3) as executor:
        tasks = [executor.submit(load_url, url) for url in URLS]
        for future in as_completed(tasks):
            print future.result() #yield once complete
    ```

  * wait

    ```python
    from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, wait, ALL_COMPLETED, FIRST_COMPLETED
    from concurrent.futures import as_completed
    import requests
    URLS = ['http://httpbin.org', 'http://example.com/', 'https://api.github.com/']
    def load_url(url):
        requests.get(url)
        print url
    with ThreadPoolExecutor(max_workers=3) as executor:
        tasks = [executor.submit(load_url, url) for url in URLS]
        wait(tasks, return_when=ALL_COMPLETED)
        print 'all_cone'
    ```

*   `ProcessPoolExecutor`

  The `__main__` module must be importable by worker subprocesses. This means that [`ProcessPoolExecutor`](https://docs.python.org/3/library/concurrent.futures.html#concurrent.futures.ProcessPoolExecutor) will not work in the interactive interpreter.

  ```python
  def main():
      with ProcessPoolExecutor() as executor:
          tasks = [executor.submit(load_url, url) for url in URLS]
          for f in as_completed(tasks):
              ret = f.done()
              if ret:
                  print f.result().status_code
   
  if __name__ == '__main__':
      main()
  ```

  

