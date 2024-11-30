
### mspan 과 크기 

#### 1. **Go의 페이지와 `mspan` 구조**

- **페이지(page)**: 메모리를 관리하는 기본 단위. Go는 **8KB 페이지**를 사용합니다.
- **런(run)**: 페이지가 여러 개 연속으로 묶인 것.
    - 예를 들어, 3개의 페이지를 묶어 하나의 "런"으로 관리할 수 있습니다.
- **`mspan`**: Go에서 페이지나 런을 관리하기 위한 데이터 구조.
    - 페이지나 런이 **어떤 용도로 사용되고 있는지**, 또는 **얼마나 많은 메모리가 할당되었는지**를 추적.

#### 2. **크기 클래스(size classes)와 메모리 절약**

- **크기 클래스**:
    - Go는 메모리를 관리하기 위해 미리 정의된 다양한 크기 단위를 사용.
    - 예를 들어, 8바이트, 16바이트, 32바이트 등, 정해진 "크기 클래스"로 메모리를 나눔.
    - 크기 클래스마다 특정한 개수의 페이지(런)를 할당하고, 이를 **작은 객체 블록**으로 잘라 사용.


- **`mspan`**: 페이지와 런을 추적하고 관리하는 데이터 구조.
- **크기 클래스**: 메모리를 정해진 크기로 나눠 효율적으로 관리.

.

### **Go의 mcache와 mcentral 작동 원리**

- **`mcache`**: 각 스레드가 사용하는 **빠른 메모리 할당 캐시**. 하지만, 필요한 크기의 메모리 블록이 모두 사용 중이라 **빈 슬롯이 없으면**, 새로운 메모리 블록(`mspan`)이 필요.  **`mcentral`**에서 새로운 `mspan`을 가져옴.  

- **`mcentral`**: 특정 크기 클래스에 맞는 모든 `mspan`을 관리하는 구조체.
- 각 **크기 클래스**에 대해 두 개의 `mspan` 리스트를 관리:
    1. **empty mspanList**: 모든 객체가 사용 중인 `mspan` 목록.
    2. **nonempty mspanList**: 아직 할당되지 않은 객체(빈 슬롯)가 남아 있는 `mspan` 목록.

- `mcache`에 빈 슬롯이 없을 때, **`mcentral`의 `nonempty mspanList`**에서 `mspan`을 가져옴.
- 가져온 `mspan`은 이제 **`empty mspanList`**로 이동.
    - 이유: `mcache`에 들어간 `mspan`은 더 이상 `mcentral` 관점에서 사용 가능한 빈 슬롯이 없다고 간주됨.
-  `mspan`에 있던 객체가 프로그램에서 해제되면, 빈 슬롯이 다시 생김.
- 빈 슬롯의 수에 따라 **`mcentral`의 리스트**가 업데이트:
    - 만약 `mspan`에 빈 슬롯이 생기면, **`empty mspanList`**에서 **`nonempty mspanList`**로 이동.
    - 이렇게 하면 `mcentral`은 항상 사용 가능한 `mspan`을 효율적으로 관리.


### **mheap, mcentral, mspan의 관계와 작동 방식**

![[Pasted image 20241130174028.png]]

- **`mheap`**:
    - Go에서 메모리 관리의 최상위 레벨 구조로, 모든 메모리 할당의 근원.
    - `mheap`에는 **`mcentral` 배열**을 갖고 있음. 이 배열은 각 **크기 클래스(span class)**에 대한 `mcentral`을 저장.
    - `mheap`은 사용 가능한 페이지들을 관리하기 위해 두 가지 구조를 사용

		1. **`free` 배열**:
		    - 크기별로 사용 가능한 페이지를 리스트 형태로 관리.
		    - 예: `free[3]` → 3페이지로 구성된 `mspan`들의 연결 리스트.
		    - 이 배열은 **1 ~ 127 페이지**를 가진 `mspan`을 관리.
		2. **`freelarge`**:
		    - 127 페이지 이상을 가진 큰 `mspan`들의 목록.
		    - 트리 자료구조인 **`mtreap`**으로 관리.
		    - 큰 메모리 블록을 효율적으로 검색하고 할당 가능

![[Pasted image 20241130174148.png]]
- **`mcentral`**:
	-  Go provides each **Logical Processors**(**P**) a Local Thread Cache of Memory known as **mcache**
	- 특정 크기 클래스에 속하는 **메모리 블록(`mspan`)**을 관리.
		- class size에 따라 두 타입이 나뉨. GC 실행 시 이점을 위해서 구분해둠 
		- **noscan** object doesn’t need to be traversed to find any containing live object.
			1. **scan** — Object that contains a pointer.
			2. **noscan** — Object that doesn’t contains a pointer.
	

 `mheap` → `mcentral` → `mspan` 구조로 메모리가 내려감

#### **mcache와 mcentral의 상호작용**
- **mcache**가 빈 슬롯이 없으면, 필요한 크기의 `mspan`을 `mcentral`에서 요청.
- 요청 과정에서 **락(lock)**이 사용되지만, 락은 **요청된 크기 클래스의 `mcentral`에만 적용**.
    - 다른 크기 클래스의 요청은 동시에 처리 가능.
    - ex - 16바이트 블록의 `mcentral`과 32바이트 블록의 `mcentral`은 동시에 요청을 처리할 수 있음.

#### mcentral에 사용 가능한 `mspan`이 없을 때
- `mcentral`은 **`mheap`에서 새로운 페이지(run of pages)**를 가져옴.
- 이렇게 가져온 페이지를 필요한 크기 클래스로 나누어 `mspan`으로 사용.


https://medium.com/@ankur_anand/a-visual-guide-to-golang-memory-allocator-from-ground-up-e132258453ed