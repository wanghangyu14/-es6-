### Symbol

`es5`中有五种原始类型：字符串、数字型、布尔型、`null`和`undefined`，`es6`中加入了第六种原始类型：`Symbol`。

#### 1. 创建Symbol

```javascript
let firstName = Symbol("first name");
let person = {};
person[firstName] = "Jack";
console.log("first name" in person);//false
console.log(person[firstName]);  //"jack"
console.log(firstName);  //"Symbol(first name)"
```

`Symbol`是原始值，用`new Symbol()`会导致程序抛出错误。

`Symbol`函数接收一个可选参数，作为其文本描述，该文本描述存储在内部的`[[Description]]`中，只有调用`Symbol`的`toString()`方法才能读取，`console.log()`隐式的调用了该方法。

识别是否为`Symbol`类型，可以用`typeof`操作符。

***

#### 2. Symbol的使用方法

`Symbol`可用于可计算对象字面量属性名、`Object.defineProperty()`和`Object.defineProperties()`。

```javascript
let firstName = Symbol("first name");

let person = {
    [firstName]: "jack"
}

Object.defineProperty(person, firstName, { writable: false });

let lastName = Symbol("last name");

Object.defineProperties(person, {
    [lastName]: {
        value: "zakas",
        writable: false
    }
})

```

***

#### 3. Symbol共享体系

使用`Symbol.for()`可以创建一个可共享的`Symbol`，`Symbol.for()`先在全局`Symbol`注册表中搜索键为`"uid"`的`Symbol`是否存在，如果存在则直接返回已有的`Symbol`；否则，创建一个新的`Symbol`,并使用这个键在`Symbol`全局注册表中注册，返回新的`Symbol`。

```javascript
let uid = Symbol.for("uid");
let o = {
    [uid]: "12345"
}
console.log(object[uid]);  //"12345"
console.log(uid);   //"Symbol(uid)"

let uid2 = Symbol.for("uid");
console.log(uid === uid2); //true
console.log(object[uid2]); //"12345"
console.log(uid2)  //"Symbol(uid)"
```

使用`Symbol.keyFor()`在`Symbol`全局注册表中检索与`Symbol`有关的键。

```javascript
let uid = Symbol.for("uid");
console.log(Symbol.keyFor(uid))  //"uid"
let uid2 = Symbol.for("uid");
console.log(Symbol.keyFor(uid2)) //"uid"
let uid3 = Symbol("uid");
console.log(Symbol.keyFor(uid3)) //undefined
```

***

#### 4. Symbol与类型强制转换

不能将`Symbol`强制转换为字符串和数字类型：

```javascript
let uid = Symbol.for("uid"),
    desc = String(uid);
console.log(desc);  //"Symbol(uid)"
```

`String()`方法调用了`uid.toString()`。

```javascript
let uid = Symbol.for("uid"),
    desc = uid + "";  //报错
```

`Symbol`和字符串拼接会报错。

```javascript
let uid = Symbol.for("uid"),
    desc = uid/1;
```

`Symbol`也不能参与数值运算。但是逻辑操作符除外，`Symbol`的布尔值为`true`。

***

#### 5. Symbol属性检索

`Object.keys()`和`Object.getOwnPropertyNames()`，前者检索自身可枚举属性，后者检索包括继承和不可枚举属性，但是两者都不能检索`Symbol`属性。

```javascript
let uid = Symbol.for("uid");
let o = {
    [uid]: "12345"
}

let symbols = Object.getOwnPropertySymbols(o);
console.log(symbols.length)； //1
console.log(symbols[0])； //"Symbol(uid)"
console.log(o[symbols[0]])； //"12345"
```

***

#### 6. 通过well-known Symbol暴露内部操作

##### `Symbol.hasInstance()`

```javascript
obj instanceof Array
//等价于
Array[Symbol.hasInstance](obj)
```

`instanceof`是`Symbol.hasInstance()`的简写语句，只有通过`Object.defineProperty()`才能改变`instanceof`的默认行为。

```javascript
function SpecialNumber(){}
Object.defineProperty(SpecialNumber,Symbol.hasInstance,{
    value: function(v){
        return (v instanceof Number)&&(v>=1 && v<=100);
    }
})
var two = new Number(2),
    zero = new Number(0);
console.log(two instanceof SpecialNumber); //true
console.log(zero instanceof SpecialNumber); //false
```

##### `Symbol.isConcatSpreadable`属性

如果`Symbol.isConcatSpreadable`属性为`true`,则表示对象有`length`属性和数字键，它的数值型属性值应该被独立添加到`concat()`调用中。

```javascript
let collection = {
    0: "hello",
    1: "world",
    length: 2,
    [Symbol.isConcatSpreadable]: true
};
let messages = ["hi"].concat(collection);
console.log(messages.length); // 3
console.log(messages); //["hi","hello","world"]
```

##### `Symbol.math`、`Symbol.replace`、`Symbol.search`和`Symbol.split`属性

可以使用开发者自定义的对象来替代正则表达式进行字符串匹配。

```javascript
//等于/^.{10}&/
let hasLengthOf10 = {
    [Symbol.match]: function(v){
        return v.length === 10? [v] : null;
    },
    [Symbol.replace]: function(v,r){
        return v.length === 10? r : v;
    },
    [Symbol.search]: function(v){
        return v.length === 10? 0 : -1;
    },
    [Symbol.split]: function(v){
        return v.length === 10? [, ] : [v]
    },
}
let message1 = "hello world"; //11个字符
let message2 = "hello john"; //10个字符
let match1 = message1.match(hasLengthOf10), //null
    match2 = message2.match(hasLengthOf10); //["hello john"]

let replace1 = message1.replace(hasLengthOf10), //"hello world"
    replace2 = message2.replace(hasLengthOf10); //"hello john"

let search1 = message1.search(hasLengthOf10),  //-1
    search2 = message2.search(hasLengthOf10);  //0

let split1 = message1.split(hasLengthOf10),  //["hello world"]
    split2 = message2.split(hasLengthOf10);  //["",""]
```

***

##### `Symbol.toPrimitive`方法

`Symbol.toPrimitive`方法规定了当对象转换为原始值时应当执行的操作，该方法的参数只有三种选择："number"、"string"和"default",对应分别返回：数字、字符串和无类型偏好的值。

数字模式有以下特性：

1. 调用`valueOf()`方法，如果结果为原始值，则返回。
2. 否则，调用`toString()`方法，如果结果为原始值，则返回。
3. 如果无可选值，则抛出错误。

字符串模式的特点：

1. 调用`toString()`方法，如果结果为原始值，则返回。
2. 否则，调用`valueOf()`方法，如果结果为原始值，则返回。
3. 如果无可选值，则抛出错误。

多数情况下，标准对象默认以数字模式处理，除了`Date`对象默认以字符串模式处理。

如果要覆写默认的转换特性，可以将函数的`Symbol.toPrimitive`属性赋值为一个新的函数：

```javascript
function Temperature(degrees){
    this.degrees = degrees;
}
Temperature.prototype[Symbol.toPrimitive] = function(hint){
    switch(hint){
            case "string":
            	return this.degrees + "\u00b0";
        case "number":
            return this.degrees;
        case "default":
            return this.degrees + "degrees";
    }
}
var freezing = new Temperature(32);
console.log(freezing + "!");  //"32 degrees"
console.log(freezing / 2 );   //16
console.log(String(freezing));  //"32°"
```

##### `Symbol.toStringTag`

定义了`Object.prototype.toString()`返回的身份标识。

```javascript
function Person(name){
    this.name = name;
}
Person.prototype[Symbol.toStringTag] = "Person";
var me = new Person("Jack");
console.log(me.toString());  //"[object Person]"
console.log(Object.prototype.toString.call(me)); //"[object Person]"
```

自定义`toString()`方法不影响`Object.prototype.toString.call()`方法的使用。

```javascript
function Person(name){
    this.name = name;
}
Person.prototype[Symbol.toStringTag] = "Person";
Person.prototype.toString = function(){
    return this.name
}
var me = new Person("Jack");
console.log(me.toString());  //"Jack"
console.log(Object.prototype.toString.call(me)); //"[object Person]"
```

##### `Symbol.unscopables`属性

`Symbol.unscopables`通常用于`Array.prototype`,以在`with`语句中标示出要忽略的标识符。

```javascript
Array.prototype[Symbol.unscopables] = Object.assign(Object.create(null),{
    values: true
})
```

`es6`中数组有`values`方法，设置了`values`为`true`，可以在`with`中忽略默认的`values`方法，转而使用`with`外部的`values`数组：

```javascript
var values = [1,2,3],
    colors = ["red","green","blue"],
    color = "black";
with(colors){
    push(color);
    push(...values);
}
```

