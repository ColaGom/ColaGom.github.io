---
layout: post
title: "Spring, opencsv 활용하여 csv 내보내기"
categories: [Dev, Spring]
tags: [spring, opencsv]

---

# OpenCSV

jvm을 지원하는 csv 라이브러리 중 가장 지원하는 커스텀 기능과 사용성이 좋아 csv를 다룰 일이 있으면 사용 중.

# Comparator

간단한 Comparator이며 reflection을 이용하여 클래스의 프로퍼티 순서데로 컬럼이 생성되도록하기위해 구현

```kotlin
class MyComparator<T>(private val clazz: Class<T>) : Comparator<String> {

    private val memberOrder: List<String> by lazy {
        FieldUtils.getAllFields(clazz)
            .map { it.getDeclaredAnnotation(CsvBindByName::class.java) }
            .map { it?.column ?: "" }
            .map { it.uppercase(Locale.US) }
    }

    override fun compare(field1: String?, field2: String?): Int {
        return memberOrder.indexOf(field1) - memberOrder.indexOf(field2)
    }

    companion object {
        inline fun <reified T> of() = StatisticsComparator(T::class.java)
    }
}
```

# Extension

csv 추출 시 중복코드가 많이 발생하여 HttpServletResponse의 extension으로 작업

```kotlin
inline fun <reified T> HttpServletResponse.csv(filename: String, items: List<T>) {
    val contentDisposition =
        ContentDisposition.builder("attachment")
            .filename(filename, StandardCharsets.UTF_8)
            .build().toString()

    val mappingStrategy = HeaderColumnNameMappingStrategy<T>().apply {
        setColumnOrderOnWrite(StatisticsComparator.of<T>())
        type = T::class.java
        setErrorLocale(Locale.US)
    }

    contentType = "text/csv; charset=UTF-8"
    characterEncoding = "UTF-8"
    addHeader(HttpHeaders.CONTENT_DISPOSITION, contentDisposition)

    OutputStreamWriter(outputStream).use {
        val beanToCsv = StatefulBeanToCsvBuilder<T>(it)
            .withMappingStrategy(mappingStrategy).build()
        beanToCsv.write(items)
    }
}
```

# Use

```kotlin
//somewhere in Controller :)
@GetMapping("csv")
fun getCsv(
    parameters: ParametersForCSV
    response: HttpServletResponse
) {
   response.csv("MY_CSV.csv", service.getAll(parameters))
}
```
