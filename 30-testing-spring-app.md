# TESTING YOUR SPRING APPS

- Unit tests: Focus only on an isolated piece of logic
- Integration tests: Focus on validating that multiple components correctly
  interact with each other

**Regression testing** is the approach of constantly testing existing functionality to validate it still works correctly.

## IMPLEMENTING UNIT TESTS

Every test comprises three fundamental elements:

- **Assumptions**: Define inputs and manage dependencies to control the desired flow scenario. Determine necessary inputs and expected behavior of dependencies to enable the specific behavior of the tested logic.

- **Call/Execution**: Invoke or execute the logic being tested to observe and evaluate its behavior.

- **Validations**: Specify and define all expected outcomes and behaviors for the tested logic under the given conditions. Verify whether the actual behavior aligns with the expected behavior when the logic is called or executed.

> These are called call "arreange, act and assert" or "given, when and then".

In a general sense, dependencies for any method or function can be categorized into:

- **Method parameters**: External values passed as inputs to the method, influencing its behavior or output.

- **External object instances or services**: Objects or services utilized by the method, not created within it.

---

In testing the `transferMoney()` method, we have a dependency on AccountRepository's `findById()` method, but in a unit test, we shouldn't directly call this method. To handle this dependency, we use `mocks`- fake objects whose behavior we can control. By employing `mocks`, we substitute the real `AccountRepository` object with a controlled fake object. This allows us to control the behavior of `findById()` and test various executions of the `transferMoney()` method without directly invoking the actual database or repository operations. The goal is to assert that the method under test behaves as expected for different scenarios without relying on real data or external systems.

```java
@Transactional
public void transferMoney(long senderAccountId, long receiverAccountId, BigDecimal amount){

    Account sender = accountRepository.findById(senderAccountId)
            .orElseThrow(() -> new AccountNotFoundException());

    Account receiver = accountRepository.findById(receiverAccountId)
            .orElseThrow(() -> new AccountNotFoundException());

    accountRepository.changeAmount(senderAccountId, sender.getAmount().subtract(amount));
    accountRepository.changeAmount(receiverAccountId, receiver.getAmount().add(amount));

}
```

To create a `mock` we will use a method `mock()` which is provided by a dependency called `Mockito`. To make is simpler, we can import static using: `import static org.mockito.Mockito.mock`.

For creating the test cases, we will use `@Test` annotated method in the Java Files under `src/main/test`.

```java
package com.example.demo;

import com.example.demo.service.TransferService;
import org.junit.jupiter.api.Test;

import static org.mockito.Mockito.mock;

public class TransferServiceUnitTests {


    @Test
    public void moneyTransferHappyFlow(){

        //Instead of a real AccountRepository instance,
        //we create the object using a mock AccountRepository.
        AccountRepository accountRepository = mock(AccountRepository.class);
        TransferService transferService = new TransferService(accountRepository);
    }

}
```

Now we can "control" this mock object using `given()` method. In our case, we
want the AccountRepository’s `findById()` method to return a specific `Account`
instance for a given parameter value. We will also add `@DisplayName` annotation to describe the method.

```java
package com.example.demo;

import com.example.demo.model.Account;
import com.example.demo.service.TransferService;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import java.math.BigDecimal;
import java.util.Optional;

import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.mock;

public class TransferServiceUnitTests {


    @Test
    @DisplayName("Test the amount transfer from one account to another if no exception occurs.")
    public void moneyTransferHappyFlow(){

        AccountRepository accountRepository = mock(AccountRepository.class);
        TransferService transferService = new TransferService(accountRepository);

        //create sender Account instance
        Account sender = new Account();
        sender.setId(1);
        sender.setAmount(new BigDecimal(1000));

        //create receiver Account instance
        Account destination = new Account();
        destination.setId(2);
        destination.setAmount(new BigDecimal(1000));

        //If one calls the findById() method with the sender ID parameter, then return the sender account instance
        given(accountRepository.findById(sender.getId()))
                .willReturn(Optional.of(sender));

        //If one calls the findById() method with the destination ID parameter, then return the destination account instance
        given(accountRepository.findById(destination.getId()))
                .willReturn(Optional.of(destination));

        //test transfer service
        transferService.transferMoney(sender.getId(), destination.getId(), new BigDecimal(100));

    }

}
```

Next, we need to define: What should happen when the test method is run?

To verify a mock's object's method has been called, we can use `verify()` method.

In this example, we expect `changeAmount()` method of the `accountRepository` instance to be called with

1. accountId: 1 and amount 900
2. accountId: 2 and amount 1100

So, this is what we will verify using the Mockito's `verify()` method which we can import using `import static org.mockito.Mockito.verify`.

```java
package com.example.demo;

import com.example.demo.model.Account;
import com.example.demo.service.TransferService;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import java.math.BigDecimal;
import java.util.Optional;

import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.verify;

public class TransferServiceUnitTests {


    @Test
    @DisplayName("Test the amount transfer from one account to another if no exception occurs.")
    public void moneyTransferHappyFlow(){

        AccountRepository accountRepository = mock(AccountRepository.class);
        TransferService transferService = new TransferService(accountRepository);

        //create sender Account instance
        Account sender = new Account();
        sender.setId(1);
        sender.setAmount(new BigDecimal(1000));

        //create receiver Account instance
        Account destination = new Account();
        destination.setId(2);
        destination.setAmount(new BigDecimal(1000));

        //If one calls the findById() method with the sender ID parameter, then return the sender account instance
        given(accountRepository.findById(sender.getId()))
                .willReturn(Optional.of(sender));

        //If one calls the findById() method with the destination ID parameter, then return the destination account instance
        given(accountRepository.findById(destination.getId()))
                .willReturn(Optional.of(destination));

        //test transfer service
        transferService.transferMoney(
                sender.getId(),
                destination.getId(),
                new BigDecimal(100)
        );

        //Verify that the changeAmount() method in the AccountRepository was called with the expected parameters.
        verify(accountRepository)
                .changeAmount(1, new BigDecimal(900));
        verify(accountRepository)
                .changeAmount(2, new BigDecimal(1100));

    }

}
```

Now, just right click on the test Java file and run it:

![intellij-test-case](images/intellij-test-case.png)

You will see the following test case (passed) in the output:

![first-test-case-pass](images/first-test-case-pass.png)

## USING ANNOTATION FOR MOCK DEPENDENCIES

- `@ExtendWith(MockitoExtension.class)`: Enables the use of Mockito annotations like `@Mock` and `@InjectMocks`.
- `@Mock`: Generates mock objects for dependencies (`AccountRepository`) that the `TransferService` relies on.
- `@InjectMocks`: Creates an instance of the class under test (`TransferService`) and injects the mocked dependencies (`AccountRepository`) into it.

```java
package com.example.demo;

import com.example.demo.model.Account;
import com.example.demo.service.TransferService;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.math.BigDecimal;
import java.util.Optional;

import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.verify;

//Enable the use of @Mock and @InjectMocks annotations
@ExtendWith(MockitoExtension.class)
public class TransferServiceUnitTests {

    @Mock
    private AccountRepository accountRepository;

    //Use the @InjectMocks to create the tested object and inject it into the class’s annotated field.
    @InjectMocks
    private TransferService transferService;

    @Test
    @DisplayName("Test the amount transfer from one account to another if no exception occurs.")
    public void moneyTransferHappyFlow(){

        //create sender Account instance
        Account sender = new Account();
        sender.setId(1);
        sender.setAmount(new BigDecimal(1000));

        //create receiver Account instance
        Account destination = new Account();
        destination.setId(2);
        destination.setAmount(new BigDecimal(1000));

        //If one calls the findById() method with the sender ID parameter, then return the sender account instance
        given(accountRepository.findById(sender.getId()))
                .willReturn(Optional.of(sender));

        //If one calls the findById() method with the destination ID parameter, then return the destination account instance
        given(accountRepository.findById(destination.getId()))
                .willReturn(Optional.of(destination));

        //test transfer service
        transferService.transferMoney(
                sender.getId(),
                destination.getId(),
                new BigDecimal(100)
        );

        //Verify that the changeAmount() method in the AccountRepository was called with the expected parameters.
        verify(accountRepository)
                .changeAmount(1, new BigDecimal(900));
        verify(accountRepository)
                .changeAmount(2, new BigDecimal(1100));

    }

}

```
