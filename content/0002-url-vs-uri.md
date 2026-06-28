---
title: "URL과 URI의 차이점은 무엇인가요?"
preview: "URL과 URI는 혼용되는 경우가 많지만 엄밀히는 다른 개념입니다"
tags: [network, web, http]
---

URI는 자원을 식별하는 문자열의 총칭이고, URL은 URI의 하위 개념으로 자원의 위치와 접근 방법까지 포함한 것입니다. 즉, 모든 URL은 URI이지만 모든 URI가 URL은 아닙니다.

```text
URI
└── URL  (위치 + 접근 방법 포함)
└── URN  (이름으로만 식별, 위치 무관)
```

RFC 3986은 URI를 다음 구문으로 정의합니다.

```text
URI = scheme ":" hier-part [ "?" query ] [ "#" fragment ]

https://maeilham.kr/content/0001?page=1#section
└─────┘ └──────────────────────┘ └──────┘ └──────┘
scheme      hier-part            query   fragment
```

`https://maeilham.example/content/0001` 처럼 scheme과 위치 정보가 포함된 형태가 URL이고, `/content/0001` 처럼 경로만 있는 형태는 URL이 아닌 URI reference입니다.

실무에서는 대부분 URL을 다루기 때문에 두 용어를 혼용해도 큰 문제는 없지만, HTTP 명세(RFC 9110)에서 요청 대상을 "request-target"으로 표현하는 것처럼 스펙 수준에서는 URI 개념을 정확히 구분해서 사용합니다.

## 참고

- [RFC 3986 — Uniform Resource Identifier (URI): Generic Syntax](https://www.rfc-editor.org/rfc/rfc3986)
