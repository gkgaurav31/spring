# UNDERSTANDING THE USE OF INTERFACES

## Scenario

Implement an object that needs to print packages’ details to be delivered for a shipping app. The printed details must be sorted by their destination address.

### THOUGHT PROCESS

The most straight forward way to implement is by having two classes:

- `DeliveryDetailsPrinter` class whose main responsibility is to `printDetails()`.
- `DeliveryDetailsPrinter` delegates the sorting task to another class `SorterByAddress` which can have a method `sortDetails()` to sort by address.

The main challenge we can easily see is:

### UNDERSTANDING THE PROBLEM

> What happens if later we need to sort by sendder's name instead of destination address?

You will need to change the code in `DeliveryDetailsPrinter` as well, which is not ideal.

### CODE TO INTERFACE

Instead of saying "Give me this" (concrete implementation) and rather say "I want to do this" (interface) it would allow us to change the implementation of how the main task is done later. The idea is not to rely directly on a concrete implementation as it can change later.

The classes would then look something like this:

- `Sorter` interface

```java
public interface Sorter {
 void sortDetails();
}
```

- `DeliveryDetailsPrinter` class

```java
public class DeliveryDetailsPrinter {

 private Sorter sorter;

 public DeliveryDetailsPrinter(Sorter sorter) {
    this.sorter = sorter;
 }

 public void printDetails() {
    sorter.sortDetails();
    // printing the delivery details
 }
}
```

- This gives us an option to use any implementation of `Sorter`.
