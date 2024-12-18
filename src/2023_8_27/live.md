git，并行开发

大部分开发者和公司都是使用git管理代码。

在git上创建分支，让不同的开发者可以并行开发。
git提供了管理分支、合并代码等管理代码的命令，但并没有提供分支模型。当团队规模较小，代码的迭代不是很频繁的情况下，分支模型的优劣不是很明显。但当团队规模较大，需求迭代频繁时，一个合适的分支模型将尤为重要。

即使一个人开发，也需要选择合适的分支模型。

没有一个完美的分支模型，需要根据项目的特点、团队的特点选择适合的分支模型。

1. 是否支持并行开发。
2. 是否支持回滚，以及如何回滚。
3. 如何跟踪历史。

需要考虑的问题：1. 有哪些分支、这些分支的角色是什么？2. 在哪个分支上发布代码？3. 在测试阶段，如何处理bug？4. 在线上bug，如何处理bug？

## github flow

在github上流行的flow。

在master分支上发布代码，feature分支只开发代码。

github，library或framework，不是最终面向用户。

代码贡献者素质比较高，自动化测试脚本较为全面。

## git flow

最复杂的分支模型。非常严谨。

release切出去之后，develop可以继续开发下一个版本。

## gitlab flow

pre-production分支上测试单独的代码。

production分支上测试代码合并线上代码时的最终结果。

## Trunk-based flow

长期维护的业务分支，差距将越来越大。

## 总结

github - library or framework
git/gitlab - xxx

分支命名需要规范。会造成沟通困难。所有的分支都要汇集到主干分支上！
