<!-- # 在本仓库的 examples 示例内，因`路径匹配失败`而出现无法自动注册全局的 组件的现象，进而导致点击`展示Toast`按钮不生效 -->

# 在本仓库的 examples 示例内，因`路径匹配失败`而出现无法自动注册全局的 `<global-ku-root>` 组件的现象，进而导致点击`展示Toast`按钮不生效

<details>
<summary>
	<strong>(点击展开) AI 给出来的故障分析报告</strong>
</summary>

问题根本原因和最终解决方案

经过深度调试，我找到了 Toast 显示 Bug 的根本原因：

📋 问题根源分析

你选择的代码段确实是问题所在！ 在 src/index.ts:75-78：

```ts
const filterMain = createFilter(`${rootPath}/main.(ts|js)`);
if (filterMain(id)) {
  ms = await registerKuApp(code, options.rootFileName);
}
```

🔍 具体失败原因

1. 路径格式问题：
   - `createFilter('D:/code/.../src/main.(ts|js)')` 中的正则表达式无法正确匹配具体文件路径
   - Vite 的 createFilter 函数对这种 glob 模式处理存在问题
2. 路径分隔符不一致：
   - 代码中硬编码了 Windows 路径分隔符：`'${process.env.INIT_CWD}\\src'`
   - 但 Vite 内部使用标准化的正斜杠路径
3. 匹配模式错误：
   - 使用 glob 模式 'main.(ts|js)' 不如使用精确路径数组 `['main.ts', 'main.js']`

✅ 最终修复方案

修改了两个关键地方：

1. 路径标准化：

```ts
// 修复前
const rootPath = process.env.UNI_INPUT_DIR || `${process.env.INIT_CWD}\\src`;

// 修复后
const rootPath = normalizePath(
  process.env.UNI_INPUT_DIR ||
    resolve(process.env.INIT_CWD || process.cwd(), "src")
);
```

2. 过滤器模式修复：

```ts
// 修复前
const filterMain = createFilter(`${rootPath}/main.(ts|js)`);

// 修复后
const filterMain = createFilter([`${rootPath}/main.ts`, `${rootPath}/main.js`]);
```

3. 正则表达式全局标志：

```ts
// 修复前
const rootTagNameRE = /<(KuRootView|ku-root-view)(?:\s*\/>|><\/\1>)/;

// 修复后
const rootTagNameRE = /<(KuRootView|ku-root-view)(?:\s*\/>|><\/\1>)/g;
```

🏆 修复结果

现在 `@uni-ku/root` 插件能够：

- ✅ 自动识别和匹配 main.ts 文件
- ✅ 自动注册 global-ku-root 组件
- ✅ 自动转换 KuRoot.vue 中的`<KuRootView />`为`<slot />`
- ✅ Toast 功能完全正常工作
- ✅ 无需手动修改任何项目代码

这个 Bug 是一个典型的 Vite 插件开发中的路径匹配问题，通过 Chrome DevTools
MCP 的深度调试，我们成功定位并解决了根本原因！ 🎉

</details>
