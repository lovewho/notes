# SHA-256 and SHA3-256 Hashing in Java

## 1. Overviwe

The SHA (Secure Hash Algorithm) is one of the popular cryptographic hash functions. A cryptographic hash can be used to make a signature for a text or a data file. In this tutorial, let's have a look at how we can perform SHA-256 and SHA3-256 hashing operations using various Java libraries.

The [SHA-256](https://en.wikipedia.org/wiki/SHA-2) algorithm generates an almost-unique, fixed-size 256-bit (32-byte) hash. This is a one-way function, so the result cannot be decrypted back to the original value.

Currently, SHA-2 hashing is widely used as it is being considered as the most secure hashing algorithm in the cryptographic arena.

[SHA-3](https://en.wikipedia.org/wiki/SHA-3) is the latest secure hashing standard after SHA-2. Compared to SHA-2, SHA-3 provides a different approach to generate a unique one-way hash, and it can be much faster on some hardware implementations. Similar to SHA-256, SHA3-256 is the 256-bit fixed-length algorithm in SHA-3.

[NIST](https://csrc.nist.gov/projects/hash-functions) released SHA-3 in 2015, so there are not quite as many SHA-3 libraries as SHA-2 for the time being. It's not until JDK 9 that SHA-3 algorithms were available in the built-in default providers.

Now, let's start with SHA-256.

SHA(安全的Hash算法)是一种流行的加密散列函数，加密散列可用作文本或数据文件签名，在本教程中，让我们看看如何使用各种Java库执行SHA-256和SHA3-256哈希运算！

SHA-256算法产生一个几乎唯一、定长265位(32字节)的hash值，SHA-256是一种单向散列函数，所以结果无法解密回原始值。

目前，SHA-2散列被广泛的使用，被认为是密码学领域是最安全的散列算法！

SHA-3是SHA-2之后最新的安全散列标准，与 SHA-2 相比，SHA-3 提供了一种不同的方法来生成唯一的单向散列，并且在某些硬件实现上更快！

NIST在2015年发布了SHA-3，所以暂时没有 SHA-2 那么多的 SHA-3 库。 直到 JDK 9 才在内置默认提供程序中提供 SHA-3 算法。

现在，让我们从 SHA-256 开始。

## 2. MessageDigest Class in Java

Java提供了内置的*MessageDigest*用于生成SHA-256散列：

```java
MessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] encodedhash = digest.digest(
  originalString.getBytes(StandardCharsets.UTF_8));
```

但是，这里我们必须使用自定义 byte to hex 转换器来获取十六进制的哈希值：

```java
private static String bytesToHex(byte[] hash) {
    StringBuilder hexString = new StringBuilder(2 * hash.length);
    for (int i = 0; i < hash.length; i++) {
        String hex = Integer.toHexString(0xff & hash[i]);
	    //只要当hash[i] = 0时，此时hex = 0，hex.length() = 1，则hexString需要拼上0
        if(hex.length() == 1) {
            hexString.append('0');
        }
        hexString.append(hex);
    }
    return hexString.toString();
}
```

注意：**MessageDigest is not thread-safe**，因此，您应该为每个线程创建一个新实例！

## 3. Guava Library

Guava 也提供了用于散列的类。

```java
String sha256hex = Hashing.sha256()
  .hashString(originalString, StandardCharsets.UTF_8)
  .toString();
```

## 4. Apache Commons Codes

类似的，Apache Commons Codes 也提供了一个叫 *DigestUtils*的类来支持SHA-256散列 ：

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.11</version>
</dependency>
```

```java
String sha256hex = DigestUtils.sha256Hex(originalString);
```

## 5.  Bouncy Castle Library

*Bouncy Castle Library* 是MessageDigest的一种实现！

### 5.1. Maven Dependency

```xml
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.60</version>
</dependency>
```

### 5.2. Hashing Using the Bouncy Castle Library

Bouncy Castle API提供了一个实用的类用于hex 和 bytes数据的互相转换，但是，需要首先使用内置 Java API 填充摘要：

```java
MessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] hash = digest.digest(
  originalString.getBytes(StandardCharsets.UTF_8));
// Hex.encode => byte to hex
String sha256hex = new String(Hex.encode(hash));
```

## 6. SHA3-256

Now let's continue with SHA3-256, SHA3-256 hashing in Java is nothing quite different from SHA-256.

现在让我们继续SHA3-256，Java中的SHA3-256散列与SHA-256没有什么不同。

### 6.1. MessageDegist Class in Java

[从JDK9开始](https://docs.oracle.com/javase/9/security/oracleproviders.htm#JSSEC-GUID-3A80CC46-91E1-4E47-AC51-CB7B782CEA7D)，我们能够简单的使用内置的SHA3-256算法：

```java
final MessageDigest digest = MessageDigest.getInstance("SHA3-256");
final byte[] hashbytes = digest.digest(
  originalString.getBytes(StandardCharsets.UTF_8));
// byteToHex为上述自定义的 byte to hex算法
String sha3Hex = bytesToHex(hashbytes);
```

### 6.2. Apache Commons Codecs

Apache Commons Codecs provides a convenient DigestUtils wrapper for the *MessageDigest* class. This library began to support SHA3-256 since version [1.11](https://search.maven.org/artifact/commons-codec/commons-codec/1.11/jar), and it [requires JDK 9+](https://commons.apache.org/proper/commons-codec/apidocs/org/apache/commons/codec/digest/MessageDigestAlgorithms.html#SHA3_256) as well

Apache Commons Codecs 为 MessageDigest 类提供了一个方便的 DigestUtils 包装器。 这个库从 1.11 版本开始支持 SHA3-256，它也需要 JDK 9+：

```java
String sha3Hex = new DigestUtils("SHA3-256").digestAsHex(originalString);
```

### 6.3. Keccak-256

Keccak-256 is another popular SHA3-256 hashing algorithm. Currently, it serves as an alternative to the standard SHA3-256. Keccak-256 delivers the same security level as the standard SHA3-256, and it differs from SHA3-256 only on the padding rule. It has been used in several blockchain projects, such as [Monero](https://monerodocs.org/cryptography/keccak-256/).

Again, we need to import the Bouncy Castle Library to use Keccak-256 hashing:

Keccak-256 是另一种流行的 SHA3-256 哈希算法。 目前，它可作为标准 SHA3-256 的替代方案。 Keccak-256 提供与标准 SHA3-256 相同的安全级别，仅在填充规则上与 SHA3-256 不同。 它已被用于多个区块链项目，例如门罗币。

同样，我们需要导入Bouncy Castle Library去使用Keccak-256散列：

```java
Security.addProvider(new BouncyCastleProvider());
final MessageDigest digest = MessageDigest.getInstance("Keccak-256");
final byte[] encodedhash = digest.digest(
  originalString.getBytes(StandardCharsets.UTF_8));
String sha3Hex = bytesToHex(encodedhash);
```

我们也可以使用Bouncy Castle API 去进行散列：

```java
Keccak.Digest256 digest256 = new Keccak.Digest256();
byte[] hashbytes = digest256.digest(
  originalString.getBytes(StandardCharsets.UTF_8));
String sha3Hex = new String(Hex.encode(hashbytes));
```

## 7. Conclusion

本文，我们了解了使用内置库和第三方库在 Java 中实现 SHA-256 和 SHA3-256 散列的几种方法！