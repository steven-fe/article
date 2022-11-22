# Git - 规范commit message

每次开发完一个独立功能之后，都会提交commit，并且会写上commit message。但不同的人对commit message有着不同的理解：中英混合使用、fix bug等各种笼统的message司空见惯。这也导致code review、后期维护等场景下，必须通过代码才能得知提交的主要内容，代码维护成本比较大。因为规范的commit message是非常有必要的。

通过在技术社区里搜索得知，Angular规范是目前使用最广泛的写法，比较合理和系统化。结合本人的开发经验，总结出简化版本的commit message规范。

## commit message 格式

commit message: `<type>: <subject>`

### type

| type | 说明 |
| - | - |
| feat | 新功能（feature） |
| fix | 修复bug |
| docs | 文档（documentation） |
| style | 修改格式（不影响代码的功能） |
| refactor | 重构 |
| perf | 优化相关，比如提升性能、体验 |
| revert | 回滚到上一个版本 |
| merge | 代码合并 |

### subject

subject是commit目的的简短描述，结尾不加句号或其他标点符号。

## 好处

一旦约束了commit message，意味着开发者将慎重地进行每一次提交，不能再一股脑地把各种各样的改动都放在一个commit里，这样整个代码改动的历史也将更加清晰。
