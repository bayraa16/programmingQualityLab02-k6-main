Лаборатори 2 — k6

B222270019 
Б.Баяржавхлан

Зорилго

k6 ашиглан https://test.k6.io дээр ачааллын тест хийж, VU нэмэгдэхэд latency, throughput, error rate яаж өөрчлөгдөхийг харьцуулсан. Мөн baseline үр дүндээ тулгуурлаж SLO болон threshold тохируулж шалгасан.

Орчин

Ubuntu
k6.exe v2.4.0 (commit/dd9395906d, go1.26.11, windows/amd64)
Туршсан хаяг: https://test.k6.io

Файлууд

script.js — baseline болон тогтмол VU тест
script-stages.js — үе шаттай ачааллын тест
script-threshold.js — PASS болох threshold
script-threshold-fail.js — зориуд FAIL болох threshold
results/ — бүх тестийн бүтэн гаралт
screenshots/ — k6 summary зургууд

Ажиллуулсан командууд

bash
k6 run script.js | tee results/run-05vu-baseline.txt
k6 run --vus 5 --duration 1m script.js | tee results/run-05vu.txt
k6 run --vus 30 --duration 1m script.js | tee results/run-30vu.txt
k6 run --vus 100 --duration 1m script.js | tee results/run-100vu.txt
k6 run script-stages.js | tee results/run-stages.txt
k6 run script-threshold.js | tee results/run-threshold-pass.txt
k6 run script-threshold-fail.js | tee results/run-threshold-fail.txt

Baseline — 5 VU, 30 секунд

avg	p90	p95	Throughput	Error rate
237.49 мс	724.65 мс	744.41 мс	6.32 req/s	0.00%

Бүтэн үр дүн

<details> <summary>Baseline screenshot</summary>

5 VU baseline үр дүн

</details>

SLO ба threshold

Baseline p95 нь 744.41 мс байсан. Үүнийг 1.5-аар үржүүлээд 744.41 × 1.5 = 1116.62 мс болсон тул SLO-г p(95) < 1117 мс гэж авсан. Error rate-ийн threshold-ийг rate < 0.01 гэж тохируулсан.

Туршилт	Threshold	Бодит үр дүн	Төлөв
Энгийн SLO	p(95) < 1117 мс	p95 = 1.10 сек, error = 0.00%	PASS
Зориуд хатуу SLO	p(95) < 50 мс	p95 = 1.11 сек, error = 0.00%	FAIL
PASS бүтэн үр дүн · screenshot
FAIL бүтэн үр дүн · screenshot

FAIL тестийг threshold үнэхээр алдаа барьж байгааг шалгахын тулд зориуд хатуу утгатай ажиллуулсан.

5, 30, 100 VU харьцуулалт

Тест бүрийг 1 минут ажиллуулсан.

VU	p90	p95	Throughput	Error rate
5	735.34 мс	753.56 мс	6.21 req/s	0.00%
30	712.71 мс	740.38 мс	36.15 req/s	0.00%
100	744.45 мс	787.56 мс	123.73 req/s	0.00%
5 VU бүтэн үр дүн · screenshot
30 VU бүтэн үр дүн · screenshot
100 VU бүтэн үр дүн · screenshot

Stages тест

Ачааллыг 30s → 5 VU, 1m → 30 VU, 30s → 100 VU, 30s → 0 VU гэсэн дарааллаар өөрчилсөн. Нийт p90 752.97 мс, p95 948.85 мс, throughput 37.89 req/s, error rate 0.00% гарсан.

Бүтэн үр дүн · screenshot

Дүгнэлт

Энэ лабораторийн гол үр дүн нь test.k6.io сервер 5-аас 100 VU хүртэлх ачааллыг алдаагүй даасан явдал юм. Бүх туршилтад error rate 0.00% байсан бөгөөд checks 100% амжилттай болсон.

Throughput нь VU-ийн тоотой бараг пропорциональ өссөн: 5 VU дээр 6.21 req/s, 30 VU дээр 36.15 req/s, 100 VU дээр 123.73 req/s байв. Харин p95 latency 740–788 мс-ийн явцуу хүрээнд л хэлбэлзсэн. 30 VU-ийн p95 (740.38 мс) нь 5 VU-ийнхээс (753.56 мс) бага гарсан нь ачааллын нөлөөнөөс илүү сүлжээ болон серверийн тухайн үеийн хэлбэлзэл байж болзошгүйг харуулж байна. Харин 100 VU дээр p95 787.56 мс хүрсэн нь ачаалал өсөхөд хүлээх хугацаа бага зэрэг уртсаж эхэлсний шинж юм. Тиймээс энэ ачаалал серверийн хязгаарт хүрээгүй гэж үзэж болно.

Stages тест бол хамгийн сонирхолтой үр дүн өгсөн. Ачаалал огцом өөрчлөгдөх үед p95 948.85 мс хүртэл нэмэгдэж, хамгийн удаан хүсэлт 3.59 секунд болсон нь тогтмол 100 VU-ийн туршилтаас (787.56 мс, 2.49 сек) илүү муу байна. Иймд тогтмол ачаалал бус, ачааллын огцом өөрчлөлт tail latency-д илүү нөлөөлдөг гэж харагдаж байна. Гэхдээ нэг удаагийн хэмжилт учраас үүнийг батлахын тулд давтан туршилт хэрэгтэй.

SLO-ийн хувьд baseline p95-ыг 1.5-аар үржүүлж p(95) < 1117 мс гэж тавьсан бөгөөд бодит p95 1.10 сек байж PASS болсон. Гэвч энэ нь хязгаартаа маш ойр байсан тул дахин ажиллуулахад FAIL болох магадлалтай. Бодит төсөлд илүү зайтай утга (жишээ нь 1500 мс) сонгох нь зүйтэй. Харин зориуд тавьсан p(95) < 50 мс threshold нь p95 1.11 сек дээр FAIL болсон. HTTP error 0% байсан ч threshold нь амжилтгүй болсноор k6 нь performance quality gate болж чадахыг харууллаа. Үүнийг CI pipeline-д ашиглавал удаан ажилладаг хувилбарыг автоматаар барьж чадна.

Дүгнэж хэлбэл, error rate нэг ч удаа нэмэгдээгүй ч зөвхөн алдааны хувиар системийг "хэвийн" гэж үнэлж болохгүй: latency-ийн threshold-ийг хамт хянах шаардлагатай нь энэ лабораториор харагдсан.
