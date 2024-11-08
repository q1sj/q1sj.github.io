---
layout: post
title: "Java文件大小转换"
date: 2024-11-08
tags: [java]
---

```java

/**
 * 文件大小单位枚举
 *
 * @author Q1sj
 * @date 2024/11/6 上午9:58
 */
@AllArgsConstructor
public enum FileSizeUnit {
	BYTE(1) {
		@Override
		public long toByte(long size) {
			return size;
		}

		@Override
		public long toKB(long size) {
			return size / KB.bytes;
		}

		@Override
		public long toMB(long size) {
			return size / MB.bytes;
		}

		@Override
		public long toGB(long size) {
			return size / GB.bytes;
		}

		@Override
		public long toTB(long size) {
			return size / TB.bytes;
		}

		@Override
		public long convert(long size, FileSizeUnit fileSizeUnit) {
			return fileSizeUnit.toByte(size);
		}
	},
	KB(1024) {
		@Override
		public long toByte(long size) {
			return size * (KB.bytes / BYTE.bytes);
		}

		@Override
		public long toKB(long size) {
			return size;
		}

		@Override
		public long toMB(long size) {
			return toByte(size) / MB.bytes;
		}

		@Override
		public long toGB(long size) {
			return toByte(size) / GB.bytes;
		}

		@Override
		public long toTB(long size) {
			return toByte(size) / TB.bytes;
		}

		@Override
		public long convert(long size, FileSizeUnit fileSizeUnit) {
			return fileSizeUnit.toKB(size);
		}
	},
	MB(1024 * KB.bytes) {
		@Override
		public long toByte(long size) {
			return size * MB.bytes;
		}

		@Override
		public long toKB(long size) {
			return toByte(size) / KB.bytes;
		}

		@Override
		public long toMB(long size) {
			return size;
		}

		@Override
		public long toGB(long size) {
			return toByte(size) / GB.bytes;
		}

		@Override
		public long toTB(long size) {
			return toByte(size) / TB.bytes;
		}

		@Override
		public long convert(long size, FileSizeUnit fileSizeUnit) {
			return fileSizeUnit.toMB(size);
		}
	},
	GB(1024 * MB.bytes) {
		@Override
		public long toByte(long size) {
			return size * GB.bytes;
		}

		@Override
		public long toKB(long size) {
			return toByte(size) / KB.bytes;
		}

		@Override
		public long toMB(long size) {
			return toByte(size) / MB.bytes;
		}

		@Override
		public long toGB(long size) {
			return size;
		}

		@Override
		public long toTB(long size) {
			return toByte(size) / TB.bytes;
		}

		@Override
		public long convert(long size, FileSizeUnit fileSizeUnit) {
			return fileSizeUnit.toGB(size);
		}
	},
	TB(1024 * GB.bytes) {
		@Override
		public long toByte(long size) {
			return size * TB.bytes;
		}

		@Override
		public long toKB(long size) {
			return toByte(size) / KB.bytes;
		}

		@Override
		public long toMB(long size) {
			return toByte(size) / MB.bytes;
		}

		@Override
		public long toGB(long size) {
			return toByte(size) / GB.bytes;
		}

		@Override
		public long toTB(long size) {
			return size;
		}

		@Override
		public long convert(long size, FileSizeUnit fileSizeUnit) {
			return fileSizeUnit.toTB(size);
		}
	};

	public final long bytes;

	/**
	 * 将字节数转化为人类可读文件大小(向下取整)
	 * 如:10GB 10MB
	 *
	 * @param bytes
	 * @return
	 */
	public static String toReadableSize(long bytes) {
		long readableSize;
		if ((readableSize = BYTE.toTB(bytes)) > 0) {
			return readableSize + TB.name();
		} else if ((readableSize = BYTE.toGB(bytes)) > 0) {
			return readableSize + GB.name();
		} else if ((readableSize = BYTE.toMB(bytes)) > 0) {
			return readableSize + MB.name();
		} else if ((readableSize = BYTE.toKB(bytes)) > 0) {
			return readableSize + KB.name();
		} else {
			return bytes + BYTE.name();
		}
	}

	public abstract long toByte(long size);

	public abstract long toKB(long size);

	public abstract long toMB(long size);

	public abstract long toGB(long size);

	public abstract long toTB(long size);

	public abstract long convert(long size, FileSizeUnit fileSizeUnit);
}

```