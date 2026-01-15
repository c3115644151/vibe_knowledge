# 示例：MockBukkit 单元测试

## 1. 依赖配置 (pom.xml)
```xml
<dependency>
    <groupId>com.github.seeseemelk</groupId>
    <artifactId>MockBukkit-v1.21</artifactId>
    <version>3.102.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.0</version>
    <scope>test</scope>
</dependency>
```

## 2. 测试类 (GeneticsTest.java)
```java
class GeneticsTest {
    private ServerMock server;
    private CuisineFarming plugin;

    @BeforeEach
    void setUp() {
        server = MockBukkit.mock();
        plugin = MockBukkit.load(CuisineFarming.class);
    }

    @AfterEach
    void tearDown() {
        MockBukkit.unmock();
    }

    @Test
    void testMendelianInheritance() {
        GeneticsManager manager = new GeneticsManager(plugin);
        
        // 模拟两个亲本基因
        GenePair parent1 = new GenePair(Allele.DOMINANT, Allele.RECESSIVE); // Aa
        GenePair parent2 = new GenePair(Allele.DOMINANT, Allele.RECESSIVE); // Aa
        
        // 运行 1000 次杂交，验证概率
        int dominantCount = 0;
        for (int i = 0; i < 1000; i++) {
            GenePair child = manager.cross(parent1, parent2);
            if (child.isDominant()) {
                dominantCount++;
            }
        }
        
        // 期望: 75% 显性 (AA or Aa)
        // 允许 5% 的误差
        assertTrue(dominantCount > 700 && dominantCount < 800, 
            "杂交概率异常: " + dominantCount + "/1000");
    }
}
```
