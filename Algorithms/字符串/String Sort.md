[toc]

# 字符串排序

## 字母表

```java
public class Alphabet {

    /**
     *  The binary alphabet { 0, 1 }.
     */
    public static final Alphabet BINARY = new Alphabet("01");

    /**
     *  The octal alphabet { 0, 1, 2, 3, 4, 5, 6, 7 }.
     */
    public static final Alphabet OCTAL = new Alphabet("01234567");

    /**
     *  The decimal alphabet { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 }.
     */
    public static final Alphabet DECIMAL = new Alphabet("0123456789");

    /**
     *  The hexadecimal alphabet { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C, D, E, F }.
     */
    public static final Alphabet HEXADECIMAL = new Alphabet("0123456789ABCDEF");

    /**
     *  The DNA alphabet { A, C, T, G }.
     */
    public static final Alphabet DNA = new Alphabet("ACGT");

    /**
     *  The lowercase alphabet { a, b, c, ..., z }.
     */
    public static final Alphabet LOWERCASE = new Alphabet("abcdefghijklmnopqrstuvwxyz");

    /**
     *  The uppercase alphabet { A, B, C, ..., Z }.
     */

    public static final Alphabet UPPERCASE = new Alphabet("ABCDEFGHIJKLMNOPQRSTUVWXYZ");

    /**
     *  The protein alphabet { A, C, D, E, F, G, H, I, K, L, M, N, P, Q, R, S, T, V, W, Y }.
     */
    public static final Alphabet PROTEIN = new Alphabet("ACDEFGHIKLMNPQRSTVWY");

    /**
     *  The base-64 alphabet (64 characters).
     */
    public static final Alphabet BASE64 = new Alphabet("ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/");

    /**
     *  The ASCII alphabet (0-127).
     */
    public static final Alphabet ASCII = new Alphabet(128);

    /**
     *  The extended ASCII alphabet (0-255).
     */
    public static final Alphabet EXTENDED_ASCII = new Alphabet(256);

    /**
     *  The Unicode 16 alphabet (0-65,535).
     */
    public static final Alphabet UNICODE16      = new Alphabet(65536);


    private char[] alphabet;     // the characters in the alphabet
    private int[] inverse;       // indices
    private final int R;         // the radix of the alphabet

    /**
     * Initializes a new alphabet from the given set of characters.
     *
     * @param alpha the set of characters
     */
    public Alphabet(String alpha) {

        // check that alphabet contains no duplicate chars
        boolean[] unicode = new boolean[Character.MAX_VALUE];
        for (int i = 0; i < alpha.length(); i++) {
            char c = alpha.charAt(i);
            if (unicode[c])
                throw new IllegalArgumentException("Illegal alphabet: repeated character = '" + c + "'");
            unicode[c] = true;
        }

        alphabet = alpha.toCharArray();
        R = alpha.length();
        inverse = new int[Character.MAX_VALUE];
        for (int i = 0; i < inverse.length; i++)
            inverse[i] = -1;

        // can't use char since R can be as big as 65,536
        for (int c = 0; c < R; c++)
            inverse[alphabet[c]] = c;
    }

    /**
     * Initializes a new alphabet using characters 0 through R-1.
     *
     * @param radix the number of characters in the alphabet (the radix R)
     */
    private Alphabet(int radix) {
        this.R = radix;
        alphabet = new char[R];
        inverse = new int[R];

        // can't use char since R can be as big as 65,536
        for (int i = 0; i < R; i++)
            alphabet[i] = (char) i;
        for (int i = 0; i < R; i++)
            inverse[i] = i;
    }

    /**
     * Initializes a new alphabet using characters 0 through 255.
     */
    public Alphabet() {
        this(256);
    }

    /**
     * Returns true if the argument is a character in this alphabet.
     *
     * @param  c the character
     * @return {@code true} if {@code c} is a character in this alphabet;
     *         {@code false} otherwise
     */
    public boolean contains(char c) {
        return inverse[c] != -1;
    }

    /**
     * Returns the number of characters in this alphabet (the radix).
     * 
     * @return the number of characters in this alphabet
     * @deprecated Replaced by {@link #radix()}.
     */
    @Deprecated
    public int R() {
        return R;
    }

    /**
     * Returns the number of characters in this alphabet (the radix).
     * 
     * @return the number of characters in this alphabet
     */
    public int radix() {
        return R;
    }

    /**
     * Returns the binary logarithm of the number of characters in this alphabet.
     * 
     * @return the binary logarithm (rounded up) of the number of characters in this alphabet
     */
    public int lgR() {
        int lgR = 0;
        for (int t = R-1; t >= 1; t /= 2)
            lgR++;
        return lgR;
    }

    /**
     * Returns the index corresponding to the argument character.
     * 
     * @param  c the character
     * @return the index corresponding to the character {@code c}
     * @throws IllegalArgumentException unless {@code c} is a character in this alphabet
     */
    public int toIndex(char c) {
        if (c >= inverse.length || inverse[c] == -1) {
            throw new IllegalArgumentException("Character " + c + " not in alphabet");
        }
        return inverse[c];
    }

    /**
     * Returns the indices corresponding to the argument characters.
     * 
     * @param  s the characters
     * @return the indices corresponding to the characters {@code s}
     * @throws IllegalArgumentException unless every character in {@code s}
     *         is a character in this alphabet
     */
    public int[] toIndices(String s) {
        char[] source = s.toCharArray();
        int[] target  = new int[s.length()];
        for (int i = 0; i < source.length; i++)
            target[i] = toIndex(source[i]);
        return target;
    }

    /**
     * Returns the character corresponding to the argument index.
     * 
     * @param  index the index
     * @return the character corresponding to the index {@code index}
     * @throws IllegalArgumentException unless {@code 0 <= index < R}
     */
    public char toChar(int index) {
        if (index < 0 || index >= R) {
            throw new IllegalArgumentException("index must be between 0 and " + R + ": " + index);
        }
        return alphabet[index];
    }

    /**
     * Returns the characters corresponding to the argument indices.
     * 
     * @param  indices the indices
     * @return the characters corresponding to the indices {@code indices}
     * @throws IllegalArgumentException unless {@code 0 < indices[i] < R}
     *         for every {@code i}
     */
    public String toChars(int[] indices) {
        StringBuilder s = new StringBuilder(indices.length);
        for (int i = 0; i < indices.length; i++)
            s.append(toChar(indices[i]));
        return s.toString();
    }

}
```

## 键索引计数法

适用于小整数键。只要R在N的一个常数因子范围内，它是一个线性时间级别的排序方法。

学生被分为若干组，标号为1，2，3，将全班同学按组分类。假设a[]中的每个元素都保存了一个名字和组号。a[i].key()返回组号。

1.  频率统计

    使用int数组count[]计算每个键出现的频率。对于数组中的每个元素，都使用它的键访问count[]中的相应元素并将其加一。键为r，count[r+1]加一。

2.  将频率转换为索引

    使用count[]计算每个键在排序结果中的起始索引位置。任何给定的键的起始索引均为所有较小的键所对应的出现频率之和。

3.  数据分类

    每个元素的位置是由它的键对应的count[]值决定的，移动后对应元素加一，以保证count[r]总是为下一个键位r的元素的索引位置。是稳定的，因为相对位置没有变

4.  回写

```java
int N = a.length;
String[] aux = new String[N];
int[] count = new int[R+1];

//计算出现频率
for (int i = 0; i < N; i++)
    count[a[i].key() + 1]++;
//将频率转换位索引
for (int r = 0; r < R; R++)
    count[r+1] = count[r]
//将元素分类
for (int i = 0; i < N; i++)
    aux[count[a[i].key()]++] = a[i]
//回写
for (int i = 0; i < N; i++)
    a[i] = aux[i];
```

## 低位优先排序

如果字符串长度均为W，从右向左以每个位置的字符为键，用键索引计数法将字符串排序W遍。因为键索引计数法是稳定的，低位优先排序也是稳定的。

```java
    public static void sort(String[] a, int w) {
        int n = a.length;
        int R = 256;   // extend ASCII alphabet size
        String[] aux = new String[n];

        for (int d = w-1; d >= 0; d--) {
            // sort by key-indexed counting on dth character

            // compute frequency counts
            int[] count = new int[R+1];
            for (int i = 0; i < n; i++)
                count[a[i].charAt(d) + 1]++;

            // compute cumulates
            for (int r = 0; r < R; r++)
                count[r+1] += count[r];

            // move data
            for (int i = 0; i < n; i++)
                aux[count[a[i].charAt(d)]++] = a[i];

            // copy back
            for (int i = 0; i < n; i++)
                a[i] = aux[i];
        }
```

## 高位优先排序

字符串长度不相同时，考虑从左向右遍历所有字符。首先用键索引计数法将所有字符串按照字母排序，然后（递归地）再将每个首字母所对应地子数组排序。（忽略首字母，因为每一类中的所有字符串的首字母都是相同的）。

要注意字符串达到末尾的情况，方法是将所有字符都已经检查过的字符串所在的子数组排在所有子数组的前面。使用charAt来将字符串中字符索引转化为数组索引，指定位置超过字符串末尾时返回-1。再将所有返回值加一。所以count要是R+2，0代表字符串末尾，1代表字母表第一个字符

```java
public class MSD {
    private static final int R             = 256;   // extended ASCII alphabet size
    private static final int CUTOFF        =  15;   // cutoff to insertion sort

    public static void sort(String[] a) {
        int n = a.length;
        String[] aux = new String[n];
        sort(a, 0, n-1, 0, aux);
    }

    // return dth character of s, -1 if d = length of string
    private static int charAt(String s, int d) {
        assert d >= 0 && d <= s.length();
        if (d == s.length()) return -1;
        return s.charAt(d);
    }

    // sort from a[lo] to a[hi], starting at the dth character
    private static void sort(String[] a, int lo, int hi, int d, String[] aux) {

        // cutoff to insertion sort for small subarrays
        if (hi <= lo + CUTOFF) {
            insertion(a, lo, hi, d);
            return;
        }

        // compute frequency counts
        int[] count = new int[R+2];
        for (int i = lo; i <= hi; i++) {
            int c = charAt(a[i], d);
            count[c+2]++;
        }

        // transform counts to indicies
        for (int r = 0; r < R+1; r++)
            count[r+1] += count[r];

        // distribute
        for (int i = lo; i <= hi; i++) {
            int c = charAt(a[i], d);
            aux[count[c+1]++] = a[i];
        }

        // copy back
        for (int i = lo; i <= hi; i++) 
            a[i] = aux[i - lo];

        // recursively sort for each character (excludes sentinel -1)
        for (int r = 0; r < R; r++)
            sort(a, lo + count[r], lo + count[r+1] - 1, d+1, aux);
    }
}

```

## 三向字符串快速排序

```java
    public static void sort(String[] a) {
        sort(a, 0, a.length-1, 0);
    }

    // return the dth character of s, -1 if d = length of s
    private static int charAt(String s, int d) { 
        assert d >= 0 && d <= s.length();
        if (d == s.length()) return -1;
        return s.charAt(d);
    }

    // 3-way string quicksort a[lo..hi] starting at dth character
    private static void sort(String[] a, int lo, int hi, int d) { 

        // cutoff to insertion sort for small subarrays
        if (hi <= lo + CUTOFF) {
            insertion(a, lo, hi, d);
            return;
        }

        int lt = lo, gt = hi;
        int v = charAt(a[lo], d);
        int i = lo + 1;
        while (i <= gt) {
            int t = charAt(a[i], d);
            if      (t < v) exch(a, lt++, i++);
            else if (t > v) exch(a, i, gt--);
            else              i++;
        }

        // a[lo..lt-1] < v = a[lt..gt] < a[gt+1..hi]. 
        sort(a, lo, lt-1, d);
        if (v >= 0) sort(a, lt, gt, d+1);
        sort(a, gt+1, hi, d);
    }
```

