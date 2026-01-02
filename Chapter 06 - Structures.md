<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [6.1 Basic of structures](#61-basic-of-structures)
- [6.2 Structures and functions](#62-structures-and-functions)
- [6.3 Arrays of structures](#63-arrays-of-structures)
- [6.4 Pointers to structures](#64-pointers-to-structures)
- [6.5 Self-referential structures](#65-self-referential-structures)
- [Exercises](#exercises)
    - [Exercise 6-1](#exercise-6-1)

<!-- markdown-toc end -->

# 6.1 Basic of structures

```c
struct point { // <== "point" is called "structure tag"
    int x;     // <== variables in it are called "members"
    int y;
}
```

The keyword `struct` introduces a structure declaration, which is **a list of declarations enclosed in braces**. An optional name called a *structure tag* may follow the word `struct`. The tag names this kind of structure, and can be used subsequently as a shorthand for the part of the declaration in braces.

The variables named in a structure are called *members*.

一个 `struct` 的名字叫作 tag，`struct` 里面的变量叫作 members。

A `struct` declaration defines a type. The right brace that terminates the list of members may be followed by a list of variables, just as for any basic type. That is

```c
struct { ... } x, y, z;
```

is syntactically analogous to

```c
int x, y, z;
```

in the sense that each statement declares `x`, `y`, `z` to be variables of the named type and causes space to be set aside for them.

---

A structure delcaration that is not followed by a list of variables reserves no storage; it merely describes a template or the shape of a structure. If the declaration is tagged, however, the tag can be used later in the definitions of instances of the structure. For example, given the declaration of `point`,

```c
struct point pt;
```

**defines** a variable `pt` which is a structure of type `struct point`.

---

Structures can be nested.

```c
struct rect {
    struct point pt1;
    struct point pt2;
}
```

# 6.2 Structures and functions

The only legal operations on a structure are copying it or assigning to it as a unit, taking its address with `&`, and accessing its members. Copy and assignment include passing arguments to functions and returning values from functions as well.

---

If a large structure is to be passed to a function, it's generally more efficient to pass a pointer than to copy the whole structure. Structure pointers are just like pointers to ordinary variables.

```c
struct point *pp;
```

says that `pp` is a pointer to a structure of type `struct point`.

If `pp` points to a `point` struct, `*pp` is the structure, and `(*pp).x` and `(*pp).y` are the memebers. To use `pp`, we might write, for example

```c
struct point origin, *pp;

pp = &origin;
printf("origin is (%d, %d)\n", (*pp).x, (*pp).y);
```

The parentheses are necessary in `(*pp).x` because the precedence of the structure member operator `.` is higher than `*`. The expression `*pp.x` means `*(pp.x)`, which is illegal here because `x` is not a pointer.

---

Pointers to structures are so frequently used that an alternative notation is provided as a shorthand. If `p` is a pointer to a structure, then

```c
p->member-of-structure
```

refers to the particular member. So we could write instead

```c
printf("origin is (%d, %d)\n", pp->x, pp->y);
```

Both `.` and `->` associate from left to right, so if we have

```c
struct rect r, *rp = &r;
```

then these four expressions are equivalent:

```c
r.pt1.x
rp->pt1.x
(r.pt1).x
(rp->pt1).x
```

`pp->x` 是 `(*pp).x` 的快捷写法。后者先解引用 `pp` 这个指针，然后 access 到 `x` 这个 member。

---

The structure operators `.` and `->`, together with `()` for function calls and `[]` for subscripts, are at the top of the precedence hierarchy and thus bind very tightly.

```c
struct {
    int len;
    char *str;
} *p;
```

then

```c
++p->len
```

increments `len`, not `p`, because the implied parenthesization is `++(p->len)`. Parentheses can be used to alter the binding: `(++p)->len` increments `p` before accessing `len`, and `(p++)->len` increments `p` afterward. (This last set of parentheses is unnecessary.)

In the same way, `*p->str` fetches whatever `str` points to; `*p->str++` increments `str` after accessing whatever it points to (just like `*s++`); `(*p->str)++` increments whatever `str` points to; and `*p++->str` increments `p` after accessing whatever `str` points to.

# 6.3 Arrays of structures

```c
struct key {
    char *word;
    int count;
} keytab[NKEYS];
```

declares a structure type `key`, defines an array `keytab` of structures of this type, and sets aside storage for them. Each element of the array is a structure. This could also be written

```c
struct key {
    char *word;
    int count;
};

struct key keytab[NKEYS];
```

---

C provides a compile-time unary operator called `sizeof` that can be used to compute the size of *any* object. The expressions

```c
sizeof object
```

and

```c
sizeof(type name)
```

yield an integer equal to the size of the specified object or type in bytes. An object can be a variable or array or structure. A type name can be the name of a basic type like `int` or `double`, or a derived type like a structure or a pointer.

---

```c
// getword: get next word or character from input
int getword(char *word, int lim)
{
    int c, getch(void);
    void ungetch(int);
    char *w = word;

    while (isspace(c = getch()))
        ;
    if (c != EOF)
        *w++ = c;
    if (!isalpha(c))
    {
        *w = '\0';
        return c;
    }
    for (; --lim > 0; w++)
        if (!isalnum(*w = getch()))
        {
            ungetch(*w);
            break;
        }
    *w = '\0';
    return word[0];
}
```

`getword` fetches the next "word" from the input, where a word is either a string of letters and digits beginning with a letter, or a single non-white space character.

`getword` 会 pickup 的：

1. a string of letters;
2. digits beginning with a letter;
3. single non-white space character.

---

The function value is the first character of the word, or `EOF` for end of file, or the character itself if it is not alphabetic.

`getword` 可能返回的：

1. first character of the word;
2. `EOF`;
3. character if it's not alphabetic (for example digits, and symbols: `+`, `&`...).

---

上面可能 `getword` 可能返回第 2 第 3 点，具体会从以下代码返回：

```c
    if (!isalpha(c))
    {
        *w = '\0';
        return c;
    }
```

第 1 点会从最后一行代码返回：`return word[0];`。

---

这个函数一个一个的从 input 当中提取关键词。每运行一次，提取一个关键词。具体工作原理如下：

第一个 while loop 排除掉行首的空格。如果 `c` 不等于 `EOF`，那么把 `c` 赋值给此时 `w` 所在的位置，替换掉原本可能存在的值。然后把 `w` 这个指针进一位，指向下一个元素。再检查 `c` 是否是字母，如果不是，那么将此时 `w` 指向的值设为 null character，然后直接返回 `c` 值。

---

`getword` also uses `isspace` to skip white space, `isalpha` to identify letters, and `isalnum` to identify letters and digits.

---

```c
// count C keywords
int main(void)
{
    int n;
    char word[MAXWORD];

    while (getword(word, MAXWORD) != EOF)
        if (isalpha(word[0]))
            if ((n = binsearch(word, keytab, NKEYS)) >= 0)
                keytab[n].count++;

    for (n = 0; n < NKEYS; n++)
        if (keytab[n].count > 0)
            printf("%4d %s\n", keytab[n].count, keytab[n].word);

    return 0;
}
```

`main` 函数当中，while loop 用来 count，for loop 用来 print。在 while loop 中，`getword` 有可能提取到各种字符或字符串，包括那些非 C 关键词的字符或字符串。但在 `if` 的检查中，只有第一个字符是字母的字符串才可以通过。通过这个 `if` 只会进入第二个 `if` 的检查。这里在做一个 binary search，它会在 `keytab` 这个装了很多个叫作 `key` 的 structure 的数组当中，搜寻 `word`。如果在其中找到了的话，那么把 `keytab` 中对应的 `count` 加 1。如果没有找到，那么跳过。

跳过 whitespace 之后，做一个 `if` 检查，如果该字符不是 `EOF`，把第一个字符赋值给 `w`。接着做第二个检查，如果这个字符不是字母（C 的关键词需以字母开头——不允许数字开头的关键词），那么准备把这个字符（可能是 symbol 和 digit）直接返回。

# 6.4 Pointers to structures

```c
struct key *binsearch(char *word, struct key *tab, int n)
{
    int cond;
    struct key *low = &tab[0];  // the address of the 1st item in array tab
    struct key *high = &tab[n]; // the address of the n-st item in array tab
    struct key *mid;
...
```

The problem is that `&tab[-1]` and `&tab[n]` are both outside the limits of the array `tab`. The former is strictly illegal, and it's illegal to dereference the latter. The language definition does guarantee, however, that pointer arithmetic that involves the first element beyond the end of an array (that is, `&tab[n]`) will work correctly.

`tab[-1]` 和 `tab[n]` 都是超出了 tab 这个数组的，但是 C 会保证你如果刚好超出 tab 的限制，刚好超出一个位置的话，那么对于涉及 `tab[n]` 的加减，还是会正常工作的。

---

根据我的实测，解引用 `tab[-1]` 会显示 warning（实际上这个操作在 C 中应该说也是不合法的）。不像 Python，使用 `tab[-1]` 可以直接 access 到最后一个元素。

```c
#include <stdio.h>

struct key
{
    char *word;
    int count;
} keytab[] = {
    "auto", 1,
    "break", 100,
};

int main()
{
    struct key *p = &keytab[-1];
    printf("%d\n", p->count);
}
```

相反，C 会告诉你，你使用 -1 是在 access 数组边界前面的元素。报错如下：

```sh
$ gcc tmp.c -o tmp && ./tmp
tmp.c:14:22: warning: array index -1 is before the beginning of the array [-Warray-bounds]
    struct key *p = &keytab[-1];
                     ^      ~~
tmp.c:3:1: note: array 'keytab' declared here
struct key
^
1 warning generated.
0
```

---

因为是 0 base 的，所以在 `keytab` 这个数组中，想要获取第一个元素，使用 `keytab[0]`。0 => 第一个元素。所以 `keytab[2]` 实际上是超出了 `keytab` 这个数组的边界的。但当你 access `keytab[2]` 时，却不会像上面 `keytab[-1]` 一样直接显示 warning。

```c
int main()
{
    struct key *p = &keytab[2];
    printf("%d\n", p->count);
}
```

而是只输出了一个 0：

```sh
$ gcc tmp.c -o tmp && ./tmp
0
```

如果我们更进一步，`keytab[3]` 的话，那么 warning 又会显示出来：

```c
int main()
{
    struct key *p = &keytab[3];
    printf("%d\n", p->count);
}
```

输出：

```sh
$ gcc tmp.c -o tmp && ./tmp
tmp.c:14:22: warning: array index 3 is past the end of the array (which contains 2 elements) [-Warray-bounds]
    struct key *p = &keytab[3];
                     ^      ~
tmp.c:3:1: note: array 'keytab' declared here
struct key
^
1 warning generated.
0
```

---

```c
int main(void)
{
...
    for (p = keytab; p < keytab + NKEYS; p++)
...
}
```

If `p` is a pointer to a structure, arithmetic on `p` takes into account the size of the structure, so `p++` increments `p` by the correct amount to get the next element of the array of structures, and the rest stops the loop at the right time.

Don't assume, however, that the size of a structure is the sum of the sizes of its members. Because of alignment requirements for different objects, there may be unnamed "holes" in a structure. Thus, for instance, if a `char` is one byte and a `int` four bytes, the structure

```c
struct {
    char c;
    int i;
};
```

might well require require eight bytes, not five. The `sizeof` operator returns the proper value.

# 6.5 Self-referential structures

Suppose we want to handle the more general problem of counting the occurrences of *all* the words in some input. Since the list of words isn't known in advance, we can't conveniently sort it and use a binary search. Yet we can't do a linear search for each word as it arrives, to see if it's already been seen; the program would take too long. (More precisely, its running time is likely to grow quadratically with the number of input words.) How can we organize the data to cope efficiently with a list of arbitrary words?

假设我们要处理一个更普遍的问题，即计算某个输入中所有单词的出现次数。由于事先并不知道单词列表，我们无法方便地对其进行排序，也无法使用二进制搜索。但我们也不能在每个单词出现时对其进行线性搜索，以确定是否已经出现过；这样程序的运行时间会太长。(更确切地说，程序的运行时间可能会随着输入单词数量的增加而呈二次曲线增长）。我们该如何组织数据，以高效处理任意单词列表呢？

一种解决方案是将迄今为止看到的单词集始终保持排序，在每个单词到达时将其按顺序排列到适当的位置。不过，这不应该通过在线性数组中移位单词来实现——那样也会耗时太长。相反，我们将使用一种名为二叉树 *binary tree* 的数据结构。

---

```c
struct tnode *addtree(struct tnode *p, char *w)
{
    int cond;

    if (p == NULL) // a new word has arrived
    {
        p = talloc(); // make a new node
        p->word = strdup(w);
        p->count = 1;
        p->left = p->right = NULL;
    }
    else if ((cond = strcmp(w, p->word)) == 0)
        p->count++;    // repeat word
    else if (cond < 0) // less than into left subtree
        p->left = addtree(p->left, w);
    else // greater than into right subtree
        p->right = addtree(p->right, w);

    return p;
}
```

The function `addtree` is recursive. A word is presented by `main` to the top level (the root) of the tree. At each stage, that word is compared to the word already stored at the node, and is percolated down to either the left or right subtree by a recursive call to `addtree`. Eventually the word either matches something already in the tree (in which case the count is incremeneted), or a null pointer in encountered, indicating that a node must be created and added to the tree. If a new node is created, `addtree` returns a pointer to it, which is installed in the parent node.

# Exercises

## Exercise 6-1

Our version of `getword` does not properly handle underscores, string constants, comments, or preprocessor control lines. Write a better version.

1. 把下划线当作词的一部分，而不是词与词之间的分界线。
2. 目前，string constants 被分解成好几块分别处理了，修改后的函数应完整保留字符串文字。
3. 跳过 comments，不处理。
4. 把 preprocessor directives 视为一个整体进行处理。
