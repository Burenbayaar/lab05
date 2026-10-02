# Лаб 05 — API систем тест: Postman ба Newman

**Оюутан:** Эрдэнэтөр Бүрэнбаяр
**Оюутны код:** B242270084

## Орчин

```
$ node -v
NODE_VERSION_OUTPUT
```

```
$ newman -v
NEWMAN_VERSION_OUTPUT
```

## Даалгавар 2: Тест дизайн

**Бие даан тестлэгдэх функц:** `POST /registrations`, хүсэлтийн бие нь `{"studentID", "courseID"}`.

### Сонголтууд ба төлөөлөх утгууд

| # | Сонголт (параметр / орчны нөхцөл) | Эквивалент анги | Төлөөлөх утга (collection-д) |
|---|---|---|---|
| 1 | studentID-гийн хүчинтэй байдал | идэвхтэй | `B242270084` (`status: "active"`) |
|   |   | идэвхгүй | `S03-INACTIVE` (`status: "inactive"`) |
|   |   | байхгүй | `S02-NOBODY` (PUT хийгээгүй ID) |
|   |   | хоосон / дутуу (хязгаар) | `""` эсвэл талбар огт байхгүй |
| 2 | Оюутны үзсэн хичээлүүд (coursesTaken) | урьдач нөхцлийг хангана | `["CS201"]` |
|   |   | хангахгүй (хоосон) | `[]` |
|   |   | хэсэгчлэн хангана | `["CS201"]`, шаардлага `["CS201","CS202"]` |
| 3 | courseID-гийн хүчинтэй байдал | байгаа | `CS313` (PUT хийсэн) |
|   |   | байхгүй | `S04-NOCOURSE` |
|   |   | дутуу (хязгаар) | талбар огт байхгүй |
| 4 | Хичээлийн урьдач нөхцөл (prerequisites) | бүгдийг үзсэн | `["CS201"]` + оюутан `["CS201"]` |
|   |   | огт үзээгүй | `["CS201"]` + оюутан `[]` |
|   |   | зарим нь | `["CS201","CS202"]` + оюутан `["CS201"]` |
|   |   | урьдач нөхцөлгүй (хязгаар) | `[]` |
| 5 | Хүсэлтийн формат | зөв JSON | `{"studentID":…,"courseID":…}` |
|   |   | буруу JSON | `{"studentID": "B242270084", "courseID": ` |

**Боломжгүй хослол:** "байхгүй оюутан" анги нь "үзсэн хичээлүүд" сонголттой хослох боломжгүй, учир нь байхгүй оюутанд coursesTaken гэж байхгүй. Мөн "байхгүй хичээл" анги нь "урьдач нөхцөл" сонголттой хослохгүй. Тиймээс эдгээр мөчрийг тусад нь задлаагүй.

### Спецификацийн хүснэгт

| ID | Тайлбар | Setup (PUT) | POST бие | Хүлээгдэх статус | Хүлээгдэх `result` |
|---|---|---|---|---|---|
| S01 | Happy path | оюутан active `[CS201]`, CS313 prereq `[CS201]` | `B242270084`, `CS313` | **201** | `OK` + registrationID тоо, GET жагсаалтад байна |
| S02 | Оюутан байхгүй | хичээл байгаа | `S02-NOBODY`, `S02-CS313` | 200 | `ERROR_NO_STUDENT` |
| S03 | Оюутан идэвхгүй | оюутан inactive, хичээл байгаа | `S03-INACTIVE`, `S03-CS313` | 200 | `ERROR_INACTIVE_STUDENT` |
| S04 | Хичээл байхгүй | оюутан active | `S04-ACTIVE`, `S04-NOCOURSE` | 200 | `ERROR_NO_COURSE` |
| S05 | Урьдач огт үзээгүй | оюутан `[]`, хичээл `[CS201]` | `S05-NOPREREQ`, `S05-CS313` | 200 | `ERROR_PREREQUISITES`, missing `["CS201"]` |
| S06 | Урьдачийн зарим нь дутуу | оюутан `[CS201]`, хичээл `[CS201, CS202]` | `S06-PARTIAL`, `S06-CS401` | 200 | `ERROR_PREREQUISITES`, missing `["CS202"]` |
| S07 | Давхар: идэвхгүй + хичээл байхгүй | оюутан inactive | `S07-INACTIVE`, `S07-NOCOURSE` | 200 | `ERROR_INACTIVE_STUDENT` |
| S08 | Давхар: оюутан байхгүй + хичээл байхгүй | (setup байхгүй) | `S08-NOBODY`, `S08-NOCOURSE` | 200 | `ERROR_NO_STUDENT` |
| S09 | Давхар: идэвхгүй + урьдач дутуу | оюутан inactive `[]`, хичээл `[CS201]` | `S09-INACTIVE`, `S09-CS313` | 200 | `ERROR_INACTIVE_STUDENT` |
| S10 | Хязгаар: урьдачгүй хичээл + хоосон coursesTaken | оюутан `[]`, хичээл `[]` | `S10-FRESH`, `S10-CS101` | **201** | `OK` + registrationID тоо |
| S11 | Хязгаар: courseID талбар дутуу | — | `{"studentID":"B242270084"}` | **400** | `ERROR_BAD_REQUEST` |
| S12 | Хязгаар: studentID хоосон мөр | — | `{"studentID":"","courseID":"CS313"}` | **400** | `ERROR_BAD_REQUEST` |
| S13 | Хязгаар: буруу JSON | — | `{"studentID": "B242270084", "courseID": ` | **400** | `ERROR_BAD_JSON` |

Давхар алдааны S07–S09 тохиолдлуудаас тестээр тогтоосон дараалал: **оюутан байгаа эсэх → оюутан идэвхтэй эсэх → хичээл байгаа эсэх → урьдач нөхцөл**. Эхний таарсан алдаа буцна.

## Даалгавар 3: Collection

- `lab05-collection.json`: 13 бие даасан тест (S01–S13) байгаа бөгөөд тус бүр нь тусдаа folder-т байна.
- Тест бүр өөрийн setup-ыг өөрөө хийнэ (PUT). Бусад тесттэй давхцахгүйн тулд тусдаа ID ашигладаг, жишээ нь `S03-INACTIVE`.
- Oracle бүр статус болон `result` утгыг шалгана. S05/S06 нь `missing` массивыг, S01/S10 нь `registrationID` эерэг бүхэл тоо мөн эсэхийг шалгана. **Яг утгыг шалгахгүй** тул серверийг дахин асаалгүй давтан ажиллуулж болно.
- `{{baseUrl}}` = `http://localhost:3000` (collection variable).

## Даалгавар 4: Newman

| Ажиллуулалт | Файл | requests | assertions executed | assertions failed | exit code |
|---|---|---|---|---|---|
| PASS | `results/newman-pass.txt` | PASS_REQ | **PASS_ASSERT** | PASS_FAILED | PASS_EXIT |
| FAIL (`lab05-collection-fail.json`) | `results/newman-fail.txt` | FAIL_REQ | FAIL_ASSERT | **FAIL_FAILED** | FAIL_EXIT |
| DOWN (сервер унтарсан) | `results/newman-down.txt` | DOWN_REQ | DOWN_ASSERT | DOWN_FAILED | DOWN_EXIT |

- **Тестийн / assertion-ы тоо:** **PASS_ASSERT** (`newman-pass.txt`-ийн `assertions executed`)
- **FAIL:** S01-ийн статус oracle-ийг зориуд `201` → `200` болгосон (`Статус 200 (ЗОРИУД БУРУУ ORACLE)`). Newman `expected response to have status code 200 but got 201` гэж унаж, exit code 1 буцаасан. Энэ нь CI quality gate-д pipeline-ийг зогсооно.
- **DOWN:** гаралтад `connect ECONNREFUSED 127.0.0.1:3000` гарсан. Энэ бол **интерфейсийн алдаа**: хүсэлт серверт огт хүрээгүй тул хариу ирээгүй. Харин FAIL нь **oracle-ийн алдаа**: сервер хариу өгсөн боловч хүлээгдэж буй утгатай таараагүй.

## Дүгнэлт

ДҮГНЭЛТ_ЭНД
