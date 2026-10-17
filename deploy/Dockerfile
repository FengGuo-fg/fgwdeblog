# 阶段1：构建后端
FROM maven:3.8-jdk-8 AS build
WORKDIR /build
COPY . /build
RUN mvn clean package -DskipTests

# 阶段2：运行环境（Java 8 + nginx + supervisor）
FROM eclipse-temurin:8-jre
RUN apt-get update && apt-get install -y nginx supervisor tzdata \
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
