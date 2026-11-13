# antd-table-standalone

从 antd v5.29.3 源码中提取出来的 **Table 组件**，作为独立包使用，已通过 `tsc --noEmit` 全量类型检查。

## 提取范围

以 `components/table` 为核心，通过**运行时 import 依赖闭包**自动追踪，共包含 36 个组件目录（依赖关系见 `../scripts/extract-table.js` 的运行输出），关键依赖包括：

- `table` — 核心组件（含 `RcTable/`、`hooks/`、`style/`）
- `pagination`、`checkbox`、`radio`、`dropdown`、`menu`、`spin`、`empty`、`tooltip`、`button`、`input`、`tree` — Table 直接/间接使用的组件
- `config-provider`、`locale`、`theme`、`_util`、`style`、`version`、`grid` — CSS-in-JS Token 体系、国际化、工具函数

### 类型桩（`components/_stubs/`）

antd 的 `theme/interface/components.ts`（ComponentTokenMap）与 `config-provider/context.ts` 会 type-only 引用几乎全部 50+ 组件的类型。为了不把这些组件全部拷进来，提取脚本将这些 **type-only import** 重写为 `components/_stubs/` 下的 `export type X = any` 桩文件（共 58 个）。这对 Table 的运行时行为零影响；极个别 strict 泛型表达式因 `any` 丢失结构信息，在 4 处做了显式 cast（见脚本 `POST_PATCHES`）。

### 自动重写的 import

- `table/hooks/useTitleColumns.ts`、`table/hooks/useFilter/FilterDropdown.tsx` 中 `antd/es/_util/type` → 相对路径
- `select/style/token.ts` 中 `antd/es/theme/internal` → 相对路径
- `form/hooks/useFormItemStatus.ts` 中 `antd/es/form/FormItem` → 相对路径

## 使用

```tsx
import Table from 'antd-table-standalone';
import type { ColumnsType, TableProps } from 'antd-table-standalone';

const columns: ColumnsType<{ name: string }> = [
  { title: 'Name', dataIndex: 'name' },
];

export default () => <Table dataSource={[{ name: 'antd' }]} columns={columns} />;
```

样式基于 `@ant-design/cssinjs` 自动注入，无需引入任何 CSS 文件；主题定制用法与 antd 一致（`ConfigProvider`、`theme` 已从入口导出）。

## 验证

```bash
npm install
npm run typecheck
```

## 重新提取

在仓库根目录运行：

```bash
node scripts/extract-table.js
```

脚本会重新复制运行时闭包（跳过 `__tests__`、`demo`、`design` 目录）、重写 self-referencing import、生成类型桩、应用类型修补，并做完整性校验；`package.json` / `tsconfig.json` 等脚手架文件不会被覆盖。
