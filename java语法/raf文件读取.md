自己用过的用法：

raf  = new RandomAcessFile（logfile，”rw“）；

raf.seek();改变指针的位置

raf.getpointer();获取当前指的位置可与long类型加减

raf.readLong();读一个long数据，指针移动8个字节



 `RandomAccessFile` 是 Java 中用于随机访问文件的类。它允许你以类似于数组的方式读取和写入文件中的任意位置。`RandomAccessFile` 支持多种数据类型的操作，包括基本数据类型（如 `int`、`long`、`float`、`double` 等）和字符数据。

### 主要方法

1. **构造方法**：
   
   - `RandomAccessFile(File file, String mode)`：创建一个 `RandomAccessFile` 对象，用于读取和/或写入指定的文件。
   - `RandomAccessFile(String name, String mode)`：创建一个 `RandomAccessFile` 对象，用于读取和/或写入指定名称的文件。
   
   `mode` 参数可以是以下值之一：
   - `"r"`：只读模式。
   - `"rw"`：读写模式。
   
2. **文件指针操作**：
   - `void seek(long pos)`：将文件指针移动到指定的位置。
   - `long getFilePointer()`：返回当前文件指针的位置。
   - `long length()`：返回文件的长度（以字节为单位）。

3. **读取数据**：
   - `int read()`：读取一个字节并返回其值。
   - `int read(byte[] b)`：读取多个字节并存储在字节数组中。
   - `int read(byte[] b, int off, int len)`：从指定偏移量开始读取多个字节并存储在字节数组中。
   - `int readBoolean()`：读取一个布尔值。
   - `int readByte()`：读取一个字节。
   - `char readChar()`：读取一个字符。
   - `double readDouble()`：读取一个双精度浮点数。
   - `float readFloat()`：读取一个单精度浮点数。
   - `void readFully(byte[] b)`：读取指定数量的字节并存储在字节数组中。
   - `void readFully(byte[] b, int off, int len)`：从指定偏移量开始读取指定数量的字节并存储在字节数组中。
   - `int readInt()`：读取一个整数。
   - `String readLine()`：读取一行文本（已过时，建议使用 `BufferedReader`）。
   - `long readLong()`：读取一个长整数。
   - `short readShort()`：读取一个短整数。
   - `int readUnsignedByte()`：读取一个无符号字节。
   - `int readUnsignedShort()`：读取一个无符号短整数。
   - `String readUTF()`：读取一个 UTF-8 编码的字符串。

4. **写入数据**：
   - `void write(int b)`：写入一个字节。
   - `void write(byte[] b)`：写入多个字节。
   - `void write(byte[] b, int off, int len)`：从指定偏移量开始写入多个字节。
   - `void writeBoolean(boolean v)`：写入一个布尔值。
   - `void writeByte(int v)`：写入一个字节。
   - `void writeBytes(String s)`：写入一个字符串的字节。
   - `void writeChar(int v)`：写入一个字符。
   - `void writeChars(String s)`：写入一个字符串的字符。
   - `void writeDouble(double v)`：写入一个双精度浮点数。
   - `void writeFloat(float v)`：写入一个单精度浮点数。
   - `void writeInt(int v)`：写入一个整数。
   - `void writeLong(long v)`：写入一个长整数。
   - `void writeShort(int v)`：写入一个短整数。
   - `void writeUTF(String str)`：写入一个 UTF-8 编码的字符串。

5. **文件操作**：
   - `void close()`：关闭 `RandomAccessFile`。
   - `void setLength(long newLength)`：设置文件的长度。
   - `void truncate(long length)`：截断文件到指定的长度。

### 示例代码

以下是一个简单的示例，展示了如何使用 `RandomAccessFile` 进行读写操作。

```java
import java.io.RandomAccessFile;
import java.io.File;
import java.io.IOException;

public class RandomAccessFileExample {
    public static void main(String[] args) {
        File file = new File("example.dat");
        
        try (RandomAccessFile raf = new RandomAccessFile(file, "rw")) {
            // 写入一些数据
            raf.writeInt(123456789);
            raf.writeDouble(1234.5678);
            raf.writeUTF("Hello, World!");
            
            // 输出当前文件指针位置
            System.out.println("After writing, file pointer is at: " + raf.getFilePointer());
            
            // 移动文件指针到开头
            raf.seek(0);
            
            // 读取数据
            int intValue = raf.readInt();
            double doubleValue = raf.readDouble();
            String stringValue = raf.readUTF();
            
            System.out.println("Read int value: " + intValue);
            System.out.println("Read double value: " + doubleValue);
            System.out.println("Read string value: " + stringValue);
            
            // 输出当前文件指针位置
            System.out.println("After reading, file pointer is at: " + raf.getFilePointer());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 输出示例

假设你运行上述代码，输出可能如下：

```
After writing, file pointer is at: 27
Read int value: 123456789
Read double value: 1234.5678
Read string value: Hello, World!
After reading, file pointer is at: 27
```

### 解释

1. **写入数据**：
   - 使用 `writeInt`、`writeDouble` 和 `writeUTF` 方法写入不同类型的数据。
   - 写入完成后，文件指针位于文件的末尾。

2. **读取数据**：
   - 使用 `seek` 方法将文件指针移动到文件的开头。
   - 使用 `readInt`、`readDouble` 和 `readUTF` 方法读取不同类型的数据。
   - 读取完成后，文件指针位于文件的末尾。

### 注意事项

- **文件指针**：在读写操作中，文件指针会自动移动。确保在读写之前将文件指针移动到正确的位置。
- **异常处理**：在使用 `RandomAccessFile` 时，务必处理可能抛出的 `IOException`。
- **资源管理**：使用 `try-with-resources` 语句确保 `RandomAccessFile` 被正确关闭。

通过这些方法，你可以灵活地读取和写入文件中的任意位置，实现高效的文件操作。