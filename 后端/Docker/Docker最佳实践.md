[TOC]



# Docker最佳实践


## Dockerfile

### 减少构建时间

**调整指令顺序，将容易变动的指令排列在后面**
（更好的利用编译缓存）
```dockerfile
FROM debian
RUN apt-get update
RUN apt-get -y install openjdk-8-jdk ssh vim
COPY . /app
CMD ["java", "-jar", "/app/app.jar"]
```


**只COPY需要的文件，避免使用COPY . 指令**
（更好的避免缓存被破坏）
```dockerfile
FROM debian
RUN apt-get update
RUN apt-get -y install openjdk-8-jdk ssh vim
COPY target/app.jar /app
CMD ["java", "-jar", "/app/app.jar"]
```

**尽量合并RUN中的命令，减少不必要的RUN指令**
（更好的利用缓存）
```dockerfile
FROM debian
RUN apt-get update \
    && apt-get -y install openjdk-8-jdk ssh vim 
COPY target/app.jar /app
CMD ["java", "-jar", "/app/app.jar"]
```

### 减少镜像大小

**移除不必要的依赖**
```dockerfile
FROM debian
RUN apt-get update \
    && apt-get -y install --no-install-recommends \
    openjdk-8-jdk 
COPY target/app.jar /app
CMD ["java", "-jar", "/app/app.jar"]
```

**移除包管理器的缓存**
```dockerfile
FROM debian
RUN apt-get update \
    && apt-get -y install --no-install-recommends \
    openjdk-8-jdk \
    && rm -rf /var/lib/apt/lists/* 
COPY target/app.jar /app
CMD ["java", "-jar", "/app/app.jar"]
```

**使用最小化的基础镜像**
```dockerfile
FROM openjdk:8-jre-alpine
COPY target/app.jar /app
CMD ["java", "-jar", "/app/app.jar"]
```

### 可维护性

**尽可能的使用官方的镜像**
**尽可能的指定版本**
(避免意外的错误)
```dockerfile
FROM openjdk:8
COPY target/app.jar /app
CMD ["java", "-jar", "/app/app.jar"]
```

**在统一环境中，从源码构建出镜像**
**使用多阶段构建，移除构建依赖**
```dockerfile
FROM maven:3.6-jdk-8-alpine AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn -e -B dependency:resolve
COPY src ./src
RUN mvn -e -B package

FROM openjdk:8-jre-alpine
COPY --from=builder /target/app.jar /
CMD ["java", "-jar", "/app/app.jar"]
```




