# JAVASCRIPT & jQuery 부분



# 1. 식별자

## 1.1 식별자 유형

 메소드

 속성 (cf. 변수)

> ex. math.PI : `PI` 는 속성
>
> EX. PI : `PI` 변수
>
> 

  

- JAVASCRIPT 에서 따옴표로 묵여있는 것을 `문자열` 이라고 한다
  - `''` : 따옴표
  - `"" ` : 쌍따옴표
  - `/``/` : BACKCTICK

```html
console.log(300>200) //true
console.log(300>200>100) //false -> 300>200 => 1로 변환 , 1 > 100 : false
console.log(30>20 && 20> 10) // true
```



![image-20200207131744815](C:\Users\HPE\AppData\Roaming\Typora\typora-user-images\image-20200207131744815.png)



## boolean 함수 - p64

- 다른 자료형을 불(bool) 자료형으로 변환 → 명시적인 형변환

- **0, NaN, '', null, undefined ⇒ false로 변환**나머지 ⇒ true로 변환





## **짧은 조건식(short-circuit)** - p89

> "xxx" && "yyy" : 만약 왼쪽 항이 true 라면, 오른쪽항의 값을 검사한다



- A || B ⇐ A 또는 B, 둘 중 하나가 참이면 참 = A와 B, 둘 모두 거짓이어야 거짓

- A && B ⇐ A와 B, 둘 모두 참이어야 참 = A 또는 B 둘 중 하나가 거짓이면 거짓 



##  `this` keyword예제

```html
<script>
    let score = {
        korean : 90,
        math : 100,
        science : 80,
        
        sum : function () {
            	return this.korean + this.math + this.science;
        },
        average : function () {
            		return this.sum() /3;
        },
    };
    
    console.log(`총점 : ${score.sum()}, 평균 : ${score.average()}`);
    
</script>    
```





## 전개연산자 -예제 - p200

```html
<script>
    function sum(i,j,k,l=10,m=20,n=30) {
        return i+j+k+l+m+n;
    }
    
    let data = [1, 2, 3];
    
    sum(...data); ==> 1+2+3+10+20+30
    sum(...data, ...data); ==> 1+2+3+  1+ 2+ 3
    sum(4,5, ... data);  ==> 4 + 5 + 1 + 2 + 3 + 30
</script>
```



## CSS 의 Selector 예제 - 13장 jQuery의 기본 

```html
<script>
    //두번째 xyz의 글자색을 붉은색으로 만드는 jQuery 코드는?
    $(function() {
       		$('#a2').css('color','red'); 
        	$('#abc #a2').css('color','red'); //a2는 abc의 자손
        	$('p#a2').css('color','red');
        	$('.b:odd').css('color','red');  //인덱스가 홀수인 p
        	$('div#abc p#a2').css('color','red');  //이벤트 핸들링
    });
</script>

<div id="abc">
    <p id="a1" class="b">xyz</p>
    <p id="a2" class="b">xyz</p>
    <p id="a3" class="b">xyz</p>
</div>
```





# React  - createElement 메소드 - p9 이후

- react.createElement 가독성 x -> jsx 구문으로 작성  -> babel이 react.createElement 구문으로 트렌스컴파일-> webpack -> 배포



```html
<script>
   class Hello extends React.Component {
       render() {
           return <h1>Hello React !! </h1>
           
       };
   }
    ReactDOM.render(
        <HELLO/>,
        document.getElementById('xyz');
    );
    
</script>
```





# npm 명령어 - p21

	1. npm start : 개발 모드로 시작
 	2. npm run build : 배포할 수 있는 형식으로 준비
 	3. npm test: 테스트 코드 실행하기
 	4. npm run eject : 설정파일 추출하기



# JSX 구문

```JS
<h1>HELLO React!!</h1>
React.createElement(  //3개의 요소
	'h1',
	{},
    'Hello React!!'

);
```



# 중요! 모든 element는 시작과 끝이 있다!!

> <Car color='red'/>





JSX에서는 반드시 닫는 태그를 사용해야 한다.

```js
<input type="text" /><input type="text"></input><br /><br></br>
```



JSX에서는 반드시 하나의 태그(엘리먼트)로 감싸져 있어야 한다.

[ 잘못된 예 ]

```js
<div> … </div><div> … </div>
```



[ 올바른 예 ]<div>	<div> … </div>	<div> … </div></div>

엘리먼트를 묶어줄 때 다른 태그(<div>)를 사용하는 경우

 ⇒ 2개의 <div> 엘리먼트를 묶어주는 역할의 <div>가 생성

 → 불필요한 DOM 객체가 사용(생성)됨

```js
class App extends React.Component {  render() {    return (      <div>        <div>          abc        </div>        <div>          xyz        </div>      </div>    );  }}
```



- <></> 또는 <Fragment></Fragment>를 사용해서 엘리먼트를 묶을 수 있음 

  → 불필요한 DOM 요소 생성을 방지할 수 있음

  ```js
  class App extends React.Component {
      render() {  
          return (     
              <>          { /* 또는 <Fragment> */ }     
              <div>         
               abc       
              </div>        
              <div>          
               xyz        
              </div>    { /* 또는 </Fragment> */ }     
      </>    );  }}
  ```

  

  



# 부모 컴포넌트 $ `props` - 예제

```js
class Car extends React.Component {
    render() {
        return <div>It's _this.props.color_____ color.</div>;
    }
}

ReactDOM.render(<Car color='red'/>, document.getElementById("root"));

//It's red color.이 출력하게 하려면 ___에 어떻게 코딩해야할까?

//부모 컴포넌트에서 전달된 값을 출력하려면 props 값을 이용한다
```





# `this.props.brand.name ` 예제

```js
class Car extends React.Component {
    render() {
        return <h2>{__this.props.brand.name__} {__this.props.brand.model__}</h2>
    } 
    
}


class Garage extends React.Component {
    render() {
        const carinfo = {name : "SM5", model : "2019"};
        return (
        		<div>
        				<Car brand={carinfo}/>
 				</div>			
				);
		}
}

ReactDOM.render(<Garage />, document.getElementById("root"));

```





# REACT 공부 소스

- W3School site:  https://www.w3schools.com/react/default.asp

  - 공부 방법:

    1. `react ` example 의 code를 `visual studio code` 에서 수동으로 작성

    2. `web` 에서 출력하여 확인

  

- react 한글 블로그 : https://ko.reactjs.org/docs/hello-world.html

  

  - **추천 실습**: 

    - 미리보기 사이트 (preview site)

    - 일정관리 어플 (todo application)  -> `리액트를 다루는 기술 ` 의 최종 프로젝트

      