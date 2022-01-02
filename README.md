# 프런트 성능
## 로딩 최적화
### 1. 브라우저 기준 최적화의 문제점Cancel changes
- Navigation Timing
![](https://images.velog.io/images/jgi0105/post/ef68ae7d-de7b-4c12-8438-4ad783cf74bd/image.png)
- processing && load 이 두 이벤트를 앞 단기고 빨리하는데 목표가 있다.
	- domContentLoadedEvent(processing) - 브라우저가 html을 파싱을 완료하면 발생
    - loadEvent(load) - html이 포함하고 있는 자원들이 모두 로딩 되었을 때 발생
![](https://images.velog.io/images/jgi0105/post/78652c7c-c5fb-45c3-a4f9-6e810407540b/image.png)

#### 1. critical rendering path : js - 자바스크립트 로드 시점 최적화
1. 위치 변경
	- /body맨 끝
    - script 속성 [ async / defer ] 사용 - 브라우저가 block 시키지 않도록 만듭니다.

#### 2. critical rendering path : css
1. TTFB([Time to first byte](https://en.wikipedia.org/wiki/Time_to_first_byte)): 브라우저가 다운로드 요청을 하고 첫 바이트를 받아서 처리하는 데까지 걸리는 시간
	- 외부 스타일을 인라인 스타일을 만들어라! critical css

![](https://images.velog.io/images/jgi0105/post/1e65d201-af88-4e26-80a8-f3e1232212f7/image.png)



> **흰 화면이 나오는 이유는???** 
HTML이 파싱이 되는 중간에 block 요소(CSS, JS)가 들어와서 파싱을 지연시키기 때문에다.
브라우저가 화면을 렌더 하는 순서는 다음과 같다.
**DOM TREE + CSSOM TREE > DOM TREE  > Layout > Paint > Composition**
위 과정을 거쳐 렌더링 되기 때문에 **CSS**는 렌더링 필수 요소기 때문에 block요소가 된다.
**그렇다면? JS는**
이유는, Dom과 Css를 변경할 수 있기 때문에 최악의 상황을 보호하기 위해서! block 리소스가 됩니다.


### ⚜️ 사용자 기준 최적화 - 사용자에게 의미 있는 콘텐츠가 로딩되는 첫 순간.
1. First Meaningful Paint(FMP) : 사용자에게 의미 있는 콘텐츠가 로딩되는 첫 순간.
	- 고전 적인 방법(Run-time에서 HTML생성 )이 아닌 SPA에서의 [Server Side Rendering](https://developers.google.com/web/updates/2019/02/rendering-on-the-web?hl=ko)
    - 프리 렌더러
    
![](https://images.velog.io/images/jgi0105/post/e7112dc5-4308-4859-aa87-be64af1ae0fa/image.png)
> 1. First Paing : 흰 화면에서 처음으로 화면에 무엇이든 그려지는 첫 순간.
2. First Contentful Paint : 이미지 또는 텍스트가 처음으로 표현되는 그 순간.
3. 💥 First Meaningful Paint : 사용자에게 의미있는 컨텐츠가 로딩되는 첫 순간.
4. Time to Interactive : 거의 모든 리소스가 로딩되어서 사용자와 인터렉션 할 수 있는 순간

### 프리 렌더러
1. 빌드 타임에 HTML을 생성한다.
	- webpck 사용시 [prerender-laoder](https://www.npmjs.com/package/prerender-loader) 참고

### PWA
1. [PRPL패턴](https://web.dev/apply-instant-loading-with-prpl/) = Web + App
	- Push Render Pre-cache Lazy-load
2. 빠른 로딩 속도, 사업지표 향상, 사용성개선

## 렌더링 최적화
### 레이아웃 스래싱
1. [강제 동기식 레이아웃](https://developers.google.com/web/tools/chrome-devtools/rendering-tools?hl=ko)이 빈번하게 발생되는 것.
![](https://images.velog.io/images/jgi0105/post/56e76c0a-73bc-4f1a-a0ef-a6c99388352e/image.png)

> **강제 동기 레이아웃**
자바스크립트를 실행한 후, 스타일 계산을 수행한 후에 레이아웃을 실행합니다. 하지만 자바스크립트를 사용하여 브라우저가 레이아웃을 더 일찍 수행하도록 하는 것도 가능합니다. 이를 **강제 동기식 레이아웃**이라고 합니다.

### [VDOM](https://medium.com/geekabyte/lets-better-know-the-famous-vdom-a21faf9e9157)
1. 불필요한 돔 변경을 최소화하는데 의미가 있다!
	- 돔을 변경하게 되면 렌더링 과정의 다시 발생하게 되기 때문에!
    - 부분적인 변경
![](https://images.velog.io/images/jgi0105/post/f432cbfb-127b-4133-b28c-81a8a8c269d7/image.png)
### [웹 워커](https://developer.mozilla.org/ko/docs/Web/API/Web_Workers_API)
1. 무거운 작업을 웹 워커로 위임.
	- https://developer.mozilla.org/ko/docs/Web/API/Web_Workers_API


> 60FPS = 16ms/fr => 10ms/fr
자바스크립트 코드가 10ms에 끝나야 한다.

[참고](https://www.youtube.com/watch?v=G1IWq2blu8c)


# image optimization
![이미지 사이즈 변환 사이트](https://squoosh.app/editor)
```jsx
const imageRef = useRef(null);
useEffect(()=>{
	const callback = (entries,observer) => {
		entries.forEach(entry => {
			if(entry.isIntersecting){
				entry.target.src = entry.target.dataset.src;
				observer.unobserve(entry.target)
			}
		})
	}
	const options = {}
	const observer = new IntersectionObserver(callback, options)	
	observer.observe(imageRef.current)
},[])
```
