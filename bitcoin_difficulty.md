# Cложностью в сети Bitcoin 
Сложность в сети Bitcoin — это мера, показывающая, насколько трудно найти новый блок. Эта система автоматически корректируется сетью таким образом, чтобы время, затрачиваемое на поиск одного блока, составляло примерно 10 минут.

### Основные определения
1. Target (Цель) — это 256-битное число, которое используют все клиенты Bitcoin. Хеш блока должен быть меньше или равен текущей цели, чтобы блок был принят сетью.

2. Difficulty (Сложность) — это мера, которая показывает, насколько трудно найти хеш, который меньше или равен текущей цели.

3. Hash (Хеш) — каждый блок в сети Bitcoin имеет уникальный хеш, который служит его идентификатором. Майнеры генерируют хеши, пытаясь найти тот, который соответствует текущей цели.

### Связь между сложностью и целью
Сложность напрямую связана с целью. Сложность определяется как отношение базовой цели к текущей цели и выражается следующей формулой:
```sh
difficulty = difficulty_1_target / current_target
```

Базовая цель (difficulty_1_target) — это константа, и ее значение фиксировано: `00000000FFFF0000000000000000000000000000000000000000000000000000`

Таким образом чем меньше target, тем больше difficult, и соответственно тем сложнее найти хэш блока.
### Java код для расчета цели на основе сложности
Вот пример кода на Java, который показывает, как можно рассчитать цель, основываясь на сложности сети или сложности, установленной для майнеров в пуле.

```java
import java.math.BigInteger;

public class BitcoinDifficulty {

  private static final String DIFFICULTY_1_TARGET = "00000000FFFF0000000000000000000000000000000000000000000000000000";

    private static BigInteger calculateTarget(BigInteger difficultyHex) {
        BigInteger maxTargetHex = new BigInteger(DIFFICULTY_1_TARGET, 16);

        return maxTargetHex.divide(difficultyHex);
    }

    public static void main(String[] args) {
        BigInteger poolDifficulty = BigInteger.valueOf(500_000L);
        BigInteger networkDifficulty = BigInteger.valueOf(62_463_471_666_732L);
        BigInteger poolTargetHex = calculateTarget(poolDifficulty);
        BigInteger networkTargetHex = calculateTarget(networkDifficulty);

        String foundBlockHash = "000000000000000000039113e2995e8673734db9e65e9ac1835a267da42bf60a";
        BigInteger foundBlockHex = new BigInteger(foundBlockHash, 16);

        if (isBlockHashLessOrEqualsTarget(foundBlockHex, poolTargetHex)) {
            System.out.println("The share is valid");
        }

        if (isBlockHashLessOrEqualsTarget(foundBlockHex, networkTargetHex)) {
            System.out.println("New block hash is found");
        }

        if (!isBlockHashLessOrEqualsTarget(foundBlockHex, poolTargetHex)) {
            System.out.println("The share is not valid");
        }

        System.out.println("Pool target hash    : " + toHash(poolTargetHex));
        System.out.println("Network target hash : " + toHash(networkTargetHex));
        System.out.println("Found block hash    : " + toHash(foundBlockHex));
    }

    private static boolean isBlockHashLessOrEqualsTarget(BigInteger blockHex, BigInteger targetHex) {
        return blockHex.compareTo(targetHex) < 0;
    }

    private static String toHash(BigInteger target) {
        String value = target.toString(16);

        int length = 64;
        if (value.length() >= length) {
            return value;
        }
        StringBuilder sb = new StringBuilder();
        while (sb.length() < length - value.length()) {
            sb.append('0');
        }
        sb.append(value);

        return sb.toString().toUpperCase();
    }
}
```

В этом коде метод calculateTarget принимает сложность в качестве параметра и возвращает цель. 
В качестве примера взять хэш существующего блока сети и сложность сети действующая, когда этот блок был найден. 
Из результата видно, что хеш блока соответствует условию "меньше текущей цели". 
При этом, если хэш сблока соответствует условию "меньше текущей цели" для цели рассчитанной на основании сложности сети,
то это же условие выполняется и для цели рассчитанной на основании сложности выставленной пулом (500000 в примере выше).

```java
Pool target hash    : 000000000000218dcdb37c99ae924f227d028a1dfb9389b52007dd441355475a
Network target hash : 0000000000000000000481940000000eb0bbbbfbbc2ba032a4367cd675458cbb
Found block hash    : 000000000000000000039113e2995e8673734db9e65e9ac1835a267da42bf60a
```
### Ссылки на источники
- [Target](https://en.bitcoin.it/wiki/Target)
- [Difficulty](https://en.bitcoin.it/wiki/Difficulty)
