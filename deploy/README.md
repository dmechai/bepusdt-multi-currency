# BEpusdt 运维

`dmechai/bepusdt-multi-currency` 是唯一维护仓库；`v03413/bepusdt` 是上游。多币种收银台、EasyPay 查单、反代 HTTPS 和 TRON HTTP/gRPC 兼容改动直接维护在源码中。

## 上游更新

GitHub Actions 定期在临时检出中合并最新正式版、构建并运行 `go test ./...`，不推送、不发布、不部署。需要升级时，从 `main` 开分支，运行 `git fetch --no-tags upstream refs/tags/<tag>` 并合并 `FETCH_HEAD`（首次可用 `git remote add upstream https://github.com/v03413/bepusdt.git`）。处理冲突后验证本地改动和数据库迁移，再合回 `main`。

## 部署和回滚

修改 `deploy/compose.yaml` 的默认版本为新的唯一镜像标签。部署前执行 `docker compose -f deploy/compose.yaml build`，用隔离容器检查 SQLite 副本；在线数据库用 SQLite `.backup` 制作一致快照。保留上一版镜像，再运行 `docker compose -f deploy/compose.yaml up -d --no-build --wait`。检查站点、查单和链扫描。

需要回滚时，可用 `BEPUSDT_VERSION=<previous-tag> docker compose -f deploy/compose.yaml up -d --no-build --wait`。先确认旧版兼容升级后的数据库；不要直接覆盖正在使用的数据库或丢弃升级后写入的订单。首次升级到 v1.24.4 的本机回滚镜像是 `rollback-pre-v1.24.4`，包含之前热替换的程序。
