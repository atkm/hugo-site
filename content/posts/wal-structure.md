+++
title = "On-disk structures of write-ahead log (WAL)"
date = "2024-01-12"
+++

## 1. How does one store WAL on disk?

How do you store WAL on disk?
This came up when I was working on a database course assignment on recovery.
The assignment closely follows the [Database System Concepts](https://codex.cs.yale.edu/avi/db-book/) textbook.

The only requirement on the WAL structure is that the LSN can be used to seek to the record on disk.
Obviously, we want locality; but that shouldn't be hard to acheive, because these records have to be written sequentially.

## 2. Don't allocate fixed-size section for each log record

What one shouldn't do is to make each log record fixed-size, say K bytes.
Then, the LSNs are contiguous integers, and computing the offset of a record is trivial: `offset(LSN) = LSN * K`.
Sadly, this will waste a lot of disk space, because the size discrepancy among record types is large.
Very roughly speaking, a log record consists of metadata and payload.
Small record types, such as transaction events (BEGIN, COMMIT, ABORT), only need to store the metadata, which is about 20 bytes.
The payload section is what makes large record types large, and its size depends on the amount of tuple info that needs to be stored.
In the worst case, two copies of an entire row need to be stored---this is for update operations, which need to remember the before- and after-state of the row in case of recovery.

So, log records for small record types create lots of wasted space.
Even for a modestly-sized table with 10 4-byte columns, at least 10 * 4 * 2 = 80 bytes need to be reserved for the payload section.
In this scenario, *every* small log record wastes *80%* of their allocated space (80 bytes out of 20+80 bytes is unused).
And space utilization gets worse as the table gets wider.

## 3. Variable-length record format

### 3-1. Let the LSN of a record be the byte offset on disk

Another obvious thing to do is to let LSN be the byte offset of the record on disk (`offset(LSN) = LSN`).
This approach works.

The byte offset of a record must be and can be computed when the record is written to the append buffer.
To do this, you just take note of the current byte offset of the log tail and the number of bytes ahead of this record in the append buffer.

Postgres splits the log into 'segments', which are 16MB files.
Figuring out which segment an LSN belong is [straightforward](https://github.com/postgres/postgres/blob/6a3631e251d1390673aee469c0cd672cac6195ef/src/include/access/xlog_internal.h#L117).

### 3-2. WAL with block structure

To support (fixed-size) blocks, we need a mapping from an LSN to the block number.
The offset of the record within the uncompressed block also needs to be encoded into an LSN.
So, logically, an LSN would be the union of a block number and an offset.

Suppose that we set the block size to be 8kB.
To compute the block number of a record, we increment the block counter on every 8kB of buffered data.
The offset into the block is obtained by mod'ing the offset into the uncompressed WAL stream by 8kB.

One thing not to do is to keep a mapping from LSN to block number in a separate sector.
First of all, there is the complexity of keeping this metadata in sync with the log records.
You really don't want that.
This mapping sector needs to be updated atomically with the records, or else WAL can be corrupted.
The second reason is performance.
Every time a log is appended, you need to seek to the mapping sector, then to the log tail, which squanders a benefit of the log structure.
If you keep this mapping on a separate disk, then you would need to write across two devices atomically.
Again, you really don't want that.

#### Block compression support

This scheme can be modified to support compression of blocks.
To support block compression, the offsets of compressed blocks need to be encoded into the LSNs.
To do so, every 8kB of buffered data must be compressed before more data is buffered.
This is to maintain the on-disk offset of the tail block.
Since an LSN has to be given out when a record is buffered, I don't think we can split compressed data into chunks (that align with filesystem blocks).
Another side note is that compression comes with a disadvantage that an entire block needs to be decompressed before records in it can be accessed.
That's not good for random access.

### 3-3. Side note: an LSN in Postgres is 64-bit.

Postgres uses a [`uint64`](https://github.com/postgres/postgres/blob/45da69371ebfc4d6982695e58791989660c1cc33/src/include/access/xlogdefs.h#L19) for its LSN, with which you can address 16EiB of space.
(
    ... In theory.
    Its LSN used to be the [union](https://pgpedia.info/x/xlogrecptr.html) of the log segment number, and the offset into the segment.
    Each segment is 16MB by default, so this scheme can address up to 16MB * max(uint32) bytes, which is a lot less than max(uint64).
)
This is far more than what typical users need.
Even a `uint32` can address up to \~4PB, when large HDDs have about 20TB of capacity.
Checkpointing ought to happen far before you accumulate that much log data.
Also... is recovery at this scale even feasible?
It takes 15 months to read 4PB of data sequentially at 100MB/s transfer rate; 5000 years for 16EB.
[Parallel recovery](https://wiki.postgresql.org/wiki/Parallel_Recovery) seems to be a thing (though experimental).
But you need 5000 workers doing work at the max disk tput to go through 16EB of data in 1 year.
Due to its sequential nature, WAL supporting that much parallelism seems unlikely.

But hey, storing four extra bytes on each record prepares Postgres for a version of the future where non-volatile storage is comically fast by today's standard, why not.

### 3-4. How does Postgres support page-level compression?

Postgres' LSN is a [byte offset](https://www.postgresql.org/docs/current/datatype-pg-lsn.html) in the WAL stream.
Postgres supports [page-level compression](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-WAL-COMPRESSION).
A page is [8kB](https://www.postgresql.org/docs/current/wal-internals.html) by default.
A page can contain [multiple records](https://github.com/postgres/postgres/blob/6a3631e251d1390673aee469c0cd672cac6195ef/src/include/access/xlog_internal.h#L36).

I don't see how LSN can remain to be an offset into the (uncompressed) WAL stream, since records in the same page are compressed together.
It doesn't make sense to speak of an offset into compressed data.
Also, compression throws off the segment number computation.
I would think the LSN needs to be (logically) the union of the segment number, the offset of the compressed page within the segment, and the offset into the uncompressed page.

## 4. LevelDB/RocksDB format

LevelDB uses a [blocked structure](https://github.com/google/leveldb/blob/main/doc/log_format.md) for its WAL.
Unlike recordio, this structure does not employ per-block headers.

```
Copy-pasted straight from [RocksDB doc](https://github.com/facebook/rocksdb/wiki/Write-Ahead-Log-File-Format#log-file-format)

       +-----+-------------+--+----+----------+------+-- ... ----+
 File  | r0  |        r1   |P | r2 |    r3    |  r4  |           |
       +-----+-------------+--+----+----------+------+-- ... ----+
       <--- kBlockSize ------>|<-- kBlockSize ------>|

  rn = variable size records
  P = Padding
```
(Side note: the log format of RocksDB is the same as LevelDB.
Makes sense, one is a fork of the other.)

This forces the database to *iterate through the entire WAL for recovery*.
The [log reader](https://github.com/facebook/rocksdb/blob/main/db/log_reader.h) class does not provide a random read API.
Instead, it only provides a front-to-back sequential read API (`ReadRecord`).
Why can it get away with this?
It may have something to do with the fact that LevelDB is internally append-only.
(A single key may exist in multiple SSTables, but queries read the latest SSTable.)

Still, they need to make sure that appends of aborted and unfinished transactions are rolled back, and transaction ordering is respected.
Upon start, LevelDB [loads all log records into the in-memory data structure](https://github.com/google/leveldb/blob/068d5ee1a3ac40dabd00d211d5013af44be55bea/db/db_impl.cc#L385).
I suppose that the normal flushing logic of memtable enforces transaction guarantees.
There is an [article](https://rocksdb.org/blog/2017/12/19/write-prepared-txn.html) that describes the concurrency protocol of RocksDB.
A question that I have is whether writes from an uncommitted transaction can be persisted, and, if so, what they do about it.

## 5. Epilogue

For a single-version database, which is what the course assignment is on, there is a natural and straightforward WAL/LSN scheme.
In general, though, the WAL structure depends on the database architecture.
For example, the WAL in LevelDB is meant to be read sequentially back-to-back.
I should read about recovery processes of MVCC databases to see if they are any different.

## 6. Links

- A [recordio](https://github.com/grailbio/base/blob/master/recordio/README.md) library in Go.
