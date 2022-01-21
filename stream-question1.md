Из списка резюме надо сформировать мапу, где ключём является название должности,
а значением является резюме с самым длительным опытом по этой должности.
(Надо учитывать, что у одного резюме есть несколько периодов опыта с разными должностями)

```java
class Experience {
  String position;
  LocalDate startDate;
  LocalDate endDate;

  public Experience(String position, String startDate, String endDate) {
    this.position = position;
    this.startDate = LocalDate.parse(startDate);
    this.endDate = LocalDate.parse(endDate);
  }
}

class Resume {
  List<Experience> experiences;

  public Resume(List<Experience> experiences) {
    this.experiences = experiences;
  }
}
```

Тестовые данные
```java
    Resume resume1 = new Resume(List.of(new Experience("грузчик", "2000-01-01", "2005-02-02"), new Experience("менеджер", "2007-01-01", "2009-02-02")));
    Resume resume2 = new Resume(List.of(new Experience("менеджер", "2000-01-01", "2005-02-02"), new Experience("директор", "2007-01-01", "2009-02-02")));
    Resume resume3 = new Resume(List.of(new Experience("продавец", "2000-01-01", "2005-02-02"), new Experience("директор", "2010-01-01", "2020-02-02")));
```

Решение:

```java
    Comparator<AbstractMap.SimpleEntry<String, Resume>> byExperienceDays =
        Comparator.comparing(e -> e.getValue().experiences.stream()
            .filter(exp -> exp.position.equals(e.getKey()))
            .mapToLong(exp -> DAYS.between(exp.startDate, exp.endDate))
            .max()
            .orElse(0)
        );

    //Вариант через toMap
    var longestExperienceByPosition1 = Stream.of(resume1, resume2, resume3)
        .flatMap(r -> r.experiences.stream().map(e -> new AbstractMap.SimpleEntry<>(e.position, r)))
        .collect(Collectors.toMap(AbstractMap.SimpleEntry::getKey, Function.identity(), BinaryOperator.maxBy(byExperienceDays)))
        .entrySet().stream()
        .collect(Collectors.toMap(Map.Entry::getKey, e -> e.getValue().getValue()));

    //Вариант через groupingBy+reduce
    var longestExperienceByPosition2 = Stream.of(resume1, resume2, resume3)
        .flatMap(r -> r.experiences.stream().map(e -> new AbstractMap.SimpleEntry<>(e.position, r)))
        .collect(
            groupingBy(Map.Entry::getKey, collectingAndThen(reducing(BinaryOperator.maxBy(byExperienceDays)), v -> v.get().getValue()))
        );


    assertEquals(resume1, longestExperienceByPosition2.get("грузчик"));
    assertEquals(resume2, longestExperienceByPosition2.get("менеджер"));
    assertEquals(resume3, longestExperienceByPosition2.get("директор"));
    assertEquals(resume3, longestExperienceByPosition2.get("продавец"));
```
