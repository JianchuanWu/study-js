## function的执行之[Declaration Binding Instantiation](http://es5.github.io/#x10.5)(声明绑定实例化)
我们只关注function code，并且先不管严格模式(strict)。<br/>
参考原文请直接点链接，这里只对关键点进行说明。<br/>
假定目前的Execution Context中的code是function code,仍然以
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
举例，假设执行到`outer('outerArg')`,创建Execution Context，**Declaration Binding Instantiation开始**，
1. 逐个将函数形参放进Environment Record.就是`outerArg`,假如
Environment Record是个js里的对象`{}`，那么经过这步之后ER.outerArg的值就是`'outerArg'`.
2. 将函数声明([FunctionDeclaration](http://es5.github.io/#x13))放进Environment Record，如果Environment Record
已经有了相同名称的identifier，就覆盖它。简单理解就是如果outer函数中有两个`function middle(){}`,后面的生效。
3. 如果Environment Record中没有"arguments" identifier,则构建arguments，并放到Environment Record中，这个构建过程比较复杂(参考[CreateArgumentsObject](http://es5.github.io/#x10.6)),先不看。
如果Environment Record有"arguments" identifier,那就什么也不做。那么我们可以推测下边代码的执行结果。
    ```javascript
    (function(a,arguments){
        console.log(arguments);//undefined
    })();
    ```
    因为在这之前，第一步就是把形参放进Environment Record中(严谨考虑的话我们还是应该照着第一步的详细步骤执行一下的下面会演示，先略过)，"arguments" identifier已经在Environment Record中了，
    那么第三步就什么都不干。所以打印出的arguments并不是通常的arguments对象。<br/>
    现在我们来看一下第一步是如何将arguments这个形参放进Enviromnent Record里的，
    
        a. Let func be the function whose [[Call]] internal method initiated execution of code. Let names be the value of func’s [[FormalParameters]] internal property.
        b. Let argCount be the number of elements in args.
        c. Let n be the number 0.
        d. For each String argName in names, in list order do
            1. Let n be the current value of n plus 1.
            2. If n is greater than argCount, let v be undefined otherwise let v be the value of the n’th element of args.
            3. Let argAlreadyDeclared be the result of calling env’s HasBinding concrete method passing argName as the argument.
            4. If argAlreadyDeclared is false, call env’s CreateMutableBinding concrete method passing argName as the argument.
            5. Call env’s SetMutableBinding concrete method passing argName, v, and strict as the arguments.
    在d.2那步我们得出arguments赋的值是undefined，并且放进了Environment Record中。
    (如果别人解释这个代码说,“因为你在形参里又声明了一次arguments，所以给arguments对象覆盖了”，我们可以明确告诉他，不对！并且所有的除了用ECMA-262解释之外的解释都不对。:joy:)
4. VariableDeclaration,也就是将变量声明(就是var 变量的那些语句)放到Environment Record里。这里仍然有细节，请看。
 
        a. Let dn be the Identifier in d.
        b. Let varAlreadyDeclared be the result of calling env’s HasBinding concrete method passing dn as the argument.
        c. If varAlreadyDeclared is false, then
            1. Call env’s CreateMutableBinding concrete method passing dn and configurableBindings as the arguments.
            2. Call env’s SetMutableBinding concrete method passing dn, undefined, and strict as the arguments.
    直接看c，就是说如果在Environment Record中没有这个identifier的话，进行如下的操作，那有的话呢，没写，就是什么也不干的意思。<br/>
    上代码：
    ```javascript
    (function(a){
       var a;
       console.log(a);//3
    })(3);
    ```
    因为`var a;`执行前，第一步已经把形参`a`放进Environment Record中了，所以`var a;`什么也没做。其他解释都不对。:joy:
    
下一节会再看闭包究竟是怎么回事。<br/>
并且我还遇到了一个奇葩的现象，看通过以后的学习能否给出正确解释,代码：
```javascript
(function(a,b) {
  function b() {}
  console.log(arguments[1]);//function b()
})(1,2);
```
要解释这个代码估计要看[CreateArgumentsObject](http://es5.github.io/#x10.6)(太长)

[下一节](function-Closure.md)