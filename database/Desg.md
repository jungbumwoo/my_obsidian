
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

---
****There are mainly two types of decompositions in DBMS**** 

1. Lossless join Decomposition
2. Lossy join Decomposition


--- 
### 1. 전체 참여(Total Participation) vs. 부분 참여(Partial Participation)

- **전체 참여(Total Participation):**
    
    - 어떤 엔터티 집합의 모든 요소(엔터티)가 적어도 하나의 관계에 반드시 참여해야 할 때, 그 엔터티 집합은 관계에 **전체 참여**한다고 합니다.
    - **예:** 대학에서는 모든 학생이 반드시 한 명 이상의 지도교수를 가져야 한다고 가정합시다. 그러면 학생(student) 엔터티는 지도(advisor) 관계에 **전체 참여**하는 것입니다. 즉, 어떤 학생도 지도교수가 없는 경우가 없으므로, 반드시 관계에 연결되어야 합니다.
- **부분 참여(Partial Participation):**
    
    - 반대로, 어떤 엔터티 집합의 일부 엔터티는 관계에 참여하지 않아도 되는 경우, 그 엔터티 집합은 관계에 **부분 참여**한다고 합니다.
    - **예:** 한 교수(instructor)는 반드시 학생의 지도교수가 될 필요가 없습니다. 즉, 어떤 교수는 지도(advisor) 관계에 전혀 참여하지 않을 수도 있으므로, 교수 엔터티는 지도 관계에 **부분 참여**하는 것입니다.

### 2. ER 다이어그램에서 참여 표현 방법

- **이중 선 (Double Lines):**
    - ER 다이어그램에서는 **전체 참여**를 표시할 때 해당 엔터티와 관계를 연결하는 선을 이중 선(double line)으로 나타냅니다.
    - 예를 들어, 학생과 지도(advisor) 관계를 연결하는 선이 이중 선이라면, 이는 모든 학생이 반드시 지도교수를 가져야 함을 의미합니다.


