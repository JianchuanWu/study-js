## function的执行之closure
提到closure就会想到function的**作用域链**，那么我们看一下作用域链是怎么构造的。<br/>
参考[Creating Function Objects](https://es5.github.io/#x13.2)，
无非是创建一个function需要给他设一堆属性，我们只关注`Scope`就好。
    
    "9. Set the [[Scope]] internal property of F to the value of Scope."
    "a Lexical Environment specified by Scope"
就是说Scope就是Lexical Environment，在[第一节](function-LexicalEnvironment.md)
已经说明了LexicalEnvironment就是一个*链*的结构,可以递归地查他里边的变量。

```javascript
function outer(outerArg){
    function middle(midArg){
        function inner(innerArg){
            console.log(outerArg,midArg,innerArg);
        }
        inner('innerArg');
    }
    middle('midArg');
}
outer('outerArg');
```
上面代码执行到`console.log`的时候，outer被调用时产生的Execution Context在栈的
最下边，middle的Execution Context在中间,最上边的是inner的Execution Context；
inner的LexicalEnvironment持有middle的LexicalEnvironment的引用，middle的LexicalEnvironment
持有outer的LexicalEnvironment的引用，因此`outerArg`,`midArg`都能在此时获取。

假如LexicalEnvironment的“生命周期”是与Execution Context一致的，那么下边的代码
```javascript
function outer(outerArg){
    return function middle(midArg){
        return function inner(innerArg){
            console.log(outerArg,midArg,innerArg);
        }
    }
}
outer('outerArg')('midArg')('innerArg');
```
会因为outer和middle的LexicalEnvironment在
`outer2('outerArg')('midArg')`执行完后，随outer和middle的Execution Context一起被干掉，
导致inner的LexicalEnvironment没有外层的LexicalEnvironment，获取不到`outerArg`和`midArg`.

因此问题回到LexicalEnvironment的行为究竟是怎样。
但目前我没有找到LexicalEnvironment是在什么时候销毁的。找了一个[靠谱的二手内容](http://dmitrysoshnikov.com/ecmascript/es5-chapter-3-1-lexical-environments-common-theory/#static-lexical-scope)，
看一下js中的LexicalEnvironment是个什么概念，它解释了静态作用域和动态作用域。<br/>
摘几个关键句子。
    
    "Lexical environments — a mechanism used in some languages to manage the static scoping."
    "Typically, scope is used to manage the visibility and accessibility of variables from different parts of a program."
    "The word “static” relates to ability to determine the scope of an identifier during the parsing stage of a program.
     That is, if we (by looking at the code) can say before the starting the program, in which scope a variable will be 
     resolved — we deal with a static scope."
现在我们可以理解js的LexicalEnvironment,他独立于Execution Context，在
function创建、还没运行的时候，LexicalEnvironment就已经确定了。
    
举例
```javascript
var x=5;
function a() {
    console.log(x);
}
function b(){
    var x=10;
    a();
}
function c(){
    x=10;
    a();
}
b();
c();
```
当a被构建的时候，a的LexicalEnvironment的外层LexicalEnvironment中有x=5.<br/>
b被构建的时候，b的LexicalEnvironment中有x=10.<br/>
当c被构建的时候，什么也没做,因为`x=10`是一个表达式，非变量声明，在[DeclarationBindingInstantiation](http://es5.github.io/#x10.5)过程中表达式不会参与。<br/>
代码表示当前的LexicalEnvironment
```javascript
//这个是全局对象的LexicalEnvironment
globalLE = {
    environmentRecord:{x:5}
};

aLE = {
    environmentRecord:{},
    outerLE : globalLE
};

bLE = {
    environmentRecord:{x:10},
    outerLE : globalLE
};

cLE = {
    environmentRecord:{},
    outerLE : globalLE
};
```
b执行时，调用a，a在LexicalEnvironment找x，在globalLE中获取x值为5.<br/>
c执行时，先执行`x=10`,这个x是在globalLE找到，并重新赋值为10,再log.

我们再解释一下这个典型的闭包的代码
```javascript
function add(x) {
    return function(y){
        console.log(x+y);
    }
}
var add5 = add(5);
add5(2);
```
add首先被构建成一个函数对象。
`add(5)`执行,将执行`return function`,此时赋值给`add5`的这个匿名fuction被构建,<br/>
这个function的LexicalEnvironment指向add的LexicalEnvironment，这个x=5的效果会一直保持在add的LexicalEnvironment，
add5被调用时就会从add的LexicalEnvironment中获取x的值。

总之一句话就是，LexicalEnvironment在函数被调用前就已经确定在那里了，是**静态**的。

[]