概述
Proxy 用于修改某些操作的默认行为，可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。
ES6 原生提供 Proxy 构造函数，用来生成 Proxy 实例。

var proxy = new Proxy(target, handler);
Proxy 对象的所有用法，都是上面这种形式，不同的只是 handler 参数的写法。其中，new Proxy()表示生成一个 Proxy 实例，target 参数表示所要拦截的目标对象，handler 参数也是一个对象，用来定制拦截行为。

Proxy 实例的方法
get()
get 方法用于拦截某个属性的读取操作，可以接受三个参数，依次为目标对象、属性名和 proxy 实例本身（严格地说，是操作行为所针对的对象），其中最后一个参数可选。

var person={name:'小明'};
var proxyPerson=new Proxy(person,{
get:function(target,property){
if(property in target){
return target[property]}
else{
throw new ReferenceError("Property \"" + property + "\" does not exist.")
}
}
})
proxy.name // "小明"
proxy.age //Uncaught ReferenceError: Property "age" does not exist.
上面代码表示，如果访问目标对象不存在的属性，会抛出一个错误。如果没有这个拦截函数，访问不存在的属性，只会返回 undefined。

get 方法可以继承。

var child=Object.create(proxyPerson);
child.name //小明
上面代码中，拦截操作定义在 Prototype 对象上面，所以如果读取 obj 对象继承的属性时，拦截会生效。

set()
set 方法用来拦截某个属性的赋值操作，可以接受四个参数，依次为目标对象、属性名、属性值和 Proxy 实例本身，其中最后一个参数可选。

let personq=new Proxy({},{
set:function(obj,prop,value){
if(prop==='age'){if(!Number.isInteger(value)){
throw new TypeError('The age is not an integer')}
if(value>200){
throw new RangeError('The age seems invalid')}
}
obj[prop]=value
}})
上面代码中，由于设置了存值函数 set，任何不符合要求的 age 属性赋值，都会抛出一个错误，这是数据验证的一种实现方法。利用 set 方法，还可以数据绑定，即每当对象发生变化时，会自动更新 DOM。

apply()
apply 方法拦截函数的调用、call 和 apply 操作。
apply 方法可以接受三个参数，分别是目标对象、目标对象的上下文对象（this）和目标对象的参数数组。

var target = function () { return 'I am the target'; };
var handler = {
apply: function () {
return 'I am the proxy';
}
};
var p = new Proxy(target, handler);
p()
// "I am the proxy"
has()
has 方法用来拦截 HasProperty 操作，即判断对象是否具有某个属性时，这个方法会生效。典型的操作就是 in 运算符。
has 方法可以接受两个参数，分别是目标对象、需查询的属性名。
下面的例子使用 has 方法隐藏某些属性，不被 in 运算符发现。

var animal=new Proxy(target,{
has(target,key){
if(key[0]==='\_'){
return false}
return target[key]
}
})
上面代码中，如果原对象的属性名的第一个字符是下划线，proxy.has 就会返回 false，从而不会被 in 运算符发现。
另外，虽然 for...in 循环也用到了 in 运算符，但是 has 拦截对 for...in 循环不生效。

deleteProperty()
deleteProperty 方法用于拦截 delete 操作，如果这个方法抛出错误或者返回 false，当前属性就无法被 delete 命令删除。

var handler = {
deleteProperty (target, key) {
invariant(key, 'delete');
delete target[key];
return true;
}
};
function invariant (key, action) {
if (key[0] === '\_') {
throw new Error(`Invalid attempt to ${action} private "${key}" property`);
}
}

var target = { \_prop: 'foo' };
var proxy = new Proxy(target, handler);
delete proxy.\_prop
// Error: Invalid attempt to delete private "\_prop" property
上面代码中，deleteProperty 方法拦截了 delete 操作符，删除第一个字符为下划线的属性会报错。
注意，目标对象自身的不可配置（configurable）的属性，不能被 deleteProperty 方法删除，否则报错。
其他实例方法就不在一一赘述，可以参考https://es6.ruanyifeng.com/#docs/proxy

使用场景
proxy 模式一般可以使用在当你想要：
拦截或者控制对一个对象的访问时
通过掩盖程序或隐藏逻辑来降低方法/类的复杂度
阻止没有经过验证/准备的重资源操作

1. 抽离校验代码
   function createValidator(target, validator) {
   return new Proxy(target, {
   \_validator: validator,
   set(target, key, value, proxy) {
   if (target.hasOwnProperty(key)) {
   let validator = this.\_validator[key];
   if (!!validator(value)) {
   return Reflect.set(target, key, value, proxy);
   } else {
   throw Error(`Cannot set ${key} to ${value}. Invalid.`);
   }
   } else {
   // prevent setting a property that isn't explicitly defined in the validator
   throw Error(`${key} is not a valid property`)
   }
   }
   });
   }
   // Now, just define validators for each property
   const personValidators = {
   name(val) {
   return typeof val === 'string';
   },
   age(val) {
   return typeof age === 'number' && age > 18;
   }
   }
   class Person {
   constructor(name, age) {
   this.name = name;
   this.age = age;
   return createValidator(this, personValidators);
   }
   }
   const bill = new Person('Bill', 25);
   这样，你就能在不改变你的类/方法的前提下无线扩展你的校验代码了。

2. JavaScript 中真正的私有
   通过 get(),set()实例方法可以保证对象里面私有属性真正实现私有。

var api = {
\_apiKey: '123abc456def',
/_ mock methods that use this.\_apiKey _/
getUsers: function(){ },
getUser: function(userId){ },
setUser: function(userId, config){ }
};
// Add other restricted properties to this array
const RESTRICTED = ['_apiKey'];
api = new Proxy(api, {
get(target, key, proxy) {
if(RESTRICTED.indexOf(key) > -1) {
throw Error(`${key} is restricted. Please see api documentation for further info.`);
}
return Reflect.get(target, key, proxy);
},
set(target, key, value, proxy) {
if(RESTRICTED.indexOf(key) > -1) {
throw Error(`${key} is restricted. Please see api documentation for further info.`);
}
return Reflect.get(target, key, value, proxy);
}
});
// throws an error
console.log(api.\_apiKey);
// throws an error
api.\_apiKey = '987654321'; 3. 静默对象访问日志
对于资源密集、运行缓慢或重度使用的方法和接口，你也许想要记录他们的使用情况或性能。Proxies 可以很容易的在后台默默的做这件事。
每当你调用一个方法，你都会先 get 这个方法。所以如果你想拦截一个方法调用，你需要拦截它的 get 先，然后再拦截 apply。

let api = {
\_apiKey: '123abc456def',
getUsers: function() { /_ ... _/ },
getUser: function(userId) { /_ ... _/ },
setUser: function(userId, config) { /_ ... _/ }
};
api = new Proxy(api, {
get: function(target, key, proxy) {
var value = target[key];
return function(...arguments) {
logMethodAsync(new Date(), key);
return Reflect.apply(value, target, arguments);
};
}
});
// executes apply trap in the background
api.getUsers();
function logMethodAsync(timestamp, method) {
setTimeout(function() {
console.log(`${timestamp} - Logging ${method} request asynchronously.`);
}, 0)
}
警告或阻止特定操作
设你想要组织某人删除 noDelete 属性，想要告诉调用 oldMethod 的用户那已经被弃用了，或者想组织某人修改 doNotChange 属性
let dataStore = {
noDelete: 1235,
oldMethod: function() {/_..._/ },
doNotChange: "tried and true"
};
const NODELETE = ['noDelete'];
const DEPRECATED = ['oldMethod'];
const NOCHANGE = ['doNotChange'];
dataStore = new Proxy(dataStore, {
set(target, key, value, proxy) {
if (NOCHANGE.includes(key)) {
throw Error(`Error! ${key} is immutable.`);
}
return Reflect.set(target, key, value, proxy);
},
deleteProperty(target, key) {
if (NODELETE.includes(key)) {
throw Error(`Error! ${key} cannot be deleted.`);
}
return Reflect.deleteProperty(target, key);

},
get(target, key, proxy) {
if (DEPRECATED.includes(key)) {
console.warn(`Warning! ${key} is deprecated.`);
}
var val = target[key];

    return typeof val === 'function' ?
      function(...args) {
        Reflect.apply(target[key], target, args);
      } :
      val;

}
});
// these will throw errors or log warnings, respectively
dataStore.doNotChange = "foo";
delete dataStore.noDelete;
dataStore.oldMethod();
组织不必要的重资源操作
假设你有个会返回一个很大的文件的服务端，你不希望在之前的请求正在进行时，或者文件正在被下载时，或者已经下载完毕的情况下进行请求。
即时取消对敏感数据的访问
Proxy 提供任何时候取消对目标对象访问功能。如果你想要完全封锁（为了安全性、权限、或性能原因）一些数据和 API 的访问这会很有用。
