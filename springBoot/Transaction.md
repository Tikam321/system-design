### Spring Transactions in Practice: Declarative, Programmatic, and Propagation
- 1) Declarative Transactions (@Transactional)
- Declarative transactions mean you describe transaction behavior with annotations, and Spring manages transaction boundaries for you.
```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder() {
        // 1) insert order
        // 2) reduce inventory
        // 3) create payment record
    }
}
```
### How it works internally
- Spring creates an AOP proxy for the bean.
- Before method execution, proxy starts a transaction.
- If method completes successfully, proxy commits.
- If runtime exception occurs, proxy rolls back (default behavior).
- Why use it
- Minimal boilerplate
- Clean business code
- Consistent behavior across services
- Limitation
- Less control over fine-grained transaction boundaries inside a single method.
- 2) Programmatic Transactions (TransactionTemplate / PlatformTransactionManager)
- Programmatic transactions mean you control begin/commit/rollback in code.

```java
@Service
public class PaymentService {

    private final TransactionTemplate txTemplate;

    public PaymentService(TransactionTemplate txTemplate) {
        this.txTemplate = txTemplate;
    }

    public void processPayment() {
        txTemplate.execute(status -> {
            // db operations
            // if (someCondition) status.setRollbackOnly();
            return null;
        });
    }
}
```
### Why use it
- Full control for complex flows
- Useful when different code blocks need different transaction behavior
- Useful for conditional rollback rules
- Tradeoff
- More verbose
- Business logic and transaction logic get mixed
- 3) Transaction Propagation (Very Important)
- Propagation defines what happens when a transactional method calls another transactional method.

-    Common Modes
-    REQUIRED (default)
-    Join existing transaction if present.
-   Otherwise create a new one.
-   REQUIRES_NEW
-   Always start a new transaction.
-    Suspends existing transaction temporarily.
-    NESTED
-    Runs inside existing transaction with a savepoint.
-    Can roll back to savepoint without rolling back outer transaction (DB support required).
-    SUPPORTS
-   Use current transaction if exists.
-    Else run without transaction.
-    NOT_SUPPORTED
-    Always run without transaction.
-    Suspends existing transaction.
-    MANDATORY
-    Must have an existing transaction.
-    Throws exception if none exists.
-    NEVER
-    Must run without transaction.
-    Throws exception if a transaction exists.
-    4) Propagation Example
```java
@Service
public class CheckoutService {

    private final AuditService auditService;

    public CheckoutService(AuditService auditService) {
        this.auditService = auditService;
    }

    @Transactional(propagation = Propagation.REQUIRED)
    public void checkout() {
        // order + inventory updates in main tx
        auditService.writeAudit(); // independent transaction
    }
}

@Service
class AuditService {

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void writeAudit() {
        // audit log commit should be independent
    }
}
```
### Behavior
checkout() runs in Tx-1.
writeAudit() runs in Tx-2 (new one).
If Tx-1 rolls back later, Tx-2 can still remain committed.
5) Interview-Ready Summary
Use declarative transactions (@Transactional) for most business methods.
Use programmatic transactions for advanced/custom transaction control.
Know propagation:
REQUIRED for normal service logic
REQUIRES_NEW for independent operations (audit/outbox)
use other modes only with clear need.


### DB loack type 
1. shareLock(read lock) - transactions T1 applied share lock on DB ROW then transaction T2 can also acquire share lock on the same db row .
2. Exclusive Lock(write lock) - if transactoin have read lock or write lock on db row then t2 cannot able to applu the write lock on that db row.

so ths is how we use the lock to maintain the isolation level.
### for more clarity please check from ipad spring framework line number from 64 to 71.
