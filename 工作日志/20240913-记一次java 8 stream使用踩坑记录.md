# 20240913

记一次java 8 stream使用踩坑记录。

先看下面这段代码：

```java
public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("1");
        list.add("2");
        list.add("3");
  
        Optional<String> any = list.stream().findAny();
        Optional<String> first = list.stream().findFirst();
  
        System.out.println(any.get().equals(first.get())); // result is true!
    }
```

## 疑惑：为什么findAny()和findFirst()获取的结果是相同的呢？

查看源码：

```java
    /**
     * Returns an {@link Optional} describing some element of the stream, or an
     * empty {@code Optional} if the stream is empty.
     *
     * <p>This is a <a href="package-summary.html#StreamOps">short-circuiting
     * terminal operation</a>.
     *
     * <p>The behavior of this operation is explicitly nondeterministic; it is
     * free to select any element in the stream.  This is to allow for maximal
     * performance in parallel operations; the cost is that multiple invocations
     * on the same source may not return the same result.  (If a stable result
     * is desired, use {@link #findFirst()} instead.)
     *
     * @return an {@code Optional} describing some element of this stream, or an
     * empty {@code Optional} if the stream is empty
     * @throws NullPointerException if the element selected is null
     * @see #findFirst()
     */
    Optional<T> findAny();

```

源码注释的意思是此操作的行为明显是不确定的；可以自由选择流中的任何元素。这是为了在并行操作中实现最大性能；代价是对同一源的多次调用可能不会返回相同的结果。如果需要稳定的结果，请改用findFirst()。

上面测试代码使用的串行流，所以findAny()和findFirst()获取的结果是一样，再看看两个方法的源码区别：

```java
    @Override
    public final Optional<P_OUT> findFirst() {
        return evaluate(FindOps.makeRef(true));
    }

    @Override
    public final Optional<P_OUT> findAny() {
        return evaluate(FindOps.makeRef(false));
    }
```

源码都使用同一个方法，只是参数不同。参数的作用是否必须取第一条。

## 总结

1. 在串行流中findAny()和findFirst()获取的结果是一样的。
2. 在并行流中findAny()和findFirst()获取的结果是不一样的，findAny()性能更高但是不保证获取第一条数据，是真正的any！！！

