# самостійна робота №5

варіант 3 - Java, обхід з кроком що дорівнює розміру рядка кешу, тобто 64 байти

## 1 програма

масив byte виділяється один раз перед вимірюванням, потім спочатку окремий прогрів для JIT, далі для кожного розміру вимір середнього часу одного проходу з акумулятором який виводиться в кінці

крок 64 елементи по 1 байту означає що кожне звернення потрапляє в новий рядок кешу

```java
public class CacheTest {
    static long pass(byte[] a) {
        long s = 0;
        for (int i = 0; i < a.length; i += 64) s += a[i];
        return s;
    }

    public static void main(String[] args) {
        long sink = 0;

        byte[] w = new byte[1 << 20];
        for (int r = 0; r < 3000; r++) sink += pass(w);

        System.out.println("bytes;ns_per_pass;ns_per_access");
        for (long bytes = 4096; bytes <= 64L * 1024 * 1024; bytes *= 2) {
            byte[] a = new byte[(int) bytes];
            for (int i = 0; i < a.length; i++) a[i] = (byte) i;

            for (int r = 0; r < 5; r++) sink += pass(a);

            long accesses = bytes / 64;
            int reps = (int) Math.max(10, 100_000_000L / accesses);
            long t0 = System.nanoTime();
            for (int r = 0; r < reps; r++) sink += pass(a);
            long t1 = System.nanoTime();

            double ns = (double) (t1 - t0) / reps;
            System.out.println(bytes + ";" + Math.round(ns) + ";" + String.format("%.3f", ns / accesses));
        }
        System.err.println(sink);
    }
}
```

запуск - `javac --release 25 CacheTest.java` а потім `taskset -c 2 java CacheTest > result.csv`

## 2 таблиця вимірювань

сюди треба вставити результати з result.csv зі свого ПК

| розмір масиву в байтах | час на прохід в нс | нс на звернення |
| ----------------------------------------- | -------------------------------- | ---------------------------- |
| 4096                                      | 64                               | 1                            |
| 8192                                      | 147                              | 1.146                        |
| 16384                                     | 152                              | 0.596                        |
| 32768                                     | 265                              | 0.518                        |
| 65536                                     | 512                              | 0.5                          |
| 131072                                    | 1001                             | 0.489                        |
| 262144                                    | 2001                             | 0.489                        |
| 524288                                    | 4592                             | 0.560                        |
| 1048576                                   | 12121                            | 0.740                        |
| 2097152                                   | 25376                            | 0.774                        |
| 4194304                                   | 171564                           | 2.618                        |
| 8388608                                   | 2071633                          | 15.805                       |
| 16777216                                  | 1114557                          | 4.252                        |
| 33554432                                  | 2352655                          | 4.487                        |
| 67108864                                  | 4619671                          | 4.406                        |

## 3 графік

```python
import pandas as pd
import matplotlib.pyplot as plt

d = pd.read_csv("result.csv", sep=";")
plt.plot(d["bytes"], d["ns_per_access"], marker="o")
plt.xscale("log", base=2)
plt.xlabel("розмір масиву в байтах")
plt.ylabel("нс на звернення")
plt.grid(True)
plt.savefig("graph.png", dpi=150)
```

## 4 межа кешу

зламу видно між 32 і 64 КБ, це межа L1, у lscpu L1d дорівнює 256 KiB на 8 ядер, тобто 32 КБ на ядро

другий злам між 512 КБ і 1 МБ, це межа L2, у lscpu L2 дорівнює 4 MiB на 8 ядер, тобто 512 КБ на ядро

третій злам між 4 і 16 МБ, це межа L3, у lscpu L3 дорівнює 8 MiB у 2 екземплярах, тобто по 4 МБ на блок, а один потік бачить вигідно лише свій бло

![1790696879090](image/individual_5/1790696879090.png)
