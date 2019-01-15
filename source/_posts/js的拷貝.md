---
title: js的拷貝
date: 2019-01-15 10:21:18
tags: 'JavaScript'
categories: 'JavaScript'
---

今天要來講的是 js 的**拷貝**，我們都知道程式語言有傳值(call by value)與傳址(call by reference)，而 js 是一個**比較特別**的語言，在不用做任何設定的情況下，在不同形態會有不同的傳遞方式：
- 使用基本型別時(Number、String、Boolean)，會表現出 call by value 的特色，並不會修改到原始的值
```javascript=
var a = 1
var b = a
b++
console.log(a) // 1
console.log(b) // 2
```
- 使用複合型別時(Object、Array)，會表現出 call by reference 的特色，兩個都是指向同一個參考，當 b 被改變時，因爲是修改指向到的位置上面的值，所以一樣指向相同位置的 a 的值也會改變
```javascript=
var a = [0,1,2]
var b = a
b[0] = 10
console.log(a) // [10,1,2]
console.log(b) // [10,1,2]
```

而探討 js 的這個行爲有許多面向，這並不是我們現在要探討的，我們現在要假定的情境是：當我在使用複合型別時，怎樣可以在不動到原始資料的情況下**拷貝**呢？
### 陣列
在一維陣列的情況下，情況會比較簡單，可以使用以下幾個方式：
1. 使用 Array.from
```javascript=
var a = [0,1,2]
var b = Array.from(a)
b[0] = 10
console.log(a) // [0,1,2]
console.log(b) // [10,1,2]
```
2. 使用 ES6 展開運算元
```javascript=
var a = [0,1,2]
var b = [...a]
b[0] = 10
console.log(a) // [0,1,2]
console.log(b) // [10,1,2]
```
3. 使用 Array.map
```javascript=
var a = [0,1,2]
var b = a.map(item => item)
b[0] = 10
console.log(a) // [0,1,2]
console.log(b) // [10,1,2]
```
4. 原生的迴圈
```javascript=
var a = [0,1,2]
var b = []
for(var i =0;i<a.length;i++){
	b.push(a[i])
}
b[0] = 10
console.log(a) // [0,1,2]
console.log(b) // [10,1,2]
```

說到這邊就知道，其實只要能產生一個**新的** array，不要指向同一個參考，做一個新的參考，就可以達到拷貝的效果

但這種情況只能用在一維陣列下，如果是多維陣列的情況下，會只拷貝一層，第二層還是以 call by reference 會去改動到值，所以我們稱這樣爲**淺拷貝**：
```javascript=
var a = [0,1,2,[3,4,5]]
var b = [...a]
b[0] = 10 // 第一層不會被修改到
b[3][0] = 10 // 第二層還是被修改了
console.log(a) // [0,1,2,[10,4,5]]
console.log(b) // [10,1,2,[10,4,5]]
```

在處理資料的時候，一定會處理到多維陣列的時候，這時候我們必須有一個可以把**每一層**都拷貝的 function：
```javascript=
function deepCopy(array){
 let newArray = []
 array.forEach((item,index)=>{
   if(Array.isArray(item)){
     newArray[index] = deepCopy(item)
	 }else{
		 newArray.push(item)
	 }
 })
 return newArray
}
```
江湖一點訣，說破不值錢，使用遞迴的方式可以達到不管基層都能做到乾淨拷貝的方式，原理很簡單：
1. 傳入一個 array
2. 先建立一個新的 array
3. 開始遍歷傳進來 array 並判斷
    - 如果遍歷到的值的型別是 array 的話，將這一個值再傳入 function 內遞迴
    - 如果是基本型別的話，就直接 push 值在新的 array 裡面
4. 重複第三步驟的判斷，直到 array 遍歷完

### 物件
物件的深拷貝原理也是一樣的，而因爲在 js 裡面， array 也是物件的一種，所以這一個程式碼可以在 array 與 object 共用
```javascript=
function deepCopy(obj){
	let newObj = Array.isArray(obj)?[]:{};
	for (let item in obj){
		if((typeof obj[item]) === "object"){
    	newObj[item] = deepCopy(obj[item])
		}else{
			newObj[item] = obj[item]
    }
	}
	return newObj
}
```

可以透過以下的測試碼測試，不論怎麼修改新創造的陣列/物件，都不會修改到最原始的值，達到我們期望的**深拷貝**
```javascript=
//測試資料
var array = [0,1,2,[3,4,[5,6,7]]]
var newArray = deepCopy(array)
var obj = {name:"kai",k:{q:1,w:2,e:[0,1,2,3]}}
var newObj = deepCopy(obj)
```

在不使用套件的情況下，除了上面寫一個 function 外，還有一個方式可以達到深拷貝的效果，就是使用 JSON 的 method：
- 用JSON.stringify把物件轉成字串
- 再用JSON.parse把字串轉成新的物件

```javascript=
var a = [0,1,2,[3,4,[5,6,7]]];
var b = JSON.parse(JSON.stringify(a));
```

這種方式也可以達到深拷貝的效果，但相對來說，可讀性比較低一點(使用 JSON.stringify 是要處理 JSON 物件還是要處理深拷貝？)，所以使用上就看個人習慣與團隊溝通就可以啦

參考資料：
[深入探討 JavaScript 中的參數傳遞：call by value 還是 reference？](https://blog.techbridge.cc/2018/06/23/javascript-call-by-value-or-reference/)
