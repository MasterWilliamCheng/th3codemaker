### 和 LinkedList的区别：

**ArrayList：**
- 底层是**动态数组，支持扩容，线程不安全**。
- 对于添加和删除方法，如果是添加到列表尾部，时间复杂度是O(1)；
- 如果是添加到指定位置i，时间复杂度就是O(n-i)，因为需要将后续数组的元素往后移动一位；
- 对于快速随机访问get(i),时间复杂度是O(1)。

**LinkedList:**
- **双向链表结构，线程不安全**。
- 对于添加和删除方法，如果是添加到列表尾部，因为是直接操作最后一个last节点，所以是O(1)；
- 如果是添加到指定位置i，因为需要先移动到链表的指定位置，所以时间复杂度是O(i)；
- 不支持快速随机访问，时间复杂度O(n)。

所以一般来说，因为支持快速随机访问，所以在查询元素方面，ArrayList有优势，相反的，在添加和删除元素方面，LinkedList不需要移动数据，所以更有优势。当然这不是绝对的，因为LinkedList也需要先遍历到指定位置（末尾节点除外），具体使用还是需要根据实际业务场景来综合考量。


----------


### 核心源码解析


```
public class ArrayList<E> {
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * 默认初始容量
     */

    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 用于指定大小为0的空实例时的数组实例.
     */

    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * 用于默认大小的空实例的共享空数组实例。
     * 将它与 EMPTY_ELEMENTDATA 区分开来，以了解添加第一个元素时要膨胀多少
     */

    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 存储ArrayList元素的数组缓冲区。
     *
     * ArrayList 的容量就是这个数组缓冲区的长度。任何带有
     * elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA 的
     * 空 ArrayList添加第一个元素时将扩展为 DEFAULT_CAPACITY 大小。
     */

    transient Object[] elementData;

    /**
     * ArrayList 的大小（它包含的元素数）
     */

    private int size;

    /**
     * 记录List对象被修改的次数，List对象每修改一次，modCount都会加1.
     */

    protected transient int modCount = 0;


    /**
     * 有参构造函数，构造一个具有指定初始容量的数组.
     */

    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                    initialCapacity);
        }
    }

    /**
     * 无参构造函数，构造一个初始容量为 10 的空数组（当添加第一个元素时才为10）
     *
     * 有参大小为0和无参的构造函数使用的空数组不一样，前者是EMPTY_ELEMENTDATA，后者是DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     */

    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }


    /**
     * 按照集合的迭代器返回的顺序构造一个包含指定集合元素的数组
     */

    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            //这里，例如当使用Arrays.asList来构建c时，c.toArray().getClass()的结果就不是Object[].class了，而是String[].class
            //如果是这种情况，就需要将非Object数组类型的elementData转成Object数据类型
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }


    /**
     * 将ArrayList实例的容量修剪为列表的当前大小。
     * 应用程序可以使用此操作来最小化实例的存储
     */

    public void trimToSize() {
        modCount++;
        //size是数组中实际元素的个数，而elementData.length代表的是数组的大小
        if (size < elementData.length) {
            elementData = (size == 0)
                    ? EMPTY_ELEMENTDATA
                    : Arrays.copyOf(elementData, size);
        //顺便提一下，Arrays.copyOf方法是创建一个大小为size的新数组，按照size和elementData.length中更小的单位复制elementData中的元素。
        //当然，如果传入的size大于elementData.length话，也就相当于扩容，反之则是裁剪
        }
    }

    /**
     * 扩容机制
     *
     * 增加实例的容量，如果必要，以确保它至少可以容纳元素的数量
     * 由最小容量参数指定
     * @param   minCapacity   所需最小容量
     */

    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
                // 这里要说明一下，只有在创建不指定大小的空实例时，扩容才会用到默认初始容量DEFAULT_CAPACITY
                ? 0 : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

    /**
     * 先判断数组缓冲区是不是没有添加过元素，如果是，则返回设置容量和默认初始容量的最大值
     * @param elementData
     * @param minCapacity
     * @return
     */
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // 如果要扩容的数组大小大于缓冲数组的大小，才可以执行扩容，否则不会扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * 要分配的数组的最大大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * 扩容机制具体实现
     */
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        // >> 1 意思是获取旧数组的一半长度，扩容相当于将数组长度扩展成原来的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //如果还是小于最小容量，则直接把最小容量作为新容量
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 按照新容量扩容
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    /**
     * 比较 minCapacity 和 MAX_ARRAY_SIZE，如果 minCapacity 大于最大容量，则新容量则为Integer.MAX_VALUE，
     * 否则，新容量大小则为 MAX_ARRAY_SIZE 即为 Integer.MAX_VALUE - 8
     */
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
                Integer.MAX_VALUE :
                MAX_ARRAY_SIZE;
    }

    /**
     * 返回此列表中的元素数
     *
     */
    public int size() {
        return size;
    }

    /**
     * 如果此列表不包含元素，则返回 true
     *
     */
    public boolean isEmpty() {
        return size == 0;
    }

    /**
     * 如果此列表包含指定的元素，则返回true
     */

    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }

    /**
     * 返回指定元素第一次出现的索引，如果此列表不包含该元素，则为 -1
     */

    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回指定元素最后一次出现的索引,如果此列表不包含该元素，则为 -1
     */

    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此实例的浅拷贝。 （元素本身不会被复制。）
     */

    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }

    /**
     * 以适当的顺序（从第一个元素到最后一个元素）返回一个包含此列表中所有元素的数组
     * 返回的数组将是“安全的”，因为列表实例没有对它的引用。（换句话说，这个方法分配了
     * 一个新数组），因此调用者可以自由地修改返回的数组。
     */

    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * 以适当的方式返回包含此列表中所有元素的数组序列（从第一个元素到最后一个元素）；
     * 返回的运行时类型数组是指定数组的数组；
     * 如果列表符合指定的数组，它在其中返回；
     * 否则，使用指定数组的运行时类型和大小分配一个新数组。
     *
     * 如果列表适合指定的数组并有剩余空间（即，数组的元素比列表多），
     * 紧跟在集合末尾之后的数组被设置为空。
     * （这在确定长度时很有用，如果调用者知道列表不包含任何空元素。）
     */

    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }


    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * 返回此列表中指定位置的元素
     */

    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    /**
     * 将此列表中指定位置的元素替换为指定的元素。
     */

    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

    /**
     * 将指定的元素附加到此列表的末尾。
     */

    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    /**
     * 在此指定位置插入指定元素。
     * 移动当前在该位置的元素（如果有）和右边的任何后续元素（在它们的索引上加一）。
     */

    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                size - index);
        elementData[index] = element;
        size++;
    }

    /**
     * 移除此列表中指定位置的元素。
     * 将任何后续元素向左移动（从它们的元素中减去一个）
     */

    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                    numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

    /**
     * 从此列表中删除第一次出现的指定元素（如果它存在）；
     * 如果列表不包含该元素，则为不变。
     */

    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    /**
     * 跳过边界检查且不执行的私有删除方法
     */

    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                    numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }

    /**
     * 从此列表中删除所有元素。
     * 该列表在调用返回后将会为空。
     */

    public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    /**
     * 将指定集合中的所有元素追加到末尾（按照它们指定集合的迭代器返回的顺序）。
     *
     */

    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 将指定集合中的所有元素插入到此列表，从指定位置开始。
     *
     */

    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                    numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 从此列表中删除索引介于两者之间的所有元素
     *
     */

    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    /**
     * 检查给定的索引是否在范围内
     */

    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * add 和 addAll 使用的 rangeCheck 版本.
     */

    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * 构造一个 IndexOutOfBoundsException 详细消息
     */

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    /**
     * 从此列表中删除指定集合的所有元素
     *
     */

    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, false);
    }

    /**
     * 仅保留此列表中包含在指定集合中的元素.
     *
     */

    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }

    private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                System.arraycopy(elementData, r,
                        elementData, w,
                        size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }

}
```
