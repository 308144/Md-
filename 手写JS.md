# 手写JS

## 防抖

触发高频事件后n秒内函数只会执行一次，如果n秒内高频事件再次被触发，则重新计算时间

- 实现方式：每次触发事件时设置一个延迟调用方法，并且取消之前的延时调用方法
- 缺点：如果事件在规定的时间间隔内被不断的触发，则调用方法会被不断的延迟

防抖代码

```ini
function debounce(fn){
	let timeout=null
	return function (){
		clearTimeout(timeout)
			timeout=setTimerout(()=>{
			fn.app(this,arguments)
		})
	}
}

function handle(){
	console.log(Math.random)
}

window.addEventListener('scroll',debounce(handle))

```

## 节流

高频事件触发，但在n秒内只会执行一次，所以节流会稀释函数的执行频率

- 实现方式：每次触发事件时，如果当前有等待执行的延时函数，则直接return

节流代码

```ini
//节流throttle代码：
function throttle(fn) {
    let canRun = true; // 通过闭包保存一个标记
    return function () {
         // 在函数开头判断标记是否为true，不为true则return
        if (!canRun) return;
         // 立即设置为false
        canRun = false;
        // 将外部传入的函数的执行放在setTimeout中
        setTimeout(() => { 
        // 最后在setTimeout执行完毕后再把标记设置为true(关键)表示可以执行下一次循环了。
        // 当定时器没有执行的时候标记永远是false，在开头被return掉
            fn.apply(this, arguments);
            canRun = true;
        }, 500);
    };
}

function sayHi(e) {
    console.log(e.target.innerWidth, e.target.innerHeight);
}
window.addEventListener('resize', throttle(sayHi));
```

## new

```ini
// 使用的例子：
function Person(name, age){
	this.name = name;
	this.age = age;
}

function myNew (constrc,..args){
	// 1,2 创建一个对象obj，将obj的[[prototype]]属性指向构造函数的原型对象
	// 即实现：obj.__proto__ === constructor.prototype
	const obj =Object.create(constrc.prototype)
	// 3.将constrc内部的this（即执行上下文）指向obj，并执行
	const result = constrc.apply(obj, args); 
	// 4. 如果构造函数返回的是对象，则使用构造函数执行的结果。否则，返回新创建的对象
	return result instanceof Object ? result : obj; 
}
const person1 = myNew(Person, 'Tom', 20)
console.log(person1)  // Person {name: "Tom", age: 20}

```

