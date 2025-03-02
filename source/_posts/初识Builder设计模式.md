---
title: 初识Builder设计模式
date: 2021-09-11 11:52:50
tags:
  - 设计模式
  - Java
categories:
  - 架构设计
---

### 问题

现在，假设我们正在一个为银行制作软件。在开始其他事情之前，我们需要用一种方式表示一个银行账户。我们第一版设计可能是下面这个样子：

    public class BankAccount {
        private long accountNumber;
        private String owner;
        private double balance;
    
        public BankAccount(long accountNumber, String owner, double balance) {
            this.accountNumber = accountNumber;
            this.owner = owner;
            this.balance = balance;
        }
    
    	//getter方法和setter方法
    }

我们可以用下面的方式来声明一个对象：

```java
BankAccount account = new BankAccount(123L, "lj", 100.00);
```

不幸的是，这样的设计是非常简单的。一个新的需求被送了过来，告诉你说我们需要记录每个账户每月的利率（interestRate），并且，还需要知道是在哪个分行（branch）。这听起来很容易，所以我们很容易就能提交我们第二个版本的BankAccount类：

```java
public class BankAccount {

    private long accountNumber;
    private String owner;
    private String branch;
    private double balance;
    private double interestRate;

    public BankAccount(long accountNumber, String owner, String branch, double balance, double interestRate) {
        this.accountNumber = accountNumber;
        this.owner = owner;
        this.branch = branch;
        this.balance = balance;
        this.interestRate = interestRate;
   }

    //getter方法和setter方法
}
```

由于我们及时更新银行账户，这使得我们获得了几个新用户。

```java
BankAccount account = new BankAccount(456L, "lj", "Springfield", 100.00, 2.5);
BankAccount anotherAccount = new BankAccount(789L, "wtx", null, 2.5, 100.00);  //这里的利息异常
```

对以上这段代码，我们的编译器不会检查出错误。但是，我们知道利息肯定不可能是100%（如果有这样一个银行，那绝对已经倒闭了！）但是为什么会出现这种情况呢？提示：注意构造器的变量顺序。

如果我们有多个连续同类型的参数，我们很容易将他们弄混。而且编译器是不会把这个错误识别出来的，这可能会在我们运行时造成一些很难调试的问题。另外，在构造器上增加过多的参数将会使可读性变得很差。如果有一个构造器有10个不同的参数，那么你很难一眼看出这些参数分别代表着什么。更糟糕的是，有一些参数还是可选的，也就是说我们有一系列重载的构造器来面临可能存在的组合情况，否则我们就只能将`null`传递给构造器，但这不是一个好习惯。

为了解决这个问题，你可能会认为这个时候我们需要调用一个无参构造器然后使用setter填充变量信息。但是这又留下了一个新的问题。如果一个程序员忘记调用特定的`setter`方法了呢？结果这会使得这个对象部分初始化并且编译器还是不会发现任何问题。

因此，我们有两件事需要解决：

* 避免过多的构造器参数
* 避免错误的对象状态

这就是Builder设计模式最初所解决的问题。



### Builder设计模式

Builder设计模式允许我们在初始化一个对象的时候写出可读性好和容易理解的代码。

建造者`builder`通常拥有`BankAccount`的所有字段。我们将会在builder中配置所有我们想要的字段，并且我们会使用builder来创建一个`account`。同时，我们将会移除`BankAccount`中的所有`public`的构造器，只留下一个`private`的构造器以便于只有builder能够创建一个`account`对象。

在之前的`BankAccount`例子中，我们使用建造者模式后，这个类将会是下面这个样子:

```java
public class BankAccount {

    public static class Builder {

        private long accountNumber; //这个字段很重要，所以我们会将其传给构造器
        private String owner;
        private String branch;
        private double balance;
        private double interestRate;

        public Builder(long accountNumber) {
            this.accountNumber = accountNumber;
        }

        public Builder withOwner(String owner){
            this.owner = owner;

            return this;  //通过返回一个Builder，可以创建一个流畅接口
        }

        public Builder atBranch(String branch){
            this.branch = branch;

            return this;
        }

        public Builder openingBalance(double balance){
            this.balance = balance;

            return this;
        }

        public Builder atRate(double interestRate){
            this.interestRate = interestRate;

            return this;
        }

        public BankAccount build(){
            //在这里我们创建一个BankAccount的对象，并且这个对象将被充分的初始化
            BankAccount account = new BankAccount();  //因为builder是BankAccount的内部类, 我们可以调用BankAccount的私有构造器。
            account.accountNumber = this.accountNumber;
            account.owner = this.owner;
            account.branch = this.branch;
            account.balance = this.balance;
            account.interestRate = this.interestRate;

            return account;
        }
    }

    //定义一个私有构造器
    private BankAccount() {
        //Constructor is now private.
    }

    //getter方法和setter方法
}
```

接下来我们用使用建造者模式的`BankAccount`创建`account`对象:

```java
BankAccount account = new BankAccount.Builder(1234L)
            .withOwner("lj")
            .atBranch("Springfield")
            .openingBalance(100)
            .atRate(2.5)
            .build();

BankAccount anotherAccount = new BankAccount.Builder(4567L)
            .withOwner("wtx")
            .atBranch("Springfield")
            .openingBalance(100)
            .atRate(2.5)
            .build();
```

虽然上面的代码看上去更长了，但是却更清晰了，更容易理解了。对于读代码时间多于写代码时间的我们来说，这显然是一种更棒的形式。

### 总结

在一个简单的银行账户的例子从简单到变复杂的过程中，我们使用了建造者模式探讨了我们发现的问题。

如果你发现你正处于在一个构造器上添加新的参数来解决问题而导致代码变得很难读的情况，那么对你来说这可能是你自己亲手实践，使用建造者模式重构你的代码的大好时机。

在笔者阅读这篇文章的时候，正在阅读一个使用Builder设计模式的项目代码，不得不说，使用Builder设计模式的代码虽然看上去长，有点使人畏惧，但是当你开始看上几眼之后，你会发现对你来说，读懂那文字，你就知道这个方法是干什么的了，这个类的构造也是十分清晰，基本是一目了然。经常也看到这样的言论：**伟大的代码是连初学者也能看懂的代码**。接触到builder设计模式以后，深以为然。


本文取自Dzone:[Design Patterns: The Builder Pattern](https://dzone.com/articles/design-patterns-the-builder-pattern)