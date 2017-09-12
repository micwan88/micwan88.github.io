---
title: How to convert IBM packed decimal / zoned decimal in java
categories: java
---

## How to convert IBM packed decimal / zoned decimal in java

There is a Java library called [IBM Toolbox for Java](https://www-03.ibm.com/systems/power/software/i/toolbox/downloads.html) or [JTOpen, a open source version](http://sourceforge.net/projects/jt400) can convert to/from IBM decimal format of packed decimal and zoned decimal in Java.

To use it, include jt400.jar in your class path and below is an example for coverting packed decimal.

``` java
/**
 * Java BigDecimal to Packed Decimal
 */
//15 means total number of digits, 5 means number of decimal places
AS400PackedDecimal packedDecimal = new AS400PackedDecimal(15, 5);
BigDecimal javaBigDecimal = new BigDecimal("1000.12345");
byte[] convertedBytesArray = packedDecimal.toBytes(javaBigDecimal);

/**
 * Packed Decimal to Java Big Decimal
 */
BigDecimal convertedBigDecimal = (BigDecimal)packedDecimal.toObject(convertedBytesArray);
```

References:
- [Toolbox for Java and JTOpen](https://www.ibm.com/developerworks/ibmi/library/i-javatoolbox/index.html)
- [Programming Java in IBM i](https://www.ibm.com/support/knowledgecenter/en/ssw_ibm_i_73/rzahh/page1.htm)
- [JTOpen](http://jt400.sourceforge.net/)
- [IBM Toolbox for Java](http://www.ibm.com/systems/i/software/toolbox/)