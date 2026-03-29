---
layout: post
---

### 📂 시도/시군구별 감기 진료 정보 합계 구하기

이 코드는 `csv` 모듈을 사용하여 지역 코드 데이터와 진료 정보를 매핑하고, 특정 날짜와 지역에 따른 감기 환자 수의 합계를 계산하는 스크립트입니다.

```python
import csv

# 파일 열기
f_sido = open('시도 지역코드.csv', 'r')
f_sigungu = open('시군구 지역코드.csv', 'r')
f_info = open('실제진료정보_감기_시군구.csv', 'r')

rdr_sido = csv.reader(f_sido)
rdr_sigungu = csv.reader(f_sigungu)
rdr_info = csv.reader(f_info)

# 데이터 구조화
sigungu_dict = {}
cold_data = {}

# 1. 시도 코드 매핑 (코드 -> 이름)
temp_sido = {}
for line in rdr_sido:
    if line:
        temp_sido[line[0]] = line[1]

# 2. 시군구 코드 매핑 (코드 -> [시군구명, 시도명])
for line in rdr_sigungu:
    if line:
        parent_code = line[0]
        sido_name = temp_sido.get(parent_code, "")
        sigungu_dict[line[1]] = [line[2], sido_name]

# 3. 진료 정보 매핑 ((날짜, 시군구코드) -> 발생건수)
for line in rdr_info:
    if line:
        cold_data[(line[0], line[1])] = line[2]

# 사용자 입력
target_sido = input('지역: ')
target_date = input('날짜: ')

total_count = 0

# 결과 출력 및 합계 계산
for code in sigungu_dict:
    name, sido = sigungu_dict[code]
    if target_sido == sido:
        count = cold_data.get((target_date, code), "0")
        print(f"{name} {count}")
        total_count += int(count)

print(f"합계 {total_count}")

# 파일 닫기
f_sido.close()
f_sigungu.close()
f_info.close()
```

## 🔍 코드 한 줄씩 뜯어보기

작성한 파이썬 스크립트의 주요 로직을 단계별로 설명합니다.

---

### 1. CSV 모듈 임포트 및 파일 열기
```python
import csv

f_sido = open('시도 지역코드.csv', 'r')
f_sigungu = open('시군구 지역코드.csv', 'r')
f_info = open('실제진료정보_감기_시군구.csv', 'r')
```
> `csv` 라이브러리를 불러오고, 분석에 필요한 세 개의 CSV 파일을 읽기 모드(`r`)로 엽니다.

### 2. 리더(Reader) 설정
```python
rdr_sido = csv.reader(f_sido)
rdr_sigungu = csv.reader(f_sigungu)
rdr_info = csv.reader(f_info)
```
> 파일을 줄 단위로 읽어 리스트 형태로 변환해주는 `reader` 객체를 생성합니다.

### 3. 시도 코드 매핑
```python
temp_sido = {}
for line in rdr_sido:
    if line:
        temp_sido[line[0]] = line[1]
```
> `temp_sido` 딕셔너리에 `지역코드: 시도명` 형태로 저장합니다. (예: `{'11': '서울특별시'}`)

### 4. 시군구와 상위 시도 연결
```python
sigungu_dict = {}
for line in rdr_sigungu:
    if line:
        parent_code = line[0]
        sido_name = temp_sido.get(parent_code, "")
        sigungu_dict[line[1]] = [line[2], sido_name]
```
> 시군구 코드(`line[1]`)를 키로 하여, 해당 시군구의 이름과 그 상위 시도 이름을 리스트로 묶어 저장합니다.

### 5. 감기 진료 데이터 구조화
```python
cold_data = {}
for line in rdr_info:
    if line:
        cold_data[(line[0], line[1])] = line[2]
```
> 날짜와 시군구 코드를 튜플`(날짜, 코드)`로 묶어 키로 만들고, 진료 건수를 값으로 저장합니다.

### 6. 사용자 입력 및 필터링
```python
target_sido = input('지역: ')
target_date = input('날짜: ')

total_count = 0

for code in sigungu_dict:
    name, sido = sigungu_dict[code]
    if target_sido == sido:
        count = cold_data.get((target_date, code), "0")
        print(f"{name} {count}")
        total_count += int(count)
```
> 입력받은 지역과 날짜에 해당하는 데이터를 찾습니다. `sigungu_dict`를 돌며 사용자가 입력한 시도명과 일치하는 시군구만 추출하여 합산합니다.

### 7. 결과 출력 및 종료
```python
print(f"합계 {total_count}")
f_sido.close()
f_sigungu.close()
f_info.close()
```
> 최종 합계를 출력하고, 열려있던 모든 파일 객체를 닫아 메모리를 해제합니다.
