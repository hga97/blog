### 背景

**开发遇到的问题:**

**规范**

每个人的风格随心所欲，代码交付质量不可控。  
提交的commit没有规范，无法从commit知晓提交开发内容。

**效率**

不断的重复工作，没有技术积累与沉淀

**项目质量**

没有规范就没有质量  
测试功能全部靠人工发现与回归，费时费力

**部署**

人工构建、部署  
依赖不统一、人为不可控  
没有版本追踪、回滚等功能  


### 工程化

**工程化：** 广义上，一切以提高效率、降低成本、保障质量为目的的手段，都属于工程化的范畴。

通过一系列的规范、流程、工具达到研发提效、自动化、保障质量、服务稳定、预警监控等等。

**DevOps:**

**常规基建**

组件库+脚手架+工具库+模版+CLI

**Git Flow**

通过常规git flow工作流程，不同branch不同功能 + code review

**CI/CD**

webhook + 脚本


### CLI工具分析

**CLI工具:**

**质量**

自动化测试 + eslint校验

**构建**

提供本地构建功能，接管发布构建

**模版**

创建模版、创建区块

**工具合集**

其他可以内置的工具类