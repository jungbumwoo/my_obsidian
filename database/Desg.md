
### **Mandatory Participation (필수 참여) vs. Optional Participation (선택적 참여)**

어떤 테이블이 다른 테이블과의 관계에서 반드시 존재해야 하는지 여부를 따지는 개념.

#### **(1) 필수 참여 (Mandatory Participation)**

- **TABLE_A가 존재해야만 TABLE_B에 데이터를 추가할 수 있는 경우**.
- 즉, `TABLE_B`가 `TABLE_A`의 데이터를 **반드시 참조해야만** 하는 경우.
- **예제**: `AGENTS`(에이전트)와 `CLIENTS`(고객) 테이블
    - 만약 `CLIENTS` 테이블의 고객이 반드시 `AGENTS` 테이블의 에이전트에게 할당되어야 한다면, `AGENTS` 테이블은 **필수 참여**를 해야 합니다.
    - 즉, **에이전트가 존재하지 않으면 고객을 추가할 수 없음**.
- 데이터베이스에서는 `CLIENTS` 테이블이 `AGENTS` 테이블의 `agent_id`를 **외래 키(Foreign Key)로 설정하고, NOT NULL 제약 조건을 적용**할 수 있음.

#### **(2) 선택적 참여 (Optional Participation)**

- **TABLE_A에 데이터가 없어도 TABLE_B에 데이터를 추가할 수 있는 경우**.
- 즉, `TABLE_B`가 `TABLE_A`와 관계를 맺을 수도 있고, 맺지 않을 수도 있음.
- **예제**: `AGENTS`(에이전트)와 `CLIENTS`(고객) 테이블
    - 만약 고객이 반드시 에이전트를 가져야 하는 것이 아니라면, `AGENTS` 테이블의 존재 여부는 **선택적 참여**가 됩니다.
    - 즉, **에이전트가 없어도 고객을 추가할 수 있음**.
- `CLIENTS` 테이블에서 `agent_id`는 **NULL을 허용**할 수 있음.

---

O : 0 or more.
| : 1 or more.
|O : 1 or 0.
|| : exactly one.
<-(tri): many.