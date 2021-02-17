## 继承关系

ArrayList是List的子类，实现了List,RandomAccess,Cloneable,Serializable

继承自AbstractList。



## 属性和方法

这里初始的大小为10，不过不会在你声明的时候给你创建，当你添加第一个元素的时候会给你创建出这10个空间







## AbstractCollection

如果不是List接口下的类，那么会调用到这边

当调用toArray()的时候，会调用到这边

```java
    public Object[] toArray() {
        // Estimate size of array; be prepared to see more or fewer elements
        Object[] r = new Object[size()];
        Iterator<E> it = iterator();
        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) // fewer elements than expected
                return Arrays.copyOf(r, i);
            r[i] = it.next();
        }
        return it.hasNext() ? finishToArray(r, it) : r;
    }
```

对于这些经常调用的方法，我们可以将对参数校验的方法抽离出来，形成单独的方法

```java
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```



