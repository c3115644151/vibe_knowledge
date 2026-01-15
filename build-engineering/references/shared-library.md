# 示例：创建共享的基因库 (Shared Genetics Library)

## 1. 目录结构
```text
/root
  /CoreLibrary (New Module)
    /src/main/java/com/example/core/genetics
      - GeneticsManager.java
      - Trait.java
  /CuisineFarming
    pom.xml (Depends on CoreLibrary)
  /PastureSong
    pom.xml (Depends on CoreLibrary)
```

## 2. Maven 配置 (CoreLibrary/pom.xml)
```xml
<groupId>com.example</groupId>
<artifactId>core-library</artifactId>
<version>1.0.0</version>
<packaging>jar</packaging>
```

## 3. 插件依赖 (CuisineFarming/pom.xml)
```xml
<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>core-library</artifactId>
        <version>1.0.0</version>
        <scope>compile</scope> <!-- 会被 Shade 进插件 Jar -->
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <configuration>
                <relocations>
                    <relocation>
                        <pattern>com.example.core</pattern>
                        <shadedPattern>com.example.cuisinefarming.libs.core</shadedPattern>
                    </relocation>
                </relocations>
            </configuration>
        </plugin>
    </plugins>
</build>
```
