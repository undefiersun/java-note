java8 流
=========
几乎每个Java应用都会制造和处理集合。但集合用起来并不总是那么理想。比方说，你需要 从一个列表中筛选金额较高的交易，然后按货币分组。
你需要写一大堆套路化的代码来实现这个 数据处理命令，如下所示：

      Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>(); 
        for (Transaction transaction : transactions) { 
            if(transaction.getPrice() > 1000){  
            Currency currency = transaction.getCurrency();
            List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency); 
            if (transactionsForCurrency == null) {
                transactionsForCurrency = new ArrayList<>();
                transactionsByCurrencies.put(currency, transactionsForCurrency); 
            }         
            transactionsForCurrency.add(transaction);  
        } 
      } 
      
 此外，我们很难一眼看出来这些代码是做什么的，因为有好几个嵌套的控制流指令。 有了Stream API，你现在可以这样解决这个问题了： 
 建立累积交易 分组的Map 遍历交易的List筛选金额较 高的交易 提取交易货币如果这个货币的 分组Map是空的，那就建立一个 将当前遍历的交易添加到具有同一货币的交易List中 

为什么要关心 Java 8 
 
    import static java.util.stream.Collectors.toList;
    Map<Currency, List<Transaction>> transactionsByCurrencies = 
    transactions.stream() .filter((Transaction t) -> t.getPrice() > 1000).collect(groupingBy(Transaction::getCurrency)); 

这看起来有点儿神奇，不过现在先不用担心。第4~7章会专门讲述怎么理解Stream API。现在 值得注意的是，和Collection API相比，Stream API处理数据的方式非常不同。
用集合的话，你得 自己去做迭代的过程。你得用for-each循环一个个去迭代元素，然后再处理元素。我们把这种 数据迭代的方法称为外部迭代。相反，有了Stream API，你
根本用不着操心循环的事情。数据处 理完全是在库内部进行的。我们把这种思想叫作内部迭代。在第4章我们还会谈到这些思想。 使用集合的另一个头疼的地方是，想想看，要
是你的交易量非常庞大，你要怎么处理这个巨 大的列表呢？单个CPU根本搞不定这么大量的数据，但你很可能已经有了一台多核电脑。理想的 情况下，你可能想让这些CPU内核
共同分担处理工作，以缩短处理时间。理论上来说，要是你有 八个核，那并行起来，处理数据的速度应该是单核的八倍。 
