# session2-practice-ci-cd

Session 2 · Practice 2：用 GitHub Actions 搭建 CI/CD 流水线，自动训练线性模型并发版。

> 参考示例：https://github.com/reveurmichael/session2-practice-ci-cd

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
requirements.txt              # 依赖：pandas / scikit-learn / joblib
```

## 本地复现

```bash
pip install -r requirements.txt
python train_model.py
cat linear_model.txt
```

本地跑一次的结果（30 条样本）：

```
Coefficients: [9449.96232146]
Intercept: 25792.20019866871
```

即 Salary ≈ 25792.2 + 9449.96 × YearsExperience。

## 首次推送要做的事

```bash
# 1. 推到 main
git push -u origin main

# 2. 初始化 release 分支（关键！不建的话流水线最后一步会失败）
git checkout --orphan release
git rm -rf .
git commit --allow-empty -m "create empty release branch"
git push origin release
git checkout main
```

## 测试流水线

修改 `training_data.csv`（例如末尾追加 `11.0,130000.00`）并 push 到 `main`：

- **Actions** 页：能看到 `CI/CD Pipeline` 运行成功
- **release 分支**：只有 `linear_model.txt` 一个文件
- **Releases** 页：多了一个 `v2` 之类的版本，带 `linear_model.txt` 附件

## 常见坑

| 现象 | 原因 |
|---|---|
| 流水线没被触发 | 分支不叫 `main`。本仓库已用 `git init -b main` 建好 |
| `check_changes` 步骤报错 | `github.event.before` 在首次 push 时是全 0，属正常；只要不是首次提交就没问题 |
| push 到 release 分支 403 | `permissions: contents: write` 缺失，或仓库 Settings → Actions 里 Workflow permissions 设成了只读 |
| `gh release create` 报 tag 已存在 | 同一 run_number 重复执行，删掉旧 tag 或换个 tag 名 |
| 没生成 Release | `release` 分支不存在，第 6 步没做 |
