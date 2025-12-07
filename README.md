# Korean Local Election Districts

대한민국 지방의회 선거구 데이터

## 데이터 구조

### 기본 필드

#### `id` (string)

선거구 고유 식별자

- 형식: `{sido}-{sigungu}-{metropolitan}-{local}`
- 예: `gg-sw-01-ga` (경기도 수원시 제1선거구 가선거구)
- 특별자치시/도: `{sido}-00-{number}-00`
  - 예: `sj-00-07-00` (세종), `jj-00-01-00` (제주)

**시도 코드:**

- `gg`: 경기도
- `gw`: 강원도
- `cb`: 충청북도
- `cn`: 충청남도
- `jb`: 전라북도
- `jn`: 전라남도
- `gb`: 경상북도
- `gn`: 경상남도
- `jj`: 제주특별자치도
- `sj`: 세종특별자치시

**시군구 코드:**

- 일반 시: 약어 (예: `sw` - 수원, `cj` - 청주, `chu` - 충주)
- 군: 약어 + `gun` (예: `ycgun` - 영천군)

#### `region` (object)

행정구역 정보

```json
{
  "sido": "경기도",
  "sigungu": "수원시",
  "hasGu": true // 구가 있는 경우만
}
```

- 특별자치시/도는 `sigungu: null`

#### `coverage` (object)

선거구가 포함하는 행정동 정보

**기본 형태:**

```json
{
  "administrativeDongs": ["동1", "동2"]
}
```

**구가 있는 시:**

```json
{
  "administrativeDongs": ["동1", "동2"],
  "administrativeUnits": [
    {
      "gu": "장안구",
      "units": ["동1", "동2"]
    }
  ]
}
```

**세부 분할이 있는 경우:**

```json
{
  "administrativeDongs": ["물금읍"],
  "detailRules": [
    {
      "adminUnit": "물금읍",
      "type": "ri",
      "include": ["범어리", "가촌리"],
      "mode": "auto"
    }
  ]
}
```

#### `electionDistricts` (object)

선거구 명칭

```json
{
  "metropolitanCouncil": { "name": "제1선거구" },
  "localCouncil": { "name": "가선거구" }
}
```

- 특별자치시/도는 `localCouncil` 없음
- 제주는 `metropolitanCouncil.name`에 전체 선거구명 (예: "제주시 일도1동·이도1동·건입동")

### detailRules 상세

선거구가 동/읍/면보다 세부 단위로 나뉠 때 사용

#### `type` 종류:

- `beopjeongDong`: 법정동 단위
- `ri`: 리 단위
- `tong`: 통 단위
- `village`: 자연마을 단위

#### `mode`:

- `auto`: 주소로 자동 판별 가능
- `manual`: 사용자가 직접 선택 필요 (통, 자연마을 등)

#### 예시:

**통 단위 분할 (세종/제주):**

```json
{
  "adminUnit": "도담동",
  "type": "tong",
  "include": ["1~9", "13~19", "22", "25"],
  "mode": "manual",
  "note": "통 단위는 주소 자동 판별 불가, 사용자 선택 필요"
}
```

**법정동 단위 분할:**

```json
{
  "adminUnit": "물금읍",
  "type": "beopjeongDong",
  "include": ["범어리"],
  "mode": "auto"
}
```

**자연마을 단위 분할:**

```json
{
  "adminUnit": "상림리",
  "type": "village",
  "include": ["상동"],
  "mode": "auto",
  "note": "자연마을명은 자동 판별 불가, 사용자 선택 필요"
}
```

## 엣지 케이스

### 1. 구가 있는 시 (hasGu)

수원, 성남, 고양, 창원, 포항 등

- `hasGu: true` 설정
- `administrativeUnits`로 구별 그룹핑

### 2. 군(郡) ID

- 반드시 `gun` 접미사 사용
- 예: `gb-ycgun-01-ga` (경북 영천군)

### 3. 특별자치시/도

세종특별자치시, 제주특별자치도

- `sigungu: null`
- ID: `{sido}-00-{number}-00`
- `localCouncil` 없음

### 4. 다단계 detailRules

한 선거구에 여러 detailRules 적용 가능

```json
"detailRules": [
  {
    "adminUnit": "거창읍",
    "type": "ri",
    "include": ["상림리"],
    "mode": "auto"
  },
  {
    "adminUnit": "상림리",
    "type": "village",
    "include": ["상동"],
    "mode": "auto",
    "note": "자연마을명은 자동 판별 불가, 사용자 선택 필요"
  }
]
```

## 파일 형식

- NDJSON (Newline Delimited JSON)
- 각 줄이 하나의 JSON 객체
- 들여쓰기: 2 spaces
