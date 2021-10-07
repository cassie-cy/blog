# 移动端页面高度 100%， DIV 撑满剩余高度

### 实现的效果

![实现效果图](/assets/flex.jpeg)

```html
<div className="main">
  <div className="nav">nav</div>
  <div className="content">content</div>
</div>
```

### 有如下几种情况

#### 1. 顶部 nav 高度固定

方法一：content 使用 position: absolute 绝对定位

```css
.main {
  display: relative;
  width: 100%;
  height: 100%; // 注意使用百分比时需要将 html 和 body 的高度同时设置为 100%，否则高度不生效
  .nav {
    width: 100%;
    height: 0.5rem;
  }
  .content {
    display: absolute;
    top: 0.5rem;
    left: 0;
    right: 0;
    bottom: 0;
  }
}
```

方法二： calc 计算 css 值

```css
.main {
  width: 100%;
  height: 100vh;
  .nav {
    width: 100%;
    height: 0.5rem;
  }
  .content {
    width: 100%;
    height: calc(100% - 0.5rem);
  }
}
```

方法三：使用 padding-top 或 margin-top 将头部撑开

```css
.main {
  width: 100%;
  height: 100vh;
  .nav {
    width: 100%;
    height: 0.5rem;
  }
  .content {
    width: 100%;
    height: 100%;
    padding-top: 0.5rem; // 或 margin-top： 0.5rem;
  }
}
```

#### 2. 顶部 nav 高度不固定

方法一：使用 flex 布局

```css
.main {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  .nav {
    width: 100%;
  }
  .content {
    flex: 1;
  }
}
```

方法二：nav 左浮动 float: left, content 设置高度 100%

```css
.main {
  width: 100%;
  height: 100vh;
  .nav {
    width: 100%;
    float: left;
  }
  .content {
    height: 100%;
  }
}
```
