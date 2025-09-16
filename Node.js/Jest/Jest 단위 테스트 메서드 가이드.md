---

tags: [jest, testing, javascript, unit-test]

created: 2025-07-15

updated: 2025-07-15

---

  

# Jest 단위 테스트 메서드 가이드

  

> [!info] 개요

> Jest는 JavaScript 테스팅 프레임워크로, 단위 테스트, 통합 테스트, 스냅샷 테스트 등을 지원합니다.

  

## 📑 목차

  

- [[#⚡ 전역 메서드]]

- [[#🎯 Expect 메서드]]

- [[#🎭 Mock 함수]]

- [[#⏰ 타이머 모킹]]

- [[#📦 모듈 모킹]]

- [[#📸 스냅샷 테스팅]]

- [[#⏳ 비동기 테스팅]]

- [[#🔄 Lifecycle 메서드]]

- [[#💡 유용한 팁]]

  

---

  

## ⚡ 전역 메서드

  

### `describe(name, fn)`

> 테스트 스위트를 그룹화합니다.

  

```javascript

describe('Calculator', () => {

// 테스트들이 여기에 위치

});

```

  

### `test(name, fn, timeout)` / `it(name, fn, timeout)`

> 개별 테스트를 정의합니다. `test`와 `it`은 동일합니다.

  

```javascript

test('1 더하기 2는 3이다', () => {

expect(1 + 2).toBe(3);

});

```

  

### `test.only(name, fn, timeout)`

> 해당 테스트만 실행합니다.

  

```javascript

test.only('이 테스트만 실행됨', () => {

expect(true).toBe(true);

});

```

  

### `test.skip(name, fn, timeout)`

> 해당 테스트를 건너뜁니다.

  

```javascript

test.skip('이 테스트는 건너뜀', () => {

expect(true).toBe(false);

});

```

  

### `test.todo(name)`

> 구현할 테스트를 표시합니다.

  

```javascript

test.todo('나중에 구현할 테스트');

```

  

### `test.each(table)(name, fn, timeout)`

> 여러 데이터로 동일한 테스트를 실행합니다.

  

```javascript

test.each([

[1, 1, 2],

[1, 2, 3],

[2, 1, 3],

])('%i + %i equals %i', (a, b, expected) => {

expect(a + b).toBe(expected);

});

```

  

---

  

## 🎯 Expect 메서드

  

### 🔹 기본 매처

  

#### `toBe(value)`

> [!note] 엄격한 동등성 (===)을 확인합니다.

  

```javascript

expect(2 + 2).toBe(4);

expect('hello').toBe('hello');

```

  

#### `toEqual(value)`

> [!note] 객체나 배열의 깊은 동등성을 확인합니다.

  

```javascript

expect({name: 'kim'}).toEqual({name: 'kim'});

expect([1, 2, 3]).toEqual([1, 2, 3]);

```

  

#### `toBeNull()`

> [!note] 값이 null인지 확인합니다.

  

```javascript

expect(null).toBeNull();

```

  

#### `toBeUndefined()`

> [!note] 값이 undefined인지 확인합니다.

  

```javascript

expect(undefined).toBeUndefined();

```

  

#### `toBeDefined()`

> [!note] 값이 정의되어 있는지 확인합니다.

  

```javascript

expect('hello').toBeDefined();

```

  

#### `toBeTruthy()`

> [!note] 값이 truthy인지 확인합니다.

  

```javascript

expect('hello').toBeTruthy();

expect(1).toBeTruthy();

```

  

#### `toBeFalsy()`

> [!note] 값이 falsy인지 확인합니다.

  

```javascript

expect(0).toBeFalsy();

expect('').toBeFalsy();

```

  

### 🔢 숫자 매처

  

#### `toBeGreaterThan(number)`

> [!note] 값이 특정 숫자보다 큰지 확인합니다.

  

```javascript

expect(10).toBeGreaterThan(5);

```

  

#### `toBeGreaterThanOrEqual(number)`

> [!note] 값이 특정 숫자 이상인지 확인합니다.

  

```javascript

expect(10).toBeGreaterThanOrEqual(10);

```

  

#### `toBeLessThan(number)`

> [!note] 값이 특정 숫자보다 작은지 확인합니다.

  

```javascript

expect(5).toBeLessThan(10);

```

  

#### `toBeLessThanOrEqual(number)`

> [!note] 값이 특정 숫자 이하인지 확인합니다.

  

```javascript

expect(10).toBeLessThanOrEqual(10);

```

  

#### `toBeCloseTo(number, numDigits?)`

> [!note] 부동소수점 숫자를 비교합니다.

  

```javascript

expect(0.1 + 0.2).toBeCloseTo(0.3);

```

  

### 📝 문자열 매처

  

#### `toMatch(regexpOrString)`

> [!note] 정규표현식이나 문자열과 매치되는지 확인합니다.

  

```javascript

expect('Hello World').toMatch('World');

expect('Hello World').toMatch(/world/i);

```

  

### 📚 배열과 이터러블 매처

  

#### `toContain(item)`

> [!note] 배열이나 문자열에 특정 항목이 포함되어 있는지 확인합니다.

  

```javascript

expect(['apple', 'banana']).toContain('apple');

expect('Hello World').toContain('World');

```

  

#### `toContainEqual(item)`

> [!note] 배열에 특정 객체가 포함되어 있는지 확인합니다.

  

```javascript

expect([{a: 1}, {b: 2}]).toContainEqual({a: 1});

```

  

#### `toHaveLength(number)`

> [!note] 객체의 length 속성을 확인합니다.

  

```javascript

expect('hello').toHaveLength(5);

expect([1, 2, 3]).toHaveLength(3);

```

  

### ⚠️ 예외 매처

  

#### `toThrow(error?)`

> [!note] 함수가 예외를 던지는지 확인합니다.

  

```javascript

expect(() => {

throw new Error('에러 발생!');

}).toThrow();

  

expect(() => {

throw new Error('에러 발생!');

}).toThrow('에러 발생!');

```

  

### 🏗️ 객체 매처

  

#### `toHaveProperty(keyPath, value?)`

> [!note] 객체가 특정 속성을 가지고 있는지 확인합니다.

  

```javascript

expect({name: 'kim', age: 30}).toHaveProperty('name');

expect({name: 'kim', age: 30}).toHaveProperty('name', 'kim');

```

  

#### `toMatchObject(object)`

> [!note] 객체가 특정 속성들을 포함하는지 확인합니다.

  

```javascript

expect({name: 'kim', age: 30}).toMatchObject({name: 'kim'});

```

  

#### `toBeInstanceOf(Class)`

> [!note] 객체가 특정 클래스의 인스턴스인지 확인합니다.

  

```javascript

class User {}

expect(new User()).toBeInstanceOf(User);

```

  

### 🔀 기타 매처

  

#### `.not`

> [!note] 매처를 부정합니다.

  

```javascript

expect(2 + 2).not.toBe(5);

```

  

#### `.resolves`

> [!note] Promise가 resolve되는 값을 확인합니다.

  

```javascript

await expect(Promise.resolve('success')).resolves.toBe('success');

```

  

#### `.rejects`

> [!note] Promise가 reject되는지 확인합니다.

  

```javascript

await expect(Promise.reject(new Error('fail'))).rejects.toThrow('fail');

```

  

---

  

## 🎭 Mock 함수

  

### `jest.fn(implementation?)`

> 모의 함수를 생성합니다.

  

```javascript

const mockFn = jest.fn();

mockFn('arg1', 'arg2');

  

expect(mockFn).toHaveBeenCalled();

expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');

```

  

### `mockFn.mockName(name)`

> 모의 함수에 이름을 설정합니다.

  

```javascript

const mockFn = jest.fn().mockName('myMockFunction');

```

  

### `mockFn.mockReturnValue(value)`

> 모의 함수의 반환값을 설정합니다.

  

```javascript

const mockFn = jest.fn();

mockFn.mockReturnValue(42);

expect(mockFn()).toBe(42);

```

  

### `mockFn.mockReturnValueOnce(value)`

> 모의 함수의 일회성 반환값을 설정합니다.

  

```javascript

const mockFn = jest.fn();

mockFn

.mockReturnValueOnce(10)

.mockReturnValueOnce(20)

.mockReturnValue(30);

  

expect(mockFn()).toBe(10);

expect(mockFn()).toBe(20);

expect(mockFn()).toBe(30);

```

  

### `mockFn.mockResolvedValue(value)`

> Promise를 반환하는 모의 함수를 설정합니다.

  

```javascript

const mockFn = jest.fn();

mockFn.mockResolvedValue('result');

await expect(mockFn()).resolves.toBe('result');

```

  

### `mockFn.mockRejectedValue(value)`

> 거부된 Promise를 반환하는 모의 함수를 설정합니다.

  

```javascript

const mockFn = jest.fn();

mockFn.mockRejectedValue(new Error('error'));

await expect(mockFn()).rejects.toThrow('error');

```

  

### `mockFn.mockImplementation(fn)`

> 모의 함수의 구현을 설정합니다.

  

```javascript

const mockFn = jest.fn();

mockFn.mockImplementation((a, b) => a + b);

expect(mockFn(1, 2)).toBe(3);

```

  

### `mockFn.mockClear()`

> 모든 호출 기록을 지웁니다.

  

```javascript

mockFn.mockClear();

```

  

### `mockFn.mockReset()`

> 모든 호출 기록과 구현을 초기화합니다.

  

```javascript

mockFn.mockReset();

```

  

### `mockFn.mockRestore()`

> 원래 구현으로 복원합니다 (spyOn으로 생성된 경우).

  

```javascript

mockFn.mockRestore();

```

  

### 📊 Mock 속성

  

#### `mock.calls`

> [!info] 함수 호출시 전달된 인자들의 배열입니다.

  

```javascript

expect(mockFn.mock.calls.length).toBe(2);

expect(mockFn.mock.calls[0][0]).toBe('first arg');

```

  

#### `mock.results`

> [!info] 함수 호출 결과들의 배열입니다.

  

```javascript

expect(mockFn.mock.results[0].value).toBe('return value');

```

  

#### `mock.instances`

> [!info] new로 호출된 경우 생성된 인스턴스들의 배열입니다.

  

```javascript

const MockClass = jest.fn();

const instance = new MockClass();

expect(MockClass.mock.instances[0]).toBe(instance);

```

  

### 🎯 Mock 매처

  

#### `toHaveBeenCalled()` / `toBeCalled()`

> 함수가 호출되었는지 확인합니다.

  

```javascript

expect(mockFn).toHaveBeenCalled();

```

  

#### `toHaveBeenCalledTimes(number)`

> 함수가 특정 횟수만큼 호출되었는지 확인합니다.

  

```javascript

expect(mockFn).toHaveBeenCalledTimes(3);

```

  

#### `toHaveBeenCalledWith(arg1, arg2, ...)`

> 특정 인자로 호출되었는지 확인합니다.

  

```javascript

expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');

```

  

#### `toHaveBeenLastCalledWith(arg1, arg2, ...)`

> 마지막 호출이 특정 인자로 되었는지 확인합니다.

  

```javascript

expect(mockFn).toHaveBeenLastCalledWith('last', 'call');

```

  

#### `toHaveBeenNthCalledWith(nthCall, arg1, arg2, ...)`

> N번째 호출이 특정 인자로 되었는지 확인합니다.

  

```javascript

expect(mockFn).toHaveBeenNthCalledWith(2, 'second', 'call');

```

  

---

  

## ⏰ 타이머 모킹

  

### `jest.useFakeTimers()`

> 가짜 타이머를 활성화합니다.

  

```javascript

jest.useFakeTimers();

```

  

### `jest.useRealTimers()`

> 실제 타이머로 복원합니다.

  

```javascript

jest.useRealTimers();

```

  

### `jest.runAllTimers()`

> 모든 타이머를 즉시 실행합니다.

  

```javascript

jest.useFakeTimers();

setTimeout(() => console.log('타이머 실행'), 1000);

jest.runAllTimers();

```

  

### `jest.runOnlyPendingTimers()`

> 현재 대기 중인 타이머만 실행합니다.

  

```javascript

jest.runOnlyPendingTimers();

```

  

### `jest.advanceTimersByTime(msToRun)`

> 시간을 특정 밀리초만큼 진행시킵니다.

  

```javascript

jest.advanceTimersByTime(1000);

```

  

### `jest.clearAllTimers()`

> 모든 대기 중인 타이머를 제거합니다.

  

```javascript

jest.clearAllTimers();

```

  

---

  

## 📦 모듈 모킹

  

### `jest.mock(moduleName, factory?, options?)`

> 모듈을 모킹합니다.

  

```javascript

jest.mock('./config', () => ({

apiUrl: 'http://localhost'

}));

```

  

### `jest.unmock(moduleName)`

> 모듈 모킹을 해제합니다.

  

```javascript

jest.unmock('./config');

```

  

### `jest.doMock(moduleName, factory?, options?)`

> 호이스팅 없이 모듈을 모킹합니다.

  

```javascript

jest.doMock('./config', () => ({

apiUrl: 'http://localhost'

}));

```

  

### `jest.dontMock(moduleName)`

> 호이스팅 없이 모듈 모킹을 해제합니다.

  

```javascript

jest.dontMock('./config');

```

  

### `jest.requireActual(moduleName)`

> 실제 모듈을 가져옵니다.

  

```javascript

const actualConfig = jest.requireActual('./config');

```

  

### `jest.requireMock(moduleName)`

> 모킹된 모듈을 가져옵니다.

  

```javascript

const mockedConfig = jest.requireMock('./config');

```

  

### `jest.resetModules()`

> 모듈 레지스트리를 리셋합니다.

  

```javascript

jest.resetModules();

```

  

### `jest.isolateModules(fn)`

> 격리된 모듈 레지스트리에서 코드를 실행합니다.

  

```javascript

jest.isolateModules(() => {

const config = require('./config');

config.value = 'isolated';

});

```

  

---

  

## 📸 스냅샷 테스팅

  

### `toMatchSnapshot(propertyMatchers?, hint?)`

> 스냅샷과 일치하는지 확인합니다.

  

```javascript

expect(component).toMatchSnapshot();

```

  

### `toMatchInlineSnapshot(propertyMatchers?, inlineSnapshot?)`

> 인라인 스냅샷과 일치하는지 확인합니다.

  

```javascript

expect(data).toMatchInlineSnapshot(`

Object {

"name": "John",

"age": 30

}

`);

```

  

### `toThrowErrorMatchingSnapshot(hint?)`

> 에러가 스냅샷과 일치하는지 확인합니다.

  

```javascript

expect(() => {

throw new Error('에러 메시지');

}).toThrowErrorMatchingSnapshot();

```

  

### `toThrowErrorMatchingInlineSnapshot(inlineSnapshot?)`

> 에러가 인라인 스냅샷과 일치하는지 확인합니다.

  

```javascript

expect(() => {

throw new Error('에러 메시지');

}).toThrowErrorMatchingInlineSnapshot(`"에러 메시지"`);

```

  

---

  

## ⏳ 비동기 테스팅

  

### 🔄 콜백 방식

> `done` 콜백을 사용합니다.

  

```javascript

test('비동기 콜백 테스트', done => {

function callback(data) {

try {

expect(data).toBe('data');

done();

} catch (error) {

done(error);

}

}

fetchData(callback);

});

```

  

### 🌟 Promise 방식

> Promise를 반환합니다.

  

```javascript

test('Promise 테스트', () => {

return fetchData().then(data => {

expect(data).toBe('data');

});

});

```

  

### ⚡ Async/Await 방식

> async/await를 사용합니다.

  

```javascript

test('async/await 테스트', async () => {

const data = await fetchData();

expect(data).toBe('data');

});

```

  

### ⏱️ waitFor (Testing Library와 함께 사용)

> 특정 조건을 기다립니다.

  

```javascript

await waitFor(() => {

expect(screen.getByText('로딩 완료')).toBeInTheDocument();

});

```

  

---

  

## 🔄 Lifecycle 메서드

  

### `beforeEach(fn, timeout?)`

> 각 테스트 전에 실행됩니다.

  

```javascript

beforeEach(() => {

initializeDatabase();

});

```

  

### `afterEach(fn, timeout?)`

> 각 테스트 후에 실행됩니다.

  

```javascript

afterEach(() => {

clearDatabase();

});

```

  

### `beforeAll(fn, timeout?)`

> 모든 테스트 전에 한 번 실행됩니다.

  

```javascript

beforeAll(() => {

return initializeCity();

});

```

  

### `afterAll(fn, timeout?)`

> 모든 테스트 후에 한 번 실행됩니다.

  

```javascript

afterAll(() => {

return clearCity();

});

```

  

---

  

## 💡 유용한 팁

  

### 1️⃣ 테스트 구조화

```javascript

describe('UserService', () => {

describe('createUser', () => {

it('should create a new user', () => {

// 테스트 코드

});

it('should throw error for duplicate email', () => {

// 테스트 코드

});

});

});

```

  

### 2️⃣ AAA 패턴 (Arrange, Act, Assert)

```javascript

test('should calculate total price', () => {

// Arrange

const items = [{ price: 10 }, { price: 20 }];

// Act

const total = calculateTotal(items);

// Assert

expect(total).toBe(30);

});

```

  

### 3️⃣ 테스트 데이터 팩토리

```javascript

const createUser = (overrides = {}) => ({

id: 1,

name: 'John Doe',

email: 'john@example.com',

...overrides

});

  

test('user test', () => {

const user = createUser({ name: 'Jane Doe' });

expect(user.name).toBe('Jane Doe');

});

```

  

### 4️⃣ 커스텀 매처

```javascript

expect.extend({

toBeWithinRange(received, floor, ceiling) {

const pass = received >= floor && received <= ceiling;

return {

pass,

message: () => `expected ${received} to be within range ${floor} - ${ceiling}`

};

}

});

  

test('custom matcher', () => {

expect(100).toBeWithinRange(90, 110);

});

```

  

### 5️⃣ 테스트 커버리지

```bash

# package.json에 추가

"scripts": {

"test:coverage": "jest --coverage"

}

```

  

> [!tip] Best Practice

> - 테스트는 독립적이어야 합니다

> - 테스트 이름은 명확하고 구체적이어야 합니다

> - AAA 패턴을 따라 구조화하세요

> - 모킹은 필요한 경우에만 사용하세요

> - 테스트 커버리지는 품질의 지표가 아닙니다

  

---

  

## 📚 관련 링크

- [[JavaScript Testing]]

- [[Unit Testing Best Practices]]

- [[Test Driven Development]]

- [Jest 공식 문서](https://jestjs.io/docs/getting-started)

  

---

  

> [!quote]

> "테스트는 버그를 찾는 것이 아니라, 신뢰를 쌓는 것이다." - Kent C. Dodds