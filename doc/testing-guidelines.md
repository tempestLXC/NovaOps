# NovaOps 单元测试规范

> 目标：让 NovaOps 的核心逻辑 **可验证、可重构、可维护**，而不是为覆盖率数字而测。

---

## 1. 总体原则

1. **优先测试业务逻辑，而不是框架行为**  
2. 能不用 Spring 容器就不用，优先纯单元测试（不依赖网络/DB/MQ 等外部资源）。  
3. 优先覆盖“容易出 bug 的地方”：复杂分支、金额 / 状态计算、边界条件。  
4. 测试代码也是代码，需要可读、可维护、可重构。

---

## 2. 技术栈与目录结构

### 2.1 测试技术栈

统一约定：

- 单元测试框架：**JUnit 5 (Jupiter)**
- Mock 框架：**Mockito**
- 断言：
  - 默认使用 JUnit 5 自带断言
  - 如有需要可以引入 **AssertJ** 提升可读性
- 需要 Spring 支持时使用：
  - `@SpringBootTest`
  - `@WebMvcTest`
  - `@DataJpaTest`
  - `@JdbcTest`

### 2.2 目录与包结构

- 生产代码：`src/main/java`
- 测试代码：`src/test/java`
- 单元测试类包路径应**镜像**生产代码包路径，例如：

```text
src/main/java/com/novaops/order/service/OrderService.java
src/test/java/com/novaops/order/service/OrderServiceTest.java
```

---

## 3. 命名规范

### 3.1 测试类命名

- 基础规则：`{被测类名}Test`
  - `OrderService` → `OrderServiceTest`
  - `UserController` → `UserControllerTest`

- 集成测试类可使用后缀 `IT`：
  - `OrderServiceIT`
  - `UserControllerIT`

### 3.2 测试方法命名

推荐格式（统一使用这一种）：

```text
methodName_whenCondition_expectedResult
```

示例：

```java
@Test
void createOrder_whenStockEnough_shouldSucceed() { ... }

@Test
void createOrder_whenUserBlacklisted_shouldThrowException() { ... }
```

配合 `@DisplayName` 增强可读性：

```java
@Test
@DisplayName("库存充足时创建订单成功")
void createOrder_whenStockEnough_shouldSucceed() { ... }
```

---

## 4. 单元测试 vs 集成测试

### 4.1 单元测试（Unit Test）

**定义**：  
不依赖网络、数据库、MQ、Redis 等外部系统，不起完整 Spring 容器，只验证**一个类或少量类的行为**。

**规范：**

- 不使用 `@SpringBootTest`
- 使用 Mockito + 构造器注入：
  - `@ExtendWith(MockitoExtension.class)`
  - `@Mock` + `@InjectMocks`
- 外部依赖统一 Mock：
  - Repository、FeignClient、Redis、MQ Producer/Consumer 等

**示例：**

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    StockClient stockClient;

    @Mock
    OrderRepository orderRepository;

    @InjectMocks
    OrderService orderService;

    @Test
    @DisplayName("库存充足时创建订单成功")
    void createOrder_whenStockEnough_shouldSucceed() {
        // given
        when(stockClient.getStock("item-001")).thenReturn(100);

        // when
        Order order = orderService.createOrder("user-001", "item-001", 2);

        // then
        assertNotNull(order.getId());
        assertEquals(OrderStatus.CREATED, order.getStatus());

        verify(orderRepository).save(any(Order.class));
    }
}
```

### 4.2 集成测试（Integration Test）

**定义**：  
涉及 Spring 容器启动，或真实访问数据库 / MQ / HTTP 等外部系统的测试。

**规范：**

- 使用 `@SpringBootTest` 或更细粒度注解：
  - `@DataJpaTest`：只测 JPA Repository
  - `@WebMvcTest`：只测 MVC Controller 层
- 测试数据库：
  - 优先使用 **H2** 或 **Testcontainers**
  - 使用 `@Transactional` 或测试结束清理数据，保证用例互不影响
- 命名规则：
  - 类名以 `IT` 结尾：`OrderServiceIT`

---

## 5. 覆盖范围与优先级

### 5.1 必须覆盖的部分

1. **核心业务逻辑**
   - 金额、折扣、税费计算
   - 状态机 / 审批流 / 订单流转等
   - 重试、熔断、限流等关键分支

2. **公共组件**
   - 自定义工具类（时间、金额、签名、加解密）
   - 自定义注解、拦截器、过滤器等

3. **复杂条件分支**
   - `if / else if / switch` 较多的逻辑
   - 策略模式 / 工厂模式中的路由选择逻辑

### 5.2 可以弱化或不测的部分

- 纯 DTO/VO、简单的 getter/setter（不含业务逻辑）
- 简单委托逻辑（例如明确只包了一层调用）

---

## 6. Mock 与外部依赖规范

1. **所有不确定因素要抽象并可 Mock**
   - 时间：避免直接 `LocalDateTime.now()`，封装为 `TimeProvider` 或注入 `Clock`
   - 随机数：注入 `Random` 或封装服务
   - 外部 HTTP/RPC：通过 Client 接口封装，在单元测试中 Mock

2. **禁止在单元测试中访问真实外部资源**
   - 真实 DB、Redis、消息队列、第三方 HTTP 接口等  
   - 如确有必要访问，归类为集成测试（`*IT`），避免与单元测试混在一起

3. **Verify 使用原则**
   - 仅对关键副作用进行 `verify`（如发送消息、写库、调用第三方）
   - 不要对所有调用都 `verify`，避免测试过度脆弱

---

## 7. 测试数据与断言规范

### 7.1 测试数据

- 避免魔法数字，使用有业务含义的常量或参数名：
  - ✅ `new Order("user-123", "item-001", 2)`
  - ❌ `new Order("u1", "i1", 2)`

- 对稍复杂的数据构造，统一使用工厂 / Builder：
  - `OrderTestData.newPaidOrder()`
  - `UserTestData.createBlacklistedUser()`

### 7.2 断言规范

- 使用 JUnit 5 或 AssertJ 进行断言，避免混用 JUnit 4 旧 API。
- 对核心断言给出解释说明（可用 AssertJ 的 `as`）：

```java
assertThat(order.getStatus())
    .as("创建订单后状态应为 CREATED")
    .isEqualTo(OrderStatus.CREATED);
```

---

## 8. 覆盖率指标（建议）

> 覆盖率是结果，不是目标；不要为了覆盖率写“假测试”。

建议指标（后续可接入 Jacoco + Sonar）：

- 项目整体：
  - 行覆盖率：**≥ 70%**
  - 分支覆盖率：**≥ 60%**
- 核心模块（如认证、网关、订单结算等）：
  - 行覆盖率：**≥ 80%**
  - 分支覆盖率：**≥ 70%**

新增逻辑要求：

- 新增非 trivial 的业务逻辑 → PR 中应包含对应单元测试；
- 修 bug → 必须新增/补充一个回归测试用例。

---

## 9. CI 集成要求（预留）

后续在 CI（如 GitHub Actions）中集成：

- 执行 `mvn test` 或 `mvn verify`，保证测试全部通过；
- 生成 Jacoco 报告（例如 `target/site/jacoco`）；
- 如使用 SonarQube，可在质量门中配置最低覆盖率和重复代码限制。

---

## 10. 示例：标准单元测试模板

```java
package com.novaops.order.service;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    StockClient stockClient;

    @Mock
    OrderRepository orderRepository;

    @InjectMocks
    OrderService orderService;

    @Test
    @DisplayName("库存充足时创建订单成功")
    void createOrder_whenStockEnough_shouldSucceed() {
        // given
        when(stockClient.getStock("item-001")).thenReturn(100);

        // when
        Order order = orderService.createOrder("user-001", "item-001", 2);

        // then
        assertNotNull(order);
        assertEquals(OrderStatus.CREATED, order.getStatus());
        verify(orderRepository).save(any(Order.class));
    }
}
```

以上规范适用于 NovaOps 全仓，如需例外情况（如 POC、Spike），需要在 PR 说明中说明原因。
