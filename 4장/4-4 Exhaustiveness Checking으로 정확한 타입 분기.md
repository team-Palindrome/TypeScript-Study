# 4-4 Exhaustiveness Checking으로 정확한 타입 분기 유지하기

Exhaustiveness는 사전적으로 철저함, 완전함을 의미한다. 따라서 Exhaustiveness Checking은 모든 케이스에 대해 철저하게 타입을 검사하는 것을 말하며 타입 좁히기에 사용되는 패러다임 중 하나이다.

타입 가드를 사용해서 타입에 대한 분기 처리를 수행하면 필요하다고 생각되는 부분만 분기 처리를 하여 요구 사항에 맞는 코드를 작성할 수 있게 된다. 하지만 때로는 모든 케이스에 대해 분기 처리를 해야만 유지보수 측면에서 안전하다고 생각되는 상황이 생긴다.

이때 Exhaustiveness Checking을 통해 모든 타입 검사를 강제할 수 있다.

## 상품권

배민 선물하기 서비스에는 다양한 상품권이 있다. 상품권 가격에 따라 상품권 이름을 반환해주는 함수를 작성하면 다음과 같다.

```jsx
type ProductPrice = "10000" | "20000";

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === "10000") return "배민상품권 1만 원";
  if (productPrice === "20000") return "배민상품권 2만 원";
  else {
    return "배민상품권";
  }
};
```

여기까지는 각 상품권 가격에 따라 상품권 이름을 올바르게 반환하고 있어 큰 문제가 없다고 느껴질 수 있다. 하지만 새로운 상품권이 생겨서 `ProductPrice` 타입이 업데이트 되어야 한다면

```jsx
type ProductPrice = "10000" | "20000" | "5000";

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === "10000") return "배민상품권 1만 원";
  if (productPrice === "20000") return "배민상품권 2만 원";
  if (productPrice === "5000") return "배민상품권 5천 원"; // 조건 추가 필요
  else {
    return "배민상품권";
  }
};
```

이처럼 `ProductPrice` 타입이 업데이트 되었을 때 `getProductName` 함수도 함께 업데이트 되어야 한다. productPrice가 “5000”일 경우의 조건도 검사하여 의도한 대로 상품권 이름을 반환해야 한다.

그러나 `getProductName` 함수를 수정하지 않아도 별도 에러가 발생하는 것이 아니기 때문에 실수할 여지가 있다.

이와 같이 모든 타입에 대한 타입 검사를 강제하고 싶다면 아래와 같이 코드를 작성하면 된다.

```jsx
type ProductPrice = "10000" | "20000" | "5000";

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === "10000") return "배민상품권 1만 원";
  if (productPrice === "20000") return "배민상품권 2만 원";
  // if (productPrice === "5000") return "배민상품권 5천 원";
  else {
    exhaustiveCheck(productPrice); // Error: Argument of type 'string' is not assignable to parameter of type 'never'
    return "배민상품권";
  }
};

const exhaustiveCheck = (param: never) => {
  throw new Error("type error!");
};
```

앞의 코드에서 productPrice가 “5000”일 때의 분기 처리가 주석 처리 되었고, `exhaustiveCheck(productPrice);` 에서 에러를 뱉고 있는데 `ProductPrice` 타입 중 5000이라는 값에 대한 분기 처리를 철처하게 하지 않아서 발생한 것이다.

이렇게 모든 케이스에 대한 분기 처리를 해주지 않았을 때, 컴파일타임 에러가 발생하는 것을 Exhaustiveness Checking이라고 한다.

좀 더 자세히 살펴보면 `exhaustiveCheck`라는 함수가 보일텐데, 이 함수는 매개변수를 never 타입으로 선언하고 있다. 즉, 매개변수로 그 어떤 값도 받을 수 없으며 만일 값이 들어온다면 에러를 내뱉는다. 이 함수를 타입 처리 조건문의 마지막 else문에 사용하면 앞의 조건문에서 모든 타입에 대한 분기 처리를 강제할 수 있다.

이렇게 Exhaustiveness Checking을 활용하면 예상치 못한 런타임 에러를 방지하거나 요구사항이 변경되었을 때 생길 수 있는 위험성을 줄일 수 있다. 타입에 대한 철저한 분기처리가 필요하다면 Exhaustiveness Checking 패턴을 활용해보자.
