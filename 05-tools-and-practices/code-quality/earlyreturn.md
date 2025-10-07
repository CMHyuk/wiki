### Early return

```java
if (a > 3) {
  doSomething1();
} else if (a <= 3 && b > 1) {
  doSomething2();
} else {
  doSomethig3();
}
```

```java
extracted();

void extracted() {
    if (a > 3) {
        doSomething1();
        return;
    }
    if (a <= 3 && b > 1) {
        doSomething2();
        return;
    }
    doSomething3();
}
```

**Early return으로 else의 사용을 지양**