---
title: 【小程序】自实现md预览+文字打字机效果
time: 2025-05-19 11:31:43
categories: 小程序
---

## 实现预览效果

由于后端返回的是 md 格式的文本，由于文本中的标题、列表会和//n 同时出现，因此使用`marked`会无法正确识别标题、列表等语法

````js
// 统一处理行内格式：加粗、斜体、链接、行内代码
function parseInlineMarkdown(text) {
  return text
    .replace(/\*\*(.*?)\*\*/g, "<strong>$1</strong>") // 加粗
    .replace(/\*(.*?)\*/g, "<em>$1</em>") // 斜体 *
    .replace(/_(.*?)_/g, "<em>$1</em>") // 斜体 _
    .replace(/`(.*?)`/g, "<code>$1</code>") // 行内代码
    .replace(
      /$$([^$$]+)$$$([^$$]+)$$/g,
      '<a href="$2" target="_blank" rel="noopener noreferrer">$1</a>'
    ); // 链接
}

export function markdownToHTML(markdown) {
  // 处理转义换行符并分割文本
  const text: string = (markdown.replace(/&@#\$&$/, "") || "")
    .replace(/\\\\n/g, "\\n")
    .replace(/\\n/g, "\n")
    .replace(/\s\\S/g, "");
  console.log(markdown, text);
  const lines: string[] = text.split("\n");

  let html: string[] = [];
  let inList = false; // 跟踪列表状态
  let inCodeBlock = false; // 跟踪是否在代码块中
  let codeBlockContent: string[] = []; // 暂存代码块内容

  for (let line of lines) {
    const trimmedLine = line.replace(/\s\\S/g, " ").trim();

    // 判断是否是代码块开始或结束
    if (trimmedLine.startsWith("```")) {
      if (inCodeBlock) {
        // 如果已经在代码块中，表示代码块结束
        html.push(
          "<pre style=' white-space: pre-wrap;word-wrap: break-word;overflow-x: auto;max-width: 100%;'><code>" +
            codeBlockContent.join("\n") +
            "</code></pre>"
        );
        codeBlockContent = []; // 清空暂存内容
        inCodeBlock = false; // 退出代码块模式
      } else {
        // 如果不在代码块中，表示代码块开始
        inCodeBlock = true; // 进入代码块模式
      }
      continue; // 跳过后续逻辑，直接处理下一行
    }

    // 如果在代码块中，收集代码块内容
    if (inCodeBlock) {
      codeBlockContent.push(line);
      continue;
    }

    // 匹配 # 到 ###### 并提取层级
    const headingMatch = trimmedLine.match(/^(\#{1,6})\s+(.+)$/);
    if (headingMatch) {
      // 关闭前一个列表
      if (inList) {
        html.push("</ul>");
        inList = false;
      }

      const level = headingMatch[1].length; // 获取标题级别 1-6
      const content = headingMatch[2].replace(
        /\*\*(.*?)\*\*/g,
        "<strong>$1</strong>"
      ); // 处理加粗

      html.push(`<h${level}>${content}</h${level}>`);
    }
    // 处理横线 ---
    else if (trimmedLine === "---") {
      html.push("<hr>");
      continue; // 跳过后续逻辑，直接处理下一行
    }
    // 处理列表项 (-)
    else if (/^-/.test(trimmedLine)) {
      // 首次进入列表时添加<ul>
      if (!inList) {
        html.push("<ul>");
        inList = true;
      }
      // 处理列表项内容并转换加粗
      let content = trimmedLine
        .replace(/^-\s+/, "")
        .replace(/\*\*(.*?)\*\*/g, "<strong>$1</strong>");
      content = parseInlineMarkdown(content);
      html.push(`<li>${content}</li>`);
    }
    // 处理普通文本
    else {
      // 关闭已存在的列表
      if (inList) {
        html.push("</ul>");
        inList = false;
      }
      // 非空行转为段落
      if (trimmedLine !== "") {
        const content = parseInlineMarkdown(trimmedLine);
        if (content === "<br />") {
          html.push("<br />");
        } else {
          html.push(`<p>${content}</p>`);
        }
      }
    }
  }

  // 关闭最后一个列表
  if (inList) html.push("</ul>");

  return html.join("\n");
}
````

支持常用的几种语法：

- 标题
- 列表
- 代码块（行内/块状
- 加粗、斜体

其他的可以按照需求自行扩展

## 实现打字机效果

### 将 html 转为可遍历的节点数组

上一步是把 md 文本字符串转为 html 字符串，但需要实现打字机效果的话，需要将 html 转为可遍历的节点数组，以实现打字机效果。

```js
// 将HTML转换为可遍历的节点数组
const htmlToNodes = (html: string) => {
  const tagRegex = /(<\/?[^>]+>)/g;
  return html
    .split(tagRegex)
    .filter((chunk) => chunk.trim())
    .flatMap((chunk) => {
      if (chunk.startsWith("<")) return [chunk];
      return chunk.split("").map((char) => (char === " " ? "&nbsp;" : char));
    });
};
```

### 打字机效果

使用定时器去定时添加文字

父组件调用`stopTimer`可以停止文字继续显示

```js
const [currentIndex, setCurrentIndex] = useState(0);
// displayContent是需要显示的html节点
const [displayContent, setDisplayContent] = useState<string[]>([]);

useEffect(() => {
  timer = setInterval(() => {
    if (currentIndex < nodes.length) {
      setDisplayContent((prev) => [...prev, nodes[currentIndex]]);
      setCurrentIndex((prev) => prev + 1);
    } else {
      clearInterval(timer);
      timer = null;
      if (onStop && nodes.length) onStop(); // 触发 onStop 回调
    }
  }, 50);
  return () => {
    if (timer) clearInterval(timer);
  };
}, [nodes, currentIndex]);

useImperativeHandle(ref, () => ({
    stopTimer: () => {
        if (timer) {
        clearInterval(timer);
        }
    },
}));
```

```html
<RichText className="break-all leading-[56rpx]" nodes={displayContent.join("")}
/>
```

### 完整代码

```js
import { View, RichText } from "@tarojs/components";
import { markdownToHTML } from "@/hooks/useMd";
import { FC, PropsWithChildren, useState, useEffect, useMemo, useRef, useImperativeHandle, forwardRef } from "react";

type TAIAnswerProps = {
  text: string;
  onStop?: () => void;       // 新增 prop：定时器停止时通知父组件
};

// 使用 ref 暴露方法给父组件
export type AIAnswerRef = {
  stopTimer: () => void;
};

// 正确使用 forwardRef 并定义 ref 类型
const AIAnswer: FC<PropsWithChildren<TAIAnswerProps>> & { ref?: React.Ref<AIAnswerRef> } = forwardRef<
  AIAnswerRef,
  PropsWithChildren<TAIAnswerProps>
>(({ text, onStop }, ref) => {
  const [displayContent, setDisplayContent] = useState<string[]>([]);
  const [currentIndex, setCurrentIndex] = useState(0);
  const [htmlStr, setHtmlStr] = useState("");

  // 将HTML转换为可遍历的节点数组
  const htmlToNodes = (html: string) => {
    const tagRegex = /(<\/?[^>]+>)/g;
    return html
      .split(tagRegex)
      .filter((chunk) => chunk.trim())
      .flatMap((chunk) => {
        if (chunk.startsWith("<")) return [chunk];
        return chunk.split("").map((char) => (char === " " ? "&nbsp;" : char));
      });
  };

  // 使用useMemo缓存节点数组
  const nodes = useMemo(() => htmlToNodes(htmlStr), [htmlStr]);
  let timer = null
  useEffect(() => {
    console.log("nodes", nodes)
    timer = setInterval(() => {
      if (currentIndex < nodes.length) {
        setDisplayContent((prev) => [...prev, nodes[currentIndex]]);
        setCurrentIndex((prev) => prev + 1);
      } else {
        clearInterval(timer);
        timer = null;
        if (onStop && nodes.length) onStop(); // 触发 onStop 回调
      }
    }, 50);

    return () => {
      if (timer) clearInterval(timer);
    };
  }, [nodes, currentIndex]);

  useEffect(() => {
    text && setHtmlStr(markdownToHTML(text));
  }, [text]);

  useImperativeHandle(ref, () => ({
    stopTimer: () => {
      if (timer) {
        clearInterval(timer);
      }
    },
  }));
  return (
    <>
      <View className="flex justify-start pt-[20rpx]">
        {text ? (
          <View className="inline-block py-[15rpx] text-[28rpx]">
            <RichText className="break-all leading-[56rpx]" nodes={displayContent.join("")} />
          </View>
        ) : (
          <View className="inline-block py-[15rpx] text-[28rpx]">
            正在生成...
          </View>
        )}
      </View>
    </>
  );
});

export default AIAnswer;
```
