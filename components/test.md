### 单元测试

```zsh
yarn add jest ts-jest @testing-library/react @testing-library/jest-dom
identity-obj-proxy @types/jest @types/testing-library__react --dev
```

jest: JavaScript 测试框架，专注于简洁明快；  
ts-jest：为 TypeScript 编写 jest 测试用例提供支持；  
@testing-library/react：简单而完整的 React DOM 测试工具，鼓励良好的测试实践；  
@testing-library/jest-dom：自定义的 jest 匹配器(matchers)，用于测试 DOM 的状态
（即为 jest 的 except 方法返回值增加更多专注于 DOM 的 matchers）；  
identity-obj-proxy：一个工具库，此处用来 mock 样式文件。

jest.config.js

```js
module.exports = {
  verbose: true,
  roots: ["<rootDir>/src"],
  moduleNameMapper: {
    "\\.(css|less|scss)$": "identity-obj-proxy",
    "^src$": "<rootDir>/src/index.tsx",
    "^src(.*)$": "<rootDir>/src/$1",
  },
  testRegex: "(/test/.*|\\.(test|spec))\\.(ts|tsx|js)$",
  moduleFileExtensions: ["ts", "tsx", "js", "jsx"],
  testPathIgnorePatterns: ["/node_modules/", "/lib/", "/esm/", "/dist/"],
  preset: "ts-jest",
  testEnvironment: "jsdom",
};
```

package.json

```json
"scripts": {
    "test": "jest",                         # 执行jest
    "test:watch": "jest --watch",           # watch模式下执行
    "test:coverage": "jest --coverage",     # 生成测试覆盖率报告
    "test:update": "jest --updateSnapshot"  # 更新快照
},

"lint-staged": {
  "src/**/*.ts?(x)": [
    "jest --bail --findRelatedTests", # bail：退出测试。findRelatedTests：用于预提交挂钩集成以运行最少量的必要测试。
  ],
}
```

### 编写测试用例

```tsx
import React from "react";
import { render } from "@testing-library/react";
import Alert from "../alert";

describe("<Alert />", () => {
  test("should render default", () => {
    const { container } = render(<Alert>default</Alert>);
    expect(container).toMatchSnapshot();
  });

  test("should render alert with type", () => {
    const kinds: any[] = ["info", "warning", "positive", "negative"];

    const { getByText } = render(
      <>
        {kinds.map((k) => (
          <Alert kind={k} key={k}>
            {k}
          </Alert>
        ))}
      </>
    );

    kinds.forEach((k) => {
      expect(getByText(k)).toMatchSnapshot();
    });
  });
});
```

当使用 toMatchSnapshot 的时候，Jest 将会渲染组件并创建其快照文件。这个快照文件包含渲染后组件的整个结构，并且应该与测试文件本身一起提交到代码库。当我们再次运行快照测试时，Jest 会将新的快照与旧的快照进行比较，如果两者不一致，测试就会失败，从而帮助我们确保用户界面不会发生意外改变。
