# scraper

[![crates.io](https://img.shields.io/crates/v/scraper?color=dark-green)][crate]
[![downloads](https://img.shields.io/crates/d/scraper)][crate]
[![test](https://github.com/rust-scraper/scraper/actions/workflows/test.yml/badge.svg)][tests]

HTML 解析与 CSS 选择器查询库。

`scraper` 托管在 [crates.io][crate] 和 [GitHub][github] 上。

[crate]: https://crates.io/crates/scraper
[github]: https://github.com/rust-scraper/scraper
[tests]: https://github.com/rust-scraper/scraper/actions/workflows/test.yml

Scraper 封装了 Servo 的 `html5ever` 和 `selectors` 库，提供浏览器级别的 HTML 解析和查询能力。

## 示例

### 解析文档

```rust
use scraper::Html;

let html = r#"
    <!DOCTYPE html>
    <meta charset="utf-8">
    <title>Hello, world!</title>
    <h1 class="foo">Hello, <i>world!</i></h1>
"#;

let document = Html::parse_document(html);
```

### 解析片段

```rust
use scraper::Html;
let fragment = Html::parse_fragment("<h1>Hello, <i>world!</i></h1>");
```

### 解析选择器

```rust
use scraper::Selector;
let selector = Selector::parse("h1.foo").unwrap();
```

### 选择元素

```rust
use scraper::{Html, Selector};

let html = r#"
    <ul>
        <li>Foo</li>
        <li>Bar</li>
        <li>Baz</li>
    </ul>
"#;

let fragment = Html::parse_fragment(html);
let selector = Selector::parse("li").unwrap();

for element in fragment.select(&selector) {
    assert_eq!("li", element.value().name());
}
```

### 选择后代元素

```rust
use scraper::{Html, Selector};

let html = r#"
    <ul>
        <li>Foo</li>
        <li>Bar</li>
        <li>Baz</li>
    </ul>
"#;

let fragment = Html::parse_fragment(html);
let ul_selector = Selector::parse("ul").unwrap();
let li_selector = Selector::parse("li").unwrap();

let ul = fragment.select(&ul_selector).next().unwrap();
for element in ul.select(&li_selector) {
    assert_eq!("li", element.value().name());
}
```

### 获取元素属性

```rust
use scraper::{Html, Selector};

let fragment = Html::parse_fragment(r#"<input name="foo" value="bar">"#);
let selector = Selector::parse(r#"input[name="foo"]"#).unwrap();

let input = fragment.select(&selector).next().unwrap();
assert_eq!(Some("bar"), input.value().attr("value"));
```

### 序列化 HTML 和内部 HTML

```rust
use scraper::{Html, Selector};

let fragment = Html::parse_fragment("<h1>Hello, <i>world!</i></h1>");
let selector = Selector::parse("h1").unwrap();

let h1 = fragment.select(&selector).next().unwrap();

assert_eq!("<h1>Hello, <i>world!</i></h1>", h1.html());
assert_eq!("Hello, <i>world!</i>", h1.inner_html());
```

### 获取后代文本

```rust
use scraper::{Html, Selector};

let fragment = Html::parse_fragment("<h1>Hello, <i>world!</i></h1>");
let selector = Selector::parse("h1").unwrap();

let h1 = fragment.select(&selector).next().unwrap();
let text = h1.text().collect::<Vec<_>>();

assert_eq!(vec!["Hello, ", "world!"], text);
```

### 操作 DOM

```rust
use html5ever::tree_builder::TreeSink;
use scraper::{Html, Selector, HtmlTreeSink};

let html = "<html><body>hello<p class=\"hello\">REMOVE ME</p></body></html>";
let selector = Selector::parse(".hello").unwrap();
let mut document = Html::parse_document(html);
let node_ids: Vec<_> = document.select(&selector).map(|x| x.id()).collect();
let tree = HtmlTreeSink::new(document);
for id in node_ids {
    tree.remove_from_parent(&id);
}
let document = tree.finish();
assert_eq!(document.html(), "<html><head></head><body>hello</body></html>");
```

## 贡献指南

欢迎提交 Pull Request。如果您计划实现较大的改动（如不只是修复拼写错误、小 Bug 或小幅重构），请先提交 Issue 讨论。
