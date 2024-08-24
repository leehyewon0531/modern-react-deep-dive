## 1. ESLint를 활용한 정적 코드 분석
- **정적 코드 분석이란?** 코드 실행과는 별개로 코드 자체로 문제의 소지가 있는 부분을 찾아 사전에 수정하는 것
- 즉, JavaScript 코드 정적 분석, 잠재적인 문제 파악, 수정까지
<br/> <br/>

> 💡 **[ESLint의 원리]**
  1. 자바스크립트 코드를 문자열로 읽는다
  2. 코드를 분석할 수 있는 **파서(parser)로 코드를 구조**화한다
  3. 2번에서 구조화한 트리를 AST라 하고, 이를 기준으로 각종 규칙과 대조한다
  4. 규칙과 대조하여 위반한 코드를 **알리거나(report), 수정한다(fix)**

  (eslint는 [espree](https://github.com/eslint/espree)를 이용해 코드를 구조화하며, 이걸 참고해서 규칙을 만든다) 

<br/> <br/>


- **Debugger 예시**
    - Debugger만 있는 코드를 espree로 분석
    
    ```jsx
    {
    	"type": "Program",
    	"body": [
    		{
    			"type": "DebuggerStatement", // ✎ 분석하면 DebuggerStatement 타입으로 나온다.
    			"range": [0, 8]
    		}
    	],
    	"sourceType": "module",
    	"range": [0, 8]
    }
    ```
    
    - 참고하여 만든 no-debugger 규칙
    
    ```jsx
    module.exports = {
    	meta: {
    		type: 'problem',
    		docs: {
    			description: 'Disallow the use of `debugger`',
    			recommended: true,
    			url: 'https://eslint.org/docs/rules/no-debugger',
    		},
    		fixable: null,
    		schema: [],
    		messages: {
    			unexpected: "Unexpected 'debugger' statement.',
    		}
    	},
    	create(context) {
    		return {
    			DebuggerStatement(node) { // ✎ 해당 타입을 발견하면 
    				context.report({        // ✎ 알린다(report)
    					node,
    					messageId: 'unexpected',
    				})
    			}
    		}
    	}
    }
    ```
    
<br/> <br/>

### `eslint-plugin`과 `eslint-config`

- `eslint-plugin`은 규칙을 모아놓은 패키지 (특정 프레임워크, 도메인 관련)
    
    
    | eslint-plugin-import |  eslint-plugin-react |
    | --- | --- |
    | import과 관련된 다양한 규칙 제공 | 리액트 관련 규칙을 제공 ex. JSX 배열에 키 선언하지 않음  |

- `eslint-config`은 이러한 플러그인들을 묶어서 한 세트로 제공하는 패키지
    
  -  [eslint-config-airbnb](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb)
     -  가장 유명한 eslint-config, 압도적인 다운로드 수
  - [@titicaca/triple-config-kit](https://github.com/titicacadev/triple-config-kit)
    - 한국 커뮤니티에서 운영되는 config 중 가장 유지보수가 활발, 트리플에서 개발하고 있음
    - 대부분의 config는 airbnb 기반인데, 이건 airbnb 기반이 아님에도 사용하는 데 지장이 없음
    - Prettier와 Stylelint도 별도로 제공
  - [eslint-config-next](https://nextjs.org/docs/app/building-your-application/configuring/eslint)
    -   Next.js 11버전부터 제공
    -   단순히 자바스크립트 코드를 정적분석 할 뿐만 아니라
    -   JSX 구문, HTML 코드까지 분석 ⇒ 웹 서비스의 성능 향상에 도움이 됨


<br/> <br/>

### 주의할 점

- **Prettier와의 충돌**
    - Prettier와 ESLint는 엄연히 다르다!
        
        
        | ESLint | Prettier |
        | --- | --- |
        | 에러 발생 가능성을 찾아주는 분석 도구 (Linter) | 줄바꿈, 들여쓰기, 따옴표 등 코드의 포매팅을 도와주는 도구 (Formatter) <br/> 즉, JS뿐만 아니라 HTML, CSS, 마크다운, JSON에도 가능 |

    - ESLint에서도 Prettier가 처리하는 (들여쓰기, 줄바꿈, 따옴표 등)을 처리할 수 있기 때문에 충돌이 발생한다
        1. ESLint에서는 포매팅 규칙을 끈다
        2. JS, TS는 ESLint에 // 나머지는 Prettier에 맡긴다
           (하지만, [해당 블로그](https://velog.io/@yrnana/prettier%EC%99%80-eslint%EB%A5%BC-%EA%B5%AC%EB%B6%84%ED%95%B4%EC%84%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EC%9E%90)에서 ESLint의 포매팅 기능을 이용하면 오류도 많고 느려진다고 한다.)
      

- **규칙에 대한 예외 처리** `eslint-disable-주석`
    - 특정 줄, 여러 줄, 파일 전체 등에서 제외할 수 있다
    - But, 이 규칙들을 끌 때는 정말 옳은 건지 생각해보고 꺼야 한다!
    
    ```jsx
    useEffect(() => {
    	// eslint-disable-line no-exhaustive-deps 
    }, [])
    ```
    
<br/> 

- **ESLint 버전 충돌**
    - 프로젝트에서 사용하는 ESLint의 버전과 config의 버전 사전에 확인 必
    - 따라서 peerDependencies로 명시하도록 권장
  
<br/> <br/>

## 2. 리액트 팀이 권장하는 테스트 라이브러리
- 프론트엔드는 HTML, CSS 뿐만 아니라 사용자의 인터랙션, 의도치 않은 작동 등 브라우저에서 발생할 수 있는 다양한 시나리오를 고려해야 함

<br/> <br/>

### React Testing Library
- 리액트를 기반으로 한 테스트를 수행하기 위해 만들어짐
- DOM Testing Library 기반
    - jsdom - HTML이 없는 자바스크립트만 존재하는 환경 (ex. Node.js)에서 HTML과 DOM을 사용할 수 있도록 해주는 라이브러리
    
    ```jsx
    const jsdom = require('json')
    
    const { JSDOM } = jsdom
    const dom = new JSDOM(`<!DOCTYPE html><p>Hello World</p>`)
    
    console.log(dom.window.document.querySelector('p').textContent) 
    // 'Hello World'
    ```
    
<br/> <br/>
### JavaScript 테스트의 기초

```jsx
// Node.js에서 기본적으로 제공하는 assert
const assert = require('assert')

function sum(a, b){
	return a + b
}

assert.equal(sum(1, 2), 3)
assert.equal(sum(1, 2), 4) 
// AssertionError [ERR_ASSERTION] [ERR_ASSERTION]: 3 == 4
```

- 테스트는 대충 이런 식으로 돌아감
- 무슨 테스트를 어떻게 수행했고 등 테스트 관련 정보들을 일목요연하게 확인하기 위해 ⇒ 테스팅 라이브러리 사용 ex. Jest, Mocha, Karma, Jasmine
- **Jest**
    페이스북에서 만들어서 React와 더불어 많은 자바스크립트 개발자들로 부터 좋은 반응을 얻고 있는 테스팅 라이브러리
    출처: https://blog.imqa.io/testing-framework-jest/
    

<br/> <br/>
### 리액트 컴포넌트 테스트 코드 작성하기

- 컴포넌트를 테스트하는 파일의 이름 규칙 *ex. App.tsx → App.**test**.tsx // 번들링에서 자동 제외*
- 일반적으로 같은 디렉터리 상에 위치

> **정적 컴포넌트 -** 별도의 상태가 존재하지 않아 항상 같은 결과를 반환하는 컴포넌트
> 

```jsx
// beforeEach - 각 테스트(it)을 수행하기 전에 실행하는 함수 
beforeEach(() => {
	render(<StaticComponent />)
})

// describe - 비슷한 속성을 가진 테스트를 하나의 그룹으로 묶는 역할 (중첩 O)
describe('링크 확인', () => {
	// it - test의 축약어 
	it('링크가 3개 존재한다', () => {
		const ul = screen.getByTestId('ul')
		
		expect(ul.children.length).toBe(3)
	})
	
	it('링크가 목록 스타일이 square다.', () => {
		const ul = screen.getBy**TestId**('ul')
		
		expect(ul).toHaveStyle('list-style-type: square;')
	})
})
```

```html
<ul **data-testid="ul"** style={{ listStyleType: 'square' }} > ... </ul>
// testId - 리액트 테스팅 라이브러리의 예약어, get 등의 선택자로 선택하기 어려운 요소
```

> 📌 AAA (Arrange, Act, Assert) 패턴
  ![image](https://github.com/user-attachments/assets/66adb538-cca5-4afc-b2b0-4585044dab4f)

 
<br/>

> **동적 컴포넌트 -** useState를 이용해 상태값을 관리하는 컴포넌트

```jsx
describe('링크 확인', () => {
	// **setup** - 내부에서 컴포넌트 렌더링 후, 테스트에 필요한 요소들을 반환하도록 함수로 묶어둠 
	const **setup** = () => {
		const screen = render(<InputComponent />)
		const input = screen.getByLabelText('input') as HTMLInputElement
		const button = screen.getByText(/제출하기/i) as HTMLButtonElement
		
		return {
			input,
			button,
			...screen,
		}
	}
	
	it('아이디를 입력하면 버튼이 활성화된다.', () => {
		const { button, input } = setup();
		
		// **userEvent.type** - 사용자가 타이핑하는 것을 흉내냄 
		const inputValue = 'helloWorld'
		**userEvent.type**(input, inputValue) 
		
		expect(input.value).toEqual(inputValue)
		expect(button).toBeEnabled()
	})
	
	it('버튼을 클릭하면 alert가 해당 아이디로 표시된다. ', () => {
		// **spyOn** - 메서드를 관찰 
		// **mockImplementation** - 해당 메서드의 mocking을 도움 
		const alertMock = jest
			.**spyOn**(window, 'alert')
			.**mockImplementation**((_: string) => undefined)
			
		const { button, input } = setup()
		const inputValue = 'helloWorld' 
		
		// **fireEvent vs userEvent** 
		// - fireEvent.mouseOver, mouseMove, Up.. click -> userEvent.Click 
		**userEvent**.type(input, inputValue)
		**fireEvent**.click(button)
		
		expect(alertMock).toHaveBeenCalledTimes(1)
		expect(alertMock).toHaveBeenCalledWith(inputValue)
	})
})
```
<br/>

> **동적 컴포넌트** - 비동기 이벤트가 발생하는 컴포넌트 (fetch)

```jsx
// msw
const server = setupServer(
	rest.get('/todos/:id', (req, res, ctx) => {
		const todoId = req.params.id
		
		if (Number(todoId)) {
			return res(ctx.json({ ...MOCK_TODO_RESPONE, id: Number(todoId) }))
		} else {
			return res(ctx.status(404))
		}
	}),
)
```

<br/>

```jsx
beforeAll(() => server.listen())
afterEach(() => server.resetHandlers()) // 덮어쓰기 한 것 초기화 
afterAll(() => server.close())

beforeEach(() => {render (<FetchComponent />)})
```

<br/>

```jsx
describe('FetchComponent 테스트', () => {
	it('버튼을 클릭하면 데이터를 불러온다.', async () => {
		const button = screen.getByRole('button', { name: /1번/ })
		fireEvent.click(button)
		
		const data = await screen.findByText(MOCK_TODO_RESPONSE.title)
		expect(data).toBeInTheDocument()
	})
	
	it('버튼을 클릭하고 에러가 발생하면 에러 문구를 노출한다.', () => {
		server.use(
			rest.get('/todos/:id', (req, res, ctx) => {
				return res(cts.status(503))
			})
		)
	
		const button = screen.getByRole('button', { name: /1번/ })
		fireEvent.click(button)
		
		const error = await screen.findByText(/에러가 발생했습니다/)
		expect(error).toBeInTheDocument()
	})
})
```

<br/> <br/>

> **사용자 정의 훅 테스트하기** - 어떤 props의 변경으로 리렌더링 되었는지 콘솔에 찍어주는 커스텀 훅

```jsx

const consoleSpy = jset.spyOn(console, 'log')
const componentName = 'TestComponent'

describe('useEffectDebugger', () => {
	it('props가 없으면 호출되지 않는다.', () => {
		renderHook(() => useEffectDebugger(componentName))
		
		expect(consoleSpy).not.toHaveBeenCalled()
	})
	
	it('최초에는 호출되지 않는다.', () => {
		const props = { hello : 'world' }
		renderHook(() => useEffectDebugger(componentName))
		
		expect(consoleSpy).not.toHaveBeenCalled()
	})
	
	it('props가 변경되지 않으면 호출되지 않는다.', () => {
		const props = { hello : 'world' }
		const { rerender } = renderHook(() => useEffectDebugger(componentName))
		
		expect(consoleSpy).not.toHaveBeenCalled()
		
		rerender()
		
		expect(consoleSpy).not.toHaveBeenCalled()
	})
	
	it('props가 변경되면 다시 호출된다.', () => {
	
		const props = { hello : 'world' }
		const { rerender } = renderHook(({ componentName, props} ) 
			=> useEffectDebugger(componentName), { 
			initialProps: {
				componentName,
				props,
			}
		})
		
		const newProps = { hello : 'world2' }
		
		rerender({ componentName, props: newProps })
		
		expect(consoleSpy).toHaveBeenCalled()
	})
})
```

- 훅을 렌더링하기 위해서는 **renderHook**를 래핑해서 사용해야 함
    - 내부에 컴포넌트를 생성해 거기에서 훅을 실행한다 → 따라서, 사용자 정의 훅 오류가 발생하지 않는다
    - rerender, unmount라는 함수를 반환한다

<br/> <br/>

### **테스트를 작성하기에 앞서 고려해야 할 점**

- 테스트 커버리지를 맹신하지 말고, 100%까지 끌어올릴 수 있는 상황은 생각보다 드물다.
- 최우선 과제는 **애플리케이션에서 가장 취약하거나 중요한 부분을 파악**하는 것. *ex. 전자상거래 - 결제*
- 테스트 코드는 반드시 사용자의 작업과 최대한 유사하게 작성돼야 한다.
- 테스트가 이뤄야 할 목표는 **애플리케이션이 비즈니스 요구사항을 충족하는지 확인**하는 것

<br/> <br/>

### **그 밖에 해볼 만한 여러 가지 테스트**

- 유닛 테스트 : 각각의 코드나 컴포넌트가 독립적으로 분리된 환경에서 의도된 대로 정확히 작동하는지 검증하는 테스트
    - 컴포넌트가 잘 렌더링된다.
    - 컴포넌트의 특정 함수를 실행하면 상태가 우리가 원하는 형태로 바뀐다
- 통합 테스트 : 유닛 테스트를 통과한 여러 컴포넌트가 묶여서 하나의 기능으로 정상적으로 작동하는지 확인하는 테스트
    - 여러 컴포넌트들을 렌더링하고 서로 상호 작용을 잘 하고 있다
    - DOM 이벤트를 발생 시켰을 때 우리의 UI 에 원하는 변화가 잘 발생한다
- 엔드 투 엔드 테스트 : 실제 사용자처럼 작동하는 로봇을 활용해 애플리케이션의 전체 기능을 확인하는 테스트
    - ex. Cypress

<br/> <br/>
<hr/>

> 기타 참고할 글

- [코드와 함께 살펴보는 프론트엔드 단위 테스트 – Part 2. 실전 편 | 우아한형제들 기술블로그](https://techblog.woowahan.com/17721/)
- [프론트엔드 테스트 자동화 전략 - 3. 구현하기](https://devblog.kakaostyle.com/ko/2023-08-04-3-frontend-testing-3-implement/)

<br/>

> 코멘트

- 테스트 코드 작성을 꼭 해야 할까? -> 작은 규모에서는 오히려 시간이 많이 든다. 큰 회사에서는 필요함
- mockAPI는 종종 사용한다 -  msw, jsonServer
  - API가 늦게 나오는 경우에!    
