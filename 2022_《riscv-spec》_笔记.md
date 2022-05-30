# 《riscv-spec》_(书上看不太懂的)

## chapter 8.2(66/283)
Load-Reserved/Store-Conditional Instructions

31--27 | 26       | 25  | 24--20 | 19--15 | 14--12 | 11--7 | 6--0   | **bit range**
-------|----------|-----|--------|--------|--------|-------|--------|--------------
5      | 1        | 1   | 5      | 5      | 3      | 5     | 7      | **bit count**
funct5 | aq       | rl  | rs2    | rs1    | funct3 | rd    | opcode |
LR.W/D | ordering | 0   | addr   | width  | dest   | AMO   | --     | **exampel-1**
SC.W/D | ordering | src | addr   | width  | dest   | AMO   | --     | **exampel-2**


Complex atomix memory operations on a _single memory word_ or _doubleword_ with the load-reserved(LR) and store-conditional(SC) instructions.  
* 使用 load-reserved(LR) 和 store-conditional(SC) 指令对 single memory word 或 store-conditional(SC) 进行复杂的 atomix memory operations。

**LR.W** loads a _word_ from the address in _rs1_, places the _sign-extended value_ in _rd_, and registers a reservation set—a set of bytes that subsumes the bytes in the addressed word. 
* **LR.W** 从 _rs1_ 中的地址 加载 1个 _word_ ，
* 将 _sign-extended value_ 放置在 _rd_ 中，
* and registers(寄存器、注册) a reservation(保留) set — a set of bytes that subsumes(把...归入) the bytes in the addressed word. 

**SC.W** conditionally writes a _word_ in _rs2_ to the address in rs1: 
the **SC.W** succeeds only if the reservation is still valid and the reservation set contains the bytes being written.  
* **SC.W** 有条件地将 _rs2_ 中的一个 _word_ 写入 _rs1_ 中的地址:
  * 只有当 reservation(保留) 仍然有效且 reservation set(保留set) 包含正在写入的字节时，**SC.W** 才会成功。

If the **SC.W** succeeds, the instruction _write_ the word in _rs2_ to memory, and it writes _zero_ to _rd_. 
* 如果 **SC.W** 成功，指令将 _rs2_ 2中的字写入内存，并将 0 写入 _rd_。


If the **SC.W** fails, the instruction does not _write_ to memory, and it writes a _nonzero_ value to _rd_. 
* 如果SC.W失败，指令不会写入内存，而是将一个 非0值 写入 _rd_ 。



Regardless of success or failure, executing an **SC.W** instruction invalidates any reservation held by this hart. 
* 不管成功还是失败，执行 **SC.W** 指令将使 该hart持有的 任何reservation(保留) 失效。


**LR.D** and **SC.D** act analogously on doublewords and are only available on RV64.  
For RV64, **LR.W** and **SC.W** sign-extend the value placed in rd.
* **LR.D** 和 **SC.D** 类似地作用于 doublewords，并且只在RV64上可用。
* 对于RV64， **LR.W** 和 **SC.W** 的 sign-extend的值 放在rd中。

------👇 横线内容 👇------  

Both compare-and-swap (CAS) and LR/SC can be used to build lock-free data structures. 

After extensive discussion, we opted for LR/SC for several reasons:

(1) CAS suffers from the ABA problem, which LR/SC avoids because it monitors all accesses to the address rather than only checking for changes in the data value; 

(2) CAS would also require a new integer instruction format to support three source operands (address, compare value, swap value) as well as a different memory system message format, which would complicate microarchitectures; 

(3) Furthermore,to avoid the ABA problem, other systems provide a double-wide CAS (DW-CAS) to allow a counter to be tested and incremented along with a data word.  
This requires reading five registers and writing two in one instruction, and also a new larger memory system message type, further complicating implementations; 


(4) LR/SC provides a more efficient implementation of many primitives as it only requires one load as opposed to two with CAS (one load before the CAS instruction to obtain a value for speculative computation, then a second load as part of the CAS instruction to check if value is unchanged before updating).  
The main disadvantage of LR/SC over CAS is livelock, which we avoid, under certain circumstances, with an architected guarantee of eventual forward progress as described below.  
Another concern is whether the influence of the current x86 architecture, with its DW-CAS, will complicate porting of synchronization libraries and other software that assumes DW-CAS is the basic machine primitive. 

A possible mitigating factor is the recent addition of transactional memory instructions to x86, which might cause a move away from DW-CAS. 

More generally, a multi-word atomic primitive is desirable, but there is still considerable debate about what form this should take, and guaranteeing forward progress adds complexity to a system. 

Our current thoughts are to include a small limited-capacity transactional memory buffer along the lines of the original transactional memory proposals as an optional standard
extension “T”.

------👆 横线内容 👆------  


31--27 | 26         | 25         | 24--20 | 19--15 | 14--12 | 11--7 | 6--0   | **bit range**
-------|------------|------------|--------|--------|--------|-------|--------|--------------
5      | 1          | 1          | 5      | 5      | 3      | 5     | 7      | **bit count**
funct5 | aq         | rl         | rs2    | rs1    | funct3 | rd    | opcode |
LR.W/D | ordering_1 | ordering_2 | 0      | addr   | width  | dest  | AMO    | **exampel-1**
SC.W/D | ordering_1 | ordering_2 | src    | addr   | width  | dest  | AMO    | **exampel-2**


The failure code with value 1 is reserved to encode an unspecified failure.  
* reserved(保留) 为1的 failure code(故障码)，用于编码未指定的故障。


Other failure codes are reserved at this time, and portable software should only assume the failure code will be non-zero.

------👇 横线内容 👇------  

We reserve a failure code of 1 to mean “unspecified” so that simple implementations may return
this value using the existing mux required for the SLT/SLTU instructions.  
More specific failure codes might be defined in future versions or extensions to the ISA.

------👆 横线内容 👆------  


For **LR** and **SC**, the A extension requires that the address held in _rs1_ be naturally aligned to the size of the operand (i.e., _eight-byte_ aligned for **64-bit words** and four-byte aligned for 32-bit words).  
*  对于 **LR** 和 **SC** , A extension 要求 _rs1_ 中保存的地址 自然对齐到操作数的大小(即，
    * 对于 64-bit words，8字节对齐，
    * 对于32位单词，4字节对齐)。

If the address is not naturally aligned, an address-misaligned exception or an access-fault exception will be generated.  
* 如果地址没有自然对齐，则会生成 
  * address-misaligned 异常 
  * 或 access-fault(访问错误) 异常。

The **access-fault exception** can be generated for a **memory access** that would otherwise be able to complete except for the misalignment, if the misaligned access should not be emulated.
* 可以为 **memory access** 生成 **access-fault exception** that would otherwise be able to complete except for the misalignment, if the misaligned access should not be emulated.


------👇 横线内容 👇------  

Emulating **misaligned LR/SC sequences** is impractical in most systems.  
* 在大多数系统中，模拟 misaligned LR/SC sequences 是不切实际的。

**Misaligned LR/SC sequences** also raise the possibility of accessing multiple **reservation sets** at once, which present definitions do not provide for.
* **Misaligned LR/SC sequences** 还增加了一次访问多个 **reservation(保留) sets** 的可能性，这是目前的定义还不提供的。

------👆 横线内容 👆------  

An implementation can register an arbitrarily large reservation set on each **LR**, provided the reservation set includes all bytes of the addressed data word or doubleword. 

An **SC** can only pair with the most recent **LR** in program order.  
An **SC** may succeed only if no store from another hart to the reservation set can be observed to have occurred between the **LR** and the **SC**, and if there is no other **SC** between the **LR** and itself in program order.  
An **SC** may succeed only if no write from a device other than a hart to the bytes accessed by the **LR** instruction can be observed to have occurred between the **LR** and **SC**.  
Note this **LR** might have had a different effective address and
data size, but reserved the **SC’s** address as part of the reservation set.



------👇 横线内容 👇------  

Following this model, in systems with memory translation, an SC is allowed to succeed if the earlier LR reserved the same location using an alias with a different virtual address, but is also allowed to fail if the virtual address is different.

------👆 横线内容 👆------  



------------👇 横线内容 👇------------

To accommodate legacy devices and buses, writes from devices other than RISC-V harts are only required to invalidate reservations when they overlap the bytes accessed by the LR.  
These writes are not required to invalidate the reservation when they access other bytes in the reservation set.

------------👆 横线内容 👆------------  


The SC must fail if the address is not within the reservation set of the most recent LR in program order.  
The SC must fail if a store to the reservation set from another hart can be observed to occur between the LR and SC.  
The SC must fail if a write from some other device to the bytes accessed by the LR can be observed to occur between the LR and SC.  
(If such a device writes the reservation set but does not write the bytes accessed by the LR, the SC may or may not fail.) An SC must fail if there is another SC (to any address) between the LR and the SC in program order.  
The precise statement of the atomicity requirements for successful LR/SC sequences is defined by the Atomicity Axiom in Section 14.1.


------👇 横线内容 👇------  

The platform should provide a means to determine the size and shape of the reservation set.
A platform specification may constrain the size and shape of the reservation set.  
For example, the Unix platform is expected to require of main memory that the reservation set be of fixed size,
contiguous, naturally aligned, and no greater than the virtual memory page size.  
------👆 横线内容 👆------  

------------👇 横线内容 👇------------

A store-conditional instruction to a scratch word of memory should be used to forcibly invalidate any existing load reservation:

* during a preemptive(先发制人的) context switch, and if necessary when changing virtual to physical address mappings, such as when migrating

pages that might contain an active reservation.  

------------👆 横线内容 👆------------  


------👇 横线内容 👇------  
The invalidation of a hart’s reservation when it executes an LR or SC imply that a hart can only hold one reservation at a time, and that an SC can only pair with the most recent LR, and LR with the next following SC, in program order.  
This is a restriction to the Atomicity Axiom in Section 14.1 that ensures software runs correctly on expected common implementations that operate in this manner.

------👆 横线内容 👆------  


An SC instruction can never be observed by another RISC-V hart before the LR instruction that established the reservation.  
The LR/SC sequence can be given acquire semantics by setting the aq bit on the LR instruction.  
The LR/SC sequence can be given release semantics by setting the rl bit on the SC instruction.  
Setting the aq bit on the LR instruction, and setting both the aq and the rl bit on the SC instruction makes the LR/SC sequence sequentially consistent, meaning that it cannot be reordered with earlier or later memory operations from the same hart.

If neither bit is set on both LR and SC, the LR/SC sequence can be observed to occur before or after surrounding memory operations from the same RISC-V hart. This can be appropriate when the LR/SC sequence is used to implement a parallel reduction operation.  

Software should not set the rl bit on an LR instruction unless the aq bit is also set, nor should software set the aq bit on an SC instruction unless the rl bit is also set. LR.rl and SC.aq instructions are not guaranteed to provide any stronger ordering than those with both bits clear, but may result in lower performance.

```
    # a0 holds address of memory location
    # a1 holds expected value
    # a2 holds desired value
    # a0 holds return value, 0 if successful, !0 otherwise
cas:
    lr.w  t0, (a0)        # Load original value.
    bne   t0, a1, fail    # Doesn’t match, so fail.
    sc.w  t0, a2, (a0)    # Try to update.
    bnez  t0, cas         # Retry if store-conditional failed.
    li    a0, 0           # Set return to success.
    jr ra                 # Return.
fail:
    li a0, 1      # Set return to failure.
    jr ra         # Return

                    Figure 8.1: Sample code for compare-and-swap function using LR/SC
```

LR/SC can be used to construct lock-free data structures.  
An example using LR/SC to implement a compare-and-swap function is shown in Figure 8.1.  
If inlined, compare-and-swap functionality need only take four instructions.



## chapter 8.3(69/283)
Eventual Success of Store-Conditional Instructions

The standard A extension defines constrained LR/SC loops, which have the following properties:

* The loop comprises only an LR/SC sequence and code to retry the sequence in the case of failure, and must comprise at most 16 instructions placed sequentially in memory.

* An LR/SC sequence begins with an LR instruction and ends with an SC instruction.  
The dynamic code executed between the LR and SC instructions can only contain instructions from the base “I” instruction set, excluding loads, stores, backward jumps, taken backward branches, JALR, FENCE, and SYSTEM instructions.  
If the “C” extension is supported, then compressed forms of the aforementioned “I” instructions are also permitted.  

* The code to retry a failing LR/SC sequence can contain backwards jumps and/or branches to repeat the LR/SC sequence, but otherwise has the same constraint as the code between the LR and SC.

* The LR and SC addresses must lie within a memory region with the LR/SC eventuality property. The execution environment is responsible for communicating which regions have this property.

* The SC must be to the same effective address and of the same data size as the latest LR executed by the same hart.

LR/SC sequences that do not lie within constrained LR/SC loops are unconstrained.  
Unconstrained LR/SC sequences might succeed on some attempts on some implementations, but might never succeed on other implementations.


------👇 横线内容 👇------  

We restricted the length of LR/SC loops to fit within 64 contiguous instruction bytes in the base ISA to avoid undue restrictions on instruction cache and TLB size and associativity.  
Similarly, we disallowed other loads and stores within the loops to avoid restrictions on data-cache associativity in simple implementations that track the reservation within a private cache.  
The restrictions on branches and jumps limit the time that can be spent in the sequence. Floating-point operations and integer multiply/divide were disallowed to simplify the operating system’s emulation of these instructions on implementations lacking appropriate hardware support.
Software is not forbidden from using unconstrained LR/SC sequences, but portable software must detect the case that the sequence repeatedly fails, then fall back to an alternate code sequence that does not rely on an unconstrained LR/SC sequence.  
Implementations are permitted to unconditionally fail any unconstrained LR/SC sequence.  
If a hart H enters a constrained LR/SC loop, the execution environment must guarantee that one of the following events eventually occurs:

* H or some other hart executes a successful SC to the reservation set of the LR instruction in
H’s constrained LR/SC loops.
* Some other hart executes an unconditional store or AMO instruction to the reservation set of
the LR instruction in H’s constrained LR/SC loop, or some other device in the system writes
to that reservation set.
* H executes a branch or jump that exits the constrained LR/SC loop.
* H traps.

Note that these definitions permit an implementation to fail an SC instruction occasionally for any reason, provided the aforementioned guarantee is not violated.
As a consequence of the eventuality guarantee, if some harts in an execution environment are executing constrained LR/SC loops, and no other harts or devices in the execution environment execute an unconditional store or AMO to that reservation set, then at least one hart will eventually exit its constrained LR/SC loop.  
By contrast, if other harts or devices continue to write to that reservation set, it is not guaranteed that any hart will exit its LR/SC loop.  
Loads and load-reserved instructions do not by themselves impede the progress of other harts’ LR/SC sequences.  
We note this constraint implies, among other things, that loads and load-reserved instructions executed by other harts (possibly within the same core) cannot impede LR/SC progress indefinitely. For example, cache evictions caused by another hart sharing the cache cannot impede LR/SC progress indefinitely. Typically, this implies reservations are tracked independently of evictions from any shared cache. Similarly, cache misses caused by speculative execution within a hart cannot impede LR/SC progress indefinitely.

These definitions admit the possibility that SC instructions may spuriously fail for implementation reasons, provided progress is eventually made.
One advantage of CAS is that it guarantees that some hart eventually makes progress, whereas an LR/SC atomic sequence could livelock indefinitely on some systems. 
To avoid this concern, we added an architectural guarantee of livelock freedom for certain LR/SC sequences.
Earlier versions of this specification imposed a stronger starvation-freedom guarantee.  

However, the weaker livelock-freedom guarantee is sufficient to implement the C11 and C++11 languages, and is substantially easier to provide in some microarchitectural styles
------👆 横线内容 👆------


------👇 横线内容 👇------  


------👆 横线内容 👆------  
