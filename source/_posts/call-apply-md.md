title: call()和apply()的理解
date: 2018-07-02 14:48:20
tags: 前端
categories: 前端
---

对于call和apply的理解，永远都是遇到问题的时候查找下相关文档，看完理解了，然而一段时间过去后，脑回路又短路了，我这绝对是猪脑子😄...所以这次还是记录一下。
<!-- more -->


```javascript
obj.call(thisObj, arg1, arg2, ...);
obj.apply(thisObj, [arg1, arg2, ...]);
```
call 和 apply 都是为了改变某个函数运行时的 context 即上下文而存在的，换句话说，就是为了改变函数体内部 this 的指向。

其实看了那么多篇文章，觉得知乎上有位大佬写的比较通俗易懂，比较好记一点，嘿嘿。

猫吃鱼，狗吃肉，奥特曼打小怪兽。

有天狗想吃鱼了

猫.吃鱼.call(狗，鱼)

狗就吃到鱼了

猫成精了，想打怪兽

奥特曼.打小怪兽.call(猫，小怪兽)

举个例子方便理解：

```javascript
function Cat() {
    this.type = 'cat',
    this.eatFish = function() {
        console.log(`我是${this.type}，我喜欢吃🐟！！！`)
    }
}

function Dog() {
    this.type = 'dog',
    this.eatMeat = function() {
        console.log(`我是${this.type}，我喜欢吃肉！！！`)
    }
}

var cat = new Cat(),
    dog = new Dog();
cat.eatFish.call(dog) // 我是dog，我喜欢吃🐟！！！
dog.eatMeat.apply(cat) // 我是cat，我喜欢吃肉！！！
```
