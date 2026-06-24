# useEffect 란?

말 그대로 쓴다 + 효과다. <p>
특정 파트의 변화가 감지될 경우, useEffect 내부의 내용을 실행하도록 하는 함수(react hook)이다.

의존성 배열의 값이 바뀔 때만 실행된다.
(첫번째 렌더링에는 무조건 실행된다.)

```js

useEffect((매개변수) => {
    
}, [변화를 감지할 변수(=의존성 배열)])

```

사용한 예시:

```js

  useEffect(() => {
    if (open) {
      const r = existingRegion ?? null
      setRegion(r)
      setShape(r?.shape ?? "rectangle")
      setIntensity(r?.intensity ?? 100)
    }
  }, [open, existingRegion])

```
- 만일 의존성 배열을 비워놓을 경우에는 첫 렌더링시에만 발동한다.


### 뭐할때 쓰나?

보통 데이터를 가져올 때 (Data Fetching)서버에서 사용자 목록이나 상품 데이터를 가져와서 화면에 보여줄 때 사용. \
화면이 일단 먼저 그려진 뒤에 데이터를 채워야 사용자가 덜 기다리는 느낌을 받는다.

```js
import { useState, useEffect } from 'react';

function App() {
  const [data, setData] = useState([]); // 데이터를 담을 바구니

  useEffect(() => {
    // 1. 인터넷(API)에서 데이터 가져오기
    fetch('https://typicode.com')
      .then(response => response.json())
      .then(result => setData(result)); // 2. 바구니에 담기
  }, []); // 3. 중요: 처음 켤 때 딱 한 번만 실행해라!

  return (
    <div>
      {/* 4. 화면에 보여주기 */}
      {data.map(item => <p key={item.id}>{item.title}</p>)}
    </div>
  );
}

```