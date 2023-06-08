---
layout: post
author: Hyejin Kim
tags: [java, list, array]
---

### Cracking the Code Interview 67페이지
> ArrayList는 배열로 구현되어 있다. 배열의 용량이 꽉 찼을 때, ArrayList 클래스는 기존보다 크기가 두 배 더 큰 배열을 만든 뒤, 이전 배열의 모든 원소를 새 배열로 복사한다.


### 정말 그럴까?
``` java
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// 그럼 elementData는?

transient Object[] elementData; // non-private to simplify nested class access

// DEFAULTCAPACITY_EMPTY_ELEMENTDATA 는?

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

```
	- 일단 new ArrayList<>() 하면 Object[] 인 elementData가 초기화되고, 배열에 값이 저장되므로
> ArrayList는 배열로 구현되어 있다. (O)

### 배열의 용량이 꽉 찼을 때
``` java
public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
}

// 내부적으로 사용하는 add (private)
private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}

// 배열의 용량이 꽉 찼다면 grow(). 그럼 grow() 는 뭘까?
private Object[] grow() {
    return grow(size + 1);
}

private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length;
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                minCapacity - oldCapacity, /* minimum growth */
                oldCapacity >> 1           /* preferred growth */);
        return elementData = Arrays.copyOf(elementData, newCapacity);
    } else {
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
}

```

- minimum growth = 1
- preferred growth = oldCapacity >> 1 = 기존 배열 크기/2 
- 각 jdk의 세부적인 ArrayList의 구현에 따라 다를 수도 있음 
	- Oracle OpenJDK 17에서는 (배열 크기가 매우 큰 일부 케이스를 제외하면) 기존 크기의 1.5배 크기 배열을 만들어서 새 배열로 복사하게 된다.

---
{: data-content="footnotes"}
