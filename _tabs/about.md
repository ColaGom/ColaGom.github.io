---
title: About
icon: fas fa-info-circle
order: 4
---

```kotlin
fun me() = developer {
  profile {
    name = "Geunho"
    +role(Role.Android)
    +role(Role.Backend)
    +role(Role.DevOps)
  }
  +language(Php, 3.years)
  +language(Java, 4.years)
  +language(Kotlin, 5.years)

  skills("android", "kmm", "spring", "aws", "sql", "kafka", "ktor")
}
```
{: file="about.kt" }
