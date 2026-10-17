# ==========================================
# 阶段1：构建后端
# ==========================================
# 使用阿里云同步的 Maven 镜像，加速拉取
FROM registry.cn-hangzhou.aliyuncs.com/k8s/maven:3.8-jdk-8 AS build
WORKDIR /build

# 把代码复制进容器
COPY . /build

# 配置 Maven 使用阿里云镜像源，极大加速依赖下载
RUN mkdir -p /root/.m2 && echo '<settings><mirrors><mirror><id>aliyunmaven</id><mirrorOf>*</mirrorOf><name>阿里云公共仓库</name><url>https://maven.aliyun.com/repository/public</url></mirror></mirrors></settings>' > /root/.m2/settings.xml

# 执行 Maven 打包
RUN mvn clean package -DskipTests


# ==========================================
# 阶段2：运行环境（Java 8 + nginx + supervisor）
# ==========================================
# 同样使用阿里云同步的 JDK 镜像
FROM registry.cn-hangzhou.aliyuncs.com/k8s/eclipse-temurin:8-jre

# 替换 Debian 系统源为阿里云源，并安装 Nginx 和 Supervisor
RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list \
    && apt-get update && apt-get install -y nginx supervisor tzdata \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /home/poetize

# 从阶段1的构建产物中复制 jar 包
COPY --from=build /build/poetry-web/target/*.jar app.jar
# 复制数据库初始化脚本
COPY sql/poetry.sql /home/poetize/poetry.sql
# 复制 Nginx 和 Supervisor 的配置文件
COPY deploy/nginx.conf /etc/nginx/nginx.conf
COPY deploy/supervisord.conf /etc/supervisor/conf.d/app.conf

# 挂载卷（防止上传图片丢失）
VOLUME /home/poetize/file
EXPOSE 80

# 启动 supervisor 来同时运行 Nginx 和 Java
CMD ["supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf"]
