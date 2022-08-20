---
title: "Javascript Prototypes and Classes"
date: 2022-08-20T10:12:04+05:00
draft: false
featured_image: '/images/js_proto_class.png'

---

By using prototypes we can share properties and methods.

for example X class is parent of all. and A,B,C classes can inherit from X so it can share properties and methods from X class.

to inherit any A,B,C from X we use always use `new` keyword along with constructor. just like to create an object from class we have to use new keyword.

```javascript
//step1: define the parent constructor function named `X`
function X(){}

//step2: define common properties and methods that will be used by its child
//also note the syntax. parent.prototype.method or parent.prototype.property
X.prototype.name = "X"
X.prototype.print = function(name="X"){
    console.log( "Root Print prop,arg: ", this.name, name);
}


//step3: Define child constructor function
let A = function(){
    this.age = 10
    console.log('this is A')    
}

//step4: Inherit A from X or
//Notice the syntax where `new X()` is the parent constructor function being called.
A.prototype = new X();

let a = new A();
//now we can access the print method on child although we have not defined `print` method on child
a.print('hello')


let B = function (){
    console.log("this is b");
    this.name = 'B'
}

B.prototype = new X();

let b = new B();

b.print('hello')




```



classes are syntactic sugar for prototypes.
Even though we use class based syntax but behind the scenes it uses prototypes. you can check any transpiled code by typescript or babel etc.


```javascript
class X{
    constructor(name="hello"){
        this.name = name
    }

    print(){
        console.log("classX printing:  ", this.name)
    }
}


class A extends X{
    constructor(name){
        super(name)
    }

    print(){
        console.log("classA printing: ", this.name)
    }
}

let a = new A("AA");

a.print()
```