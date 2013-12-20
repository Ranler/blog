layout: post
title: Java NIO Channel
date: 2013-5-8
categories: JVM
---

A `Channel` is a conduit that tansport data efficiently between byte buffers and the entity on the other end of the channel (usually a file or socket).

### Interfaces

- `Channel`
  - `InterruptibleChannel`: most channels are interruptible
  - `WritableByteChannel`: Channel operate only on byte buffers.
	- `GetheringByteChannel`
  - `ReadableByteChannel`
	- `ReadableByteChannel`

```java
public interface Channel {
	public boolean isOpen();
	public void close() throws IOException;
}

public interface ReadableByteChannel extends Channel {
	public int read(ByteBuffer dst) throws IOException;
}

public interface WritableByteChannel extends Channel {
	public int write(ByteBuffer src) throws IOException;
}

public interface ByteChannel
	extends ReadableByteChannel, WritableByteChannel{
}

public interface ScatteringByteChannel
	extends ReadableByteChannel {
	public long read(ByteBuffer[] dsts) throws IOException;
	public long read(ByteBuffer[] dsts, int offset, int length) throws IOException;
}

public interface GatheringByteChannel
	extends WritableByteChannel {
	public long write(ByteBuffer[] srcs) throws IOException;
	public long write(ByteBuffer[] scrs, int offset, int length) throws IOException;
}
```

**Scatter/Gather** refers to performing a single I/O operate across multiple buffers.


### Channels

There are two types of channels:

- file I/O: file
  - `FileChannel`
- stream I/O: socket
  - `SocketChannel`
  - `ServerSocketChannel`
  - `DatagramChannel`


Channels can operate in *blocking* or *nonblocking* modes.
Only stream-oriented channels, such as sockets and pipes , can be placed in nonblocking mode.


