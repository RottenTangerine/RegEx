## 常见问题

正则表达式对于某些应用程序来说是一个强大的工具，但在某些方面，它们的行为并不直观，有时它们的行为方式与你的预期不同。 本节将指出一些最常见的陷阱。

### 使用字符串方法

有时使用 `re` 模块是一个错误。 如果你匹配固定字符串或单个字符类，并且你没有使用任何 `re` 功能，例如 `IGNORECASE` 标志，那么正则表达式的全部功能可能不是必需的。 字符串有几种方法可以使用固定字符串执行操作，它们通常要快得多，因为实现是一个针对此目的而优化的单个小 C 循环，而不是大型、更通用的正则表达式引擎。

一个例子可能是用另一个固定字符串替换一个固定字符串；例如，你可以用 `deed` 替换 `word` 。 `re.sub 看起来像是用于此的函数，但请考虑 replace() 方法。 注意 `replace()` 也会替换单词里面的 `word` ，把 `swordfish` 变成 `sdeedfish` ，但简单的正则 `word` 也会这样做。 （为了避免对单词的部分进行替换，模式必须是 `\bword\b`，以便要求 `word` 在任何一方都有一个单词边界。这使得工作超出了 `replace()` 的能力。）

另一个常见任务是从字符串中删除单个字符的每个匹配项或将其替换为另一个字符。 你可以用 `re.sub('\n', ' ', S)` 之类的东西来做这件事，但是 [`translate()`](https://docs.python.org/zh-cn/3/library/stdtypes.html#str.translate) 能够完成这两项任务，并且比任何正则表达式都快。

简而言之，在转向 `re` 模块之前，请考虑是否可以使用更快更简单的字符串方法解决问题。

### match() 和 search()

有时你会被诱惑继续使用 `re.match()`，只需在你的正则前面添加 `.*` 。抵制这种诱惑并使用 `re.search()` 代替。 正则表达式编译器对正则进行一些分析，以加快寻找匹配的过程。 其中一个分析可以确定匹配的第一个特征必须是什么；例如，以 `Crow` 开头的模式必须与 `'C'` 匹配。 分析让引擎快速扫描字符串，寻找起始字符，只在找到 `'C'` 时尝试完全匹配。

添加 `.*` 会使这个优化失效，需要扫描到字符串的末尾，然后回溯以找到正则的其余部分的匹配。 使用 `re.search()` 代替。

### 处理html

（请注意，使用正则表达式解析 HTML 或 XML 很痛苦。快而脏的模式将处理常见情况，但 HTML 和 XML 有特殊情况会破坏明显的正则表达式；当你编写正则表达式处理所有可能的情况时，模式将非常复杂。使用 HTML 或 XML 解析器模块来执行此类任务。）

### 使用 re.VERBOSE

到目前为止，你可能已经注意到正则表达式是一种非常紧凑的表示法，但它们并不是非常易读。 具有中等复杂度的正则可能会成为反斜杠、括号和元字符的冗长集合，使其难以阅读和理解。

对于这样的正则，在编译正则表达式时指定 [re.VERBOSE](https://docs.python.org/zh-cn/3/library/re.html#re.VERBOSE) 标志可能会有所帮助，因为它允许你更清楚地格式化正则表达式。

`re.VERBOSE` 标志有几种效果。 正则表达式中的 *不是* 在字符类中的空格将被忽略。 这意味着表达式如 `dog | cat` 等同于不太可读的 `dog|cat` ，但 `[a b]` 仍将匹配字符 `'a'` 、 `'b'` 或空格。 此外，你还可以在正则中放置注释；注释从 `#` 字符扩展到下一个换行符。 当与三引号字符串一起使用时，这使正则的格式更加整齐:

这更具有可读性:

```python
pat = re.compile(r"\s*(?P<header>[^:]+)\s*:(?P<value>.*?)\s*$")

pat = re.compile(r"""
 \s*                 # Skip leading whitespace
 (?P<header>[^:]+)   # Header name
 \s* :               # Whitespace, and a colon
 (?P<value>.*?)      # The header's value -- *? used to
                     # lose the following trailing whitespace
 \s*$                # Trailing whitespace to end-of-line
""", re.VERBOSE)
```

