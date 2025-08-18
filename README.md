# svg-quick

`svg-quick` 是一个轻量级的命令行工具，能将你的 SVG 图标文件夹快速打包成一个高效的、完全支持 Tree-shaking 的 JavaScript/TypeScript 模块。告别繁琐的 SVG 管理，拥抱现代化的图标工作流。

## 🤔 为什么选择 svg-quick? (Why svg-quick?)

在现代前端项目中，管理和使用 SVG 图标常常面临诸多挑战：

- **性能问题**: 大量独立的 SVG 文件会导致过多的 HTTP 请求。
- **打包效率**: 在大型项目中，每次构建都实时处理上百个 SVG 会拖慢构建速度。
- **体积问题**: 作为 React/Vue 组件直接导入会增加额外的运行时代码，导致产物体积膨胀。
- **灵活性差**: 不同的使用场景（如 `<img>` 标签, CSS `background`, 数据可视化库 ECharts）需要不同格式的图标。

`svg-quick` 正是为解决这些痛点而生，它为你带来：

- **⚡️ 极致性能**: 通过预构建，极大提升项目构建速度。所有图标打包为单一模块，经 Gzip 压缩后体积更小。
- **🌳 完美按需打包 (Tree-shaking)**: 只将你实际使用到的图标打包进最终产物，确保包体积最小化。
- **✨ 高度灵活性**: 提供多种输出模式，你可以选择生成按需打包的独立导出，或者包含所有图标的全量对象，满足任何使用场景。
- **💻 优秀的开发体验**: 提供统一的 API 和完整的 TypeScript 类型定义，享受完美的自动补全和类型检查。
- **🔧 强大的优化**: 内置 `svgo`，在打包过程中自动对所有 SVG 进行深度压缩和优化，确保代码一致性和高质量。

## 📦 安装 (Installation)

推荐使用 `npx` 在项目中按需执行，无需全局安装。

```bash
# 无需安装，直接通过 npx 运行
npx svg-quick --input ./svgs --output ./dist
```

如果你希望全局安装：

```bash
npm install -g svg-quick
```

## 🚀 使用方法 (Usage)

假设你的项目结构如下：

```
my-project/
├── svgs/
│   ├── user.svg
│   ├── arrow-left.svg
│   └── ...
└── ...
```

1.  在你的项目根目录下运行以下命令：

    ```bash
    npx svg-quick -i ./svgs -o ./src/generated-icons -m t
    ```

2.  `svg-quick` 将会处理 `svgs` 目录下的所有图标，并生成优化后的模块到 `src/generated-icons` 目录。

## ⚙️ 命令行参数 (API)

| 参数       | 简写 | 描述                     | 可选值                                  | 默认值   |
| :--------- | :--- | :----------------------- | :-------------------------------------- | :------- |
| `--input`  | `-i` | 包含源 SVG 文件的目录。  | (路径)                                  | **必需** |
| `--output` | `-o` | 生成产物模块的输出目录。 | (路径)                                  | **必需** |
| `--mode`   | `-m` | 输出模式。               | `all`(a), `treeshakeable`(t), `full`(f) | `all`    |

**模式 (Mode) 详解:**

- `treeshakeable` (`t`): **（推荐）** 生成支持按需打包的独立图标导出。支持完美的 Tree-shaking，最终产物体積最小。
- `full` (`f`): 生成包含所有图标的全量对象和实用工具函数 (`getIconData`, `getIconSrc`)。所有图标都会被打包，适用于需要在运行时动态访问大量图标的场景。
- `all` (`a`): 同时生成以上两种模式的文件，提供最大灵活性。

> **💡 重要提示**:
>
> - `treeshakeable` 模式：只包含独立的图标导出，支持完美的按需打包，打包工具只会包含你实际使用的图标
> - `full` 模式：包含完整的 `icons` 对象和工具函数 (`getIconData`, `getIconSrc`)，所有图标都会被打包，但提供动态访问能力

## 📖 产物使用示例 (Using the Output)

### Treeshakeable 模式 (推荐)

当 `mode` 设置为 `treeshakeable` 时，你可以这样使用：

```javascript
// 在你的 React / Vue / Svelte 组件中
import { user, arrowLeft } from '../generated-icons';

// 'user' 和 'arrowLeft' 都是优化后的 SVG 字符串
// 你可以轻松地封装成自己的 <Icon> 组件

// React 示例:
const Icon = ({ svgString, ...props }) => (
  <span {...props} dangerouslySetInnerHTML={{ __html: svgString }} />
);

const App = () => (
  <div>
    <Icon svgString={user} style={{ color: 'blue' }} />
    <Icon svgString={arrowLeft} style={{ color: 'red' }} />
  </div>
);
```

**优势**: 打包工具构建你的应用时，只有被 `import` 的 `user` 和 `arrowLeft` 图标会被包含进来，实现完美的按需打包。

### Full 模式

当 `mode` 设置为 `full` 时，你可以使用实用工具函数：

```javascript
// 导入实用工具函数，支持动态获取图标
import { getIconData, getIconSrc } from '../generated-icons';

// 获取 SVG 字符串
const userIconSvg = getIconData('user');

// 获取 base64 编码的 data URL，可直接用于 <img> 标签或 CSS background
const userIconSrc = getIconSrc('user');

// 在组件中使用
const DynamicIcon = ({ iconName }) => {
  const iconSrc = getIconSrc(iconName);
  return iconSrc ? <img src={iconSrc} alt={iconName} /> : null;
};

// 或者在 CSS 中使用
const iconUrl = getIconSrc('user');
// background-image: url(data:image/svg+xml;base64,...)
```

### 在 ECharts 中使用图标 (需要 Full 模式)

```javascript
// 导入 ECharts 和图标工具函数 (需要使用 full 模式生成)
import * as echarts from 'echarts';
import { getIconSrc } from '../generated-icons';

// 在 ECharts 配置中使用图标
const option = {
  // 1. 在系列数据中使用图标作为 symbol
  series: [
    {
      type: 'scatter',
      data: [
        { value: [10, 20], symbol: getIconSrc('user') },
        { value: [30, 40], symbol: getIconSrc('star') },
      ],
      symbolSize: 30,
    },
  ],

  // 2. 在图例中使用自定义图标
  legend: {
    data: [
      {
        name: '用户数据',
        icon: getIconSrc('user'),
      },
      {
        name: '收藏数据',
        icon: getIconSrc('star'),
      },
    ],
  },

  // 3. 在工具箱中使用自定义图标
  toolbox: {
    feature: {
      myTool: {
        show: true,
        title: '自定义工具',
        icon: getIconSrc('settings'),
        onclick: function () {
          console.log('自定义工具被点击');
        },
      },
    },
  },

  // 4. 在标记点中使用图标
  series: [
    {
      type: 'line',
      data: [120, 200, 150, 80, 70, 110, 130],
      markPoint: {
        data: [
          {
            type: 'max',
            symbol: getIconSrc('arrowUp'),
            symbolSize: 25,
          },
          {
            type: 'min',
            symbol: getIconSrc('arrowDown'),
            symbolSize: 25,
          },
        ],
      },
    },
  ],
};

// 初始化图标
const chart = echarts.init(document.getElementById('chart'));
chart.setOption(option);
```

> **注意**: ECharts 中的动态图标使用需要 `full` 模式，因为需要 `getIconSrc` 工具函数。如果你只需要在 ECharts 中使用少量固定图标，也可以考虑使用 `treeshakeable` 模式直接导入图标字符串。

## 📄 许可证 (License)

本项目采用 [MIT](https://github.com/King-GD/svg-quick/blob/main/LICENSE) 许可证。
