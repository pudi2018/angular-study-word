# Rxjs常用操作符
>常用操作符
这些操作符具体流的走向可以参考http://rxmarbles.com/#sequenceEqual
## 一、创作类操作符
##### from
>from可以把数组、Promise、以及Inerable转换为Observable
```typescript
const array$ = Rx.Observable.from([1, 2, 3, 4])
array$.subscribe(val => console.log(val))
// 1
// 2
// 3
// 4
```
##### formEvent
>formEvent可以把事件转化为Observable
```typescript
const height = document.getElementById('height')
const height$ = Rx.Observable.fromEvent(height, 'keyup')
```
##### of
>of接受一系列的数据，并把它们emit出去
```typescript
const obj$ = Rx.Observable.of({id:1, value: 20})
```
## 二、常见转化类操作符
##### map
>map把流里的元素做一个处理返回新的元素的一个流
```typescript
const array$ = Rx.Observable.from([1, 2, 3, 4]).map(val=> val * 2);
array$.subscribe(item => console.log(item))
//2
//4
//6
//8
```
##### mapTo
>mapTo使流的值为某一个值
```typescript
event$.mapTo(1) // 1
```
##### pluck
>pluck从得到流中取出属性里面的值可以是多层级的
```typescript
const height = document.getElementById('height')
const height$ = Rx.Observable.fromEvent(height, 'keyup')
                             .pluck('target', 'value');
//从height$流中取得其target属性的value属性的值
```
## 三、常见创建类操作符
##### Interval
>interval每隔x毫秒发射出一个值
返回一个发出无限自增的序列整数, 你可以选择固定的时间间隔进行发送。 第一次并 没有立马去发送, 而是第一个时间段过后才发出。 默认情况下, 这个操作符使用 async 调度器来 提供时间的概念，但也可以给它传递任意调度器。
```typescript
Rx.Observable.interval(100).subscribe(
    val => console.log(val)
)
 // 0
 // 1
 // 2
 // interval(x)每隔x毫秒发射出一个值，该值为它在流中的位置
```
##### timer
>timer创建一个 Observable，该 Observable 在初始延时（initialDelay）之后开始发送并且在每个时间周期（ period）后发出自增的数字。
timer(3000,1000)第一个参数代表多少ms之后开始发，第二个参数代表没多少ms在自增长的发出
```typescript
// 每隔1秒发出自增的数字，3秒后开始发送。
const numbers = Rx.Observable.timer(3000, 1000);
numbers.subscribe(x => console.log(x));
```
## 四、过滤类操作符
##### filter
>filter类似js的原生filter
```typescript
Rx.Observable.interval(100)
.filter(val => val % 2 === 0)
```
##### take
>take取流中的前几个元素，需要个数字参数
```typescript
Rx.Observable.interval(100).take(3).subscribe(
    val => console.log(val)
)
// 取流中的前三个值
```
##### takeUntil
>takeUntil 订阅并开始镜像源 Observable 。它还监视另外一个 Observable，即你 提供的 notifier 。如果 notifier 发出值或 complete 通知，那么输出 Observable 停止镜像源 Observable ，然后完成。
```typescript
const interval = Rx.Observable.interval(1000);
const clicks = Rx.Observable.fromEvent(document, 'click');
const result = interval.takeUntil(clicks);
result.subscribe(x => console.log(x));
//每秒都发出值，直到第一次点击发生
```
##### switchMap
>switchMap将每个源值投射成 Observable，该 Observable 会合并到输出 Observable 中， 并且只发出最新投射的 Observable 中的值。
```typescript
// 每次点击返回一个 interval Observable点击后又是从0开始上次的interval Observable就中断
var clicks = Rx.Observable.fromEvent(document, 'click');
var result = clicks.switchMap((ev) => Rx.Observable.interval(1000));
result.subscribe(x => console.log(x));
```
##### first/last
>取流中的第一个或者最后一个的值
```typescript
Rx.Observable.interval(100)
.filter(val => val % 2 === 0)
.first()
```
##### skip
>skip过滤掉流中的前几个元素
```typescript
Rx.Observable.interval(100).skip(2);
// 过滤掉流中的前两个元素
```
##### debounce
>debounce有一定条件，满足这个条件的就保留没有的就过滤掉接收的是一个Observable
```typescript
const height = document.getElementById('height')
const height$ = Rx.Observable.fromEvent(height, 'keyup')
                             .pluck('target', 'value')
                             .debounce(() => Rx.Observable.interval(300))
height$.subscribe(val => console.log(val))
```
##### debounceTime
>debounceTime有一定时间间隔的，大于这个时间间隔的就保留没有的就过滤掉，只能是个时间
```typescript
const height = document.getElementById('height')
const height$ = Rx.Observable.fromEvent(height, 'keyup').debounceTime(300)
height$.subscribe(val => console.log(val.target.value))
// 只有输入间隔时间大于300ms的才会被打印出来
```
##### distinct
>distinct只保留流中不一样的元素，重复的抛弃掉，使用需要十分慎重，对于无尽序列会十分消耗内存
```typescript
const array$ = Rx.Observable.from([1, 2,2, 3, 4,4,6,2,1]).distinct();
array$.subscribe(item => console.log(item));
// 1 2 3 4 6
```
##### distinctUntilChanged
>distinctUntilChanged保留跟前一个元素不一样的元素，如果有间隔是不过滤掉的
```typescript
const array$ = Rx.Observable.from([1, 2,2, 3, 4,4,6,2,1]).distinctUntilChanged();
array$.subscribe(item => console.log(item));
// 1 2 3 4 6 2 1
```
## 五、合并类操作符
##### merge
>merge两个流按自己的方式合并哪个有更新了输出那个
```typescript
const length = document.getElementById('length');
const width = document.getElementById('width');
const length$ = Rx.Observable.fromEvent(length, 'keyup')
                             .pluck('target', 'value');
const width$ = Rx.Observable.fromEvent(width, 'keyup')
                            .pluck('target', 'value');
const merged$ = Rx.Observable.merge(length$, width$);
merged$.subscribe(val => console.log(val))
```
##### concat
>concat只有运行完前面的流，才会运行后面的流，就算后面的流比较快也要先运行前面的流
```typescript
const length = document.getElementById('length');
const width = document.getElementById('width');
const length$ = Rx.Observable.fromEvent(length, 'keyup')
                             .pluck('target', 'value');
const width$ = Rx.Observable.fromEvent(width, 'keyup')
                            .pluck('target', 'value');
const first$ = Rx.Observable.form([1,2,3,4]);
//怎么在第二个输入框的输入流都不会显示
// const concat$ = Rx.Observable.concat(length$, width$);
const concat$ = Rx.Observable.concat(first$, width$);// 再次输入在第二个输入框输入内容就会显示了
concat$.subscribe(val => console.log(val))
```
##### startWitch
>startWitch给流设置个默认值初始值
##### combineLatest
>combineLatest只要任何一个流里有新元素产生就使用一个流里最新的值及另外一个流的值，作为最后函数的参数。
```typescript
const length = document.getElementById('length');
const width = document.getElementById('width');
const length$ = Rx.Observable.fromEvent(length, 'keyup')
                             .pluck('target', 'value');
const width$ = Rx.Observable.fromEvent(width, 'keyup')
                            .pluck('target', 'value');
const combineLatest$ = Rx.Observable.combineLatest(length$, width$, (l, w) => l * w);
combineLatest$.subscribe(val => console.log(val))
// 每个输入框的值变化都会返回l*w的值
```
##### withLatesFrom
>withLatesFrom是以一个流为主流另外一个流更新了值时去取另外流的值然后产生新值
```typescript
const withLatestForm$ = length$.withLatestFrom(width$)
withLatestForm$.subscribe(val => console.log(val))
// 当length$ 改变时，length$ 的新值会和width$ 此时的最新值反应
```
##### zip
>zip表示两个流的时候每个流都有值更新才会使用两个流的最新值，作为最后函数的参数，必须成对出现，一定是一一相对，最慢的流决定了zip最终的速度
```typescript
const combineLatest$ = Rx.Observable.zip(length$, width$, (l, w) => l * w);
combineLatest$.subscribe(val => console.log(val))
// 两个输入框都输入新的值的时候才会返回l*w的值
```
区别：zip必须两个流都改变才会发生改变，withLatestFrom只有当主流的值发生改变才会改变，combineLatest随便哪个流改变就会改变
## 六、常见工具类操作符
##### do
>do可以当做流与外部交流的桥梁，因为使用subscribe之后就不能对流进行其他的操作
```typescript
Rx.Observable.interval(100)
.map(val => val * 2)
.do(v => console.log('value is' + v).take(3)
```
## 七、 常见变换类操作符
##### scan
>scan对源 Observable 使用累加器函数， 返回生成的中间值， 可选的初始值。
跟递归有点像的接收一个函数(x,y)会吧上次的结果作为x新的流的值作为y进行计算
```typescript
Rx.Observable.interval(100).scan((x, y) => {
    return x + y
}).take(4);
// x为累加器， y为流传入的值
// 0 1 2 6
```
##### reduce
>reduce累计方式和scan一样只是值只输出最后一个计算的接口
```typescript
Rx.Observable.interval(100).take(4).reduce((x, y) => {
    return x + y
});
// 6
// 还可以这样用
Rx.Observable.interval(100).take(4).reduce((x, y) => {
    return [...x, y]
}, []);
// [0, 1, 2, 3]
```
##### Observable的性质
>1、有三种状态：next、error、complete，是subscribe的三个参数
next每个值发送，error出现错误，complete流结束了

```typescript
Rx.Observable.interval(100).take(3).subscribe(
    val => console.log(val),
    err => console.log(err),

    () => console.log('I am complete')
)
```
2、几种特殊的Observable: 永不结束（比如计时器）、Never、Empty、Throw
### 备注

```typescript
const height = document.getElementById('height')
// 有这个的html文件里都要加入下面这段
<input id="length" type="text">
<input id="height" type="text">
<script src="https://unpkg.com/@reactivex/rxjs@5.0.3/dist/global/Rx.min.js"></script>
```
>angular6版本时要注意

```typescript
// 在angular里要用这些的话要写成这样
import { combineLatest, from } from 'rxjs';

const array$ = form([1,2,3,4]);
// map,take,filter....要放在pipe里面
array$.pipe(
    map(val => val * 2),
    take(2)
)
```