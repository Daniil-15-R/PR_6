#PR_6.1

#1. Создаем файл для тестирования

```
 using System;
namespace BankAccountNS
{
    /// <summary>
    /// Bank account demo class.
    /// </summary>
    public class BankAccount
    {
        private readonly string m_customerName;
        private double m_balance;


        private BankAccount() { }


        public BankAccount(string customerName, double balance)
        {
            m_customerName = customerName;
            m_balance = balance;
        }


        public string CustomerName
        {
            get { return m_customerName; }
        }


        public double Balance
        {
            get { return m_balance; }
        }


        public void Debit(double amount)
        {
            if (amount > m_balance)
            {
                throw new ArgumentOutOfRangeException("amount");
            }


            if (amount < 0)
            {
                throw new ArgumentOutOfRangeException("amount");
            }


            m_balance += amount;
        }


        public void Credit(double amount)
        {
            if (amount < 0)
            {
                throw new ArgumentOutOfRangeException("amount");
            }


            m_balance += amount;
        }


        public static void Main()
        {
            BankAccount ba = new BankAccount("Mr. Roman Abramovich", 11.99);


            ba.Credit(5.77);
            ba.Debit(11.22);
            Console.WriteLine("Current balance is ${0}", ba.Balance);
            Console.ReadLine();
        }
    }
}
```
#2.Создаем еще один файл, который нужен для создания теста
```
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;

namespace BankTests
{
    using BankAccountNS;
    [TestClass]
    public class BankAccountTests
    {
        [TestMethod]
        public void Debit_WithValidAmount_UpdatesBalance()
        {
            // Arrange
            double beginningBalance = 11.99;
            double debitAmount = 4.55;
            double expected = 7.44;
            BankAccount account = new BankAccount("Mr. Roman Abramovich", beginningBalance);

            // Act
            account.Debit(debitAmount);

            // Assert
            double actual = account.Balance;
            Assert.AreEqual(expected, actual, 0.001, "Account not debited correctly");
        }
    }
}
```
#3. Запускаем и выходит ошибка
![scrin1](https://github.com/user-attachments/assets/930eb45c-f3e6-46a5-832e-0286b2025e4a)
#4. Исправляем ошибку в предудыщем файле и запускаем заново
Код:
```
using System;

namespace BankAccountNS
{
    /// <summary>
    /// Bank account demo class.
    /// </summary>
    public class BankAccount
    {
        private readonly string m_customerName;
        private double m_balance;


        private BankAccount() { }


        public BankAccount(string customerName, double balance)
        {
            m_customerName = customerName;
            m_balance = balance;
        }


        public string CustomerName
        {
            get { return m_customerName; }
        }


        public double Balance
        {
            get { return m_balance; }
        }


        public void Debit(double amount)
        {
            if (amount > m_balance)
            {
                throw new ArgumentOutOfRangeException("amount");
            }


            if (amount < 0)
            {
                throw new ArgumentOutOfRangeException("amount");
            }


            m_balance -= amount;
        }


        public void Credit(double amount)
        {
            if (amount < 0)
            {
                throw new ArgumentOutOfRangeException("amount");
            }


            m_balance -= amount;
        }


        public static void Main()
        {
            BankAccount ba = new BankAccount("Mr. Roman Abramovich", 11.99);


            ba.Credit(5.77);
            ba.Debit(11.22);
            Console.WriteLine("Current balance is ${0}", ba.Balance);
            Console.ReadLine();
        }
    }
}
```
Результат
![scrin2](https://github.com/user-attachments/assets/90779d81-4a53-4db9-a111-23946224cc00)

#5. Добавляем новый метод:
```
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System;

namespace BankTests
{
    using BankAccountNS;

    [TestClass]
    public class BankAccountTests
    {
        [TestMethod]
        public void Debit_WhenAmountIsLessThanZero_ShouldThrowArgumentOutOfRange()
        {
            // Arrange
            double beginningBalance = 11.99;
            double debitAmount = -100.00;
            BankAccount account = new BankAccount("Mr. Roman Abramovich", beginningBalance);

            // Act and assert
            Assert.ThrowsException<System.ArgumentOutOfRangeException>(() => account.Debit(debitAmount));
        }

        [TestMethod]
        public void Debit_WhenAmountIsMoreThanBalance_ShouldThrowArgumentOutOfRange()
        {
            // Arrange
            double beginningBalance = 11.99;
            double debitAmount = 20.0; // Сумма, превышающая баланс
            BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);

            // Act and assert
            Assert.ThrowsException<System.ArgumentOutOfRangeException>(() => account.Debit(debitAmount));
        }
    }
}
```
Результат:
![image](https://github.com/user-attachments/assets/c9cc59a1-bae5-4872-a25d-8d712c3bbe05)

#6 Производим рефакторинг кода метода Debit:
BankAccount:
```
public void Debit(double amount)
{
    if (amount > m_balance)
    {
        throw new System.ArgumentOutOfRangeException("amount", amount, DebitAmountExceedsBalanceMessage);
    }

    if (amount < 0)
    {
        throw new System.ArgumentOutOfRangeException("amount", amount, DebitAmountLessThanZeroMessage);
    }

    m_balance -= amount;
}
```
BankAccountTests:
```
[TestMethod]
public void Debit_WhenAmountIsMoreThanBalance_ShouldThrowArgumentOutOfRange()
{
    // Arrange
    double beginningBalance = 11.99;
    double debitAmount = 20.0;
    BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);

    // Act
    try
    {
        account.Debit(debitAmount);
    }
    catch (System.ArgumentOutOfRangeException e)
    {
        // Assert
        StringAssert.Contains(e.Message, BankAccount.DebitAmountExceedsBalanceMessage);
        return;
    }

    Assert.Fail("The expected exception was not thrown.");
}
```
Результат:
Тест показал положительный резултат

#7. Производим Credit:
```
 public void Credit_WhenAmountIsLessThanZero_ShouldThrowArgumentOutOfRange()
 {
     // Arrange
     double beginningBalance = 11.99;
     double creditAmount = -5.77;
     BankAccount account = new BankAccount("Mr. Roman Abramovich", beginningBalance);

     // Act and assert
     Assert.ThrowsException<System.ArgumentOutOfRangeException>(() => account.Credit(creditAmount));
 }
```
Результат:
Тест показал положительный результат
![Снимок экрана 2025-03-24 141910](https://github.com/user-attachments/assets/8ab725bb-3a8b-471d-b53a-abbdb382f1c0)
