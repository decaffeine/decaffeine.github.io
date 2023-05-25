---
layout: post
author: Hyejin Kim
tags: [java, list, null]
---

### 발생한 오류  
- 아래와 같은 코드에서 value가 null일 때 NullPointerException이 발생하였다.  

```java
public boolean isXXX() {
    return List.of("A", "B").contains(value);
}

```
  
  
### 원인 파악  
- null에 직접 접근한 것이 아닌, List에 null이 있느냐 없느냐를 판단하는 코드라 NPE가 발생할 것이라고 예측하지 못했다.  
- List의 contains 상단의 주석을 보면 다음과 같다.  

```java
  Returns true if this list contains the specified element. More formally, returns true if and only if this list contains at least one element e such that Objects.equals(o, e).
  Params:
  o – element whose presence in this list is to be tested
  Returns:
  true if this list contains the specified element
  Throws:
  ClassCastException – if the type of the specified element is incompatible with this list (optional)
  NullPointerException – if the specified element is null and this list does not permit null elements (optional)
  
          boolean contains(Object o);
```

- o가 null이고 *list가 null을 허용하지 않을 때* NPE를 발생시킬 수 있다고 되어 있다.
- List.of로 생성된 리스트가 null을 허용하지 않는지 List.of의 코드를 확인해 보면 다음과 같다.

```java
    /**
     * Returns an unmodifiable list containing three elements.
     *
     * See <a href="#unmodifiable">Unmodifiable Lists</a> for details.
     *
     * @param <E> the {@code List}'s element type
     * @param e1 the first element
     * @param e2 the second element
     * @param e3 the third element
     * @return a {@code List} containing the specified elements
     * @throws NullPointerException if an element is {@code null}
     *
     * @since 9
     */
    static <E> List<E> of(E e1, E e2, E e3) {
        return ImmutableCollections.listFromTrustedArray(e1, e2, e3);
    }

```
 - 요소가 null일 경우 NPE를 발생시킨다고 명시되어 있다.

### 테스트 작성
- 테스트를 통하여 위 내용을 검증해 보자.

```java
    @Test
    public void containsNull() {
        List<String> testList = Arrays.asList("A", "B", "C", null);
        assertThat(testList.contains(null)).isEqualTo(true);
    }

    @Test
    public void notContainsNull() {
        List<String> testList = Arrays.asList("A", "B", "C");
        assertThat(testList.contains(null)).isEqualTo(false);
    }
```
- list에 null이 이미 들어있는 경우 contains(null) 해도 NPE가 방지하지 않는다. (contains의 설명과 일치한다.)

```java 
        @Test
    public void immutableList() {
        List<String> testList = List.of("A", "B", "C");
        assertThatThrownBy(() -> testList.contains(null))
                .isInstanceOf(NullPointerException.class);
    }
    
        @Test
    public void immutableListDoesNotAllowNull() {
        assertThatThrownBy(() -> List.of("A", "B", "C", null))
                .isInstanceOf(NullPointerException.class);
    }
```
- List.of로 리스트를 생성할 경우 불변(immutable) 객체로 생성되고, 불변 리스트는 null을 허용하지 않는다.
  - 그러므로 List.of로 생성시 null을 포함해서 생성하려고 하면 NPE가 발생한다.
  - contains(null)할 경우 List.of로 생성되는 불변 리스트가 null을 허용하지 않는 리스트이므로 NPE가 발생한다.


---
{: data-content="footnotes"}
