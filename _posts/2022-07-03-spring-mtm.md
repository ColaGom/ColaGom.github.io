---
layout: post
title: "Spring: ManyToMany 다대다관계 사용시 주의사항"
categories: [Dev, Spring]
tags: [spring, jpa, entity]

---

> **ToMany 관계는 독이든성배와 같아서 N+1, 재귀적 관계, 성능문제 등 제대로 사용하지않으면 여러 문제에 직면 할 수있으며 본 글에서 관련내용은 따로 다루지 않는다.*
>

# 다대다 관계

![relation](/assets/img/220703-1-1.png)

```kotlin
@Entity
class Student(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    var id: Long = 0,
    @ManyToMany(cascade = [CascadeType.ALL])
    @JoinTable(
        name = "student_courses",
        joinColumns = [JoinColumn(name = "student_id")],
        inverseJoinColumns = [JoinColumn(name = "course_id")],
    )
    val courses: MutableSet<Course> = mutableSetOf(),
    var name: String,
)

@Entity
class Course(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    var id: Long = 0,
    @ManyToMany(mappedBy = "courses")
    val students: MutableSet<Student> = mutableSetOf(),
    var name: String
)
```

위 처럼 JoinTable을 사용하여 ManyToMany 관계를 구현 할 수있다.

### 주인 엔티티?

양방향 ManyToMany 관계를 정의할때 한 쪽은 관계의 주인으로 정의하는것이 필수이다.

여기선 `Student` 엔티티가 관계의 주인이며 ***Hibernate는 이를 확인하여 주인인 경우에만 persistence 동작을 수행한다.***

# 주의 할 점

> *주인 엔티티를 통해 두 엔티티사이의 관계를 관리해야된다.*
>

```kotlin
course.students.add(student) // not work
student.courses.add(course) // okay
//addAll, delete 등 전부 마찬가지
```

[ManyToMany (Java(TM) EE 7 Specification APIs)](https://docs.oracle.com/javaee/7/api/javax/persistence/ManyToMany.html#mappedBy)
