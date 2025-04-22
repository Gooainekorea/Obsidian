---
tags: 
list:
  - "[[Frontmatter]]"
Day: 2025-04-18
---
데이터를 저장하고 전달하기 위한 형식으로 간단한 구조로 다양한 데이터 형식을 지원한다.
1. Scalar : 문자열, 숫자, 불리언 값과 같은 기본 데이터 타입
	```
	ex) JSON 데이터 변환
	name: John Doe
	age: 30
	isEmployee: true
	```
2. List : 배열, 목록
	```
	각 항목은 동일한 들여쓰기 레벨에서 화이트로 시작
	hobbies:
	- Reading
	- Hiking
	- Consel
	```
3. Map : 키-값 쌍으로 데이터를 구성.

- 들여쓰기로 데이터의 계층구조를 나타내기 때문에 매우 중요
- 앵커(`&`)와 별칭(`*`)을 사용하여 데이터를 재사용가능
	```
	# 앵커
	defaultAddress: &address
	  street: 123 Main St
	  city: Anytown
	  zip: 12345
	
	# 앵커참조
	johnsAddress: *address
	janesAddress:
	  <<: *address
	  street: 456 Elm St
	```
- 줄바꿈 ( | )
	줄바꿈을 유지하며, 문자열을 그대로 보존
	```
	|
	  여러 줄의
	  텍스트를
	  그대로 보존합니다.
	```
- 폴디드 스타일(Folded Style) ( > )
	```
	> 
	  이것은 하나의
	  긴 문장입니다. (1, 2줄은 한 문장으로 결합)
	  이 줄바꿈은 유지됩니다.
	
	  새로운 단락입니다. (새로운 단락은 별도유지)
	```
- 다중 문서지원 ( --- )
	--- 로 구분된 여러 문서를 한 파일 내에서 지원.
	관련된 여러 설정을 한 파일 내에서 관리 가능
	```
	# Document 1
	---
	name: John Doe
	age: 30
	---
	# Document 2
	name: Jane Smith
	age: 25
	```
- 주석 사용 ( # )

들여쓰기와 데이터 타입의 혼동을 피해야한다.