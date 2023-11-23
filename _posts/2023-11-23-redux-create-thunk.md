---
layout: post
title: "Redux: createAsyncThunk?"
categories: [Dev, Frontend]
tags: [react, redux]

---

# Redux?

JS 진영의 대표적인 상태 관리 라이브러리 중 하나, Flux 패턴을 활용한 또 하나의 variant 정도로 보여지며 Flux에서 이야기하는 component들을 잘 추상화하여 제공해준다. 완성도가 꽤나 높은 라이브러리.

# createAsyncThunk

![Flux](/assets/img/231123_1_1.png)

Flux pattern에서 State를 변경하는 코드로 나타내면 아래와 같다

`dispatch(Action): State` 흔히 reducer라고 표현하는 그것이다.

여기서, 네트워크 통신 또는 파일 IO 등이 필요한 비동기 처리를 하는과정을 SideEffect 라고 얘기하며 일반적으로 관련 구현을 살펴보면 아래와 같다.

```jsx
const fetchData = () => {
  dispatch(setLoading(true))
  try {
    const data = fetchData()
    dispatch(loaded(data))
  } catch(e) {
    dispatch(failure(e))
  }
  dispatch(setLoading(false))
}
```

fetchData의 parameter관련 로직을 구현하거나 response 결과에 따라 에러 핸들링을 구현한다거나하면 코드의 복잡도는 높아지고 불필요한 중복이 많이 발생하는 문제가 있다.

관련해서 실제 프로젝트 작업시에는 관련 공통 구현을 묶어 delegate를 구현하는식으로 작업을 많이하는데 `Redux`에서는 관련 구현이 포함되어있다.

`createAsyncThunk`는 이런 비동기 요청과 관련된 높은 추상화를 지원해주며 reducer 또는 view logic을 작성할 때 관련 의존성을 줄여준다.

### 적용해보자

위 예시를 그대로 적용해보자면

```jsx

const fetchData = createAsyncThunk(
  URL,
  async () => {
    const resp = await apiGetData()
    return resp.data
  }
}

...

const mySlice = createSlice({
  name: 'users',
  initialState,
  reducers: { ... },
  extraReducers: (builder) => {
    builder.addCase(fetchData.fulfilled, (state, action) => {
      state.data = action.payload
      state.loading = false
    }),
    builder.addCase(fetchData.pending, (state, action) => {
      state.loading = true
    }),
    builder.addCase(fetchData.rejceted, (state, action) => {
      state.error = {...}
      state.loading = false
    }),
  },
})
```

위 처럼 Slice를 구현하고 thunk를 dispatch하여 사용하면된다. Good!

```jsx
dispatch(fetchData)
```

# 결론
최근 주로 `Next.js`를 활용하여 작업을 많이하는데 웹 진영은 standard가 너무 많다. 타 진영을 보면 기본적으로 활용되어지는 core는 비슷하되 아키텍처 수준에서의 variant가 많은데 웹은 core한 부분부터 선택지가 베라 x 10정도 되는듯; 너무 많다. `Next.js + jotai` 기반 프론트엔드 작업을 쭉하다가 `react + redux` 조합의 프로젝트를 진행하려니 난감하다. 어려움을 떠나 다른점이 너무많다. 같은 플랫폼 작업을 하는게 맞나 싶을정도.. 아무튼 하루빨리 진짜 standard가 정해지길..

# Reference

https://redux-toolkit.js.org/api/createAsyncThunk
