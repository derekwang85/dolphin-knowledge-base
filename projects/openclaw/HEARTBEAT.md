```markdown
# Keep this file empty (or with only comments) to skip heartbeat API calls.

# Add tasks below when you want the agent to check something periodically.

# HEARTBEAT TASKS:
# - Media server 健康检查：确保 port 18888 上 media-server.py 在运行。
#   如果挂了，重启：cd ~/.openclaw/workspace && nohup python3 scripts/media-server.py --port 18888 --bind 0.0.0.0 > /tmp/media-server.log 2>&1 &
#
# - derekgitworkspace 同步检查：工作区变更自动提交到 GitHub。
#   每 2 小时由独立 cron 自动执行（脚本路径：skills/derekgitworkspace/scripts/auto-sync.sh）。
#   在此仅做 git 健康状态巡检，不执行推送。
#
# - TradeOMS Agent 巡检：每 10 次 heartbeat 检查一次所有 Agent 状态。
#   流程: 查 git status → 查 /tmp/ 下编译状态 → 如有Agent空闲按WBS派下一任务
#   派活逻辑见 WBS: /home/cbnb/TradeOMS/tasks/project-wbs-full.md
#   （核心巡检由独立 cron tradeoms-night-watch 每10分钟执行，此处仅做补充）
```

## Related

- [Heartbeat config](/gateway/config-agents)
