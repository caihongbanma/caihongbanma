# 使用官方 Python 轻量级镜像作为基础
FROM python:3.9-slim AS builder

# 设置工作目录
WORKDIR /app

# 安装系统依赖（根据实际需要调整）
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gcc \
    python3-dev && \
    rm -rf /var/lib/apt/lists/*

# 复制依赖文件并安装（利用 Docker 层缓存）
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# --------------------------
# 第二阶段：构建运行时镜像
# --------------------------
FROM python:3.9-slim

# 设置容器内环境变量
ENV PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app \
    FLASK_APP=app.py \
    FLASK_ENV=production \
    PORT=5000

# 设置工作目录
WORKDIR /app

# 从构建阶段复制已安装的依赖
COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

# 复制应用代码
COPY . .

# 创建非 root 用户并设置权限
RUN useradd -m appuser && \
    chown -R appuser:appuser /app
USER appuser

# 暴露端口（实际映射在运行容器时指定）
EXPOSE $PORT

# 健康检查（可选）
HEALTHCHECK --interval=30s --timeout=5s \
  CMD curl --fail http://localhost:$PORT/health || exit 1

# 启动命令
CMD ["gunicorn", "--bind", "0.0.0.0:${PORT}", "--workers", "4", "app:create_app()"]