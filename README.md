# session2-practice-ci-cd

Session 2 · Practice 2：用 GitHub Actions 搭建 CI/CD 流水线，自动训练线性模型并发版。

## 工作流做什么

1. 推送到 `main` 分支（或向 `main` 提 PR）时触发
2. 检查 `training_data.csv` 是否有改动
3. 有改动 → 运行 `train_model.py` 训练线性回归模型
4. 把 `linear_model.txt` 推到 `release` 分支（该分支为孤立分支，只存模型文件）
5. 用 `gh release create` 自动创建 Release，tag 为 `v<run_number>`

## 目录

```
.github/workflows/ci-cd.yml   # 流水线定义
training_data.csv             # 训练数据（YearsExperience, Salary）
train_model.py                # 训练脚本，输出 linear_model.txt / linear_model.pkl
requirements.txt              # 依赖
```

## 本地复现

```bash
pip install -r requirements.txt
python train_model.py
cat linear_model.txt
```

## 测试流水线

修改 `training_data.csv`（例如追加一行数据）并 push 到 `main`，然后在仓库的
**Actions** 页看运行记录，在 **Releases** 页看新生成的版本。
