#### src/main/java/org/apache/commons/io/input/SwappedDataInputStream.java

```
@Override
public byte readByte()
   throws IOException, EOFException
{
   return (byte)in.read();
}
```

`in.read()` returns a value in [-1, 255], unsafe to cast it to byte. 

  &nbsp;

  &nbsp;


#### src/main/java/org/apache/commons/io/input/NullInputStream.java :
```
public int read(final byte[] bytes, final int offset, final int length) throws IOException {
   if (eof) {
       throw new IOException("Read after end of file");
   }
   if (position == size) {
       return doEndOfFile();
   }
   position += length;    
   int returnLength = length;
   if (position > size) {
       returnLength = length - (int)(position - size);
       position = size;
   }
   processBytes(bytes, offset, returnLength);
   return returnLength;
}
```

Input `length` can be negative integer. `position += length` use it without validating.

&nbsp;

&nbsp;

#### src/main/java/org/apache/commons/io/output/ProxyOutputStream.java :
```
@Override
public void write(final byte[] bts, final int st, final int end) throws IOException {
    try {
        beforeWrite(end); // end is possibly negative, may cause subclass CountingOutputStream.count to be negative
        out.write(bts, st, end);
        afterWrite(end);
    } catch (final IOException e) {
        handleIOException(e);
    }
}
```
Input `end` can be negative integer. By calling `beforeWrite(end)`, which is defined as
```
@Override
protected synchronized void beforeWrite(final int n) {
   count += n; // count is the number of bytes that are being written
}
```
`count` may add a negative integer



#### src/main/java/org/apache/commons/io/output/ByteArrayOutputStream.java
```
@Override
public void write(final byte[] b, final int off, final int len) {
   if ((off < 0)
           || (off > b.length)
           || (len < 0)
           || ((off + len) > b.length)
           || ((off + len) < 0)) {      
       throw new IndexOutOfBoundsException();
   } else if (len == 0) {
       return;
   }
   synchronized (this) {
       final int newcount = count + len;
       int remaining = len;
       int inBufferPos = count - filledBufferSum;
       while (remaining > 0) {
           final int part = Math.min(remaining, currentBuffer.length - inBufferPos);
           System.arraycopy(b, off + len - remaining, currentBuffer, inBufferPos, part);
           remaining -= part;
           if (remaining > 0) {
               needNewBuffer(newcount);
               inBufferPos = 0;
           }
       }
       count = newcount;
   }
}
```

The if condition `(off + len) < 0` is redundant. 