---
title: 文档/知识笔记
icon: teenyicons:doc-outline
createTime: 2024/03/04 15:45:34
permalink: /guide/document/
tags:
  - 指南
  - 快速开始
---

## 概述

主题提供了 `笔记` 的功能，它用于聚合 同一个系列的文章、或者作为站点的 **子文档** 。

`笔记`  以 文件结构 作为划分依据，默认以 `notes/` 作为根目录，
存放在 `notes` 目录下的 文档不会作为 博客文章，不会出现在 博客文章列表页中。

## 文件结构与配置

我们有一个项目中，有以下文件结构：

::: file-tree

- docs
  - notes
    - typescript \# typescript 笔记
      - basic.md
      - types.md
    - rust \# rust 笔记
      - tuple.md
      - struct.md
  - blog-post.md \# 博客文章
  - README.md \# 站点首页

:::

在 `docs/notes` 目录下，有两个子目录，分别用于存放 `typescript` 和 `rust` 的系列内容。

接下来，在配置文件中配置 `notes`：

```js title=".vuepress/config.ts" twoslash
import { defineUserConfig } from 'vuepress'
import { plumeTheme } from 'vuepress-theme-plume'

export default defineUserConfig({
  theme: plumeTheme({
    notes: {
      // 声明所有笔记的目录，(默认配置，通常您不需要声明它)
      dir: '/notes/',
      link: '/', // 声明所有笔记默认的链接前缀， 默认为 '/' （默认配置，通常您不需要声明它）
      notes: [
        // 每个笔记都是 `notes` 数组中的一个对象
        {
          // 声明笔记的目录，相对于 `notes.dir`，这里表示 `notes/typescript` 目录
          dir: 'typescript',
          // 声明笔记的链接前缀，与 `notes.link` 拼接，这里表示 `/typescript/`
          // 笔记内的所有文章会以 `/typescript/` 作为访问链接前缀。
          link: '/typescript/',
          // 配置 笔记侧边导航栏，用于导航向笔记内的所有文档
          // 声明为 `auto` 的，将根据目录结构自动生成侧边栏导航
          sidebar: 'auto'
        },
        {
          dir: 'rust',
          link: '/rust/',
          sidebar: [
            { text: '简介', items: ['foo'] }
          ]
        }
      ]
    }
  })
})
```

::: tip

你应该在创建文件之前，建议先把笔记的目录和链接前缀等配置好。
主题默认启用了 [auto-frontmatter](../../config/theme.md#autofrontmatter)，
需要根据配置，为目录中的 md 文件生成永久链接，以及侧边栏。

:::

## 编写notes配置

由于 `notes` 配置全部写在 `plumeTheme({  })` 中可能会导致 代码层级嵌套过深，因此更推荐使用主题提供的
`defineNotesConfig()` 和 `defineNoteConfig()` 将 notes 配置提取到外部，它们还能帮助你获得更好的类型提示，
更具可读性和便于维护。

::: code-tabs

@tab .vuepress/notes.ts

```ts twoslash
import { defineNoteConfig, defineNotesConfig } from 'vuepress-theme-plume'

/**
 * 配置 单个 note
 */
const typescript = defineNoteConfig({
  dir: 'typescript',
  link: '/typescript/',
  sidebar: [
    '/guide/intro.md',
    '/guide/getting-start.md',
    '/config/config-file.md',
  ]
})

/**
 * 配置 notes
 */
export default defineNotesConfig({
  // 声明所有笔记的目录，(默认配置，通常您不需要声明它)
  dir: '/notes/',
  link: '/',
  // 在这里添加 note 配置
  notes: [typescript] // [!code ++]
})
```

@tab .vuepress/config.ts

```ts twoslash
// @filename: ./notes.ts
export default {}

// ---cut---
import { defineUserConfig } from 'vuepress'
import { plumeTheme } from 'vuepress-theme-plume'
import notes from './notes' // [!code ++]

export default defineUserConfig({
  theme: plumeTheme({
    notes // [!code ++]
  }),
})
```

:::

:::: details 笔记比较多时怎么配置？
如果您拥有比较多的笔记，全部放到一个 `notes.ts` 中配置，可能会显得文件比较大且不好维护。
您可以对文件进行拆分，以 `.vuepress/notes/` 目录作为 笔记配置的目录。
::: file-tree

- docs
  - .vuepress
    - notes
      - typescript.ts
      - rust.ts
      - index.ts
      - …
  - notes
    - typescript/
    - rust/
:::

代码如下所示：

::: code-tabs
@tab .vuepress/config.ts

```ts
import { defineUserConfig } from 'vuepress'
import { plumeTheme } from 'vuepress-theme-plume'
import notes from './notes/index.ts' // [!code ++]

export default defineUserConfig({
  theme: plumeTheme({
    notes // [!code ++]
  }),
})
```

@tab .vuepress/notes/index.ts

```ts
import { defineNotesConfig } from 'vuepress-theme-plume'
import rust from './rust' // [!code ++]
import typescript from './typescript' // [!code ++]

export default defineNotesConfig({
  // 声明所有笔记的目录，(默认配置，通常您不需要声明它)
  dir: '/notes/',
  link: '/',
  // 在这里添加 note 配置
  notes: [ // [!code ++:4]
    typescript,
    rust,
  ]
})
```

@tab .vuepress/notes/typescript.ts

```ts
import { defineNoteConfig } from 'vuepress-theme-plume'

export default defineNoteConfig({
  dir: 'typescript',
  link: '/typescript/',
  sidebar: [
    '/guide/intro.md',
    '/guide/getting-start.md',
    '/config/config-file.md',
  ]
})
```

@tab .vuepress/notes/rust.ts

```ts
import { defineNoteConfig } from 'vuepress-theme-plume'

export default defineNoteConfig({
  dir: 'rust',
  link: '/rust/',
  sidebar: [
    '/guide/intro.md',
    '/guide/getting-start.md',
    '/config/config-file.md',
  ]
})
```

:::
::::

### 侧边栏配置

以 `typescript` 目录为例，它拥有如下的文件结构：

::: file-tree

- typescript
  - guide
    - intro.md
    - getting-start.md
  - config
    - config-file.md
    - configuration.md
  - reference
    - basic.md
    - syntax.md
    - modules.md
  - built-in
    - types
      - Required.md
      - Omit.md
  - README.md

:::

#### 自动生成侧边栏

一种最简单的配置方式是 `sidebar: 'auto'` ， 主题会自动根据 文件结构生成侧边栏，并根据 首个字符的编码 来排序。

如果想要修改 自动生成的侧边栏的顺序，可以直接在 目录名 或 文件名之前，添加 `1.` 或 `2.` 等前缀。

::: file-tree

- typescript
  - 1.guide
    - 1.intro.md
    - 2.getting-start.md
  - 2.config
    - 1.config-file.md
    - 2.configuration.md
  - …
:::

主题将根据 这部分的前缀的 数字 进行排序，前缀部分不会显示在侧边栏中。

#### 自定义侧边栏

有时候自动生成侧边栏 不能完全满足需求，你可以自定义侧边栏。

以下是 侧边栏的 类型定义：

```ts
interface ThemeBadge {
  /* 徽章文本 */
  text?: string
  /* 徽章类型，内置： 'info' | 'tip' | 'danger' | 'warning' */
  type?: string
  /* 文本颜色 */
  color?: string
  /* 背景颜色 */
  bgColor?: string
  /* 边框颜色 */
  borderColor?: string
}

type ThemeIcon = string | { svg: string }

type Sidebar = (string | SidebarItem)[]

interface SidebarItem {
  /**
   * 侧边栏文本
   */
  text?: string

  /**
   * 侧边栏链接
   */
  link?: string

  /**
   * 侧边栏图标
   */
  icon?: ThemeIcon

  /**
   * 侧边栏徽章
   */
  badge?: string | ThemeBadge

  /**
   * 当前分组的链接前缀，链接前缀会拼接在 `items` 内的 `link` 之前
   * 当且仅当 `items` 内的 `link` 为 相对路径时，才会拼接
   */
  prefix?: string
  /**
   * 次级侧边栏分组
   */
  items?: 'auto' | (string | SidebarItem)[]

  /**
   * 如果未指定，组不可折叠。
   * 如果为`true`，组可折叠，并默认折叠。
   * 如果为`false`，组可折叠，但默认展开。
   */
  collapsed?: boolean
}
```

当 传入类型为 `string` 时，表示 markdown 文件的路径：

```ts title=".vuepress/notes.ts" twoslash
import { defineNoteConfig } from 'vuepress-theme-plume'

const typescript = defineNoteConfig({
  dir: 'typescript',
  link: '/typescript/',
  sidebar: [
    '/guide/intro.md',
    '/guide/getting-start.md',
    '/config/config-file.md',
    // ...
  ]
})

// ... other code
```

你也可以省略 `.md` 文件后缀，简写为 `/guide/intro` 。主题会解析 对应的文件，获取 **标题** 和 **页面链接地址**
并将其转换为 `{ text: string, link: string }` 的数据形式。

当传入类型为 `SidebarItem` 时:

```ts title=".vuepress/notes.ts" twoslash
import { defineNoteConfig } from 'vuepress-theme-plume'

const typescript = defineNoteConfig({
  dir: 'typescript',
  link: '/typescript/',
  sidebar: [
    { text: '介绍', link: '/guide/intro' },
    { text: '快速上手', link: '/guide/getting-start' },
  // ...
  ]
})

// ... other code
```

也可以进行多层嵌套：

```ts title=".vuepress/notes.ts"  twoslash
import { defineNoteConfig } from 'vuepress-theme-plume'

const typescript = defineNoteConfig({
  dir: 'typescript',
  link: '/typescript/',
  sidebar: [
    {
      text: '指南',
      prefix: '/guide', // 使用 prefix 拼接，可以简写 下面的 items 中的 link 为相对路径
      items: [
      // 可以混用 string 和 SidebarItem
        { text: '介绍', link: 'intro' },
        'getting-start',
      ],
    },
    {
      text: '配置',
      prefix: '/config',
      items: 'auto', // items 为 'auto'，会根据 prefix 的文件结构自动生成侧边栏
    },
  ]
})

// ... other code
```

### 关于 `prefix`

`prefix` 的目的是为了简写与其同层级的 `items` 项内的 链接，它允许你将这些链接的相同的前缀提取到
`prefix` 中，由主题帮您完成完整链接的拼接。

需要注意的是，`items` 中的链接 仅有 相对路径的链接才会与 `prefix` 拼接，而绝对路径则不进行处理。

```ts title=".vuepress/notes.ts" twoslash
import { defineNoteConfig } from 'vuepress-theme-plume'

const typescript = defineNoteConfig({
  dir: 'typescript',
  link: '/typescript/',
  sidebar: [
    {
      prefix: '/guide',
      items: [
        'intro', // 相对路径, 最终拼接为 /guide/intro
        '/config/config-file', // 绝对路径，不拼接
        {
          text: '博客',
          link: 'blog', // 相对路径, 最终拼接为 /guide/blog
        },
        {
          text: '配置',
          link: '/config', // 绝对路径，不拼接
        }
      ]
    }
  ]
})
```

同时，`items` 内还支持 深层嵌套，内部还依然支持 `prefix`，这里也遵循相同的规则，`prefix` 如果是相对路径，
则会与 上一层的 `prefix` 拼接，再与 当前层级 `items` 内的 `link` 拼接，如果 `prefix` 是绝对路径，则不与
上一层级 `prefix` 拼接。

```ts title=".vuepress/notes.ts" twoslash
import { defineNoteConfig } from 'vuepress-theme-plume'

const typescript = defineNoteConfig({
  dir: 'typescript',
  link: '/typescript/',
  sidebar: [
    {
      prefix: '/guide',
      items: [
        'intro', // 相对路径, 最终拼接为 /guide/intro
        {
          prefix: '/config',
          items: [
            'config-file', // 相对路径, 最终拼接为 /config/config-file
            'configuration', // 相对路径, 最终拼接为 /config/configuration
          ]
        },
        {
          prefix: 'blog',
          items: [
            'intro', // 相对路径, 最终拼接为 /guide/blog/intro
            'getting-start', // 相对路径, 最终拼接为 /guide/blog/getting-start
          ]
        }
      ]
    }
  ]
})
```

**是否是绝对路径的判断标准是，如果以 `/` 开头，则为绝对路径，否则为相对路径**

:::warning
不建议 侧边栏的层级过深，超过 3 层的侧边栏 可能会导致 糟糕的 UI 效果。
:::

### 侧边栏图标

为侧边栏添加 ==图标== 有助于 侧边栏更好的呈现。得益于 [iconify](https://iconify.design/) 这个强大的开源图标库，
你可以使用超过 `200k` 的图标，仅需要添加 `icon` 配置即可。

```ts title=".vuepress/notes.ts" twoslash
import { defineNoteConfig } from 'vuepress-theme-plume'

const typescript = defineNoteConfig({
  dir: 'typescript',
  link: '/typescript/',
  sidebar: [
    {
      text: '指南',
      prefix: '/guide',
      icon: 'ep:guide', // iconify icon name // [!code hl]
      items: [
        { text: '介绍', link: 'intro', icon: 'ph:info-light' }, // [!code hl]
      ],
    },
  ]
})
```

也可以使用本地图标，或者本地图片：

```ts title=".vuepress/notes.ts" twoslash
import { defineNoteConfig } from 'vuepress-theme-plume'

const typescript = defineNoteConfig({
  dir: 'typescript',
  link: '/typescript/',
  sidebar: [
    {
      text: '指南',
      prefix: '/guide',
      icon: '/images/guide.png', // iconify icon name // [!code hl]
      items: [
        { text: '介绍', link: 'intro', icon: '/images/info.png' }, // [!code hl]
        // 也可以是一个远程图片
        { text: '快速上手', link: 'getting-start', icon: 'https://cn.vuejs.org/images/logo.png' },
      ],
    },
  ]
})
```

**请注意，使用本地图片必须以 `/` 开头，表示为 静态资源路径，它将从 `.vuepress/public/` 目录中加载。**

::: file-tree

- docs
  - .vuepress
    - public  \# 在这个位置保存静态资源
      - images
        - guide.png
        - info.png
  - …
:::

当 `sidebar: auto` 时，可在 md 文件的 `frontmatter` 部分，添加 一个 `icon` 字段：

```md title="intro.md"
---
title: 介绍
icon: ep:guide
---
```

### 侧边栏徽章 <Badge text="v1.0.0-rc.143 +" />

主题支持为侧边栏添加徽章，徽章可以用于辅助提供更多的信息。

```ts title=".vuepress/notes.ts" twoslash
import { defineNoteConfig } from 'vuepress-theme-plume'

const typescript = defineNoteConfig({
  dir: 'typescript',
  link: '/typescript/',
  sidebar: [
    {
      text: '指南',
      prefix: '/guide',
      badge: { text: '徽章', type: 'danger' }, // [!code hl]
      items: [
        { text: '介绍', link: 'intro', badge: '徽章' }, // [!code hl]
      ],
    },
  ]
})
```

当 `sidebar: auto` 时，可在 md 文件的 `frontmatter` 部分，添加 一个 `badge` 字段：

```md title="intro.md"
---
title: 介绍
badge:
  text: 徽章
  type: danger
---
```

```md title="intro.md"
---
title: 介绍
badge: 徽章
---
```

### 侧边栏组内分隔

在组内对 项 进行分隔 是一个相对小众的需求，它在组的项比较多，但又不适合拆分为多个组，或者组内拆分多组的情况下，
可能会比较适用，它提供了一个平级的，使用辅助文本颜色显示一个分隔项名 的方式，对项进行简单的分隔。

```ts title=".vuepress/notes.ts" twoslash
import { defineNoteConfig } from 'vuepress-theme-plume'

const typescript = defineNoteConfig({
  dir: 'typescript',
  link: '/typescript/',
  sidebar: [
    {
      text: '指南',
      items: [
        '项目 1',
        '项目 2',
        '项目 3',
        { text: '分隔', link: '---' }, // [!code ++]
        '项目 4',
        '项目 5',
        // ...
      ],
    },
  ]
})
```

在组内完成分隔非常简单，你只需要在合适的位置插入一个 `{ text: 'xxxx', link: '---' }` 即可，
它的重点仅是将 `link` 设置为 连续的 `---` 即可，至少三个 `-` 。
你可以随意定义文本，还可以添加图标。

## 笔记首页

你可能注意到，在 `typescript` 目录下，有一个 `README.md` 文件，它会被作为笔记首页显示。

::: file-tree

- typescript
  - README.md
  - …
- …
:::

默认情况下，它与 普通的文档页面 没有区别，这是因为 主题 默认对 所有页面 设置了 `pageLayout: docs`。

但你可以直接配置 `pageLayout: 'home'`，就像配置 [站点首页](../custom/home.md) 一样，为 笔记配置一个个性化的首页！

```md title="typescript/README.md"
---
pageLayout: home
config:
  - type: hero
  - type: features
---
```
