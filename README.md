
## 发布流程

1. 代码在 E5 上编写、构建、测试完成。
2. 把成品文件复制到 Pi 的 `~/eps-inbox/`。
3. 在 Pi 上执行 `eps-push --from ~/eps-inbox "<commit message>"`。
4. 脚本负责 stage / commit / push，不做任何编辑。

Pi 只承担提交与推送，编辑、构建、测试一律在 E5 完成。
