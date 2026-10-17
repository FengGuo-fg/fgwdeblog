# 阶段1：构建后端
# 把官方的 maven 换成阿里云同步的镜像，避免拉取超时
FROM registry.cn-hangzhou.aliyuncs.com/library/maven:3.8-jdk-8 AS build
WORKDIR /build
COPY . /build

# 给 Maven 配置阿里云镜像源，大幅加速依赖下载
RUN mkdir -p /root/.m2 && echo '<settings><mirrors><mirror><id>aliyunmaven</id><mirrorOf>*</mirrorOf><name>阿里云公共仓库</name><url>https://maven.aliyun.com/repository/public</url></mirror></mirrors></settings>' > /root/.m2/settings.xml

RUN mvn clean package -DskipTests

# 阶段2：运行环境（Java 8 + nginx + supervisor）
# 同样换成阿里云同步的基础镜像
FROM registry.cn-hangzhou.aliyuncs.com/library/eclipse-temurin:8-jre

# 配置 Debian 系统源为阿里云（防止 apt-get 更新时超时）
RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list \
    && apt-get update && apt-get install -y nginx supervisor tzdata \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /home/poetize
COPY --from=build /build/poetry-web/target/*.jar app.jar
COPY sql/poetry.sql /home/poetize/poetry.sql
COPY deploy/nginx.conf /etc/nginx/nginx.conf
COPY deploy/supervisord.conf /etc/supervisor/conf.d/app.conf

VOLUME /home/poetize/file
EXPOSE 80

CMD ["supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf"]
