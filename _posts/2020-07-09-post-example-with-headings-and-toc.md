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
