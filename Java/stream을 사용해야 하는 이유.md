# Stream을 사용해야하는 이유

스트림은 자바8 버전에 첫 등장하게 된 새로운 API입니다. 스트림을 이용하여 **선언적**으로 **간편**하고 **가독성** 있게 데이터를 처리할수 있다. 선언적이라는게 무슨말일까요?



> **선언적 프로그래밍과 명령형 프로그래밍**

명령형 프로그래밍이란 익숙한 방식의 프로그래밍입니다. 예를들어 1부터 100까지 합을 구하는 코드를 작성하려면 어떻게 해야할까요?

```java
int sum = 0;
for (int i = 1; i <= 100; i++) {
    sum = sum + i;
}
```

 필자는 처음 프로그래밍을 배울때 한참 코드를 봐야 위의 코드가 어떤의미를 가지는지 이해할수 있었습니다.

for문으로 반복문을 돌려 초기값이 0인 sum이라는 변수에 1을 더해서 다시 sum에 대입하고 다시 2를 더하고..... 3를 더하고... 마지막으로 100을 더한다!!!

1부터 100까지 더한다(무엇을 할지)와 어떻게 할지의 의미적 차이가 커 괴리감이 있기 때문에 이해하기 어려웠던것 같습니다. 지금은 처음 프로그래밍을 배웠던 때보다는 코드를 많이 작성해 보았기 때문에 위의 예제는 금방 이해할수 있습니다. 하지만 코드가 더 복잡해진다면 빠르게 이해할수 있을까요?

```java
Map<Currency, List<Transaction>> transactionByCurrencies = new HashMap<>();

for(Transaction transaction : transactions) {
    Currency currency = transaction.getCurrency();
    List<Transaction> transactionForCurrency = transactionByCurrencies.get(currency);
    if (transactionForCurrency == null) {
        transactionForCurrency = new ArrayList<>();
        transactionByCurrencies.put(currency, transactionForCurrency);
    }
    transactionForCurrency.add(transaction);
}
```

긴 코드를 다 읽은 후에야 해당 코드가 *통화별로 트랜잭션 리스트를 그룹화*한다는것을 알수 있습니다. 이것을 스트림을 이용하여 이해하기 쉽게 작성할수 있습니다.

```java
Map<Currency, List<Transaction>> transactionByCurrencies = transactions.stream()
        .collect(groupingBy(Transaction::getCurrency));
```

해당 코드를 그대로 읽어 보겠습니다. 

*transactions를 transaction의 currency 별로 groupingBy하여 collect한다.* 

어떻게 할지가 아니라 **무엇을 할지** 를 코드로 작성하여 코드를 이해하기 훨씬 쉬워진것 같습니다. 이러한 방식을 **선언적 프로그래밍** 이라고 합니다.