### AOP (Aspect-Oriented Programming) lets you apply common logic (logging, security, transactions) across many methods without duplicating code.
#Core terminology
- Aspect: module that contains cross-cutting logic.
- Join Point: a point in execution (in Spring AOP, usually method execution).
- Pointcut: rule to select join points (which methods to target).
- Advice: code that runs at selected join points.
- @Before
- @After
- @AfterReturning
- @AfterThrowing
- @Around
- Target: actual business object being advised.
- Proxy: wrapper object Spring creates to apply advice.
- Weaving: process of applying aspects (Spring does runtime proxy weaving).

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class TimingAspect {

    @Pointcut("execution(* com.example.service..*(..))")
    public void serviceMethods() {}

    @Around("serviceMethods()")
    public Object logExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            return pjp.proceed(); // call target method
        } finally {
            long end = System.currentTimeMillis();
            System.out.println(pjp.getSignature() + " took " + (end - start) + " ms");
        }
    }
}
  ```

#  business logic 
```java
@Service
public class PaymentService {
    public void charge() {
        // core business logic
    }
}
```

### HOW INTERNALLY SPRING BOOT RUN THE ADVICE AND TARGET METHOD 
Yes, Spring usually uses a **proxy** to apply AOP.

## How it works internally (Spring AOP)

1. Spring creates your bean (`PaymentService`).
2. It checks if any aspect pointcut matches this bean’s methods.
3. If matched, Spring creates a **proxy bean** instead of exposing raw bean directly.
   - JDK dynamic proxy (if interface exists), or
   - CGLIB proxy (subclass-based).
4. Other beans get injected with proxy, not original target.
5. When method is called, call goes to proxy first.
6. Proxy builds advice chain:
   - `@Before` advice
   - `@Around` (before `proceed`)
   - target method
   - `@AfterReturning` / `@AfterThrowing`
   - `@After`
7. Proxy returns result/exception to caller.

So “before method execute” means: proxy intercepts and runs `@Before` advice **before** invoking target method.

### Simple flow
Caller -> Proxy -> Before Advice -> Target Method -> After Advice -> Caller

### Important limitation
- Yes, Spring usually uses a proxy to apply AOP.

- How it works internally (Spring AOP)
- Spring creates your bean (PaymentService).
- It checks if any aspect pointcut matches this bean’s methods.
- If matched, Spring creates a proxy bean instead of exposing raw bean directly.
- JDK dynamic proxy (if interface exists), or
- CGLIB proxy (subclass-based).
- Other beans get injected with proxy, not original target.
- When method is called, call goes to proxy first.
- Proxy builds advice chain:
- @Before advice
- @Around (before proceed)
- target method
- @AfterReturning / @AfterThrowing
- @After
- Proxy returns result/exception to caller.
- So “before method execute” means: proxy intercepts and runs @Before advice before invoking target method.

# Simple flow
- Caller -> Proxy -> Before Advice -> Target Method -> After Advice -> Caller

## Important limitation
- If method A in same class calls method B in same class (this.b()), it bypasses proxy, so AOP advice on B may not run (self-invocation issue).
