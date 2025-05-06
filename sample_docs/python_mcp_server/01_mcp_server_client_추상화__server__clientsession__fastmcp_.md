# Chapter 1: MCP Server/Client 추상화 (Server, ClientSession, FastMCP)

---

## 동기: 왜 서버와 클라이언트 추상화가 중요할까요?

여러분이 챗봇, AI 모델, 도구 자동화 등의 시스템을 만든다고 상상해 보세요. 외부에서 내 프로그램의 기능을 사용할 수 있도록 만들고 싶으면, "서버"와 "클라이언트"라는 개념이 꼭 필요합니다.

- **서버(Server)**: 기능(도구, 리소스, 프롬프트 등)을 외부에 공개(노출)합니다. 즉, 제공자 역할을 합니다.
- **클라이언트(ClientSession)**: 서버에 연결해서 기능을 요청하고 결과를 받는 소비자입니다.
- **FastMCP**: 서버를 더 쉽게 만들 수 있게 도와주는 고수준(상위레벨) 래퍼입니다.

MCP 프로젝트에서는 각각의 역할을 잘 구분하고, 쉽게 다룰 수 있도록 추상화해서 제공합니다. 즉, 복잡함을 감추고 심플한 인터페이스로 바꿔줍니다.

---

## 핵심 아이디어

1. **Server와 ClientSession은 MCP의 핵심**  
   개발자는 Server/ClientSession 클래스 덕분에 복잡한 네트워크 코드를 작성하지 않고, 쉽게 서버를 만들거나 붙을 수 있습니다.

2. **추상화란?**  
   불필요한 복잡함을 숨기고, 꼭 필요한 '동작'만 보여주는 것.  
   실제로는 많은 코드와 통신이 숨어 있어도, 여러분은 '필요한 것만 보면' 됩니다.

3. **FastMCP로 더 쉽게!**  
   FastMCP는 데코레이터, 편리한 패턴 등 파이썬의 장점을 이용해 서버 코드를 더 쉽게 짤 수 있도록 해줍니다.

---

## 코드: 어디서 무엇을 불러와야 할까요?

### MCP 기본 구성

아래 파일에서, MCP의 주요 컴포넌트(Server, ClientSession, FastMCP 등)를 불러옵니다.

```python
# src/mcp/__init__.py 일부 발췌

from .client.session import ClientSession
from .server.session import ServerSession
from .server.fastmcp import FastMCP
```

### 서버(제공자)와 클라이언트(소비자)

- **서버(Server, FastMCP)**  
  - 내 프로그램이 외부 요청을 받고, 도구·리소스·프롬프트 등 여러 기능을 외부에 공개하려면 이걸 씁니다.

- **클라이언트(ClientSession)**  
  - 이미 준비된 MCP 서버에 붙어서, 제공하는 기능을 사용할 때 씁니다.

### 예시 코드 1) 서버 만드는 가장 간단한 방법

```python
from mcp.server.fastmcp import FastMCP

server = FastMCP(name="나의 서버", instructions="이 서버는 테스트용입니다.")

@server.tool()
def add(a: int, b: int) -> int:
    return a + b

if __name__ == "__main__":
    server.run("stdio")
```

### 예시 코드 2) 클라이언트로 서버에 접속 후 기능 사용

```python
from mcp import ClientSession, stdio_client

async def main():
    async with stdio_client() as (read, write):
        session = ClientSession(read, write)
        await session.initialize()
        result = await session.call_tool("add", {"a": 1, "b": 2})
        print(result)
```

---

## 코드 설명: 어떻게 동작할까요?

### 1. FastMCP로 만든 서버

- `FastMCP` 는 서버의 껍데기를 만듭니다.
- `@server.tool()` 데코레이터를 붙이면, 간단히 Python 함수를 외부 "도구"로 등록할 수 있습니다.
- `server.run("stdio")` 를 호출하면, 서버가 실행되고 외부에서 요청을 받을 준비가 됩니다.

> **비유:**  
> 마치 카페의 주방장(서버)이, 오늘의 메뉴(도구)를 미리 준비해놓고 손님(클라이언트)이 오면 주문을 처리하는 것과 같습니다.

### 2. ClientSession으로 서버와 대화

- `ClientSession` 객체를 만들 때 연결 스트림(read/write stream)을 넘깁니다.
- `initialize()`를 호출하면 서버 연결이 시작됩니다(서로 "준비 완료" 신호 주고받음).
- `call_tool("add", {"a": 1, "b": 2})` 함수로 서버에 있는 add라는 도구를 실제로 호출함.
- 결과를 받아서 활용합니다.

> **비유:**  
> 손님(클라이언트)이 카페 카운터(세션)에서 "커피 한 잔(add)"을 주문하고, 주방(서버)에서 "커피 한 잔"을 내줍니다.

---

## 자주하는 질문

### Q1. FastMCP와 Server의 차이점은?
- `Server`는 MCP 프로토콜의 저수준(낮은 수준) 기능을,  
- `FastMCP`는 더 쉽고 파이썬스럽게(데코레이터 등) 서버를 만들 수 있게 래핑(wrap)해줍니다.

### Q2. 클라이언트도 직접 도구를 등록할 수 있나요?
- 아니요. 클라이언트는 **사용하는 입장**이고,  
  도구/리소스/프롬프트 등은 **서버**에만 등록됩니다.

---

## 마무리: 이 장의 핵심 요약

- MCP의 **서버**와 **클라이언트** 추상화 덕분에, 복잡한 네트워크 코드를 몰라도  
  손쉽게 "기능 제공자(서버)"와 "기능 소비자(클라이언트)"를 구현할 수 있습니다.

- `FastMCP`를 쓰면 파이썬스러운 코드로, 그냥 함수를 만들고 데코레이터만 붙여도 서버 기능이 생깁니다.

- 클라이언트는 준비된 서버에 붙어서, 제공된 기능을 마음껏 사용할 수 있습니다.

---

다음 장에서는 **툴/리소스/프롬프트 매니저**가 실제로 MCP 서버에서 어떤 역할을 하는지,  
그리고 어떻게 등록과 관리가 이루어지는지 알아보겠습니다! 🚩

---